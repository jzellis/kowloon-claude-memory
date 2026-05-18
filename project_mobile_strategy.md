---
name: project-mobile-strategy
description: "Mobile is React Native, not a PWA. Don't propose service workers, offline queues, manifest.json, or vite-plugin-pwa work for the web frontend."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

The Kowloon mobile play is **native (React Native) apps**, not a PWA on the web frontend. jzellis confirmed 2026-05-18: "Facebook on web isn't a PWA, is it?" — i.e. once you have native apps, web becomes the share-link / desktop / quick-check surface, and offline-first on web is wasted effort.

**Why:** React Native work is starting soon. Offline queueing, background sync, push notifications, and the rich "mobile-first" features all belong to the native app. The `@kowloon/client` library is already isomorphic (works in Node, browsers, and React Native — see `client/CLAUDE.md`), so most of the auth/activity/feed plumbing carries over.

**How to apply:**
- Don't suggest PWA scaffolding for the web frontend: no `vite-plugin-pwa`, no `manifest.json` install prompts, no service worker, no IndexedDB offline queue, no background sync.
- For web frontend network resilience, prefer cheap in-page mitigations: localStorage drafts (`frontend/src/hooks/useDraft.js`), idempotency keys via `dedupeKey` on retry-prone Create activities (server handles dedupe generically in `methods/activities/create.js`), surfacing errors clearly. These are already wired into PostComposer/ReplyComposer/BookmarkComposer as of 2026-05-18.
- Reserve "real" offline/queue work for the React Native repo when it starts.
- Don't gold-plate the web UI with native-app patterns (pull-to-refresh, swipe gestures, etc.) past what works naturally in a browser.
