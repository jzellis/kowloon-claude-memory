---
name: Default placeholder icons (resolved)
description: Resolved 2026-05-06 — server/public/images/{user,circle,group}.svg ship in the repo and are referenced by schema defaults.
type: project
---

**Resolved 2026-05-06.** Default placeholder icons now ship in the repo.

**What was done:**
- Added `server/public/images/user.svg`, `circle.svg`, `group.svg` — hex-bordered SVGs matching the existing icon language. User: head + shoulders silhouette. Circle: three overlapping rings (Venn-style). Group: three clustered figures.
- Added unconditional `/images` static mount in `server/index.js` (separate from the SERVE_FRONTEND-gated public dir mount, so it works in dev too).
- Schema defaults flipped from `.png` → `.svg` in `schema/User.js`, `schema/Circle.js`, `schema/Group.js`.
- `scripts/seed-sample-icons.js` regex updated to recognize both `.png` and `.svg` placeholder variants when deciding whether to overwrite.

**No to-do remaining.** Kept this file as a record; the MEMORY.md index entry has been removed.
