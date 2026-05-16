---
name: SEO / social preview strategy
description: How Kowloon handles SEO, OG tags, and social media previews
type: project
---
Bot-detection approach — no SSR. Express middleware checks User-Agent for crawlers/scrapers (Googlebot, Twitterbot, Slackbot, facebookexternalhit, etc.) and returns a lightweight HTML shell with full <head> meta tags. Real users always get the SPA.

**Why:** SSR would require major architecture changes. Bot detection is ~1 day of work, zero frontend impact, covers both SEO and social link previews.

**Scope:**
- robots.txt (static, served by Express)
- sitemap.xml (dynamic, generated from DB — public posts, users, groups, pages)
- OG / Twitter Card meta tags per route (title, description, og:image)
- JSON-LD structured data (Article, Person, Organization) — nice to have
- Server info page (/) — server name, description, icon as og:image

**Routes to handle:**
- `/` — server info (name, description, icon)
- `/posts/:id` — post title/body preview, author, og:image from featured image
- `/users/:id` — display name, bio, avatar as og:image
- `/groups/:id` — group name, description, icon
- `/pages/:id` — page title, body excerpt, featured image

**How to apply:** When touching any of these routes on the server side, check if the bot-detection middleware is in place before adding new public routes.
