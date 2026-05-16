---
name: outbox-api-format
description: Activity payload shapes accepted by POST /outbox — quick reference for the canonical types
metadata: 
  node_type: memory
  type: project
  originSessionId: 441e186f-92df-4648-b5ed-e6a45209573c
---

Cheat-sheet for what `POST /outbox` accepts. Verify against `routes/outbox/` if anything looks stale.

**Why:** these shapes are easy to get wrong — especially `Reply` (not `Create` with `objectType: Reply`) and the `to: "post:<id>@domain"` convention for replies/reacts.

**How to apply:** when scripting against the outbox (seed scripts, federation tests, manual curl), copy from here rather than guessing.

```js
// Create a post
{ type: "Create", objectType: "Post", to: "@public",
  object: { type: "Note", content: "..." } }

// Reply to a post (use type "Reply", NOT "Create" with objectType "Reply")
{ type: "Reply", objectType: "Reply",
  to: "post:<id>@domain",   // target post ID goes here
  object: { type: "Reply", content: "..." } }

// React to a post
{ type: "React", objectType: "React",
  to: "post:<id>@domain",
  object: { type: "React", emoji: "👍", name: "thumbsup" } }

// Add to circle (following someone)
{ type: "Add", object: "@bob@kwln2.local", target: "circle:<id>@domain" }

// Update a user profile
{ type: "Update", objectType: "User", target: "@alice@domain",
  object: { profile: { name: "Alice", bio: "..." } } }

// Change password (owner only)
{ type: "Update", objectType: "User", target: "@alice@domain",
  object: { password: { current: "oldpass", new: "newpass123" } } }

// Delete/deactivate (owner or admin)
{ type: "Delete", target: "post:<id>@domain" }
{ type: "Delete", target: "@alice@domain" }  // sets active:false, NOT type:Tombstone

// Flag content
{ type: "Flag", target: "post:<id>@domain",
  object: { reason: "spam", notes: "optional note" } }
```

### Shell gotcha
`@` and `!` in shell strings — use `python3 -c 'import json; print(json.dumps({...}))'` to generate JSON safely; avoids zsh history expansion of `!`.

Related: [[codebase-gotchas]] covers the Update/Delete patch-injection bug that's tied to these shapes.
