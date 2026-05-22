---
name: project-mobile-app-scaffold
description: "Kowloon mobile app at /home/jzellis/Projects/kowloon/mobile (repo github.com:jzellis/kowloon-mobile). Expo SDK 55 + Expo Router + NativeWind v4, plain JS. Multi-account Redux + namespaced AsyncStorage. Full auth trilogy (login + register + QR invite scan) working end-to-end as of 2026-05-21."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

The mobile codebase is a fourth Kowloon repo sibling to server/frontend/client. Scaffolded 2026-05-20. Login flow shipped 2026-05-21 (commit `bf02db4`).

- **Repo**: `github.com:jzellis/kowloon-mobile` (public)
- **Path on jarvis**: `/home/jzellis/Projects/kowloon/mobile`
- **Stack**: Expo SDK 55 (managed) · Expo Router (file-based) · React Native 0.83 · React 19 · plain JS (no TS) · NativeWind v4 (Tailwind v3 under the hood) · Redux Toolkit + react-redux
- **Network layer**: `@kowloon/client` linked from `../client` via `file:` dep. Never call fetch directly from mobile code — go through the client modules. See [[project-mobile-strategy]] for the broader strategy.

### Architectural commitments — settled and durable

- **Multi-account from day one.** State model: `accounts.list[] + accounts.activeId`. Per-account auth tokens live in a namespaced AsyncStorage adapter (key prefix `kowloon:<accountId>:`) — *not* in Redux. Each account gets its own `KowloonClient` instance, cached by account ID in `src/lib/client.js`. v1 UX exposes a single account; switching is a 1-action Redux dispatch when the UI lands.
- **Push notifications**: per-account, not per-device. App registers its Expo push token with each Kowloon server it has an account on. Server-side `PushSubscription` collection — not yet built.
- **Web is NOT a target.** Expo's `react-native-web` exists but the web frontend at `../frontend` is the production web.
- **Admin is split**: full admin stays web-only; mobile gets a small "moderation tray" for time-sensitive actions. Not yet built.

### Status as of 2026-05-21

**Working end to end (tested on Android via Expo Go over Tailscale):**
- Welcome screen, three CTAs (Log in / Register / Scan invite)
- **Login**: parses `@user@domain`, infers baseUrl, optional server URL override for local dev, POST `/auth/login` via `@kowloon/client`, token written to namespaced AsyncStorage, account stored in Redux, redirect to feed
- **Register**: two-stage — enter server domain, fetch `GET /` for server info + rules, then form with username/email/password/confirm/(optional)invite-code + one checkbox per rule. Submit disabled until every rule ticked. Payload includes `acknowledgedRules: [ruleIds]`. Handles both `{ token, user }` (auto-login) and `{ requiresVerification: true }` (parks on `/verify-email` holding screen) response paths.
- **QR scan**: `expo-camera` `CameraView` + `onBarcodeScanned` + permission gating. `parseInviteUrl` in `src/lib/inviteUrl.js` handles two formats: `kowloon://register?server=...&inviteCode=...&serverUrl=...` and `https://<server>/invite/<code>`. Recognized QR latches and routes to `/register` with prefill; unrecognized ones surface a transient error and keep scanning.
- Placeholder feed showing active account + Sign out (which tears down client cache + namespaced storage + Redux entry)
- Account hydration from AsyncStorage on app boot; survives restart

**Not yet built:**
- Real feed (currently just shows account info + sign-out)
- Account picker / switcher chrome (architecture supports it; no UI yet)
- Post composer
- Notifications
- `verify-email` is a holding screen only — the verification link itself is handled by the web frontend, after which the user comes back and logs in.

### Gotchas — hard-earned

- **`.npmrc` pins `legacy-peer-deps=true`**. Without it `npm install` errors on React 19 strict peer resolution (expo-router pulls in vaul + radix transitively).
- **legacy-peer-deps breaks hoisting**, so several transitive deps must be installed explicitly at the top level: `babel-preset-expo`, `react-native-worklets` (split out of reanimated 4.x), and occasionally others. Symptom: Metro bundling fails with "Cannot find module X" pointing at a deeply nested path. Fix: `npx expo install <pkg>` to add it to the top-level `package.json`.
- **NEVER name a sibling folder `src/app/`**. Expo Router's auto-detector treats any `app/` directory it finds as a potential routes root and picks one over the other unpredictably ("Using src/app as the root directory for Expo Router" was the symptom). Redux store lives at `src/state/` instead.
- **`metro.config.js` needs `unstable_enableSymlinks: true` and `unstable_enablePackageExports: true`** for the `file:../client` dep to resolve. Also `watchFolders` must include `../client` so hot reload picks up client lib changes. Do NOT set `disableHierarchicalLookup: true` — it breaks Metro's ability to find nested transitive deps like `@expo/metro-runtime`.
- **`@kowloon/client` hardcodes its AsyncStorage key as `kowloon_token`.** For multi-account, the wrapper in `src/lib/storage.js` (`makeAccountStorage(id)`) namespaces every key with `kowloon:<accountId>:`. Pass this adapter explicitly to every `new KowloonClient({ storage })` — never let the client fall back to its own default storage.
- **Dev-server reachability from a phone**: phone can't resolve "localhost"; it must hit jarvis's Tailscale IP `100.83.23.39:3000`. The "Use a different server URL" toggle on the login screen handles this. (Production flow needs no override — `@user@kwln.org` auto-derives `https://kwln.org`.)
- **Hermes-to-Metro `console.log` bridge is unreliable** — log lines sometimes don't appear in the Metro tmux pane even when the JS is definitely executing. For debugging hangs, render trace lines in the UI itself (a `useState`-backed list with a `log()` setter) — much more reliable than `console.log`.

### Theme tokens

`tailwind.config.js` mirrors the web frontend's editorial kowloon theme as hex approximations of the source OKLCH values:
- `base-100` `#FAF4E8` (warm cream paper)
- `primary` `#5588B1` (desaturated steel blue)
- `secondary` `#393B7A` (medium navy)
- `accent` / `error` `#C0394A` (vermillion)
- `success` `#2F9956`
- Post type accents `post-{note,article,media,link,event}`
- `borderRadius` zeroed out everywhere — no pill shapes
- Custom font families (`reading`, `ui`, `mono`) resolve to system serif/sans for now; real font assets via `expo-font` is future work

### Files worth knowing

- `app/_layout.js` — Provider + Stack + status bar + hydration boot
- `app/index.js` — splash → redirect (welcome vs feed) based on accounts state
- `app/welcome.js`, `app/login.js`, `app/register.js`, `app/scan.js`, `app/verify-email.js`, `app/feed.js` — wired screens
- `src/state/store.js`, `src/state/accountsSlice.js` — Redux
- `src/lib/identity.js` — `parseKowloonId`, `inferBaseUrl`, `domainFromUrl`
- `src/lib/inviteUrl.js` — `parseInviteUrl` (QR + deep-link → server/inviteCode/serverUrl)
- `src/lib/storage.js` — `makeAccountStorage`, `purgeAccountStorage`, root-level `ROOT_KEYS`
- `src/lib/client.js` — `ensureClient`, `forgetClient`, `getCachedClient` (in-memory map keyed by account ID)
- `src/lib/useActiveClient.js` — `useActiveClient` hook returning the active account's `KowloonClient`
- `src/components/ui/{Button,Field,Heading}.jsx` — primitives in editorial style
- `babel.config.js`, `metro.config.js`, `tailwind.config.js`, `global.css` — NativeWind config
- `mobile/CLAUDE.md` — project-level conventions doc
