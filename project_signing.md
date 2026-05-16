---
name: Content signing architecture
description: How Post, Circle, and Bookmark content is signed; utility location and UserSchema methods
type: project
---

All user-authored content (Posts, Circles, Bookmarks) is signed with the author's RSA private key on every save. Signatures are stored as `Buffer` in the schema.

**Utility:** `methods/utils/signing.js` — `signData(privateKey, data)` and `verifyData(publicKey, data, signature)`. Use these when you have raw keys (e.g. verifying a remote actor's content during federation).

**UserSchema methods:** `user.sign(data)` → Buffer, `user.verify(data, signature)` → boolean. Thin wrappers around the utility. Use these whenever you have a User model instance.

**Signing payloads:**
- Post: `${id}|${source.content}` — signed in `pre("save")`, `pre("updateOne")`, `pre("findOneAndUpdate")`
- Circle: `${id}|${name}|${to}|${sorted-member-ids}` — re-signed on every save (covers membership changes); non-fatal if actor unavailable
- Bookmark: `${id}|${href||target}|${source.content}` — signed in `pre("save")`

**verifySignature():** Instance method on all three models. Looks up actor by `actorId`, calls `actor.verify(payload, this.signature)`.

**Why:** Tamper detection for federated content — HTTP Signatures only prove the delivering server, not the original author. Content signatures prove authorship independently of which server serves the content. Also load-bearing for migration: signed exports prove authenticity when re-imported to a new server.

**Note:** `createUserSignature` / `verifyUserSignature` on UserSchema are separate — they sign `userId:timestamp` for auth tokens, not content.
