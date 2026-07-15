---
name: origin-domain-convention
description: "Every remote-pullable object has originDomain (local domain by default, source domain when hydrated); discovery = local-only, search = includes cached remotes."
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

Kowloon caches remote objects locally (via `getObjectById` hydration / `GET
/lookup`). To tell local from remote-cached, every remote-pullable object type —
**Circle, Post, Group, Bookmark, User** — has an `originDomain` String field
(added 2026-07-15).

- **Local objects** default it to the local domain: schema
  `default: () => getServerSettings()?.domain` (runs on `save()`/create).
- **Remote-hydrated objects** get it set to the SOURCE domain explicitly in
  `methods/core/getObjectById.js` (`originDomain: parsedDomain` / `handle.server`).

**Design use:**
- **Discovery endpoints return LOCAL-only.** `GET /circles`
  (`routes/circles/collection.js`) filters `originDomain: { $in: [null, localDomain] }`
  — the `null` covers legacy rows created before the field existed (no backfill
  was run). So a remote circle pulled in via `/lookup` never leaks into your
  server's discovery.
- **Search intentionally INCLUDES cached remotes** — deliberately no originDomain
  filter on `/search`. Surfacing remote content you've fetched is wanted there.

**Gotcha history:** the hydration code set `originDomain` for a long time, but the
schemas never had the field, so Mongoose strict-mode silently dropped it — a
latent no-op. Adding the field made it real. Same class as
[[remote-actor-hydration-shape]] (hydration writing fields the schema lacked).

**Watch list:** only `GET /circles` filters local-only so far. `GET /users`,
`GET /groups`, `GET /pages` do NOT — if cached remotes of those should also be
excluded from discovery, add the same `originDomain: { $in: [null, localDomain] }`
filter. Related: [[federation-pull-architecture]].
