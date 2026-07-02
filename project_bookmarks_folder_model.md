---
name: bookmarks-folder-model
description: "Folder-visibility inheritance, depth cap, and cascade-delete rules for bookmarks. Settled 2026-06-16; mirror these constraints on both ends if you touch the tree."
metadata: 
  node_type: memory
  type: project
  originSessionId: ecff8eb7-af36-4331-8a4a-794961a2a9da
---

The bookmark tree is governed by four settled rules. Don't drift from them without checking back in.

1. **Read-time folder visibility inheritance**, not write-time override. A bookmark or sub-folder is only visible if its own `to` passes the viewer check AND every ancestor folder's `to` also passes. Move a bookmark into a private folder and it becomes invisible without rewriting the bookmark's `to`; move it back out and it returns. Enforced server-side in `methods/bookmarks/visibility.js` (`canSeeFolderChain`, `canSeeBookmark`).
2. **Cascade delete on folders.** Deleting a Folder soft-deletes every descendant the same owner owns (recursive walk in `ActivityParser/handlers/Delete/index.js`). The mobile delete prompt warns about this.
3. **5-level folder depth cap with cycle detection.** Enforced in both `schema/Bookmark.js`'s `pre("save")` AND the Update handler — `findOneAndUpdate` skips the schema hook, so moves go through `assertFolderDepthOk` in `methods/bookmarks/visibility.js` directly. Bookmarks themselves aren't depth-counted; only folders.
4. **Lazy folder loading.** The mobile tree fetches one folder's children per expand via `GET /users/:id/bookmarks?parentFolder=<id>`. Three query modes: omitted = flat-all (owner) / root-only (non-owner); `=root` = explicit root; `=<id>` = that folder's children with ancestor-visibility check.

**Why:** chosen 2026-06-16 with explicit user confirmation. Read-time inheritance keeps user intent intact across moves; cascade-delete matches every other bookmark manager; lazy loading scales past a few thousand bookmarks; 5 levels is the conventional cap and gives natural cycle protection.

**How to apply:** if you add a new bookmark surface (top-level browse, search, federated import, anything), wire it through `canSeeBookmark` or replicate the ancestor check. Don't trust `to` alone — the leak path is "@public bookmark inside @private folder." If you touch the schema, remember the Update handler's `ALLOWED_FIELDS.Bookmark` includes `parentFolder`, `image`, and `source` — silently dropping any of those re-breaks moves or body edits. Related: [[project-bookmarks-personal-only]] (federation/feed exclusion still applies).
