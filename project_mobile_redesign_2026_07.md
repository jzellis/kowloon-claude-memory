---
name: project-mobile-redesign-2026-07
description: "Locked design decisions from the 2026-07 mobile visual redesign (Klein blue header, desaturated paper, bottom tab bar, view-selector overlays) and what's still pending."
metadata: 
  node_type: memory
  type: project
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

A design pass on the Kowloon mobile app (2026-07) settled a batch of visual/nav
changes. Decisions locked and folded into the token layer + design system:

- **Desaturated paper** — base-100/200/300 pulled toward off-white (100 `#FAF4E8`→`#F7F3EC`, 200→`#EAE4D8`, 300→`#D8CFBD`). Same warmth/lightness, less yellow.
- **Klein blue header** — app top header/masthead is Yves Klein blue `#002FA7` with **white sans** (Inter) title, NOT the display serif; blue extends up through the status-bar area. Token: `header` / `header-content`.
- **Near-white text fields** — all inputs + composer editor use `field` `#FCFBF7` (`bg-field`), lifting off the paper.
- **Bottom tab bar is primary nav** — Feed · Circles · Groups · Search · **Notify** (bell + unread badge). NO "You" tab: the user's own profile is reached from the account avatar (top-right), which is a deliberate low-frequency-destination call.
- **Server icon + name = drawer toggle** in the feed masthead — the hamburger is dropped; no caret (a bare tappable server badge, Slack/Discord-style). The LeftDrawer is demoted to "server context" now that the tab bar owns navigation.
- **Feed view-selector icons** — the view selector shows the current context's icon: server icon with a client-side scrim + white **globe** (Public) / **lock** (Server, local-only) overlay; **hex avatar** per circle/group. Overlay is a client-side `ServerFeedIcon` (base → scrim `rgba(0,0,0,.22)` → glyph), NOT baked server images (federation renders remote server icons too).

**Pending / not done:**
- **Hexagon compose FAB** — mocked as a point-up solid-primary hex, but the user is still deciding orientation (avatars are flat-top). NOT integrated.
- **Auth hero** — replace the text wordmark with a commissioned splash image (title baked into the art); waiting on the artwork.
- **Profile featured image + profile links** (link icon → dropdown of URLs) — both need new fields on the server `User` schema; mobile UI is designed but blocked on server work.

**Scratch harness:** `mobile/design-preview/` — `archetypes.html` (7 archetypes, live palette switch, real tokens/fonts) + `render.mjs` (puppeteer → PNG/PDF in `out/`). Local scratch, uncommitted, its own `node_modules` (Chromium). Regenerate with `node render.mjs`. Iterating happens here, then locked bits move into the real app.

**Why:** These are design-language decisions the code alone won't explain, and several are mid-flight (pending items would otherwise look like omissions).

**How to apply:** Token changes are live in `tailwind.config.js` + `design-tokens.json` (+ ds-bundle, DESIGN_SYSTEM.md). Mobile now diverges from the frontend web theme (`frontend/src/index.css` OKLCH) — the web side has NOT been updated to match the desaturation/Klein blue. See [[project-install-frontend]] and the design-sync setup. Relates to [[feedback-design-aesthetic]].
