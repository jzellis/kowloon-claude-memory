---
name: current-focus
description: Active development focus is the mobile app (Expo/React Native) unless Josh says otherwise
metadata: 
  node_type: memory
  type: project
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

Primary development focus as of 2026-07-07 is the **mobile app** (`~/Projects/kowloon/mobile`), returning to it after a server-side federation debugging session.

**Why:** Josh specified mobile as default. The 2026-07-07 session was an exception -- debugging federation pull bugs (circle feed showing nothing on kwln.social). Those fixes are now committed.

**How to apply:** Any UI/UX work, component changes, or feature additions should target the mobile app first. Web frontend (`~/Projects/kowloon/frontend`) changes only when explicitly requested or when the change is server/client-lib level and applies to both.
