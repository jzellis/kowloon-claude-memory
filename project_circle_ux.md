---
name: Circle UX — create modal and member lookup
description: Two-step circle creation modal, user lookup endpoint, homepage feed design
type: project
---

Circle creation UX shipped 2026-04-18.

**Why:** Circles are the core social primitive in Kowloon (no follow graph). Creation needed to be contextual (modal, not new page) and immediately useful (add members in step 2).

**New server endpoint:** `GET /users/lookup?id=@bob@domain` (auth-required)
- Same pattern as `/preview` — server proxies outbound WebFinger/actor fetch, no CORS needed
- Calls `getObjectById` with `mode: "prefer-local"`, `hydrateRemoteIntoDB: true`, 5min cache
- Route registered before `/:id` param handler in `routes/users/index.js`

**New client methods:** `client.feeds.lookupUser({ id })` and `client.feeds.searchUsers({ q })`

**NewCircleModal** (`src/components/circles/NewCircleModal.jsx`):
- Step 1: name/description/visibility → creates circle → advances to step 2
- Step 2: add members — debounced local search (2+ chars) OR full handle `@user@domain` triggers explicit "Look up" button → server-side WebFinger fetch
- Batch adds via `addToCircle({ circleId, members })`; skip is always available
- "Done" dispatches fetchMyCircles, calls onCreated(circleId), closes

**CircleSelector fix:** now calls `onCreateCircle` prop when provided instead of always navigating to `/circles/new`. Both click and keyboard paths fixed.

**Homepage:** anon = public posts feed; auth = circle feed with selector embedded in filter bar ("Show: [circle dropdown] | All | Note | Article..."). Circle selector uses `variant="default"` (compact) with "Show:" label prefix.

**How to apply:** When touching circle creation or member management flows, the modal is the entry point. The full form at `/circles/new` (with icon upload) still exists for editing.
