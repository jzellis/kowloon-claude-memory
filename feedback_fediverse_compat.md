---
name: Kowloon fediverse compatibility philosophy
description: How to weigh Kowloon-specific features vs. fediverse interop
type: feedback
---

Fediverse interop is a good goal but not a constraint. If other fediverse software were good enough, Kowloon wouldn't need to exist. Mastodon and others recreate monolithic social network architecture in federated form, with the same limitations and relationship models. Kowloon is deliberately different (hybrid push/pull federation, richer relationship model).

**Rule:** If something isn't compatible with Mastodon/Pleroma/etc. but is genuinely valuable for Kowloon, implement it and let others catch up. Do not water down features for compatibility.

**Why:** The whole point of Kowloon is to explore better designs, not to be a Mastodon clone.

**How to apply:** When evaluating a feature, the question is "is this right for Kowloon?" not "will Mastodon support this?" Note ecosystem reality as context (e.g., nobody else does content signing yet) but don't use it as a reason to avoid the feature.
