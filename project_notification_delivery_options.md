---
name: notification-delivery-options
description: "Trade-offs for mobile notification delivery (foreground polling vs WebSocket vs push). We're shipping foreground polling first; later moves are documented here so we don't re-litigate."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

How the mobile app gets notified that *new notifications exist on the server*. This is separate from how the notification list itself renders (Phase A already handles that on focus + pull-to-refresh).

**Current state (as of 2026-06-05):** Foreground polling of `unreadCount` every 60s while the app is open. Matches the web frontend's existing behavior (Header polls every 60s with `setInterval`). Implemented as a context provider mounted in `_layout.js` so a single poll fans out to every consumer (feed masthead badge, user-menu row, etc.). Pauses when the app is backgrounded via `AppState`.

**Why we're not doing real-time / push yet:**

- **WebSocket / SSE:** The server has no WebSocket layer today. Adding one is a sizable lift on both the server (new transport, auth, fan-out, federation considerations) and the mobile client (reconnect logic, foregrounding behavior, fallback to poll). 60s polling is good enough for an alpha; revisit if testers report it feels stale.
- **Push notifications:** What we actually want long-term and what alpha testers will expect. Requires:
  1. Switching off Expo Go to an EAS-built custom dev client — same blocker as [[mobile-share-intake-todo]]. Bundle these together when we make the switch.
  2. `expo-notifications` for device-token registration.
  3. APNs cert (iOS) + FCM project (Android).
  4. Server endpoint to register/store device tokens per user (probably `/users/me/devices` or similar — there's nothing today).
  5. Integration with the existing notification create path so a push fires alongside the DB write.
  6. Notification preferences UI so users can opt out per type (the server already has `user.prefs.notifications.{reply,react,…}` flags — wire those in).

**How to apply:**
- When picking up real-time notifications, decide first whether to switch to a dev client (unlocking push *and* share-intake at once) or build a server WebSocket. Default lean: dev client + push, because users expect push and it works backgrounded.
- The foreground polling implementation lives in `mobile/src/lib/UnreadCountContext.js` — when push lands, the badge consumer doesn't need to change; the provider just learns to invalidate on push-receipt instead of polling.
- Don't propose "fix the federation case" for push — pushes are local-device events, not federated content.

Related: [[mobile-share-intake-todo]], [[mobile-groups-moderation-todo]].
