# Architecture Plan (Cloudflare-First MVP)

## 1) High-Level Components
- **Client (Web App)**: mobile-first reader/player UI.
- **API Layer (Cloudflare Workers)**: upload initiation, library APIs, playback metadata, job actions.
- **Queue (Cloudflare Queues)**: asynchronous processing orchestration.
- **Database (Cloudflare D1)**: job metadata, audiobook metadata, chapters/chunks, playback progress.
- **Object Storage (Cloudflare R2)**: uploaded source files, cleaned intermediate text, audio assets, manifests.
- **External Providers**:
  - Document parsing/OCR libraries/services as needed.
  - TTS provider (English natural voice).
  - Optional lightweight LLM cleanup pass for ambiguous text cleanup.

## 2) Processing Pipeline
1. **Upload Init**: client requests presigned upload path/token.
2. **Upload Complete**: API creates `job` and enqueues pipeline start.
3. **Parse**: extract normalized text + structure from source format.
4. **Clean**: deterministic cleanup then optional LLM cleanup fallback.
5. **Segment**:
   - synthesis segments (larger)
   - navigation chunks (2–3 sentences)
6. **Synthesize**: generate audio for synthesis segments.
7. **Package**:
   - build chapter/chunk timestamp manifest
   - write audiobook metadata
8. **Done/Failed**: finalize state and surface reason if failed.

## 3) Why No Durable Objects in MVP
- Current workload is low-volume and single user.
- Queue + D1 state machine is sufficient.
- Avoid additional architectural complexity now.

## 4) Concurrency & Reliability
- Max 3 jobs in parallel globally.
- Each job step should be idempotent.
- Retry strategy:
  - transient failures retried with backoff
  - terminal failures moved to failed queue with human-readable reason
- Graceful cancellation flag checked between step boundaries/chunk boundaries.

## 5) Data Flow (Conceptual)
- Client → Worker API: create upload/job, fetch status and manifests.
- Worker API → R2: store source and packaged assets.
- Worker API → Queue: enqueue state transitions.
- Workers/Consumers → D1: update job states and metadata.
- Client → Streaming endpoint / asset URLs in R2: playback audio.

## 6) Security Model
- Cloudflare Access protects the entire app/API surface.
- Only authenticated requests can upload/fetch metadata.
- Signed access pattern for private audio artifacts if needed.

## 7) Observability
- Structured logs per `job_id` and `audiobook_id`.
- Step timing metrics (parse/clean/synth/package).
- Failure taxonomy for quick triage (input parse, OCR, provider errors, storage errors).

## 8) Scaling Path (Post-MVP)
- Add provider abstraction for TTS/cleanup.
- Add word-level alignment service if required.
- Introduce Durable Objects only for advanced real-time session coordination.
