---
name: Reply federation lag — author's own view
description: POSSIBLY TODO — in the on-demand reply model (added 2026-05-14), the author of a cross-server reply may briefly miss seeing their own reply because the read proxies to the parent's host before federation has landed. Wait for real-world testing to see if it shows up.
type: project
---

`routes/posts/replies.js` proxies reads to the parent post's host server (so the parent is the single source of truth for replies on its posts). The author's home server (kwln1) writes the Reply locally and federates it to the parent host (kwln2). Until that federation delivery completes, viewing the reply list on kwln1 produces a request to kwln2 that won't include the just-submitted reply yet.

**Why:** the on-demand model intentionally removes third-party Reply state. Trade-off is that the author momentarily sees the post they replied to without their own reply present (until kwln2 ingests the federated activity — usually seconds).

**How to apply:** wait for real-world federation testing before fixing. If users complain or it's visible in the federation test suite, the simple fix is to union local Reply records WHERE `actorId === viewer.id` with the proxied response in `routes/posts/replies.js`. That covers the author-of-the-reply case without bringing back arbitrary third-party state. Until then, leave alone — extra code for a transient effect.
