---
name: storage-streaming-model
description: "Files stream through the app at /files/:id from INTERNAL MinIO; no presigned URLs, no public storage surface, no S3_PUBLIC_URL. Supersedes the old federation storage gap."
metadata: 
  node_type: memory
  type: project
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

As of 2026-06-20, Kowloon serves files by **streaming bytes through the app** at `GET /files/:id` (`routes/files/serve.js` uses `storage.getStream()`). Object storage (MinIO/S3) is **internal-only** — a file's sole public URL is the app endpoint, already TLS-terminated by Caddy and reachable by federation peers.

**Why:** The old model 302-redirected to presigned S3 URLs, which forced a publicly-reachable storage endpoint (`S3_PUBLIC_URL`) — an extra public surface + DNS that broke federated images when misconfigured. Streaming keeps storage private and needs zero storage config. This is the "authenticated proxy" the architecture docs always described. Chosen for the first production install (kwln.social) where "just works / fewest variables" was the goal.

**How to apply:**
- Do NOT reintroduce presigned URLs into API responses or `serve.js`. The old `S3_PUBLIC_URL` "prod/federation gap" in `server/CLAUDE.md` is now OBSOLETE — `S3_PUBLIC_URL` is unused.
- Embedded image URLs (post feeds, single post, file meta) use `methods/files/signedUrl.js#buildFileUrl`: **plain** stable `/files/:id` for public files (cacheable, federation-friendly), **short-lived HMAC-signed** (`?exp=&sig=`, secret = `JWT_SECRET`) for restricted files so `<img>` loads work without leaking the viewer's JWT. `serve.js` accepts a valid sig as access to that one file; otherwise falls back to Bearer/`?token` + parent-visibility.
- `StorageManager` is S3-only (no local adapter ever existed) — the installer MUST provision MinIO. `setup/server.js` generates an internal `minio` + `minio-init` (private `kowloon` bucket) and writes `S3_ENDPOINT=http://minio:9000` etc. Never set `STORAGE_ADAPTER=local` (no-op that left fresh installs with broken file handling).
- Keep file URLs raw (no `encodeURIComponent` on the file id) to match upload.js/proxyExternalImage.js and the frontend `sizedUrl` regex (`/\/files\/file:/`).
