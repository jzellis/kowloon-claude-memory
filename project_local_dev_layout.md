---
name: project-local-dev-layout
description: Local dev stack on this machine — host PM2 runs Kowloon server/worker/frontend; MongoDB and MinIO come from the dockerized federation compose stack.
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

The host has a dual stack:

- **Dockerized federation** (`kowloon-federation-*` containers, started long ago): MongoDB on host `:27018` (mapped from container `:27017`), MinIO on `:9000`/`:9001`. Plus three full Kowloon app containers (`kowloon1/2/3`) behind nginx on `:8080`/`:8443`, using databases `kwln1/kwln2/kwln3` and the matching MinIO buckets. These are the federation testbed and run independently.
- **Host PM2 instance** (the one to develop against): PM2 apps `kowloon` (server on `:3000`), `kowloon-worker-feed`, and `kowloon-frontend` (Vite on `:5173`). Uses a separate `kowloon` MongoDB database and `kowloon` MinIO bucket on the same containers as above — isolated from the federation data. Ecosystem files: `server/ecosystem.config.cjs` and `frontend/ecosystem.config.cjs`.

Frontend `.env.local` points `VITE_SERVER_URL` at `http://100.83.23.39:3000` (Tailscale IP + host PM2 port). Server `.env` has `MONGO_URI=mongodb://localhost:27018/kowloon` and `S3_BUCKET=kowloon`. Domain stays `localhost`.

**Why:** jzellis SSHes in over Tailscale and develops via PM2 (survives SSH logout, daemon already running). The federation containers are kept up as reference but should not be the dev target. Stopping them would also stop the MongoDB and MinIO that the host PM2 instance depends on.

**How to apply:** When adding services, write PM2 entries in the appropriate ecosystem file. Don't rebuild `docker-compose.yml` — the federation stack is intentional. If port 27018 or 9000 stops responding, check `docker ps` for the federation containers before assuming local install issues. See [[project-kowloon-seed-actor-gap]] for seed quirks specific to this host setup.
