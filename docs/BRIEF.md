# Emmy Clip Studio — Complete Product Implementation Brief

## 1. Your Role

Act as the lead software architect, senior full-stack developer, AI engineer, and QA engineer responsible for completing **Emmy Clip Studio**.

Your responsibility is to turn the existing project into a fully functional, reliable, AI-powered video clipping platform.

Do not treat this as a prototype, UI demonstration, or collection of disconnected features. Build a cohesive application in which the complete workflow works from beginning to end.

You have permission to inspect, modify, create, and improve project files as necessary.

## 2. Product Vision

Emmy Clip Studio is a desktop-friendly, local-first AI video clipping platform designed to help content creators produce multiple engaging short-form clips from long-form videos.

The platform must also understand campaign guidelines.

Users should be able to upload campaign instructions, upload source videos, allow the application to analyse the content, generate suitable clips, check those clips against campaign requirements, make adjustments, and export finished videos.

The application should automate as much of the clipping workflow as reasonably possible.

The goal is to reduce manual editing and make campaign-based content creation significantly faster.

## 3. Primary User Workflow

The complete workflow should be:

1. Create a campaign.
2. Upload campaign guidelines or paste campaign instructions.
3. Extract campaign requirements automatically.
4. Review and approve the extracted requirements.
5. Upload one or more source videos.
6. Analyse the videos automatically.
7. Identify interesting moments and potential highlights.
8. Generate multiple clip candidates.
9. Evaluate candidates against campaign requirements.
10. Allow the user to review and select clips.
11. Edit clip boundaries and configure output settings.
12. Render and export the finished clips.
13. Organise exported clips under their respective campaigns.

The user should not need to understand video editing or command-line tools to complete this workflow.

## 4. Campaign Intelligence

Users must be able to create and manage multiple campaigns.

Each campaign should support:

- Campaign name and description.
- Campaign instructions.
- Platform requirements (TikTok, YouTube Shorts, Instagram Reels, etc.).
- Required video aspect ratio and resolution.
- Required hashtags and captions.
- Required mentions and tags.
- Required audio or music.
- Minimum and maximum clip duration.
- Content restrictions.
- Other campaign-specific requirements.

Users should be able to upload PDF, DOCX, TXT, and supported image documents containing campaign guidelines.

The system should:

- Extract text from uploaded documents.
- Use OCR when necessary.
- Identify individual requirements.
- Preserve the source and evidence for each extracted requirement.
- Distinguish mandatory requirements from recommendations.
- Detect conflicting requirements.
- Allow users to resolve conflicts.
- Maintain requirement history.
- Never silently treat unapproved extracted rules as confirmed rules.

Campaign rules must be applied during clip evaluation and export configuration.

## 5. Video Ingestion and Management

Users must be able to upload local video files.

Support common formats including MP4, MOV, MKV, WebM, and AVI where technically supported.

The application should:

- Provide a reliable file picker.
- Support drag-and-drop uploads.
- Display upload progress.
- Validate file types and sizes.
- Extract video metadata.
- Generate thumbnails.
- Create lightweight preview proxies.
- Support video playback and seeking.
- Allow users to delete videos.
- Display clear processing states and useful errors.

Multiple source videos should be supported.

The upload workflow must be thoroughly tested. A clickable upload button that does not open the operating system's file picker is not acceptable.

## 6. AI Video Analysis

Analyse source videos to identify potentially useful moments.

Use a hybrid approach combining deterministic video-processing techniques and locally available AI models where practical.

Available analysis signals may include:

- Scene transitions.
- Scene duration.
- Audio energy and changes in intensity.
- Silence detection.
- Speech detection where an actual speech-recognition model is available.
- Transcription and dialogue.
- Visual composition.
- Faces and subjects.
- Relevant visual changes.
- Temporal continuity.

Do not pretend silence detection is speech recognition.

Do not claim that scene changes automatically identify interesting moments.

Clearly distinguish measured signals, model predictions, and heuristic scores.

The application should be capable of progressively improving its analysis pipeline as more local models become available.

## 7. Automatic Clip Generation

This is one of the most important features.

Generate multiple candidate clips from each source video.

Candidates should:

- Have accurate start and end timestamps.
- Avoid unnecessary overlapping selections.
- Preserve enough context to make moments understandable.
- Respect campaign duration requirements where possible.
- Have a clear reason for being selected.
- Include useful preview thumbnails.
- Be ranked using transparent scoring criteria.

Where speech transcription or visual AI is available, use it to improve highlight identification.

Do not label arbitrary scene segments as viral moments or claim heuristic scores represent probabilities of success.

Users must be able to approve, reject, regenerate, and adjust candidates.

## 8. Campaign Compliance Engine

Every generated candidate should be evaluated against the approved campaign requirements.

The system should identify:

- Requirements satisfied.
- Requirements not satisfied.
- Requirements that cannot be verified automatically.
- Potential conflicts.
- Required manual actions.

Examples:

- Incorrect aspect ratio.
- Clip duration outside campaign limits.
- Missing required hashtags.
- Missing required mentions.
- Required audio not present or not verifiable.
- Content restrictions that require human review.

Do not claim that a requirement is satisfied unless the system has sufficient evidence.

Provide a clear compliance report for each candidate.

## 9. Editing and Rendering

Provide a simple editing interface, not a complicated professional multitrack editor.

Users should be able to:

- Preview selected clips.
- Adjust start and end timestamps.
- Trim unwanted sections.
- Select aspect ratio.
- Crop or reframe video.
- Configure output resolution.
- Choose output quality.
- Add captions or subtitles where supported.
- Add text overlays.
- Configure supported audio options.
- Preview the final output.
- Export multiple clips.

Use FFmpeg for reliable rendering.

Support hardware acceleration where available, with a dependable CPU fallback.

Rendering must report progress and provide useful error messages.

Never silently produce incomplete or corrupted output files.

## 10. User Interface

Build a clean, modern, professional interface.

The application should include:

- Dashboard.
- Campaign management.
- Campaign guideline management.
- Video library.
- Video analysis workspace.
- Clip candidates.
- Compliance reports.
- Editing workspace.
- Export management.
- Application settings.

Prioritise usability over unnecessary visual effects.

Use clear navigation, consistent components, readable typography, useful empty states, progress indicators, and responsive layouts.

Avoid overwhelming beginners with technical information.

## 11. Technical Requirements

The existing project already contains substantial implementation work.

Inspect the current repository before making architectural decisions.

Existing architecture includes:

- React 18.
- TypeScript.
- Vite.
- Tailwind CSS.
- FastAPI.
- Python.
- SQLAlchemy async.
- SQLite with WAL mode.
- Persistent job queue.
- Server-Sent Events for progress updates.
- FFmpeg and FFprobe.
- Local file storage.
- Campaign intelligence and video-processing modules.

Preserve and improve working components.

Do not unnecessarily replace the database, job queue, migration system, or existing architecture.

Do not create a second database or independent job-processing system.

Use database migrations for schema changes.

Maintain proper separation between frontend, API routes, services, database models, and background processing.

## 12. AI and Resource Constraints

The application must be local-first.

Priorities:

1. Free and open-source technologies.
2. Local processing.
3. CPU compatibility.
4. Optional hardware acceleration.
5. Optional local AI models.

Do not introduce paid APIs, mandatory subscriptions, or cloud processing.

The primary development machine is a Lenovo ThinkPad X280 with:

- Intel Core i5-8250U.
- 16GB RAM.
- Intel UHD 620 graphics.
- Debian Linux.

Design the application to work within these limitations.

Do not assume that large AI models can run efficiently on this hardware.

Use lightweight models and asynchronous processing where appropriate.

Provide graceful fallbacks when a model or hardware acceleration feature is unavailable.

## 13. Reliability and Security

Implement:

- Proper upload validation.
- Safe file handling.
- Path traversal protection.
- File-size restrictions.
- Secure document processing.
- Database integrity.
- Job recovery where practical.
- Clear processing states.
- Error logging.
- Safe deletion.
- Input validation.
- Protection against duplicate processing.
- Reliable export handling.

Do not expose internal filesystem paths or sensitive system information unnecessarily.

## 14. Implementation Instructions

Before changing anything:

1. Inspect the complete repository.
2. Understand what is already implemented.
3. Identify incomplete features, bugs, duplicated logic, and architectural weaknesses.
4. Pay particular attention to the current video upload failure: clicking the upload interface does not open the file picker, and no browser-console error appears.
5. Trace the actual frontend interaction and identify the cause.

Then develop a complete implementation plan.

After that, execute the plan.

You should work autonomously through the implementation stages rather than stopping after producing another proposal or architecture document.

However:

- Implement in manageable stages.
- Test each major feature before moving on.
- Fix regressions immediately.
- Preserve working functionality.
- Do not skip tests simply because a feature appears to work.
- Do not invent successful test results.
- Do not report a feature as complete unless it has been implemented and verified.
- Do not make destructive architectural changes without explaining why.

Where a feature cannot reasonably work on the target hardware, explain the limitation and implement the best practical alternative.

## 15. Final Acceptance Criteria

The project is not complete until a user can:

1. Launch the application.
2. Create a campaign.
3. Upload campaign guidelines.
4. Review extracted campaign requirements.
5. Upload local videos successfully.
6. Play and inspect uploaded videos.
7. Analyse videos.
8. Generate multiple clip candidates.
9. Review candidate explanations and timestamps.
10. Check campaign compliance.
11. Select and adjust clips.
12. Render clips.
13. Export playable video files.
14. Find and manage their exported clips.

Test the complete workflow, not just isolated API endpoints.

At completion, provide:

- Implemented features.
- Architecture changes.
- Tests performed and actual results.
- Known limitations.
- Remaining bugs.
- Instructions for launching and using the application.

**The objective is a genuinely working Emmy Clip Studio, not another partially completed development milestone.**