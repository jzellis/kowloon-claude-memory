---
name: federation-pull-architecture
description: How server-subscription (public firehose) and user-follow federation pull works in getTimeline + pullFromRemote
metadata: 
  node_type: memory
  type: project
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

Core architecture for the on-demand federation pull system (getTimeline + pullFromRemote).
Bugs here are silent and hard to detect -- read this before touching either file.

**Why:** Multiple rounds of regressions across sessions (2026-06-20 and earlier) due to misunderstanding the dual FanOut model and the server-subscription architecture.

**How to apply:** Before editing `server/methods/feed/getTimeline.js` or `server/methods/federation/pullFromRemote.js`, re-read this entry in full.

---

## Two kinds of circle members

| Member format | Meaning |
|---|---|
| `@alice@kowloon.network` | Individual user follow |
| `@kowloon.network` | Entire server public firehose (one `@`, no second `@`) |

Detection: `id.startsWith("@") && !id.slice(1).includes("@")` = bare server entry.

---

## Server subscriptions are a public firehose

When a user adds `@kowloon.network` to a circle, they are subscribing to that server's **entire public feed** (`@public` posts). Posts are NEVER addressed to any specific kwln.social user. The remote server has no idea who on kwln.social subscribes. This means:

- The outbox query `GET /outbox?from=@kowloon.network&...` returns public posts.
- The remote's `recipients` array is irrelevant for server-only pulls -- the local server computes recipients.
- `pullFromRemote` must fan out every returned item to every local subscriber (passed in `to`).

`serverOnlyPull = from.every(isServerEntry)` detects this case in `pullFromRemote.js`.

---

## FanOut `to` values (two patterns)

`FeedFanOut.to` stores one of three values written by two different code paths:

| Value | Written by | When |
|---|---|---|
| `"@public"` | `enqueueFeedFanOut` | Local public post (one shared row) |
| `"@server"` | `enqueueFeedFanOut` | Local server-visibility post (one shared row) |
| `"@alice@kwln.social"` | `pullFromRemote` | Remote/circle-addressed post (per-subscriber row) |

The `getTimeline` FanOut query MUST be:
```js
const toFilter = { $in: ["@public", "@server", viewerId] };
```
Using `to: viewerId` alone drops ALL local public posts from every circle feed.

---

## getTimeline pull loop

`getTimeline` calls `pullFromRemote` for each remote domain in the circle. Key details:

1. `findAllCircleSubscribers(remoteAuthors)` -- finds ALL local users (not just the viewer) who have any of the remote members in any circle. This makes one pull populate FanOut for every subscriber.
2. `oldestFetchedAt` -- uses the oldest `lastFetchedAt` cursor across the remote members so the most-stale member drives the time window.
3. Cursor update: only advance `lastFetchedAt` if `gotItems || wasIncremental`. On a first-ever fetch that returns 0 items (`pullSince === null`), do NOT advance -- the remote may have had a temporary error and we want to retry the full pull next time.

---

## pullFromRemote FanOut split

```js
if (serverOnlyPull) {
  // Fan out every item to every subscriber in `to`.
  for (const itemId of upsertedIds) {
    for (const userId of to) {
      pushFanOut(itemId, feedItem, userId);
    }
  }
} else {
  // User follows: remote returned per-item recipients.
  for (const { itemId, to: recipientIds } of recipients) {
    for (const userId of recipientIds) {
      pushFanOut(itemId, feedItem, userId);
    }
  }
}
```

FanOut rows use `$setOnInsert` with a `dedupeHash` so repeat pulls are idempotent.

---

## Remote outbox S2S protocol

`GET /outbox?from=...&to=...` returns:
```json
{ "items": [...], "recipients": [{ "itemId": "...", "to": ["@alice@..."] }] }
```

For server-only pulls (`from=@domain`), `collection.js` on the remote queries `originDomain: { $in: serverDomains }` (NOT `server: { $in: serverFroms }` -- that field doesn't exist). All `to` users are set as recipients for each public item.

---

## Common regressions to watch for

- `to: viewerId` instead of `to: { $in: ["@public", "@server", viewerId] }` -- nukes local post display
- `$set: item` without stripping `_id`/`__v` -- silently fails on re-upsert, no FanOut created
- Passing only `[viewerId]` in the pull `to` list -- FanOut only reaches the viewer, not all subscribers
- Including bare `@domain` entries in `actorId: { $in: ... }` -- they never match real posts; use domain-regex condition instead

Related: [[codebase-gotchas]]
