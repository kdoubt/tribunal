## Decision memo — C10

**CALL:** No—store the recurrence in local civil time (`Monday 09:00`), its IANA zone (`America/New_York`), recurrence rule, and exceptions; derive UTC occurrences using current tzdata, optionally caching only a short horizon.

**Key risks**

- Governments can change UTC offsets or daylight-saving rules after creation. IANA explicitly warns that conversions calculated before such changes can become wrong for future meetings. Precomputed timestamps would then preserve the obsolete UTC instant rather than the promised 09:00 wall time. [IANA tzdb theory](https://www.iana.org/time-zones/theory)
- An unbounded recurrence cannot be fully materialized, while a bounded expansion creates update, cancellation, and exception-management complexity.
- Local times can be nonexistent or ambiguous during clock transitions. The product must define a stable policy for gaps and overlaps.
- Re-expansion after a tzdata update can race with reminders or invitations unless generated occurrences are versioned or atomically replaced.

**Single biggest risk:** A time-zone rule changes after scheduling, silently shifting every affected meeting away from the user’s intended local time. Rendering stored UTC back into the zone cannot recover the original intent; that information was discarded.

RFC 5545 supports the intended representation: for recurrence with local-time semantics, `DTSTART` should carry local time and a time-zone reference, with the recurrence rule applied from that basis. [RFC 5545](https://www.rfc-editor.org/rfc/rfc5545.html)

**Cheapest falsification test:** In a unit test, define zone rules where `09:00` maps to 14:00 UTC, create a weekly series, then replace the future rule so it maps to 13:00 UTC. Regenerate occurrences. The design is wrong if the meeting still fires at 14:00 UTC—or displays as 10:00 local—instead of remaining at 09:00 local.

UTC remains correct for immutable past instants and execution/reminder records, but not as the sole source of truth for future recurring civil-time intent.
