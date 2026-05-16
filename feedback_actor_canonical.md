---
name: feedback-actor-canonical
description: "For Kowloon, `actor` is the canonical author field everywhere; don't introduce `attributedTo` aliases on the server to bridge frontend gaps — fix the frontend."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

Kowloon's data model uses `actor` (a Member/Actor subdoc) as the canonical author field on Posts, Replies, Pages, FeedItems, and the API responses they produce. Do not paper over frontend bugs by emitting `attributedTo` aliases from the server.

**Why:** When a frontend component read `post.attributedTo` and got `undefined`, I changed `feedItemToPost.js` to also emit `attributedTo` as an alias. jzellis rejected this: "Nothing should be reading from attributedTo unless that's a thing in FeedItems or the fan out lookup." The frontend was wrong; the server contract is right.

**How to apply:** If frontend components reference `attributedTo`, change them to `actor` (or `reply.actor` for replies). Don't add server-side aliases. The single canonical name keeps schema, FeedItems, the fan-out lookup, and the API surface consistent. See [[project-kowloon-seed-actor-gap]] for the related issue where seed.js produces posts with no `actor` embed at all.
