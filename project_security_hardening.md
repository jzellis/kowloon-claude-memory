---
name: Security hardening — completed 2026-04-29
description: Security work completed in the session of 2026-04-29 covering XSS, CORS, rate limiting, file validation, SSRF, and storage
type: project
---

All items below completed and pushed as of 2026-04-29.

**Why:** Pre-alpha security audit identified multiple attack surfaces across XSS, federation, file handling, and SSRF.

**XSS prevention**
- All schemas (Post, Reply, Page, Bookmark) run content through `safeMarkdown()`: marked() output piped through sanitize-html with an allowlist. Applied in pre-save hooks.
- Frontend EventCard and all other post type cards use server-rendered `post.body` (not client-side marked()); EventCard was the last holdout.

**CORS**
- Replaced `cors()` wildcard with domain-aware allowlist: own domain + `CORS_ORIGIN` env var (comma-separated). Localhost ports 5173/3000 added only when `NODE_ENV !== "production"`.
- Body limit reduced from 100MB to 1MB (`express.json` + `express.urlencoded`).

**Rate limiting + deduplication**
- `routes/middleware/rateLimiter.js`: `outboxRateLimiter` on POST /outbox, `inboxRateLimiter` on POST /inbox (moved to `routes/inbox/index.js` — route() wrapper doesn't support a middleware option).
- `activityDeduplicator` middleware on POST /outbox: sha256 hash of actor+type+objectType+to+content, 30s TTL window, returns 409 on duplicate.

**File upload security**
- `methods/files/validateUpload.js`: MIME allowlist (images/audio/video only), magic byte detection via `file-type`, SVG sanitization via sanitize-html with SVG-specific element/attribute allowlist, raster image re-encoding via sharp to strip EXIF/embedded payloads.
- Wired into both `routes/files/upload.js` (direct uploads, 415 on rejection) and `methods/files/proxyExternalImage.js` (og:image proxying).

**SSRF protection**
- `proxyExternalImage.js`: `isSafeUrl()` resolves hostname via DNS and checks all private IPv4/IPv6 ranges before any outbound fetch.

**Storage — S3-only + presigned URLs**
- Removed Azure/GCS/Local adapters. `StorageManager.js` is now S3-only singleton.
- `routes/files/serve.js`: 60s presigned redirect instead of streaming (eliminates JWT-in-URL for direct file access).
- Post-serving routes (collection.js, id.js, circles/posts.js): embed 1-hour presigned URLs directly in JSON so `<img>` tags never hit the proxy.

**Federation group fan-out fix**
- `routes/inbox/post.js`: when `body.to` starts with `group:`, verify that the signing server's keyId domain matches the group's host domain. Prevents a malicious server from fan-outing posts into a group it doesn't own.

**How to apply:** These are all live. No further action needed unless a new adapter type is needed (would require re-adding to StorageManager).
