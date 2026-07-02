---
name: dev-workflow
description: "Local machine is dev-only; deployment is GitHub push -> Docker CI. No PM2, no local service restarts."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

The local machine (kwln.social host) is a development environment only. Write code here and push to GitHub. Docker CI builds and pushes images; servers pull and restart from there.

**Why:** PM2 was pre-alpha only. Production runs on Docker on remote servers (kwln.social, kowloon.network). Running PM2 restarts or testing directly against the local environment is meaningless for production deploys.

**How to apply:** After making changes, commit and push to GitHub (server -> jzellis/kowloon, frontend -> jzellis/kowloon-frontend). Never restart PM2, never rebuild the frontend locally expecting it to take effect in production. The only thing that matters is what's on GitHub main.
