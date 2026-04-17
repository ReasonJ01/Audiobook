# Audiobook Web App — Product Requirements (MVP)

## 1) Product Summary
A personal, mobile-first web application that converts uploaded documents into synchronized audiobooks with read-along text and playback controls.

## 2) Goals
- Convert uploaded docs into high-quality English audio automatically.
- Let the user read along while listening with chunk-level highlighting.
- Allow tap-to-seek navigation by selecting text chunks.
- Keep the UX simple and fast for a single personal user.

## 3) Non-Goals (MVP)
- Multi-user accounts/teams.
- Multilingual narration.
- Word-level forced alignment.
- Advanced search across library content.
- Custom pronunciation dictionary and multiple voice profiles.

## 4) Target User
- One user (owner) protected by Cloudflare Access.

## 5) Supported Inputs
- `.md`, `.txt`, `.epub`, `.docx`, `.pdf`

## 6) Functional Requirements

### 6.1 Upload & Processing
- User can upload one or more documents.
- Processing starts automatically after upload.
- Jobs support up to 3 concurrent documents.
- Job lifecycle states:
  - `queued`
  - `parsing`
  - `cleaning`
  - `segmenting`
  - `synthesizing`
  - `packaging`
  - `done`
  - `failed`
  - `canceled`
- User can gracefully cancel a running job.
- Failed jobs show a reason and a retry action.

### 6.2 Text Cleanup & Structuring
- Preserve chapter and paragraph structure where possible.
- Remove obvious layout artifacts (e.g., page numbers, repeated headers/footers, OCR noise).
- Use conservative cleanup; do not alter meaning.
- Footnotes should be read at end-of-paragraph.
- Code blocks and tables should be omitted with a brief listener note.

### 6.3 Audio Generation
- English-only TTS.
- Single high-quality natural voice.
- Persist generated audio in object storage.
- Audio generation and navigation segmentation are decoupled:
  - Larger synthesis segments for efficiency.
  - Smaller navigation chunks (2–3 sentences) for UX.

### 6.4 Playback & Read-Along
- Mobile-first, text-forward UI with controls docked at bottom.
- Controls:
  - Play/Pause
  - Skip back 10s
  - Skip forward 10s
  - Scrubber/progress bar
  - Speed control `0.5x–2.0x`
  - Chapter jump
  - Sleep timer
- Auto-scroll read-along.
- Tap a chunk to jump and immediately play.
- Persist listening progress.

### 6.5 Library
- Flat list of audiobooks.
- Recently played section.
- Each upload is a new audiobook entry (no versioning).

## 7) Non-Functional Requirements
- Responsive mobile-first UI.
- Idempotent processing steps for safe retries.
- Durable metadata in a relational store.
- Object storage for source and generated artifacts.
- Clear failure visibility and recovery path.

## 8) Success Metrics (MVP)
- ≥95% of uploads complete without manual intervention.
- Time-to-first-play acceptable for personal usage (<10 minutes for moderate docs).
- Tap-to-seek and highlight tracking stay aligned at chunk level for most content.

## 9) Open Items to Revisit Post-MVP
- Better drift correction strategies.
- Notification model (in-app push or email).
- Voice presets and pronunciation dictionary.
- Search and annotation features.
