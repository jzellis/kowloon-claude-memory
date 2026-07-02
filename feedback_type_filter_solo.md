---
name: type-filter-solo
description: "Mobile TypeFilter UX — first tap solos a type, subsequent taps add more back. Validated 2026-07-02."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

When all post types are visible, tapping one type in the TypeFilter bar **solos** it (shows only that type). Subsequent taps on dimmed types add them back one by one. Tapping the last active type returns to showing all.

**Why:** The previous behavior (first tap deselects, keeps all others) was backwards — users expect "click one to see only that" not "click one to hide that one." This is the standard filter paradigm (e.g. Instagram's type tabs).

**How to apply:** Do not revert to the deselect-on-first-tap behavior. The solo-first pattern is intentional. Implementation is in `mobile/src/components/posts/TypeFilter.jsx` — the `isAll` branch of `handlePress` calls `onSetTypes([type])` (solo), not `onSetTypes(POST_TYPE_NAMES.filter(t => t !== type))` (exclude).
