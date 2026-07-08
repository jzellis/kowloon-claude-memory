---
name: codebase-gotchas
description: Non-obvious shape-of-the-code traps in Kowloon server — current invariants that are easy to break or miss
metadata: 
  node_type: memory
  type: project
  originSessionId: 441e186f-92df-4648-b5ed-e6a45209573c
---

Compiled gotchas about *current* Kowloon server code. Each one is a property a future change could regress without anyone noticing. Verify against the code before betting the farm on any single line.

**Why:** these have all caused or nearly caused incidents. They're not historical bug fixes — they're invariants the current code depends on.

**How to apply:** when editing the named area, re-read the relevant entry before touching it.

### Update/Delete handlers must not have outbox defaults injected
`routes/outbox/post.js` injects `to/canReact/canReply` into `activity.object` only when `activity.type !== "Update" && activity.type !== "Delete"`. For Update/Delete, `object` is a **patch**, not a new entity — injecting defaults would silently corrupt updates.

### Update/Delete response strips sensitive User fields
Both handlers (`ActivityParser/handlers/Update/index.js`, `Delete/index.js`) strip `password`, `privateKey`, `publicKeyJwk`, `signature` from the returned object before responding.

### Delete of User sets active:false, not type:Tombstone
User deactivation: `{ deletedAt, deletedBy, active: false }`.
Non-User Delete: `{ deletedAt, deletedBy, type: "Tombstone" }`.

### jose needs plain objects in JWT payloads
Always call `.toObject({ depopulate: true })` on Mongoose docs before building a JWT payload. `jose` uses `structuredClone` internally and fails on Mongoose subdocs.

### JWT library is `jose`, not `jsonwebtoken`
Pure ESM. Keys come from `settings.privateKey` / `settings.publicKey` (RS256).

### User system circles are nested under `user.circles`
NOT top-level. Fields: `user.circles.following`, `user.circles.blocked`, `user.circles.muted`, `user.circles.allFollowing`, `user.circles.groups`.

### Settings cache may have stale nulls
`methods/utils/init.js` upserts settings previously saved as null/undefined. Without this, a broken first-run could leave `domain` as null in DB → 500s on `/register`.

### `getServerSettings()` falls back to env vars even when cache loaded
`methods/settings/schemaHelpers.js` — falls back to `process.env.DOMAIN` even when cache is loaded but value is falsy. Critical for Docker where `DOMAIN` is set per-container.

### FeedItems.to is a coarse enum, not a circle list
Values: `public` / `server` / `audience`. Circle IDs are **never** stored on FeedItems (privacy design). For `audience` posts, `FeedFanOut` is the authoritative grant — `canView()` checks `FeedFanOut.findOne({ feedItemId, to: viewerId })`. `enrichWithCapabilities()` uses the same check for `canReply`/`canReact`. `getFeedItem()` returns `null` on deny.

### File serving via Kowloon proxy
- All files private in MinIO/S3. Internal ID: `file:<mongoId>@domain`.
- `GET /files/:id` — Bearer header **or** `?token=` query param (for `<img>` tags).
- Visibility inherited from `file.parentObject` at serve time.
- `serveUrl(req, fileId, token, localDomain)` in post-serving routes: if `fileId`'s domain ≠ local → returns `https://<fileDomain>/files/<encoded-id>` (origin server). Local files get `?token=` appended.
- Outbound federation: `resolveFileIdsForFederation()` rewrites `file:` IDs to HTTP URLs in the activity before enqueuing.
- Two-save upload: first save triggers pre-save hook to set `file.id`; then set `file.url` and save again.

### Admin auth guard cannot use `route()` wrapper
`route()` always calls `res.json(out)` — no `next()` support. Admin middleware uses plain Express `router.use()` async middleware in `routes/admin/index.js`.

### npm overrides chain
- Direct: `multer ^2.1.x`, `ajv ^8.18.x`
- Overrides: `fast-xml-parser ^5.4.2`, `http-proxy-agent ^7.0.2`, `minimatch ^10.2.4`, `qs ^6.15.0`, `js-yaml ^4.1.1`
- `http-proxy-agent ^7.x` eliminates `@tootallnate/once` entirely (GCS dep chain). Don't downgrade — it'll reintroduce the vulnerable transitive.

### Seeding in Docker
`POST /__test/wipe` only works when `NODE_ENV !== "production"`. Docker compose sets `NODE_ENV=development` in the app service env blocks. Seed from host:
```bash
TEST_BASE_URL=http://kwln1.local:8080 node scripts/seed-test.js --wipe
```

Related: [[kowloon-seed-actor-gap]] for the deeper seed.js problem (Mongoose-direct writes bypassing the outbox pipeline).

### Group `actorId` is `@username@domain`, not a URL

`group.actorId` stores the Kowloon handle format (`@username@domain`), NOT an HTTP URL. To extract the owner's username for auth: `actorId.replace(/^@/, '').split('@')[0]`. Do NOT treat it as a URL or pass it to `fetch`.

### Groups members API returns `{ members: [...] }`, not `orderedItems`

`GET /groups/:id/members` returns `{ members: [...] }`. The response does NOT use `orderedItems` or `items`. When consuming this endpoint use a fallback chain: `res?.orderedItems || res?.items || res?.members || []`.

### Mobile `useFeed` `viewKey` for groups

The mobile `useFeed` hook accepts a `viewKey` mapped to the group/circle feed. Pass `viewKey: String(groupId)` where `groupId` is the full Kowloon ID (e.g. `group:abc123@kwln.social`). See `mobile/src/lib/useFeed.js`.

### FeedFanOut `to` has three value types -- query MUST use `$in`

`FeedFanOut.to` stores one of: `"@public"` (local public posts), `"@server"` (local server-only posts), or a specific user actorId like `"@alice@kwln.social"` (remote/circle-addressed posts). Two different code paths write these:
- `enqueueFeedFanOut`: local posts get ONE row with `to: "@public"` or `to: "@server"` -- NOT one row per user.
- `pullFromRemote`: remote content gets per-subscriber rows with `to: specificUserId`.

The FanOut query in `getTimeline` MUST be `to: { $in: ["@public", "@server", viewerId] }`. Using `to: viewerId` alone silently drops ALL local public posts from every feed. See [[federation-pull-architecture]].

### `pullFromRemote` must strip `_id`/`__v` before `$set` upsert

Remote FeedItems arrive with `_id` from the remote MongoDB. Passing them directly to `$set: item` causes MongoDB to throw "Mod on _id not allowed" on all subsequent upserts (first may succeed, inserting the remote `_id`). The error is silently caught, the item never lands in `upsertedIds`, and no FanOut rows are created. Always destructure first:
```js
const { _id, __v, ...itemFields } = item;
await FeedItems.findOneAndUpdate({ id: item.id }, { $set: itemFields }, { upsert: true });
```

### Bare server entries in circle.members must be excluded from the actorId `$in` filter

Bare server entries like `"@kowloon.network"` are never stored as `actorId` on real posts. Compute `nonServerMembers` before building the actor filter:
```js
const nonServerMembers = allMembers.filter(
  (id) => !(typeof id === "string" && id.startsWith("@") && !id.slice(1).includes("@"))
);
```
Server-member posts are matched via a domain regex condition in `fanOutOrConditions` instead.

### `Update.ALLOWED_FIELDS.Circle` must stay in sync with schema field names

`ActivityParser/handlers/Update/index.js` has `ALLOWED_FIELDS.Circle` that gates patchable fields. Circle schema uses `summary` (not `description`) and has `icon`. If either is absent from the Set, those edits succeed client-side but `filterPatch` strips them and the patch object is empty -- silent data loss.
