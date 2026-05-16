---
name: Sidebar hero image — desktop left margin still awkward
description: ServerInfo's hero image on the desktop sidebar has a visible left gap to the page edge that two attempts haven't resolved. Suspected Vite/Tailwind v4 stale cache.
type: project
---

`frontend/src/components/ui/ServerInfo.jsx` — image wrapper currently `-mt-6 -mx-6 lg:ml-[-1rem] lg:mr-[0]`.

**Goal:** on desktop, hero image should bleed flush to the page's outer left edge (escaping the layout wrapper's `px-4`). Mobile (drawer) is fine — the existing `-mx-6` already bleeds it flush to the drawer edges.

**Status as of 2026-05-09:** user reports no visual change after refresh, even after switching from `lg:-ml-4 lg:mr-0` to arbitrary values. Image still appears to sit at the column edge, ~16px from page edge, AND vertical `-mt-6` doesn't appear to be flushing the image to the bottom of the navy header either — both of those would be canceled if the wrapper div weren't being styled at all.

**Why:** strong suspicion this is Vite + Tailwind v4 + DaisyUI stale cache. Neither `-ml-4` nor `mr-0` were used elsewhere in the codebase before this change; arbitrary values were tried as a workaround but user did not yet confirm whether dev server was restarted.

**How to apply (next session):**
1. First debug step: have user run `rm -rf frontend/node_modules/.vite && npm run dev`, hard refresh, and inspect the wrapper `<div>` in DevTools. Check computed `margin-top` and `margin-left`. If `margin: 0`, Tailwind isn't generating these classes — fall back to a hand-written CSS class in `index.css` with an explicit `@media (min-width: 1024px)` rule. If margins are present but the gap still looks bigger than 16px, inspect the parent chain to find the unaccounted padding.
2. If user still sees the same broken output after restart: add a `.server-hero-bleed` class in `index.css` with explicit margins inside a media query, and use that class on the wrapper instead of Tailwind utilities. Bypasses the JIT entirely.
3. Don't bother with bigger negative-margin values until #1 confirms whether the existing `-1rem` is actually applied.

**File:** `frontend/src/components/ui/ServerInfo.jsx`
**Layout chain (for reference):** `Layout.jsx` wrapper has `flex-1 overflow-hidden px-4`; sidebar column has `lg:col-span-3 overflow-y-auto py-6` (no horizontal padding); `<aside>` has `flex flex-col gap-12` (no padding); ServerInfo container has `flex flex-col gap-3 border-b-2 border-base-300 pb-5` (no horizontal padding).
