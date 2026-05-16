---
name: Geolocation feature — completed 2026-04-29
description: Post geotagging with GPS button and Nominatim forward geocoding autocomplete
type: project
---

Full geolocation support for posts, completed 2026-04-29.

**Why:** Users wanted to geotag posts with either GPS or a typed place name search.

**How it works**
- Post schema has `location: GeoPoint` (subschema: `{ type: "Point", coordinates: [lng, lat], name }`) — GeoJSON format, longitude first.
- Frontend sends `{ type: "Place", name, lat, lon }` from the composer.
- Create handler (`ActivityParser/handlers/Create/index.js`) normalizes this to GeoPoint format: `{ type: "Point", name, coordinates: [lon, lat] }`. Plain string locations get `coordinates: [0, 0]` as sentinel.

**LocationField component** (`src/components/posts/LocationField.jsx`)
- Two variants: `variant="composer"` (borderless inline, PostComposer) and `variant="page"` (bordered input, NewPostPage/EditPostPage).
- GPS button calls `navigator.geolocation.getCurrentPosition` + Nominatim reverse geocoding for place name. Errors shown in the field ("Location access denied" / "Location unavailable") — never silent.
- Debounced forward geocoding autocomplete (400ms): Nominatim `/search` → dropdown of results. Selecting a result sets both the display name and `geo` coords.
- Dropdown uses `createPortal` + `position: fixed` to escape PostComposer's `overflow-hidden` modal. `onMouseDown` + `e.preventDefault()` on items prevents blur-before-click race.
- Skips search when `geo` is already set (GPS was used successfully).

**geocodingUrl setting**
- Added to `config/defaultSettings.js` as a public admin-configurable setting (`group: "integrations"`).
- Defaults to `https://nominatim.openstreetmap.org`. Admins can point at a self-hosted Nominatim or Photon instance.
- Exposed via `GET /` (public settings) → `state.server.settings.geocodingUrl` in Redux.
- Used by LocationField for both forward and reverse geocoding URLs.

**How to apply:** LocationField is the canonical location input component. Use it everywhere a location field is needed. Always pass `geocodingUrl` from `state.server.settings?.geocodingUrl`.
