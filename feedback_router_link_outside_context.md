---
name: feedback-router-link-outside-context
description: "Never render react-router-dom <Link> outside the <RouterProvider> tree in the Kowloon frontend. Components mounted at App.jsx root (siblings of <RouterProvider>) — toasts, global modals, anything via createPortal at that level — must navigate via the imported router instance, not Link/useNavigate."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

In the Kowloon frontend (`createBrowserRouter` + `<RouterProvider>` in `App.jsx`), any component that mounts as a SIBLING of `<RouterProvider>` is outside the router's React context. That includes the toast stack (`components/ui/ToastStack.jsx`) and anything else attached at the app root via `<Provider>` or `createPortal`. Inside such components, react-router-dom's `<Link>`, `useNavigate()`, and `useLocation()` all fail because `useContext()` returns null. `<Link>` actually throws on render — and the failing `<a href>` element appears to trigger a rogue navigation before React tears the tree down (we hit this with the toast action link causing copy-circle clicks to instantly jump to a blank page).

**Why:** React context follows the React tree, not the DOM tree. `createPortal` doesn't help — the rendering component still inherits context from where its JSX is mounted, not where the portal lands.

**How to apply:**
- For SPA navigation from a root-mounted component, import the router and call it imperatively: `import router from '../../app/router'; router.navigate(to)`. Works without context.
- Don't try to put `<ToastStack />` (or similar) inside `<RouterProvider>` — RouterProvider doesn't accept children. If you really need router context for many such components, add a root layout route in `router.jsx` and render the global UI from inside it.
- If you ever see `Uncaught TypeError: React10.useContext(...) is null` from `LinkWithRef`, this is what's happening — check where the offending component is mounted.

See `frontend/src/components/ui/ToastStack.jsx` for the `router.navigate(to)` pattern.
