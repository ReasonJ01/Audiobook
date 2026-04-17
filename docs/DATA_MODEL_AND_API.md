# Data Model and API Contract (MVP Draft)

## 1) Core Entities

### `audiobooks`
- `id` (uuid, pk)
- `title` (text)
- `source_format` (text)
- `status` (enum: processing|ready|failed|canceled)
- `created_at` / `updated_at`
- `duration_seconds` (int, nullable)
- `cover_url` (nullable)

### `jobs`
- `id` (uuid, pk)
- `audiobook_id` (fk)
- `state` (enum from pipeline states)
- `failure_code` (nullable)
- `failure_reason` (nullable)
- `cancel_requested` (bool)
- `attempt_count` (int)
- `created_at` / `updated_at`

### `chapters`
- `id` (uuid, pk)
- `audiobook_id` (fk)
- `idx` (int)
- `title` (text)
- `start_time_seconds` (float)
- `end_time_seconds` (float)

### `chunks`
- `id` (uuid, pk)
- `audiobook_id` (fk)
- `chapter_id` (fk)
- `idx` (int)
- `text` (text)
- `start_time_seconds` (float)
- `end_time_seconds` (float)

### `playback_progress`
- `audiobook_id` (pk/fk)
- `last_position_seconds` (float)
- `last_chapter_idx` (int)
- `last_chunk_idx` (int)
- `updated_at`

### `artifacts`
- `id` (uuid, pk)
- `audiobook_id` (fk)
- `type` (source|clean_text|audio_segment|manifest)
- `r2_key` (text)
- `content_type` (text)
- `bytes` (int)
- `created_at`

## 2) API Endpoints (Draft)

### Upload & Jobs
- `POST /api/uploads/init`
  - request: `{ filename, contentType }`
  - response: `{ uploadUrl, objectKey }`

- `POST /api/audiobooks`
  - request: `{ objectKey, title? }`
  - response: `{ audiobookId, jobId, status }`

- `GET /api/jobs/:jobId`
  - response: `{ id, state, failureReason?, progress? }`

- `POST /api/jobs/:jobId/retry`
- `POST /api/jobs/:jobId/cancel`

### Library & Playback
- `GET /api/audiobooks`
  - response: list with status, duration, recently played metadata

- `GET /api/audiobooks/:id`
  - response: metadata + chapters + current progress

- `GET /api/audiobooks/:id/manifest`
  - response: chunk/chapter timing map + audio source references

- `POST /api/audiobooks/:id/progress`
  - request: `{ positionSeconds, chapterIdx, chunkIdx }`

## 3) Queue Message Shapes

### `PROCESS_UPLOAD`
```json
{ "jobId": "uuid", "audiobookId": "uuid", "objectKey": "r2/path" }
```

### `RETRY_STEP`
```json
{ "jobId": "uuid", "audiobookId": "uuid", "step": "cleaning|synthesizing|..." }
```

### `CANCEL_JOB`
```json
{ "jobId": "uuid", "audiobookId": "uuid" }
```

## 4) Error Model
Standard error payload:
```json
{
  "error": {
    "code": "INVALID_FILE|PARSE_FAILED|TTS_FAILED|CANCELED|INTERNAL",
    "message": "Human-readable reason"
  }
}
```

## 5) State Transition Rules
- Legal transitions are explicit and enforced server-side.
- Any illegal transition is logged and rejected.
- `cancel_requested=true` can be set in any non-terminal state.
- Terminal states: `done`, `failed`, `canceled`.
