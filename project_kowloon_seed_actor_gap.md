---
name: project-kowloon-seed-actor-gap
description: "Rich-seed pipeline overview. seed.js bypasses the outbox (actor/FeedItems gap); seed-extra.js uses HTTP and is the source of Media posts + featured images; sample-media folder is required at repo root for the latter."
metadata:
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

The rich-seed pipeline is THREE scripts plus one asset folder. Run order matters:

1. **`scripts/seed.js`** — base data: users/circles/groups/pages and ~100 Note/Article/Link/Event posts. Direct Mongoose writes. **Has the actor-embed + FeedItems gap** (see below).
2. **`scripts/seed-extra.js`** — adds 25 more users and ~100 posts INCLUDING Media + featured images + curated real Link URLs. Uses HTTP calls to `/outbox`, so the outbox pipeline runs and `actor`/FeedItems are populated correctly. This is the source of the "rich" content jzellis values.
3. **`scripts/seed-sample-icons.js`** — backfills missing user/circle/group icons from sample-media (skips any record with a custom icon).

**`sample-media/`** lives at the REPO ROOT (sibling of `server/`, not inside it). Served at `/sample-icons` via `server/index.js:98-104` static mount. On jarvis as of 2026-05-19 the scripts are present but the folder isn't yet — jzellis is manually rsyncing the 531M of media from jzenbook. Don't run `seed-extra.js` or `seed-sample-icons.js` until the folder lands.

### The seed.js actor/FeedItems gap (still applies to step 1 only)

1. **No `actor` embed.** Posts/Replies/Pages get `actorId` but not the `actor` subdoc. Frontend `PostMeta` etc. read `post.actor` and render nothing without it.
2. **No FeedItems write.** Outbox calls `methods/feed/writeFeedItems.js`; direct `Model.create()` skips it. `GET /posts`, `GET /circles/:id/posts`, `GET /groups/:id/posts` all read FeedItems, so seeded posts are invisible until backfilled.

Backfill recipe (one node --input-type=module script):
- Load settings via `methods/settings/cache.js#loadSettings`
- For each Post/Reply/Page lacking `actor`, look up the user by `actorId` and build the Actor subdoc per `schema/subschema/Actor.js`
- Call `writeFeedItems(doc.toObject(), objectType)` to upsert the FeedItems row and enqueue fan-out

Running `seed-extra.js` after `seed.js` doesn't backfill the seed.js posts — it only adds new ones. To "rescue" the seed.js batch, run a backfill script separately.

### Other seeding gotchas

- **`seed-test.js` step 4 fails** — sets `User.to` to a circle ID, but the Update handler in `ActivityParser/handlers/Update/index.js` only allows `@public` or `@<own-domain>` for User profiles. Earlier steps succeed; everything after the Carol-profile step doesn't run.
- **DOMAIN env var** — seed-test.js defaults to `kwln.org`. On localhost dev, pass `DOMAIN=localhost`.

**Why:** seed.js predates the outbox/FeedItems separation and was never refactored. seed-extra.js was written after, which is why it does the right thing. CLAUDE.md flags the general rule ("Direct `Model.create()` skips actor embed, feed fan-out, notifications, federation").

**How to apply:** When seeding a fresh dev env, run `seed.js`, then `seed-extra.js`, then `seed-sample-icons.js`. If only the seed.js batch is in the DB, the actor/FeedItems gap is what makes posts vanish from `/posts`. See [[feedback-actor-canonical]] for why the field is named `actor` everywhere.
