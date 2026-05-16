---
name: Multi-federation helper — remaining handler wiring
description: TODO — wire the multi-federation helper into the Add (join_approved) handler so cross-server group approvals notify users on third servers. Same pattern as React/Reply, just deferred.
type: project
---

`ActivityParser/handlers/utils/getMultiFederationTargets.js` (added 2026-05-14) takes any combination of Kowloon IDs and returns a deduplicated set of remote domains an activity should land on. React and Reply Create handlers are wired up. Two callers are still pending:

- **Add (join_approved)** — when a group admin on server A approves a join request from a user on server B, the approved user needs a notification on their home server. Today the notification is created locally on whichever server processes the Add, which fails when the approved user is remote. Fix: pass the approved member's ID to the helper alongside the existing federation logic.
- **Join (join_request)** — only relevant if a group has admins on a server other than the group's home. Rare; defer until needed.

**Why:** keeps cross-server notification behavior consistent without inventing a separate "Notification" activity type — the existing activity carries enough context, and the target server's handler creates the notification as a side effect.

**How to apply:** see the React handler's call site for the pattern (`getMultiFederationTargets(targetId, targetAuthorId, parentId)`). Full case list lives in Joplin note `97449ef9bedb4d2787f1117af9703bc4` ("Federation: Multi-Server Notification Cases").
