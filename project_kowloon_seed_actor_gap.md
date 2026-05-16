---
name: project-kowloon-seed-actor-gap
description: "server/scripts/seed.js writes Posts/Replies/Pages directly via Mongoose, bypassing the outbox pipeline — so the `actor` embed and FeedItems are never populated. Posts won't render authors or appear in /posts until backfilled."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

Two known gaps in `server/scripts/seed.js`:

1. **No `actor` embed.** Posts/Replies/Pages get `actorId` but not the `actor` subdoc (id/name/icon/url/inbox/outbox/server). Frontend `PostMeta` etc. read `post.actor` and render nothing without it.
2. **No FeedItems write.** The outbox pipeline calls `methods/feed/writeFeedItems.js` after creation; direct `Model.create()` skips it. `GET /posts`, `GET /circles/:id/posts`, `GET /groups/:id/posts` all read FeedItems, not Post directly — so seeded posts are invisible until backfilled.

Backfill recipe (one node --input-type=module script):
- Load settings via `methods/settings/cache.js#loadSettings`
- For each Post/Reply/Page lacking `actor`, look up the user by `actorId` and build the Actor subdoc per `schema/subschema/Actor.js`
- Call `writeFeedItems(doc.toObject(), objectType)` to upsert the FeedItems row and enqueue fan-out

**Also note**: `seed-test.js` step 4 currently fails — it tries to set `User.to` to a circle ID, but the Update handler in `ActivityParser/handlers/Update/index.js` only allows `@public` or `@<own-domain>` for User profiles. Earlier steps (4 users, 16 user circles) succeed; everything after the Carol-profile step doesn't run.

**Why:** Same root cause — seed.js was written before the outbox/FeedItems separation and never updated. CLAUDE.md flags the general rule ("Direct `Model.create()` skips actor embed, feed fan-out, notifications, federation") but the seeder still uses it.

**How to apply:** After running seed.js, run the backfill before checking the UI, or post a CLAUDE.md TODO to fix the seeder to call writeFeedItems + populate actor. See [[feedback-actor-canonical]] for why the field is named `actor` everywhere.
