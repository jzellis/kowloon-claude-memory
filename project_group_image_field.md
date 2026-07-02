---
name: group-image-field
description: Group schema has icon (hex profile avatar) and image (hero/banner) as separate fields; both are file URLs; added 2026-07-02
metadata: 
  node_type: memory
  type: project
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

Groups have two distinct image fields:

- `icon` — square profile avatar, rendered through the hexagonal clip mask (`GroupAvatar`). Has always existed.
- `image` — wide hero/banner image (3:1 aspect ratio). Added 2026-07-02. Optional; `undefined` when not set.

Both store a Kowloon file URL (`https://domain/files/file:mongoId@domain`).

**Where each field appears:**
- Server schema: `server/schema/Group.js` — `image: { type: String, default: undefined }`
- Update handler allowlist: `server/ActivityParser/handlers/Update/index.js` line ~61 — `Group: new Set(["name", "description", "icon", "image", "to", "canReply", "canReact", "rsvpPolicy", "location"])`
- Client: `client/src/activities/index.js` — `createGroup` and `updateGroup` both accept `image` param
- Web: `frontend/src/components/groups/GroupForm.jsx` — banner picker UI (tap above icon+name row, 3:1 aspect placeholder)
- Web: `frontend/src/pages/GroupPage.jsx` — displayed above the sticky header at 3:1 via `sizedUrl(group.image, 1200)`
- Web: `frontend/src/pages/NewGroupPage.jsx` and `EditGroupPage.jsx` — upload bannerFile before calling createGroup/updateGroup
- Mobile: `mobile/src/components/groups/GroupForm.jsx` — banner picker (tap `bannerAsset`, `pickBanner()`)
- Mobile: `mobile/app/group/[id]/index.js` — hero banner rendered via `<Image>` before the group header in ListHeader; uses `resolveImageUrl(group.image, account?.baseUrl)`
- Mobile: `mobile/app/group/new.js` and `mobile/app/group/[id]/edit.js` — upload bannerAsset before calling updateGroup

**Seed script:** `server/scripts/seed-group-images.js` — logs in as each group's owner, downloads seeded picsum.photos images (400x400 icon, 1200x400 banner), uploads, and calls `updateGroup({ icon, image })`. Run with `--force` to overwrite existing. Usage: `node scripts/seed-group-images.js --server=https://kwln.social --password=Kowloon2026!`

**Why:** "icon" was already an overloaded term; keeping a distinct `image` field for hero banners matches the ActivityStreams Object schema (which also separates `icon` from `image`).

**How to apply:** When touching group display code, check both fields. Never use `icon` where `image` is meant (they're different shapes and slots). The `image` field is silently ignored by old server code (pre-2026-07-02 Update handler) — re-run seed-group-images with `--force` after any deploy that includes the schema change.
