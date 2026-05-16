---
name: Background worker — deferred jobs queue (TODO)
description: Need a dedicated worker process for deferred jobs (actor cache refresh, federation retries, image work) before scaling
type: project
---
Many maintenance jobs currently run inline (or fire-and-forget) inside request handlers. Before scaling past alpha, factor these into a single background worker with a job queue.

**Why:** today these run on the same Node process that serves requests, with no retries, no observability, no backpressure, and no separation between user-facing latency and bulk write fan-out. A user with thousands of posts editing their profile triggers a synchronous `Promise.all` of `updateMany` against half the database. A failed S3 upload or remote inbox delivery gets logged once and forgotten.

**How to apply:** when adding any new "do this asynchronously after request" path, prefer enqueuing into the (yet-to-be-built) worker over `Function().catch(() => {})` patterns. Worth introducing alongside the worker: durable jobs (mongo-backed or BullMQ), retry/backoff, dead-letter queue, structured logging, basic metrics.

**Initial backlog of jobs that should move into it:**
- Actor cache refresh (`methods/users/refreshActorCache.js`) — denormalized actor copies on profile update
- Remote actor revalidation — periodic refresh of `Member.lastFetchedAt` for stale federated users
- Federation push retries — retry deliveries that fail after the first attempt
- Federation pull (`pollWorker.js`) — already standalone via pm2, could fold into the main worker
- Thumbnail backfill / re-transcode — currently a one-off script
- Email sending — once SMTP is wired up
- File cleanup — orphaned File records / S3 objects after deletes

Already a separate pm2 process for `kowloon-worker-feed`; this is the natural place to expand or fork into a `kowloon-worker-jobs` process with a shared queue.
