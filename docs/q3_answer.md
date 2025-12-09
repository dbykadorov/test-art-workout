## 3 Technologies, Protocols, and Constraints

### Realtime channel & payloads
- Transport: WebSocket (one duplex channel per session), TLS, auth via short-lived WS token from REST.
- Encoding: compact JSON or MsgPack; server-assigned `seq`, client `op_id`, `ts`.
- Event types: `presence`, `stroke_append` (batched), `undo`, `step_change`, `score_update`, `session_snapshot`, `ack`.
- Rate/size control: batch strokes every 50–100 ms; gzip/deflate at WS level; drop/merge tiny deltas under load.

### Backend tech choices
- Gateway: Node/NestJS WS gateway (TS parity with stack), horizontal via sticky sessions + Redis pub/sub.
- Ordering/log: Redis Streams (per session) or Kafka for strict ordering, backpressure, and replay.
- State cache: Redis for session metadata (participants, current step, last_seq).
- Scoring workers: NestJS worker or Node worker pool; GPU/WebAssembly optional for stroke analysis; writes to DB + object storage (replays).
- Persistence: Postgres (session/participants/scores), S3/GCS for blobs (stroke batches, snapshots).
- Feature flags/experiments: LaunchDarkly / GrowthBook; A/B on batch size, compression, stroke tolerance.

### React-side stack
- Data fetching/mutations: React Query (REST/gRPC for auth/session bootstrap).
- Realtime client: lightweight WS client with reconnect + heartbeat; hook that exposes events stream and `send`.
- State: Zustand/Redux for session state; canvas-layer state kept separately (e.g., internal store to avoid rerenders).
- Drawing: performant canvas (e.g., Skia/CanvasKit or optimized HTML Canvas); double-buffer to apply remote ops in order.
- Local storage: IndexedDB for offline cache of lesson media; replay cache for recent ops to fast-join.

### Ties to existing mechanics
- Step validation/scoring: server authoritative; `step_change` accepted only after server validates current step with tolerances.
- Undo: modeled as tombstone on prior `op_id`; idempotent handling matches current undo behavior.
- Scoring weights per step (accuracy/quality) guide payload schema (`step_id`, `scoring_weights`, `tolerance`).
- Progress/leaderboard: rely on server-side scores; client optimistic UI but final numbers from server.

### Data-flow (end-to-end example)
1) Client loads lesson/meta via REST (cached in React Query; media in IndexedDB).  
2) Client opens WS with token, receives `session_snapshot` (state, last_seq).  
3) User draws → local buffer batches points → `stroke_append` sent with `op_id`.  
4) Gateway assigns `seq`, writes to stream, fans out; clients apply in `seq` order to canvas.  
5) On step finish, server validates strokes, computes score, persists to Postgres + stores batch blob in S3; emits `score_update` and allows `step_change`.  
6) Late joiner reconnects with `last_seq`; gateway sends delta or full snapshot + tail.  
7) Analytics/telemetry (`EventLog`) emitted asynchronously to avoid blocking UI path.

### Constraints & calculations
- Latency target <150 ms RTT for perceptible synchronicity; batch window ≤100 ms.
- Bandwidth: cap stroke payloads (quantize points, drop redundant pressure samples); consider WebRTC data channel only if P2P cost is justified (likely not needed).
- Reliability: heartbeats + server timeouts; fast resume with `last_seq`.
- Security: per-session WS tokens, rate limits per connection/session, payload size guards.
- Observability: per-session metrics (latency, drop rate, seq gaps), structured logs with `session_id`/`seq`.

### Prior experience (relevant lesson)
In real-time collab (whiteboard/graphics) the essentials are server-side ordering and snapshots: periodic snapshots plus an operation log simplify late join/replay and reduce accumulated delay in long sessions; directly applicable here.

