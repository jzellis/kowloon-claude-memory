---
name: project-mobile-app-scaffold
description: "Kowloon mobile app lives at /home/jzellis/Projects/kowloon/mobile (repo github.com:jzellis/kowloon-mobile). Expo SDK 55 + Expo Router, plain JS, linked to @kowloon/client. Scaffolded 2026-05-20."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

The mobile codebase exists as a fourth Kowloon repo sibling to server/frontend/client. Scaffolded 2026-05-20.

- **Repo**: `github.com:jzellis/kowloon-mobile` (public, follows the same pattern as the other three)
- **Path on jarvis**: `/home/jzellis/Projects/kowloon/mobile`
- **Stack**: Expo SDK 55 (managed workflow) · Expo Router (file-based) · React Native 0.83 · React 19 · plain JS (no TS, matches everything else in this org)
- **Network layer**: `@kowloon/client` linked from `../client` via `file:` dependency. Same isomorphic library the web frontend uses. Never call fetch directly from mobile code — go through the client modules. AsyncStorage is auto-detected by the client's storage adapter.

### Architectural commitments — settled and durable

- **Multi-account from day one.** Every per-user piece of state (auth token, drafts, prefs, notification subscription) keyed by `account:<id>`. v1 UX may expose a single account; the *architecture* is multi-tenant. Switching accounts = rewriting an "active account id" pointer. See [[project-mobile-strategy]] for the broader strategy.
- **Push notifications**: per-account, not per-device. App registers its Expo push token with each Kowloon server it has an account on, with the account ID. Server-side needs a `PushSubscription` collection — not yet built.
- **Web is NOT a target.** Expo's `react-native-web` exists but the web frontend at `../frontend` is the production web. The mobile `npm run web` is only for quick previewing during development.
- **Admin is split**: full admin stays web-only (forms, tables, settings — needs a big screen); the mobile app gets a small "moderation tray" for time-sensitive actions (flag approval, join requests, urgent removals) that deep-links to the web for anything more involved. Not yet built.

### Gotchas

- **`.npmrc` pins `legacy-peer-deps=true`**. expo-router's transitive deps (vaul, radix) clash with React 19's strict peer resolution. Without the .npmrc every `npm install` errors out. Don't delete the file.
- **App.js + index.js are unwired but still in the tree.** The scaffolded entry files from `create-expo-app` were left in place — `main` in package.json now points to `expo-router/entry`, so they're orphans. Safe to delete in a follow-up; left for now to respect the no-destructive-without-confirmation rule.

### Files worth knowing

- `app/_layout.js` — root layout with GestureHandlerRootView + SafeAreaProvider + Stack
- `app/index.js` — placeholder home screen
- `app.json` — Expo config (slug `kowloon`, bundle ID `org.kwln.kowloon` for both platforms)
- `mobile/CLAUDE.md` — project-level conventions doc

### Not yet wired

- NativeWind (for Tailwind-ish styling — currently using StyleSheet)
- Redux Provider + store (will be account-scoped slices)
- Account add/select onboarding flow
- Theme tokens matching the web frontend's editorial palette
- EAS Build config + signing credentials
