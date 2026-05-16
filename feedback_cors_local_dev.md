---
name: Local dev CORS — NODE_ENV=production blocks frontend
description: Server .env has NODE_ENV=production which excludes localhost:5173 from the CORS allowlist; fix with CORS_ORIGIN
type: feedback
---

Add `CORS_ORIGIN=http://localhost:5173` to `server/.env` when running the frontend dev server against a local server instance.

**Why:** `server/index.js` only adds `http://localhost:5173` and `http://localhost:3000` to the CORS allowlist when `NODE_ENV !== "production"`. The local `.env` has `NODE_ENV=production`, so those origins are excluded and browser requests from the Vite dev server get silently blocked.

**How to apply:** Whenever setting up a fresh local dev environment or after wiping `.env`, add `CORS_ORIGIN=http://localhost:5173`. The `CORS_ORIGIN` env var accepts comma-separated origins and is always merged into the allowlist regardless of NODE_ENV.
