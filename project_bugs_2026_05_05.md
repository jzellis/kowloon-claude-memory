---
name: Bug fixes 2026-05-05
description: Privacy/fan-out bug, reply count stale FeedItems, user search, circle UX, UI polish
type: project
---

Bugs fixed and shipped (commits 1056b4c8 / e3c0483), 2026-05-05.

**Why:** routine UI testing session revealed several issues.
**How to apply:** these are fixed; reference if similar symptoms reappear.

### Privacy: enqueueFanOut defaulted to public for unknown `to` values
`parseAudience()` previously returned `{ type: "public" }` as the catch-all default.
Any post addressed to a bare user ID (e.g. `@dave@kwln.org`) would be fanned out as public.
Fixed: default is now `{ type: "invalid" }` which skips fan-out entirely with a console warning.
Design rule: posts can only be addressed to `@public`, `@domain`, circle IDs, or group IDs.
Single-user addressing is only valid for Bookmarks. Private posts must use a single-member circle.

### Reply count stale in FeedItems
Reply handler bumped `replyCount` on the Post collection but not `FeedItems.object.replyCount`.
`getPost` reads from FeedItems, so the count appeared stale in the UI immediately after replying.
Fixed: Reply handler now also does `FeedItems.updateOne({ id: targetId }, { $inc: { "object.replyCount": 1 } })`.
React handler already had this fix (model: `FeedItems.updateOne({ "object.id": targetId }, ...)`).

### seed-test.js private post addressing
Seed was using `to: user.id` for "private" posts, which is invalid addressing.
Fixed: now uses `to: userCircles.private.id` (a single-member circle).

### User search endpoint
Added `GET /users/search?q=...` — case-insensitive regex on username and profile.name.
Used by circle member AddMemberRow. Bypasses profile `to` visibility (for circle lookup).
Router guard: `STATIC_SEGMENTS = new Set(["lookup", "search"])` in `routes/users/index.js`
prevents `router.param("id")` from treating "search" as a user ID.

### UI improvements
- RichTextEditor: larger toolbar buttons
- PostComposer: spacious attachment rows, REMOVE/FEATURED button clipping fixed (pr-4)
- Header: My Circles link in user dropdown
- CirclePage + CircleForm: search-based AddMemberRow replacing direct ID lookup
- UserCirclesPage: Create Circle button for owners

### JWT stale after DB wipe
After `/__test/wipe`, admin user (jzellis) is deleted. Server re-creates jzellis on next restart
(init.js checks `User.findOne({ username: adminUsername })`), but only if wipe happened before restart.
If wipe happens AFTER restart, jzellis is gone from DB but old JWT is still cryptographically valid
(RS256 key persists in settings). User can make authenticated requests but has no DB record.
Fix: log out and back in after any wipe+reseed to get a fresh JWT with correct circle IDs.

### getTimeline catch-all FeedFanOut condition
`getTimeline` has `{ to: viewerId }` as a fallback OR condition in FeedFanOut queries.
This means any post directly addressed to the viewer (e.g. via circle fan-out) appears in ANY
circle's timeline — even if the post's author isn't in that circle. Intentional for federation
pull content, but can be surprising in local testing.
