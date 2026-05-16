---
name: User profile visibility model
description: User.to is capped to @public or @<own-domain>; profile is always discoverable; personal-info fields are gated per viewer.
type: project
---

User profiles are always discoverable. The user's `to` field only gates the **personal-info subset** of their profile, not the whole record.

**Why:** Fully-private profiles broke discovery and follow flows (you can't add to a Circle what you can't find). The model now mirrors post audience: `@public` or `@<domain>` (server-only).

**How to apply:**
- `User.to` accepted values: `@public`, `@<user.domain>`. Validated in `ActivityParser/handlers/Update/index.js` (accepts shorthand `"public"` / `"server"` from clients).
- Always-visible fields: `id`, `actorId`, `username`/`preferredUsername`, `name` (display name), `icon`, `url`, `inbox`, `outbox`, `publicKey`, follower/following collections, `domain`, `type`, plus public posts/circles via their own endpoints.
- Audience-gated fields (only visible if viewer matches `to` audience): `summary` (description/bio), `urls`, `pronouns`, `location`, `subtitle`. When gated, response includes `audienceRestricted: true`.
- `methods/visibility/helpers.js#canSeeObject` returns true for any active User/Person (404 only for `deletedAt` / `active: false`).
- `methods/sanitize/object.js#sanitizeUser` does the per-field gating; consumers must pass `viewer` from `getViewerContext(req.user?.id)`.
- Endpoints wired through: `routes/users/{id,actor,lookup,collection}.js`, `routes/utils/makeGetById.js`.
- Migration: `scripts/migrate-user-visibility.js` (supports `--dry-run`) coerces any legacy non-conforming `to` values to `@<own-domain>` (server-only as the safer default).
