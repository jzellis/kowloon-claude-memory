---
name: PM2 env reload requires --update-env
description: PM2 does not pick up .env changes on plain restart; must use --update-env flag
type: feedback
---

Always use `pm2 restart all --update-env` after changing `.env`. Plain `pm2 restart` does not reload environment variables from the file.

**Why:** PM2 caches env at startup; `--update-env` forces it to re-read.

**How to apply:** Any time .env is edited, follow with `pm2 restart all --update-env`.
