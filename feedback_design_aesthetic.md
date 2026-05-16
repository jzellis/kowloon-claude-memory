---
name: Visual design taste — magazine over social-app
description: User-validated design choices for posts/avatars/typography. Lean magazine-y, not generic social-media app, but borrow proven UX from Twitter/FB where it serves the user.
type: feedback
---
The frontend `CLAUDE.md` says "feel like a beautiful magazine, not an app." Today's session refined what that means in practice. These are taste calls the user has confirmed; use them as the default direction unless told otherwise.

**Why:** the user is willing to take design hits to keep brand identity (e.g. accepted slightly-larger gap on a 2-line byline rather than going one-line social-media style), but only when they're _earning_ something. Pure aesthetic loyalty for its own sake gets called out ("too slavish to the design pattern").

**How to apply:**

- **Bylines / post meta = two lines**, magazine-style. Bold display name + small handle stacked, with very tight leading (`leading-[1.05]` or `leading-[1.1]`). Confirmed preference over a single-line `Name @handle` social layout. Use negative margin-top on the handle line to pull lines closer when leading alone hits the descender-clipping wall.

- **User avatars = circles, Circles/Groups = hexes.** People follow the universal "person = circle" convention; the hex tile stays as the brand mark on Circle/Group icons (semantically a "circle of people"). Rolled out today across UserAvatar + every direct-render site.

- **Subtle, not dramatic.** Avatar shadow is `0 1px 2px rgba(0,0,0,0.18)` — a hairline, not a chunky drop-shadow. The user explicitly cited the design notes ("no excessive shadows or 'app' chrome") when choosing.

- **Borrow Twitter/FB UX where it pulls weight**, not as wholesale aesthetic. Examples adopted today: smaller handle font than display name (Twitter sizing trick to let leading sit tighter), tap-to-zoom in lightbox (FB/IG pattern), swipe-to-navigate carousel. Examples kept distinct: hex Circle icons, magazine byline, no rounded corners anywhere.

- **Tailwind text-* utilities ship with built-in line-height** that overrides `leading-X` on a parent. To force tight leading on text-sm/text-xs elements, put `!leading-[N]` directly on the element (the `!` important modifier wins). Several rounds of debugging today were caused by this.

- **When asked "this or that?" on a design choice**, give a 2-3 sentence opinion grounded in the design notes + the tradeoff, not a both-sides-good non-answer. The user pushes back when an answer is wishy-washy and confirms when it's a clear take.
