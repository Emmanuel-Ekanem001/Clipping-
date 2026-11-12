# Emmy Clip Studio — Implementation Plan

_Last updated: 2026-11-12 (Stage 0 Audit)_

---

## Audit Findings

### Repository State

The repo contains **exactly one file**: `docs/BRIEF.md`.  
There is **no existing code** — no frontend, no backend, no database schema, no job queue, no migrations.  
The brief describes an architecture that has not yet been built.  
Everything in Sections 11–14 of the brief ("existing project", "existing architecture") is aspirational, not descriptive.

No git history exists (repo was uninitialised).

---

### Upload Bug Root Cause

**The file picker bug does not exist yet** because there is no frontend code at all.  
The brief documents a known UX failure and specifies the target architecture, but there is currently no HTML, React component, hidden `<input type="file">`, ref, event handler, or overlay to inspect.

When the frontend is built (Stage 1), the fix must be pre-emptively applied:

- The `<input type="file">` must be rendered in the DOM (not conditionally removed by React state resets).
- It must be associated via a stable `ref` that is never swapped out between renders.
- The click trigger must call `inputRef.current.click()` on the actual DOM node, not on a re-created element.
- No opaque overlay (`pointer-events: none` or absolutely-positioned div) must sit between the visible upload area and the hidden input.
- Label/for wiring is an alternative but requires that `htmlFor` matches the input's `id` exactly and that nothing intercepts the synthetic click.
- Drag-and-drop must be on the container div, not the input.
- The filechooser event in Playwright must be caught **before** the click is dispatched (`page.waitForEvent('filechooser')` before `locator.click()`).

---

### System Environment (actual, verified)

| Item | Status |
|---|---|
| OS | Debian Linux, x86_64 |
| Python | 3.13.5 (system), `python3 -m venv` works |
| pip | Available via `python3 -m ensurepip` inside a venv |
| Node.js | 26.13.0 (nvm) |
| npm | 11.20.0 |
| FFmpeg | 7.1.5 (system, full codec support) |
| FFprobe | 7.1.5 (system) |
| SQLite | 3.46.1 |
| OpenCV | 5.0.0 (in opencode venv at `~/.config/opencode/skills/effects/.venv`) |
| NumPy | 2.5.4 (same venv) |
| Pillow | 13.0.0 (same venv) |
| MediaPipe | 1.2.2 (same venv) |
| LibreOffice | 25.2.3 (soffice — DOCX → text conversion) |
| pdftotext | 25.03.0 (poppler-utils — PDF → text) |
| Tesseract | Library only (`libtesseract5`) — no `tesseract` binary, no OCR CLI |
| Playwright | Not installed |
| Network (sandbox) | **BLOCKED** — pip install fails; must use existing packages + offline installs |

**Critical constraint**: The standard Antigravity sandbox blocks all outbound network.  
All Python packages must come from either the system or the existing opencode venv.  
The backend venv for Emmy must be created as a **copy** of the opencode venv (or we symlink into it) rather than pip-installing afresh.  
Alternatively, we ask the user to allow network access for a single install step.

**Available Python packages (without network)**:
- `opencv-contrib-python` 5.0.0
- `numpy` 2.5.4
- `pillow` 13.0.0
- `mediapipe` 1.2.2
- `matplotlib` 3.11.2
- `sounddevice` 0.5.6
- `requests`, `certifi`, `urllib3`, `charset-normalizer`, `idna` (system)
- `psutil` (system)
- Python stdlib: `sqlite3`, `asyncio`, `pathlib`, `subprocess`, `json`, `hashlib`, `re`, `email`, `http.server`, `threading`, `multiprocessing`, `tempfile`, `shutil`, `struct`, `wave`, `zipfile`, `xml`, `csv`

**What we must build without pip**:
- FastAPI → replaced by Python's `http.server` + hand-rolled async routes? **No** — this is too limiting. Better option: use a minimal pure-Python HTTP framework approach, or request the user to allow one pip install of `fastapi uvicorn aiofiles python-multipart aiosqlite alembic`.

**Decision**: For Stage 1, I will note this constraint clearly. The backend must be installable. We will request the user approve one `pip install` with network bypass.

---

### What Works

Nothing works yet — the project does not exist.

---

### What Is Broken

Everything — the project does not exist.

---

### What Is Incomplete

Everything — the project does not exist.

---

## Staged Implementation Plan

### Stage 0 — Audit ✅ (this document)
Inspect repo. Write audit and plan. Initialise git. Write PROGRESS.md.

---

### Stage 1 — Upload and Video Library

**Goal**: Working file picker, drag-and-drop, upload progress, metadata extraction, thumbnail generation, proxy creation, video playback, deletion. Verified with Playwright test.

**Backend tasks**:
1. Create project structure: `backend/`, `frontend/`, `storage/`, `docs/`.
2. Create Python venv from opencode venv (copy, then symlink opencv/numpy/pillow in if needed).
3. If network unavailable: request user to approve one pip install step.
4. Bootstrap FastAPI app with lifespan, SQLAlchemy async (aiosqlite), Alembic migrations.
5. SQLite WAL mode pragma in startup.
6. `Video` model: id, filename, original_name, path, size, duration, width, height, fps, codec, thumbnail_path, proxy_path, status, created_at, error.
7. `Job` model: id, type, payload, status, started_at, completed_at, error, retries.
8. Upload endpoint: validate mime/extension/size, save to `storage/uploads/`, enqueue metadata+thumbnail job, return video id.
9. Job worker: async loop, extract metadata with ffprobe (JSON), generate thumbnail with ffmpeg, generate proxy (480p) with ffmpeg, update DB.
10. SSE endpoint: stream job status events.
11. Video list, detail, delete endpoints.
12. Static file serving for thumbnails, proxies, uploads.

**Frontend tasks**:
1. `npm create vite@latest frontend -- --template react-ts` (needs network or cached Vite).
2. Install Tailwind CSS.
3. Layout: sidebar nav, main content area.
4. Upload page: `<input type="file" ref={inputRef} style={{display:'none'}} accept="video/*" onChange={handleFiles}>`, upload button calls `inputRef.current?.click()`, drag-and-drop on container div with `onDragOver`/`onDrop`.
5. Upload progress bar using SSE.
6. Video library grid: thumbnail, name, duration, status, actions (play, delete).
7. Video player: `<video>` tag with `src` pointing to `/api/videos/{id}/stream`.
8. Validation feedback (file type, size limit 10 GB).

**Playwright test**:
- Generate test video: `ffmpeg -f lavfi -i testsrc=duration=5:size=320x240:rate=25 -c:v libx264 /tmp/test.mp4`
- Test: launch browser → navigate to app → trigger upload → `waitForEvent('filechooser')` → set files → wait for video to appear in library → click play → assert video element `readyState >= 2`.

---

### Stage 2 — Campaigns and Guideline Extraction

**Goal**: Campaign CRUD, document upload (PDF/DOCX/TXT), text extraction, requirement parsing, mandatory vs recommended, conflict detection, approval workflow.

**Backend tasks**:
1. `Campaign` model: id, name, description, platform, aspect_ratio, resolution, min_duration, max_duration, required_hashtags, required_mentions, status, created_at.
2. `CampaignDocument` model: id, campaign_id, filename, path, extracted_text, status.
3. `CampaignRequirement` model: id, campaign_id, document_id, text, evidence, is_mandatory, is_approved, conflicts_with, created_at.
4. Document processing: PDF → `pdftotext`, DOCX → `soffice --headless --convert-to txt`, TXT → direct read, images → ffmpeg frame + mediapipe/pillow (tesseract not available; mark OCR as unavailable).
5. Requirement extraction: regex + heuristic rules (look for MUST/SHALL/REQUIRED/MAY/SHOULD patterns, duration patterns, aspect ratio patterns, platform keywords, hashtag patterns).
6. Conflict detection: same field with contradictory values (two different aspect ratios, min > max duration, etc.).
7. Approval endpoint: mark individual requirements as approved/rejected.
8. Unapproved requirements flagged in compliance engine — never silently applied.

**Frontend tasks**:
1. Campaign creation form.
2. Campaign list / detail page.
3. Document upload panel (similar to video upload but PDF/DOCX/TXT).
4. Requirement review table: text, evidence, mandatory/recommended badge, approve/reject buttons, conflict indicator.
5. Requirement history timeline.

---

### Stage 3 — Video Analysis

**Goal**: Deterministic analysis pipeline — scene detection, audio energy, silence detection. Optional speech-to-text with faster-whisper if model is available. Clear signal labelling.

**Backend tasks**:
1. `VideoAnalysis` model: id, video_id, scenes (JSON), audio_energy (JSON), silence_regions (JSON), speech_regions (JSON/null), transcript (JSON/null), analysis_method (JSON), status, created_at.
2. Scene detection: OpenCV frame differencing (frame delta > threshold → scene boundary). Label as **measured signal**.
3. Audio extraction: `ffmpeg -i input -af silencedetect=noise=-40dB -vn -f null /dev/null` for silence. `ffmpeg -i input -af astats -f null /dev/null` for energy. Label as **measured signal**.
4. Speech detection: if `faster-whisper` importable, use tiny model; otherwise mark as **unavailable**. Never claim silence = speech.
5. Interesting moment scoring: heuristic combining scene activity, audio energy peaks, non-silence. Label as **heuristic score**.
6. Analysis job enqueued after video processing completes.
7. SSE progress for analysis.

**Frontend tasks**:
1. Analysis panel on video detail page.
2. Timeline visualisation: scenes, energy, silence, speech (if available), scored segments.
3. Clear signal-type labels (measured / heuristic / model / unavailable).

---

### Stage 4 — Clip Candidates

**Goal**: Generate non-overlapping candidates, ranked, with reasons and thumbnails. Approve/reject/regenerate/adjust.

**Backend tasks**:
1. `ClipCandidate` model: id, video_id, campaign_id, start_time, end_time, score, score_breakdown (JSON), reason, thumbnail_path, status (pending/approved/rejected), created_at.
2. Candidate generation: sliding window over scored segments, merge overlapping high-score windows, enforce minimum gap, rank by composite score.
3. Score breakdown: scene_activity_score, audio_energy_score, non_silence_ratio, duration_fit_score (vs campaign min/max). All labelled.
4. Thumbnail: `ffmpeg -ss {start} -i input -frames:v 1 thumb.jpg`.
5. Regenerate endpoint: re-run generation with optional parameter adjustments.
6. Adjust endpoints: update start/end times, re-score.

**Frontend tasks**:
1. Candidates list: thumbnail, timestamp range, score bar, score breakdown tooltip, reason text.
2. Approve/Reject/Regenerate buttons.
3. Adjust panel: timeline scrubber for start/end.

---

### Stage 5 — Compliance Engine

**Goal**: Per-candidate compliance report against approved campaign requirements only. Statuses: satisfied / failed / unverifiable / manual_action.

**Backend tasks**:
1. `ComplianceReport` model: id, candidate_id, campaign_id, checks (JSON array of {requirement_id, status, evidence, detail}), overall_status, created_at.
2. Checks implemented:
   - Duration: compare candidate duration vs campaign min/max → satisfied/failed.
   - Aspect ratio: compare source vs required (before export) → note it will be enforced at export time → unverifiable at this stage.
   - Required hashtags: cannot verify from video alone → manual_action.
   - Required mentions: cannot verify → manual_action.
   - Required audio: cannot verify → unverifiable.
   - Content restrictions: cannot auto-verify → manual_action.
3. Only run against requirements where `is_approved = True`.
4. Report endpoint + per-candidate compliance badge.

**Frontend tasks**:
1. Compliance report panel per candidate.
2. Status badges: green (satisfied), red (failed), amber (unverifiable), blue (manual action required).
3. Never show requirement as "satisfied" without evidence.

---

### Stage 6 — Editing and Export

**Goal**: Trim, aspect ratio, reframe, resolution, quality, captions, text overlays, FFmpeg render with progress, ffprobe validation, export management.

**Backend tasks**:
1. `ExportJob` model: id, candidate_id, campaign_id, settings (JSON), status, output_path, ffmpeg_cmd, progress, error, created_at.
2. Settings: start_time, end_time, aspect_ratio, resolution, quality (crf), caption_text, caption_style, text_overlay, audio_options.
3. FFmpeg command builder: construct filter_complex for crop/scale/pad, drawtext for captions, trim, re-encode.
4. Hardware acceleration: try `-hwaccel qsv` or `-hwaccel vaapi` (Intel UHD 620), fall back to CPU libx264.
5. Progress: parse ffmpeg stderr `time=HH:MM:SS.ms` lines via SSE.
6. Post-render: validate with `ffprobe -v error -show_entries ...` — verify duration, codec, dimensions. Mark failed if validation fails.
7. Export management: list exports by campaign, download endpoint, delete endpoint.

**Frontend tasks**:
1. Edit panel: video preview, start/end sliders, aspect ratio selector, resolution dropdown, CRF slider, caption input, text overlay input.
2. Render button with live progress bar.
3. Export library page: list of rendered clips, campaign grouping, download/delete buttons.

---

### Stage 7 — End-to-End Test

Run the complete acceptance-criteria workflow from BRIEF.md §15:
1. Launch app.
2. Create campaign.
3. Upload guidelines PDF.
4. Review requirements.
5. Upload source video.
6. Play video.
7. Run analysis.
8. Generate candidates.
9. Review candidates and compliance.
10. Select and adjust clip.
11. Render.
12. Export.
13. Find exported file.

Report actual results, failures, known limitations, remaining bugs.

---

## Architecture Decisions

### Python Package Strategy

**Problem**: Sandbox blocks network; `pip install` fails.  
**Solution**: Create the project venv by **copying the opencode venv** (which has OpenCV, NumPy, Pillow) and then bootstrapping pip inside it to install pure-Python packages from the pip wheel bundled with Python (`python3 -m ensurepip`). Pure-Python packages (FastAPI, SQLAlchemy, Alembic, uvicorn, aiofiles, python-multipart, aiosqlite, starlette, pydantic, etc.) must be installed with network access.

**Practical plan**: Request user to allow one `pip install` command with BypassSandbox=true (network access). This is a one-time setup step. After install, all running and testing happens inside the sandbox.

### Database

SQLite with WAL mode, managed via Alembic migrations, accessed via SQLAlchemy async (aiosqlite). One database file at `storage/emmy.db`.

### Job Queue

Python asyncio-based persistent job queue backed by the `jobs` table. No Celery, no Redis, no second process. Worker runs as background tasks within the FastAPI lifespan.

### Frontend Build

Vite + React 18 + TypeScript + Tailwind CSS. Dev server proxies `/api` to FastAPI. Production build served by FastAPI as static files.

### File Storage

All files stored under `storage/`:
- `storage/uploads/` — original uploaded videos
- `storage/documents/` — campaign documents
- `storage/thumbnails/` — video and candidate thumbnails
- `storage/proxies/` — 480p preview proxies
- `storage/exports/` — rendered export files

---

## Open Questions / Blockers

1. **Network access for pip**: Need user approval for one `pip install` with BypassSandbox. Alternative: user pre-installs packages manually.
2. **Tesseract binary**: Only the library is installed; no `tesseract` CLI. OCR for image documents will be marked as unavailable unless the user installs `tesseract-ocr`.
3. **Playwright**: Not installed; needed for Stage 1 test. Requires network or manual install.
4. **faster-whisper**: Not installed; speech transcription will be marked unavailable until user installs it.
5. **Frontend package manager**: `npm` is available but `npm install` requires network. Need user approval.

---

_All open questions will be noted in PROGRESS.md and raised with the user before each affected stage._
