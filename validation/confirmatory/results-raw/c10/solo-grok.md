# Decision memo — C10: storing times for future recurring events

**CALL:** Reject the proposal. Do not materialize future recurrence instances to UTC at creation. Persist civil time + IANA TZID + RRULE; convert each occurrence to a UTC instant at read/fire time with current tzdata.

Storing UTC freezes a snapshot of timezone policy into instants. “Every Monday 09:00 America/New_York” is a wall-clock rule, not a list of moments. UTC offsets for that rule change on DST, and the IANA rules themselves change after legislatures move. Precomputed UTC is therefore a stale cache of policy, not the source of truth.

**Key risks**
- DST: instances computed with the creation-time offset (or a single offset for the series) display and fire one hour off after the next transition.
- tzdata updates: a later zoneinfo change (DST abolished, offset shift) leaves every stored future stamp wrong while the wall-clock intent is unchanged.
- DST gaps/folds: “2:30 AM” on a spring-forward day does not exist; on a fall-back day it exists twice. Instants cannot encode the skip/fold policy.
- Series length: infinite or far-future RRULEs cannot be fully expanded; edits, EXDATE, and RDATE fight a bag of stamps.
- Render UTC→local shows the viewer’s equivalent instant, not the scheduled 09:00 in the event timezone, so the calendar lies about what was booked.

**Single biggest risk:** After a DST transition, the meeting is no longer Monday 09:00 in America/New_York — it is an hour early or late, and people miss it.

**Cheapest disproof:** Create “weekly Monday 09:00 America/New_York” in February (EST, UTC−5). Inspect a Monday after the March DST change. If that instance is stored as 14:00 UTC (or renders as 10:00 EDT), the design is wrong. Correct storage still says 09:00 America/New_York; correct UTC for that day is 13:00.
