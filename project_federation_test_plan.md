---
name: Federation test suite — COMPLETE
description: 3-server federation test suite for Kowloon; achieved 100% pass rate (59/59 tests) on 2026-04-27
type: project
---

## Status: COMPLETE as of 2026-04-27

100% pass rate: 59/59 tests across 7 federation categories. Committed as `137a4bee`.

**Why:** Primary pre-alpha validation for federation layer. Three servers covers multi-server fan-out (the hard cases).

## Test results (2026-04-27)

| Category | Result | Avg latency |
|----------|--------|-------------|
| post-delivery | 15/15 | 83ms |
| reply-delivery | 15/15 | 1193ms |
| react-delivery | 15/15 | 1370ms |
| group-fanout | 2/2 | 3826ms |
| profile-update | 3/3 | 7ms |
| delete-propagation | 3/3 | 15ms |
| unfollow | 6/6 | — |

## Infrastructure

Three Docker containers: `kwln1.local`, `kwln2.local`, `kwln3.local`. `/etc/hosts` must have all three.
Docker setup: `docker-compose.yml`, `docker/nginx.conf`, `docker/gen-certs.sh`.
MongoDB port 27018. nginx on 8080/8443.

```bash
# First-time setup
bash docker/gen-certs.sh && docker compose build && docker compose up -d
echo "127.0.0.1  kwln1.local  kwln2.local  kwln3.local" | sudo tee -a /etc/hosts
# Seed each server
for URL in http://kwln1.local:8080 http://kwln2.local:8080 http://kwln3.local:8080; do
  TEST_BASE_URL=$URL node scripts/seed-test.js --wipe
done

# Run federation test suite (300 users, ~5 minutes)
node scripts/fed-test/index.js
```

## Script structure

```
server/scripts/fed-test/
  index.js            — orchestrator, runs phases in order
  config.js           — server URLs, SCALE constants, timeouts
  state.js            — in-memory state + writes state.json after each batch
  client.js           — thin wrapper around kowloon-client per server/user
  wait.js             — waitFor helper (polls until assertion passes or timeout)
  db.js               — direct MongoDB connection for verification queries
  phases/
    01-users.js       — register ~100 users per server
    02-local.js       — per-server local content
    03-cross.js       — cross-server federation activities
    04-verify.js      — summarize pass/fail report
  fixtures/
    names.js, content.js, images.js
```

State is written to `scripts/fed-test/state.json` (gitignored — contains auth tokens for 300 users).

## Federation bugs fixed to achieve 100% (committed 137a4bee)

### 1. `enqueueOutbox`: remote group posts never reached group server inbox
`resolveAudience()` can't parse `group:xxx@domain` format. Added explicit `scope === "group"` branch:
```js
} else if (federation?.scope === "group" && federation.groupId) {
  const atIdx = groupId.lastIndexOf("@");
  const groupDomain = groupId.slice(atIdx + 1);
  resolved = [{ target: `@${groupDomain}`, inboxUrl: `https://${groupDomain}/inbox`, host: groupDomain }];
}
```

### 2. `inbox/post.js`: group posts not fanned out to remote member servers
Added `fanOutGroupPost()` function — when a group post arrives at kwln2's inbox, it looks up the group's members circle, finds domains that aren't local/sender, and enqueues delivery to those domains.

### 3. HTTP Signature domain bypass for fan-out re-broadcasts
kwln2 re-signs and re-broadcasts kwln1 posts using kwln2's key. But the activity's `actorId` is from kwln1. Domain consistency check failed: actorId domain (kwln1.local) ≠ keyId domain (kwln2.local). Fixed by skipping domain check when `body.to?.startsWith("group:")`.

### 4. `Join` handler: remote group not tracked in local Groups circle
Remote group federation path returned immediately without recording membership. kwln3 couldn't build FeedFanOut records for remote group posts. Fixed by adding `Circle.updateOne` to add remote group ID to user's local `circles.groups` circle before federating.

### 5. `enqueueFanOut`: remote groups fell through to "public" type
`parseAudience()` skips tokens where domain isn't local — remote group IDs were being treated as "public". Fixed by adding `remote-group` detection (any token starting with `group:` with a non-local domain) and a handler that queries `Circle.find({ "members.id": groupId })` + `User.find({ "circles.groups": ... })` to find local members.

### 6. `visibility.js`: `canView` always returned false for remote "audience" posts
`origin === "remote"` branch used `Boolean(grants[viewerId])` which is always false (grants is always `{}`). Fixed to use `isInFanOutAudience` for both local and remote posts.

### 7. `Create` handler: federated posts got new IDs on receiving server
Pre-save hook generates ID with `this.id = this.id || ...`. Activity object never had `id` set, so each server gave posts local IDs. kwln3 stored `post:xxx@kwln3.local` instead of preserving `post:xxx@kwln1.local`. Fixed by setting `activity.object.id = activity.objectId` when `activity.federated && activity.objectId && !activity.object.id`.

### 8. Fan-out test groups might have no kwln1 members
Fan-out groups were picked from kwln2's open groups, but kwln1 users joined random kwln2 groups. Fixed by explicitly joining kwln1 AND kwln3 users to the designated fan-out groups in the test setup.

### 9. Unauthenticated verification of group posts always failed
Group posts have `to: "audience"` in FeedItems — they require a FeedFanOut grant. Switched verification from `anonClient('kwln3').feeds.getPost` to `clientFor('kwln3', kwln3User.id)`.

**How to apply:** When debugging federation issues, check these 9 areas in order: delivery (enqueueOutbox scope), fan-out (inbox post.js), signature bypass, membership tracking, FeedFanOut creation, visibility check, ID preservation, test setup, auth level.
