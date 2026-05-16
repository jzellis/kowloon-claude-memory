---
name: Bookmarks are personal-only
description: Bookmarks do not fan out to feeds and do not federate. To broadcast a URL, users post a Link instead.
type: project
---

Bookmarks are a private utility, not feed content.

**Why:** A bookmark is "I want to remember this URL." A post is "I want to share this." Conflating them muddied both — bookmarks couldn't be filtered alongside posts (no Bookmark in the post-type filter), and silent fan-out meant users were broadcasting things they thought they were just saving. Decided 2026-05-06.

**How to apply:**
- Do NOT re-add `"Bookmark"` to `FEED_CACHEABLE_TYPES` in `methods/feed/writeFeedItems.js` or `ActivityParser/handlers/Update/index.js` — that re-introduces the leak.
- Do NOT route bookmark Create/Update/Delete through `getFederationTargets`. Those handlers already short-circuit `type === "Bookmark"` to `{ shouldFederate: false }`.
- If a feature genuinely needs a "shared bookmark," use a Link post — that's the right primitive.
- Bookmarks still have a `to` field (kept for future per-user-visible-folder scoping inside the user's own bookmark list) but it does NOT control federation or feed inclusion.
- Existing bookmark FeedItems were purged via `scripts/cleanup-bookmark-feeditems.js` on 2026-05-06; the script remains in the repo for any future re-runs (e.g. after a federated incoming Bookmark slips through).
