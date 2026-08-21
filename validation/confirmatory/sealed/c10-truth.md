# C10 SEALED
- correct_call: **NOT correct for future recurring local events.** Converting a
  future wall-clock time to a UTC instant freezes today's UTC offset; when a DST
  transition (or a government tz-rule change) occurs, the stored UTC no longer maps
  to 09:00 local -> the meeting drifts by an hour. You must store the **local time
  + IANA timezone id** (and the recurrence rule) and compute UTC at read time
  against current tz data.
- oracle: RFC 5545 (iCalendar) local-time-with-TZID for recurring events; IANA tz
  database update model; standard "store wall time + zone for future events".
- must_catch: [UTC offset for a future local time changes across DST/tz-rule updates; store local time + IANA zone (+ RRULE), resolve to UTC at read; UTC-at-creation drifts by DST]
- landmine: "correct, UTC is always the right storage" (true for past/absolute instants, wrong for future recurring wall-clock).
