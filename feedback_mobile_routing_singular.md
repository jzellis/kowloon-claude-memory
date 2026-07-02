---
name: mobile-routing-singular
description: "Expo Router mobile routes are all singular (group, circle, user, post) — never plural. Plural paths throw \"Unmatched Route\" at runtime."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

All Expo Router route directories in `mobile/app/` use singular names:

- `app/group/[id]/` — NOT `groups/[id]`
- `app/circle/[id]/` — NOT `circles/[id]`
- `app/user/[id]/` — NOT `users/[id]`
- `app/post/[id]/` — NOT `posts/[id]`

Any navigation call using the plural form causes an "Unmatched Route" error at runtime (Expo Router throws, not a graceful 404). The crash appeared in `LeftDrawer.jsx` where group and circle links used plural paths.

**Why:** Expo Router is file-based — the route IS the directory path on disk. There is no router config to rename routes. The singular convention was chosen when the screens were scaffolded. Plurals in navigation code silently compile and only fail at tap-time.

**How to apply:** When writing any `router.push()` / `href` / `onNavigate()` call in mobile code, always check that the segment matches the actual directory name in `mobile/app/`. Grep `mobile/app/` before assuming a path exists.
