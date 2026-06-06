---
name: mobile-groups-moderation-todo
description: "Deferred — mobile group moderation beyond the join-request queue. Member kick, role promotion, and block management aren't exposed in the activities API yet, and the right long-term path is a mobile notifications surface."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

Mobile Groups Phases A + B are complete. Phase C shipped *only* the join-request queue path. Several adjacent moderation capabilities are deferred because the API/client doesn't expose them yet, or because the right home for them is a notifications screen that doesn't exist on mobile.

**What did ship (2026-06-03):**
- Server: `GET /groups/:id/pending` — admins-only endpoint reading `group.circles.pending.members`. Bypasses the generic `getCircle` visibility (which gates on `actorId`, not group-admin membership).
- Mobile: `app/group/[id]/pending.js` with Approve / Reject buttons wired to existing `approveJoinRequest` / `rejectJoinRequest` client methods. "Pending" button on the group detail header for owners.

**Deferred — needs server + client work first:**

1. **Member removal (kick).** No `removeFromGroup` in `client.activities` and no Reject/Remove activity for an *existing* member on the server. Add a `Remove`-style activity (or wire to `removeFromCircle` against `group.circles.members` with a group-admin auth check), then a thin client wrapper.

2. **Role promotion (admin / moderator).** The five system circles (`group.circles.admins`/`moderators`/etc.) are writable today only via the server-admin endpoints (`admin.addToGroupCircle`). No activity-level path. Needs a `Promote`/`Demote` activity gated on group-admin membership, plus client wrappers.

3. **Block management.** Same story — `group.circles.blocked` is only writable via admin endpoints. Needs a `Block`/`Unblock` activity at the group scope with admin gating.

**Deferred — right long-term home is a notifications surface:**

4. **Mobile notifications screen.** The server already emits `join_request` notifications to group admins, and `client.notifications.*` is ready (`list`, `unreadCount`, `markRead`, etc.). A real notifications screen would surface join requests *and* every other notification type mobile is currently flying blind on (replies, reactions, follows, join approvals). When that screen exists, the dedicated `/group/[id]/pending` route becomes one of two ways to handle a join request — useful for in-group flows but no longer the only path.

**Untested but probably-works:**

5. **Cross-server join requests.** For a remote user requesting to join a local approval-only group, the server's existing federation pipeline should deliver the Add/Reject activities back to the requester's server. The new `/pending` endpoint returns the same shape regardless of locality. Worth a real federation smoke test once we have two mobile clients.

**How to apply:** When picking up Group moderation again, decide first whether to build the notifications screen (broader value, unblocks #4) or to grind through #1-3 sequentially (deeper moderation, no notifications win). [[project-mobile-app-scaffold]] is the umbrella; [[project-mobile-strategy]] explains why we're not rebuilding the web UI.
