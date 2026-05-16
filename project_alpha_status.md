---
name: Alpha status and next steps
description: Current state of Kowloon as of 2026-04-19 and what remains before alpha
type: project
---
As of 2026-04-19, the broad strokes are done. Server, client, and frontend all work end-to-end locally (PM2 + Vite) and via the Docker single-image install system.

**What's working**
- Full auth flow (register, login, JWT, session restore)
- Post creation, replies, reacts, feed display, type filtering, manual refresh
- Circle creation (two-step modal: create -> add members with local search + remote WebFinger lookup)
- Group creation, joining, posting, member management
- File uploads (S3/MinIO backend, authenticated proxy)
- Admin panel (users, posts, groups, circles, flags, settings, pages, invites, system stats)
- S2S federation (WebFinger, actor fetch, inbox/outbox, batch-pull)
- Install system (install.sh wizard, Docker image at ghcr.io/jzellis/kowloon:latest)
- Public homepage (anon = server posts; auth = circle feed with selector in filter bar)
- Admin Pages CRUD (create/edit/delete/restore with rich text editor + featured image upload)
- Admin batch select + hard delete across Users, Posts, Groups, Pages
- Delete activity parser accepts target as string or array of IDs
- Invite QR codes (generated on create, stored as base64 PNG, modal with save/copy in admin)
- Admin deleted-filter bug fixed across all list pages (was sending wrong param name)
- Page featured image: stored as file ID, resolved at display time via client.files.serveUrl()

**SEO / social previews (2026-04-19)**
- Bot detection middleware (Googlebot, Slackbot, Twitterbot, Discord, WhatsApp, etc.)
- OG + Twitter Card + JSON-LD meta shell for /, /posts/:id, /users/:id, /groups/:id, /pages/:id
- GET /robots.txt (static, disallows admin/auth/files)
- GET /sitemap.xml (dynamic — up to 1000 public posts + users + groups + pages)
- All in methods/seo/ — no SSR, no frontend changes

**RSS feeds (2026-04-19)**
- GET /posts?rss, /users/:id/posts?rss, /groups/:id/posts?rss
- methods/rss/index.js — toRSS() utility, 20 items, text/xml content-type (renders in browser)
- Plain Express middleware before each existing route handler, no changes to route() wrapper

**Circle UX (2026-04-19)**
- PostList accepts emptyMessage prop; HomePage passes circle-specific message
- getTimeline already pulls remote members synchronously on first fetch — no polling needed

**Email sending (2026-04-19)**
- methods/email/index.js — nodemailer + Ethereal fallback (preview URL logged in dev)
- POST /auth/forgot-password + POST /auth/reset-password (token in DB, 1hr TTL)
- Individual invites auto-email on admin creation
- SMTP configured via SMTP_HOST/PORT/USER/PASS env vars → emailServer setting

**Email verification on signup (2026-04-22)**
- User schema: emailVerified, emailVerificationToken, emailVerificationExpires fields
- requireEmailVerification boolean setting (default false, email group in admin)
- POST /register: when enabled, requires email, skips JWT, sends 24h verification email, returns { requiresVerification: true }
- POST /auth/login: returns 403 + { unverified: true } if setting on and user unverified
- GET /auth/verify-email?token=: verifies token, issues JWT (logs user in)
- POST /auth/resend-verification { email }: resends email, always 200 (prevents enumeration)
- verificationEmail() template in methods/email/templates.js

**Admin section completed (2026-04-19)**
- Pages CRUD with rich text editor and featured image upload
- Active/Deleted filter fixed across all list pages
- Batch select + hard delete on Users, Posts, Groups, Pages
- Delete activity parser accepts target as string or array
- Invite QR code modal (save/copy)
- Settings page: type-aware editing (boolean toggle, JSON textarea, click-to-edit value cells)
- Dashboard stat cards link to their respective admin list pages

**Post-edit form fix (2026-04-23)**
- Single-post endpoint now merges raw Post model data for the owner (source, title, href, to, tags, location, startTime, endTime)
- EditPostPage field names corrected: title (not name), source.content (not source), tags as string[] (not AP objects)
- client/activities.updatePost: added type and href mapping

**FeedItems hard-delete on post soft-delete (2026-04-23)**
- Delete handler now purges FeedItems on soft-delete (was tombstoning); saves storage, speeds federation
- writeFeedItems extracted to methods/feed/writeFeedItems.js (shared by Create handler + admin restore)
- Admin post restore re-populates FeedItems via writeFeedItems

**Delete timeline refresh (2026-04-23)**
- PostToolbar onDeleted callback pattern — passes deleted post id up to parent feed hook
- useFeed.removeItem removes post from local state without refetch
- PostCard/PostList/all feed pages wired up; PostPage navigates back on delete

**Profile / actor cache (2026-04-23)**
- refreshActorCache: propagates name/icon changes to denormalized actor copies in FeedItems after profile update
- GET /auth/me endpoint: returns fresh profile+prefs for session restore (avoids stale JWT snapshot)
- UserAvatar: shows live profile icon for current user; handles image load errors

**Garbage collection job (COMPLETE 2026-04-23)**
- `gcRetentionDays` setting: number, default 30, maintenance group in admin (1-365 days)
- `methods/gc/index.js`: 3-pass nightly job, starts 1min after server boot then every 24h
  - Pass 1: hard-delete soft-deleted Post/Reply/Page/Group/User/Bookmark past cutoff + their Files + FeedFanOut rows
  - Pass 2: purge orphaned remote FeedItems with no FeedFanOut records (unfollowed remote authors)
  - Pass 3: hard-delete expired soft-deleted File records from storage + DB
- Wired into index.js alongside outbox and poll workers

**Remaining before alpha**
1. Event type / RSVP system — can defer post-alpha

**Alpha checklist is otherwise complete.**

**How to apply:** No active coding task. Next session ask what's next.
