---
name: no-follow-notifications
description: "Adding someone to your Following circle is private — the followed person is never notified. Foundational design principle, not an oversight."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

When a user adds someone to their Following circle (or any other personal circle), the other party is **not notified**. This is intentional and foundational to Kowloon's design.

**Why:** Kowloon's relationship model is deliberately ambiguous. There is no public follower count, no "X followed you" social signal, no "X is following Y" graph anyone can see. Circles are a *private* act of curation by the circle owner. Conventional follow/unfollow notifications would reintroduce the exact follower-count anxiety and one-way performative-relationship dynamics that Kowloon was built to escape. This is the same reason there's no follow/unfollow API at all — `addToCircle` *is* the follow, but the relationship stays private to the owner.

**How to apply:**
- Do **not** implement a `follow` notification type, even if the Notification schema enum still lists it. Treat that enum value as vestigial.
- Do **not** propose "fix the server's Add-to-Circle handler to emit a follow notification" as a follow-up — that's an anti-feature.
- Do **not** add a "Followers" or "Followed you" surface anywhere in the UI.
- The `follow` filter / icon / label can be omitted from any notifications UI we build. If a vestigial follow notification ever does arrive over federation from a remote server with different design choices, the UI can fall back to a generic row, but it shouldn't be a first-class category.

Related: [[project-mobile-strategy]], the absence of follow/unfollow methods in the client lib (see client CLAUDE.md "No follow/unfollow methods").
