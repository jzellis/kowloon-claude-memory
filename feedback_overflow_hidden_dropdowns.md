---
name: PostComposer overflow-hidden clips dropdowns
description: Any absolute-positioned dropdown inside PostComposer must use createPortal — the modal container has overflow-hidden which clips child popups
type: feedback
---

Use `createPortal` (rendered to `document.body`) with `position: fixed` coordinates from `getBoundingClientRect()` for any dropdown, autocomplete, or popup rendered inside the PostComposer modal.

**Why:** PostComposer's modal panel (`src/components/posts/PostComposer.jsx` ~line 578) has `overflow-hidden` explicitly to prevent content escaping the modal frame. This clips all absolutely-positioned descendants regardless of z-index.

**How to apply:** Any time a new dropdown/autocomplete/tooltip is added inside PostComposer, use the pattern in `LocationField.jsx`: `useLayoutEffect` measures the container with `getBoundingClientRect()`, stores `{ position: 'fixed', top, left, width }` in state, and `createPortal` renders the popup into `document.body` with those coords. Also use `onMouseDown` + `e.preventDefault()` on dropdown items instead of `onClick` to prevent the input blurring before selection registers.
