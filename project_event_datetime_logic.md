---
name: event-datetime-logic
description: Smart-default behaviour for Event post start/end dates+times in the composer
metadata: 
  node_type: memory
  type: project
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

The Event composer's date/time fields auto-fill sensibly so the common case (an hour-long event happening soon) takes one tap. Both web and mobile composers follow the same rules.

**State shape** — four separate strings, not two Date objects:
- `startDatePart`  (`YYYY-MM-DD`)
- `startTimePart`  (`HH:MM`)
- `endDatePart`    (`YYYY-MM-DD`)
- `endTimePart`    (`HH:MM`)

Combined on submit: `startTime = ${startDatePart}T${startTimePart}` (or just `${startDatePart}` if no time). Empty parts → `undefined`.

**Helpers**:
- `pad(n) = String(n).padStart(2, '0')`
- `addOneHourToTime('14:30') = '15:30'` — wraps midnight: `'23:30' → '00:30'`
- `splitDateTime('2026-05-30T14:30') = ['2026-05-30', '14:30']` — slices time to first 5 chars

**Auto-fill rules** (only on user setting the *start*; end is independently editable after):
1. **Setting start date**:
   - End date copies the start date (same day default).
   - If no start time yet: default it to the next round hour. `now.getMinutes() > 0 ? hour+1 : hour`, wrapping at 24. End time = start + 1h.
2. **Setting start time**:
   - End time = start + 1h.
3. **Setting end date or end time**: just sets it. No auto-cascade.

**Why this design** (vs. one big timestamp): users on mobile especially tap date and time separately (native pickers are mode-specific), and "I know the date, the time can wait" is a real flow. The web's `DateTimeField` exposes a date input + a time input side by side; the mobile equivalent uses `@react-native-community/datetimepicker` in mode `"date"` and mode `"time"` from one trigger row.

**Validation**: at minimum, an Event needs a start date. The web's `canSubmit` rule: `(content.trim() || (postType === 'Event' && startDate) || (postType === 'Media' && attachments.length))`. Title is also reasonable to require for Events but isn't strictly enforced; let the composer impl decide.

**Server**:
- `createPost` accepts `startTime` and `endTime` as ISO-ish strings.
- Server's Create handler maps these to `event.startDate` / `event.endDate` on the Post schema (see `ActivityParser/handlers/Create/index.js` around the type === "Event" branch).
