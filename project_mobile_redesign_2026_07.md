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

**Implemented (2026-07-13, pushed to main on both repos):**
- Tokens (paper desaturation, `header`/`field`) — `d82a94e`.
- Bottom tab bar + server-icon drawer toggle — `3a607fb`. Bar is a router-driven `BottomTabBar` (router.replace between siblings), NOT an Expo Router `<Tabs>` layout.
- Klein header rollout on the 5 tab screens via `AppHeader`/`HeaderButton` + view-selector `ServerFeedIcon` overlay — `a2472a4`.
- Profile cover image + links dropdown (mobile) — `62d3f9c`; server `Profile.featuredImage` field — server repo `b4595513` (deployed to prod; additive/optional, `profile.urls` already existed).
- ALL UNVERIFIED ON DEVICE — no Expo on the dev box; changes were Babel-parse-checked only.

**Pending / not done:**
- **Hexagon compose FAB** — user still deciding orientation (avatars are flat-top). NOT integrated.
- **Auth hero** — waiting on commissioned artwork.
- **Klein header on SECONDARY screens** — post detail, user/server profile, settings, compose, circle/group detail + forms still have the cream `BackLink` masthead. The `AppHeader` component exists; applying it is mechanical (root drops top safe-area edge → add `<AppHeader back title=... />`).

**Scratch harness:** `mobile/design-preview/` — `archetypes.html` (7 archetypes, live palette switch, real tokens/fonts) + `render.mjs` (puppeteer → PNG/PDF in `out/`). Local scratch, uncommitted, its own `node_modules` (Chromium). Regenerate with `node render.mjs`. Iterating happens here, then locked bits move into the real app.

**Why:** These are design-language decisions the code alone won't explain, and several are mid-flight (pending items would otherwise look like omissions).

**How to apply:** Token changes are live in `tailwind.config.js` + `design-tokens.json` (+ ds-bundle, DESIGN_SYSTEM.md). Mobile now diverges from the frontend web theme (`frontend/src/index.css` OKLCH) — the web side has NOT been updated to match the desaturation/Klein blue. See [[project-install-frontend]] and the design-sync setup. Relates to [[feedback-design-aesthetic]].
