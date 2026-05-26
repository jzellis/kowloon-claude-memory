---
name: project-mobile-app-scaffold
description: "Kowloon mobile app at /home/jzellis/Projects/kowloon/mobile (repo github.com:jzellis/kowloon-mobile). Expo SDK 55 + Expo Router + NativeWind v4, plain JS. Multi-account. As of 2026-05-22: auth, typography system, home feed, composer, rich rendering all working end-to-end."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

The mobile codebase is a fourth Kowloon repo sibling to server/frontend/client. Scaffolded 2026-05-20.

- **Repo**: `github.com:jzellis/kowloon-mobile` (public)
- **Path on jarvis**: `/home/jzellis/Projects/kowloon/mobile`
- **Stack**: Expo SDK 55 (managed) · Expo Router (file-based) · React Native 0.83 · React 19 · plain JS (no TS) · NativeWind v4 (Tailwind v3 under the hood) · Redux Toolkit + react-redux
- **Network layer**: `@kowloon/client` linked from `../client` via `file:` dep. Never call fetch directly from mobile code — go through the client modules. See [[project-mobile-strategy]] for the broader strategy.

### Architectural commitments — settled and durable

- **Multi-account from day one.** State model: `accounts.list[] + accounts.activeId`. Per-account auth tokens live in a namespaced AsyncStorage adapter (key prefix `kowloon:<accountId>:`) — *not* in Redux. Each account gets its own `KowloonClient` instance, cached by account ID in `src/lib/client.js`. v1 UX exposes a single account; the account list + switcher chrome isn't built, but the data model is multi-tenant.
- **Push notifications**: per-account, not per-device. App registers its Expo push token with each Kowloon server it has an account on. Server-side `PushSubscription` collection — not yet built.
- **Web is NOT a target.** Expo's `react-native-web` exists but the web frontend at `../frontend` is the production web.
- **Admin is split**: full admin stays web-only; mobile gets a small "moderation tray" for time-sensitive actions. Not yet built.
- **Content is Markdown.** The server stores Markdown source and renders HTML itself (Create handler strips raw HTML, enforces `text/markdown`). The composer must send Markdown; feed objects come back with server-rendered HTML in `body`/`summary` (no `source` field), so the app *renders* HTML but *sends* Markdown.

### Status as of 2026-05-22 — all tested on Android via Expo Go over Tailscale

**Auth** (commits up to `16a7001`): Welcome screen; Login (`@user@domain` → infer baseUrl, server-URL override for local dev); Register (two-stage — server discovery + rules acknowledgement, handles email-verification path); QR invite scan (`expo-camera`, `parseInviteUrl`). Account hydration from AsyncStorage on boot.

**Typography system** (`793be6a`, server `aaf6198c`): Kindle-style curated reading controls. Five bundled fonts via `expo-font` (Inter default, Atkinson Hyperlegible, Lora, Merriweather, OpenDyslexic). `src/lib/typography.js` is the single source of truth. Four stepped prefs (fontFamily, fontSize, lineSpacing, columnWidth) stored at `user.prefs.typography` — a real subschema added to `server/schema/User.js`, synced via `@kowloon/client` `updateProfile`, debounced. `TypographyProvider` + `useTypography()`. Settings screen at `app/settings/typography.js`. Live on the post-detail reading surface.

**Home feed + composer + rich rendering** (`0512712`): 
- `useFeed` — paginated, pull-to-refresh, infinite scroll, refresh-on-focus. Rewritten in `3a04ac4` to take `{ viewKey, activeTypes }`: routes to `getServerPosts({ to: 'public' | 'server' })` or `getCirclePosts({ circleId })` and handles each endpoint's pagination (page-based for server, `before` cursor for circles).
- `PostCard` — editorial type-aware cards; renders `summary || body` as rich HTML (Notes have no `summary`, only `body`).
- `UserMenu` — avatar in the masthead opens it (Profile/Circles/Groups/Settings/Log out). Masthead shows the server's display name (`account.serverName`, fetched from `GET /`).
- Composer (`app/compose.js`, reworked `1e02eba`) — `PostTypeSelector` (all 5 types, hexagon icons), Note + Article composable via one **10tap** (TipTap-in-WebView) editor, `AudienceSelector` (Public/Server + circles, returns the real `to`). On submit: `editor.getJSON()` → `pmToMarkdown()` → `createPost()`. 10tap can't emit Markdown directly and RN has no DOM (so `turndown` is out) — hence the ProseMirror-JSON walker.
- `HtmlContent` — wraps `react-native-render-html` with editorial tag styles; used in cards and the detail view (detail wires in typography prefs).
- Post detail (`app/post/[id].js`) — rich body, typography applied. Reactions/replies not yet built.
- **Timeline filtering** (`3a04ac4`, server `d3f9ea0f`, client `4114662`): `FeedHeader` below the masthead — tappable view title opens a bottom sheet (Public / Server / Your circles) and a horizontal post-type filter row. Selection persists per account via `usePersistedFilter` (AsyncStorage). Server-side, `GET /posts?to=public|server` restricts visibility; circle posts via `GET /circles/:id/posts`. Saving the current view as a default to `user.prefs` is deferred (future).

**Not yet built:** account switcher chrome; Profile/Circles/Groups screens (stubs only); notifications; reactions/replies; Media/Link/Event composer types; the moderation tray.

### Known bugs / outstanding work

- **Feed-filter defaults don't reliably restore after sign-out → sign-in.** Auto-sync to `user.prefs.{defaultFeedView, defaultPostView}` *writes* correctly (verified server-side); the bug is the restore. `usePersistedFilter` has a race: when `client.auth.getUser()` returns the in-memory user before `AsyncStorage.getItem` resolves, the `awaitingFallback` ref isn't set yet when the late-fallback effect fires (`fallbackSig` already changed), so the fallback application is missed. Fix: convert `awaitingFallback` from a ref to state so the apply effect re-fires once hydrate completes and flips it true. Low priority — local AsyncStorage still persists within a session, this only affects fresh-device / post-signout cases.
- **Link composer should auto-set `target` when `href` is a Kowloon post URL.** The client lib's `createPost` already accepts a `target` field ("ID of the post being shared"), and the web composer's state has it — but nothing auto-detects "this href is a Kowloon post" and extracts the post ID. Affects both web and mobile when the Link composer gets built. When `href` matches a Kowloon post URL pattern (`https://<domain>/posts/post:<id>@<domain>`), the composer should set `target: 'post:<id>@<domain>'` so it becomes a first-class quote/share (and feed cards can render an embedded preview instead of a generic link card).
- **In-app routing for Kowloon URLs is partial.** `PostCard.openExternal` detects Kowloon *post* URLs (`/posts/post:<id>@<domain>`) and routes to `/post/[id]` in-app. Other Kowloon URL types (users `/users/<handle>`, circles `/circles/<id>`, groups `/groups/<id>`, server `/`) still fall through to the OS browser because those mobile screens don't exist yet (Profile/Circles/Groups are stubs; no user-by-handle detail; no remote-server-info screen). Once those screens land, extend `kowloonPostIdFromUrl` into a fuller parser that returns `{ type, id }` and route accordingly.

### Gotchas — hard-earned

- **`.npmrc` pins `legacy-peer-deps=true`**. Without it `npm install` errors on React 19 strict peer resolution.
- **legacy-peer-deps breaks hoisting** — some transitive deps must be installed explicitly at top level (`babel-preset-expo`, `react-native-worklets`). Symptom: Metro "Cannot find module X" at a deeply nested path. Fix: `npx expo install <pkg>`.
- **NEVER name a sibling folder `src/app/`** — Expo Router's auto-detector grabs any `app/` dir as a routes root. Redux store lives at `src/state/`.
- **`metro.config.js` needs `unstable_enableSymlinks` + `unstable_enablePackageExports`** for the `file:../client` dep; `watchFolders` includes `../client`. Do NOT set `disableHierarchicalLookup` — breaks finding nested deps like `@expo/metro-runtime`.
- **`@kowloon/client` hardcodes AsyncStorage key `kowloon_token`** — multi-account uses `makeAccountStorage(id)` in `src/lib/storage.js` to namespace it. Always pass that adapter to `new KowloonClient({ storage })`.
- **Dev-server reachability from a phone**: phone can't resolve "localhost" — must use jarvis's Tailscale IP `100.83.23.39:3000`. Login's "Use a different server URL" toggle handles it. The dev server's `DOMAIN` is `localhost`, display name "The Walled City".
- **Hermes-to-Metro `console.log` is unreliable** — logs sometimes don't reach the Metro pane. For debugging, render trace lines in the UI (`useState` list).
- **Metro restart**: after adding a dependency or changing babel/metro/tailwind config, restart with `npx expo start -c`. A bare `r` reload only suffices for JS edits. Restarting Metro drops the Expo Go connection — the phone must reconnect (re-scan / reopen) before bundling resumes.
- **`react-native-render-html` 6.3.x** — works in Expo Go (pure JS); may log harmless deprecation warnings. No bold-italic font bundled, so combined bold+italic degrades to one or the other.
- **10tap toolbar — place it as a STATIC bar on top of the editor, not a keyboard accessory.** The composer keyboard layout cost a long debugging session. Lessons: (1) 10tap's `Toolbar` self-hides via `hideToolbar = !isKeyboardUp || !editorState.isFocused`; the `isFocused` flag goes stale after the hidden-input→editor focus handoff, so pass `hidden={false}` to force it visible. (2) Its `FlatList` collapses to zero height unless wrapped in a fixed-height (44px) View. (3) Pinning it above the keyboard is fragile — putting it on top of the editor box (like the web composer) sidesteps all keyboard-position issues.
- **Expo Go doesn't reliably resize the Android window for the keyboard.** `KeyboardAvoidingView behavior="height"` double-counts / misbehaves. The composer instead pads its content area by a measured inset — `useKeyboardInset` computes `windowHeight - keyboardEvent.endCoordinates.screenY`. Exact keyboard insets on Android are genuinely hard; `react-native-keyboard-controller` is the real fix but needs a custom dev build (not Expo Go).
- **Post-type icons are rasterized PNGs** (`assets/post-icons/`), tinted per type via `Image` `tintColor` — not SVG. `react-native-svg-transformer` conflicts with NativeWind's Metro transformer, so the web SVGs were pre-rasterized with `sharp` instead.

### Theme

`tailwind.config.js` mirrors the web frontend's editorial kowloon theme: `base-100` `#FAF4E8` (cream), `primary` `#5588B1` (steel blue), `secondary` `#393B7A` (navy), `accent`/`error` `#C0394A` (vermillion), `post-{note,article,media,link,event}` accents, `borderRadius` zeroed (no pill shapes). The static `font-reading`/`font-ui` Tailwind tokens point at bundled fonts (Lora display serif / Inter sans); the user-selectable reading font is applied via inline styles, not Tailwind classes.

### Key files

- `app/_layout.js` — Provider + fonts (useFonts behind splash) + TypographyProvider + Stack
- `app/index.js` — splash → redirect (welcome vs feed)
- Screens: `welcome` `login` `register` `scan` `verify-email` `feed` `compose` `post/[id]` `profile` `circles` `groups` `settings/index` `settings/typography`
- `src/state/` — Redux store + `accountsSlice`
- `src/lib/` — `identity`, `inviteUrl`, `storage`, `client`, `useActiveClient`, `useFeed`, `useKeyboardInset`, `usePersistedFilter`, `typography`, `TypographyContext`, `pmToMarkdown`, `postTypes`, `timeAgo`
- `src/components/` — `HtmlContent`, `UserMenu`, `posts/{PostCard,Avatar,PostTypeIcon,PostTypeSelector,AudienceSelector,FeedHeader,FeedViewSelector,TypeFilter}`, `ui/{Button,Field,Heading,Checkbox,SegmentedControl}`
- `mobile/CLAUDE.md` — project-level conventions doc
