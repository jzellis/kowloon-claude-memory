---
name: project-production-servers
description: "Production server hostnames, SSH access, docker-compose location, deploy procedure, and seed credentials"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

## Production servers

| Server | Hostname | Notes |
|--------|----------|-------|
| kwln.social | kwln.social | Primary |
| kowloon.network | kowloon.network | Secondary |

## SSH access

SSH as `jzellis` with the local ed25519 key — no password needed:

```bash
ssh jzellis@kwln.social
ssh jzellis@kowloon.network
```

Root login is disabled (publickey only, no password).

## Docker compose location

`~/kowloon/docker-compose.yml` on both servers.

Services: `app`, `worker-feed`, `worker-pull`, `worker-outbox`, `worker-media`, `worker-backup`, `caddy`, `mongo`, `minio`.

## Deploy procedure

CI (GitHub Actions) builds and pushes `ghcr.io/jzellis/kowloon:latest` on every push to main. The servers do NOT auto-pull — you must SSH in and restart manually:

```bash
ssh jzellis@kwln.social "cd ~/kowloon && docker compose pull app worker-feed worker-pull worker-outbox worker-media worker-backup && docker compose up -d app worker-feed worker-pull worker-outbox worker-media worker-backup"

ssh jzellis@kowloon.network "cd ~/kowloon && docker compose down && docker compose pull && docker compose up -d"
```

Use `docker compose down && up` on kowloon.network if containers have naming conflicts from a previous failed restart.

## Seed credentials

All seed users (created by `seeder/seed.js`) share the same password: `Kowloon2026!`

The seeder lives at `~/Projects/kowloon/seeder/seed.js` (separate from `server/scripts/`).

Group owners on both servers:
- Small Press & Zines → inkandpaper
- Jazz & Improvisation → nightwriter
- Slow Web Collective → cityhacker
- Field Recording Society → recordhead
- Letterpress & Print → designthink

**Why:** Needed repeatedly for API scripts (e.g. seed-group-images.js) that must log in as group owners.

**How to apply:** When running API scripts against production, use `--password=Kowloon2026!` and look up the group's `actorId` to find the right username to log in as.
