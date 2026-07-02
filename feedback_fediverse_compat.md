---
name: kowloon-fediverse-compatibility-philosophy
description: How to weigh Kowloon-specific features vs. fediverse interop
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1d84a4ed-0961-4966-80a8-5133c21f41ef
---

Kowloon federates only with other Kowloon servers. There is no Mastodon, Pleroma, Misskey, or any other fediverse network interop planned for the foreseeable future. Do not raise cross-network compatibility as a consideration, ever.

**Rule:** Never ask "will this work with Mastodon/other AP servers?" It is not a goal. Federation means Kowloon-to-Kowloon only.

**Why:** The user has corrected this repeatedly and explicitly. Kowloon is its own ecosystem. Other networks have fundamentally different relationship models that are incompatible with Kowloon's design.

**How to apply:** When designing federation features, assume both endpoints are Kowloon servers. Do not add fallback paths, compatibility shims, or even mention other AP software unless the user explicitly raises it.
