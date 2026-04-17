# Implementation Roadmap

## Milestone 0 — Foundation (1–2 days)
- Initialize app shell and project structure.
- Configure Cloudflare Access-protected deployment target.
- Set up CI basics and lint/format scripts.

## Milestone 1 — Upload + Queue Skeleton (2–3 days)
- Build upload initiation endpoint.
- Create audiobook/job records.
- Enqueue first processing messages.
- Implement job state polling in client.

## Milestone 2 — Parsing + Cleanup (3–5 days)
- Implement parsers for `.md`, `.txt`, `.epub`, `.docx`, `.pdf`.
- Add deterministic cleanup rules.
- Add optional LLM cleanup pass for uncertain blocks.
- Save cleaned canonical text artifact.

## Milestone 3 — Segmentation + TTS (3–5 days)
- Implement synthesis segmentation and navigation chunking.
- Integrate TTS provider for English voice.
- Persist audio artifacts and build timing manifest.

## Milestone 4 — Playback UI (3–4 days)
- Mobile-first reader/player layout.
- Implement controls: play/pause, ±10s, scrubber, speed, chapter jump, sleep timer.
- Tap-to-seek on text chunks with auto-scroll highlighting.
- Save and restore playback progress.

## Milestone 5 — Reliability + Polish (2–4 days)
- Failure queue with reason and retry.
- Graceful cancellation behavior.
- Add instrumentation and failure diagnostics.
- UX refinements and regression checks.

## Definition of Done (MVP)
- Upload from all supported file formats works end-to-end.
- A produced audiobook appears in library and streams successfully.
- Read-along chunk highlighting and tap-to-seek function reliably.
- Failed jobs are visible with actionable retry.
- Progress resumes correctly after refresh/reopen.

## Risks and Mitigations
- **PDF/EPUB extraction quality** → Start with deterministic cleaners and review sample corpus early.
- **TTS latency/cost spikes** → Batch synthesis, cache artifacts, monitor per-doc timings.
- **Sync drift** → Use smaller navigation chunks and conservative timestamp interpolation.
