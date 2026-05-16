---
name: Install system + frontend serving architecture
description: Decisions made about the installer, CI/CD pipeline, and how the frontend is served
type: project
---
## Install System (COMPLETE as of 2026-04-17)

- `install.sh` — curl-pipe-bash installer, served at `https://install.kwln.social`
- `setup/` — Docker-based web wizard (port 2999) that collects domain/admin info, writes `.env` + `Caddyfile` + `docker-compose.yml`, then exits
- `install-site/` — Caddy static server on kwln.social serving `install.sh`
- Health endpoint: `/health` (returns `{status:"ok"}`) and `/__health` (legacy alias)
- Admin user: `init.js` creates the account on first boot ONLY if `ADMIN_USERNAME` + `ADMIN_PASSWORD` are in env (set by installer). No fallback to 'admin'/random password.
- `install.sh` health poll uses `curl -sfk` (k = accept self-signed certs for .local domains)
- `deploy-install.yml` uses `overwrite: true` in scp-action so re-deploys don't fail

**Why:** One-command install for non-technical users, WordPress-style.

**How to apply:** Changes to `install.sh` auto-deploy via GitHub Actions (`deploy-install.yml`) on push to main. Requires SSH secrets: `INSTALL_HOST`, `INSTALL_USER`, `INSTALL_SSH_KEY`.

## CI/CD Pipeline (COMPLETE as of 2026-04-17)

- `.github/workflows/docker.yml` — builds and pushes two images on push to `main` or version tags:
  - `ghcr.io/jzellis/kowloon:latest` — multi-stage build: Vite frontend built first, then copied into server image
  - `ghcr.io/jzellis/kowloon-setup:latest` — install wizard
  - Checks out all three repos side by side (kowloon-server, kowloon-frontend, kowloon-client); build context is repo root, Dockerfile is `kowloon-server/Dockerfile.prod`
- `.github/workflows/deploy-install.yml` — SSHes to kwln.social and copies `install.sh` to `/srv/caddy/`

## Frontend Serving Architecture (FINAL DECISION: bundled, 2026-04-17)

**Decision: frontend is bundled into the server Docker image.**

- Vite build output is copied into `/app/public` in the server image
- Express serves static files from `public/` after API routes, with SPA fallback (`app.get("*")`)
- Caddy simply does `reverse_proxy app:3000` — no path-based routing needed
- `GET /config.json` returns `{ apiUrl, domain, siteTitle, registrationIsOpen }` for runtime config
  - Route mounted at `/config.json` via special case in `routes/index.js` auto-mounter
- Login/Register pages default `serverUrl` to `window.location.origin` when no `VITE_SERVER_URL` is set

**Why:** Every Kowloon instance should always have the web UI. Simpler architecture, no separate container, no Caddy path routing.

**How to apply:** Adding new top-level API routes does NOT require updating the Caddyfile. The SPA fallback only catches routes not matched by any Express handler, so API routes always win.

## Local Dev Setup

- MongoDB runs as a system service (`sudo systemctl start mongodb`) on port 27017
- Server runs via PM2 from `~/Projects/kowloon/server`
- Frontend runs via Vite dev server from `~/Projects/kowloon/frontend`
- Federation test environment uses `~/Projects/kowloon/server/docker-compose.yml` (separate MongoDB on port 27018)
