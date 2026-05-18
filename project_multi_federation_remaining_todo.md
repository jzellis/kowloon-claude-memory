---
name: multi-federation-helper-remaining-handler-wiring
description: "One pending caller — Join (join_request) handler — only matters when a group has admins on a server other than the group's home. Add (join_approved) wired 2026-05-18."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

`ActivityParser/handlers/utils/getMultiFederationTargets.js` (added 2026-05-14) takes any combination of Kowloon IDs and returns a deduplicated set of remote domains an activity should land on. Wired into React, Reply, and **Add (Group adds)** as of 2026-05-18 (`ActivityParser/handlers/Add/index.js`). One pending caller:

- **Join (join_request)** — only relevant if a group has admins on a server other than the group's home. Rare; defer until needed.

**Why:** keeps cross-server notification behavior consistent without inventing a separate "Notification" activity type — the existing activity carries enough context, and the target server's handler creates the notification as a side effect.

**How to apply:** see the React, Reply, or Add handler's call site for the pattern (`getMultiFederationTargets(...ids)`). For Add specifically, only the Group ownerType uses the helper — personal/server-admin/mod circles keep the legacy resolveAudience path. Full case list lives in Joplin note `97449ef9bedb4d2787f1117af9703bc4` ("Federation: Multi-Server Notification Cases").
