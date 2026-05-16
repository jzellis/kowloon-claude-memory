---
name: Reply/React tombstone re-fanout — RESOLVED (2026-05-14)
description: Historical note. Resolved by switching to the on-demand reply model (parent host is the single source of truth for a post's replies). Third-party servers no longer keep Reply state, so there is nothing to fan out tombstones to.
type: project
---

**Status: resolved.** As of 2026-05-14, `routes/posts/replies.js` proxies reads to the parent post's home server. Third-party servers never accumulate Reply records for posts they don't host, so the original concern (kwln3 holding a stale copy of josh's reply on bob's post after josh deletes it) no longer applies.

Kept as a historical record only — do not put back on the active TODO list unless the on-demand model gets reverted.
