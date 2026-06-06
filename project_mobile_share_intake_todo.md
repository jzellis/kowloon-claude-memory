---
name: mobile-share-intake-todo
description: Deferred — adding Kowloon to the OS share sheet so users can share URLs/text/images from other apps into a prefilled Kowloon composer. Outbound share (Share.share) already works.
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

Inbound share-into-Kowloon support — Safari/Photos/etc. → Share → Kowloon → prefilled composer.

**Why deferred (2026-05-27):** requires moving off Expo Go to an EAS-built custom dev client (Share Extensions on iOS and intent-filters on Android both need native config that Expo Go can't load). Josh likes the current Expo Go workflow and isn't ready to switch yet. Outbound share (Share.share on the action bar) already covers the most-common direction.

**How to apply:** when we're ready to set up an EAS dev client for other reasons (push notifications, custom native modules, etc.), bundle this in the same switchover. Don't switch to a dev client just for this one feature.

**Recommended approach when revisiting:**
- Use `expo-share-intent` (community config plugin). Generates the iOS Share Extension + Android intent-filter, exposes a `useShareIntent()` hook. Couple of hours of work after the dev-client switch.
- Build a `/share-intake` route that inspects the shared payload and routes into `/compose`: URL → Link, plain text → Note, image → Media.
- Handle cold-start vs warm-start (the hook covers both), the unauth case ("log in first, your share is waiting"), and multi-account (which account receives the share).
- DIY native Share Extension is the other path — only worth it if we want the iOS extension to be its own mini-composer rendered inside the share sheet. Several days of work.

Related: [[project-mobile-app-scaffold]], [[project-mobile-strategy]].
