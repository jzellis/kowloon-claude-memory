---
name: Undo{React} handler — implement un-react
description: ActivityParser/handlers/Undo/index.js has a placeholder for Undo{React}; needs to actually remove the React record and recompute reactPreview/reactSummary
type: project
---
The `Undo` handler in `ActivityParser/handlers/Undo/index.js` has an explicit "not yet implemented" branch for `Undo{React}`. Until it's wired up, un-reacting silently no-ops on the server.

**Why:** without this, react totals can only ever go up. Users who change their mind and remove a reaction get stale state, and `reactCount`/`reactPreview`/`reactSummary` on Post + FeedItems all drift from the source of truth in the `reacts` collection.

**How to apply:** when implementing, mirror the pattern from the `React` handler:
1. Find and delete the matching `React` record (by `actorId` + `target` + `emoji`).
2. Decrement `reactCount` on the source model (Post/Page/Bookmark/Group) AND on `FeedItems.object.reactCount`.
3. Re-run `summarizeReacts(targetId)` and write the new `reactPreview` + `reactSummary` to both the source model and the FeedItems cache (use the helper from `ActivityParser/handlers/React/index.js`).
4. Federation: deliver the Undo to the same audience as the original React.

Frontend's `ReactButton` already optimistically increments on add; will need parallel decrement logic when this lands.
