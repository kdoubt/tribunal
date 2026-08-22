**CLAIM:** No. UTC instants alone are not the canonical representation of a future recurring wall-clock event.  
**EVIDENCE:** “Every Monday 09:00 America/New_York” encodes local time, IANA zone ID, and recurrence rule. RFC 5545 recommends local `DTSTART` plus time-zone reference so instances remain at the same local time across offset changes. [RFC 5545 §3.8.5.3](https://www.rfc-editor.org/rfc/rfc5545.html#section-3.8.5.3)  
**CONFIDENCE:** 0.99  
**FALSIFIER:** The product contract explicitly defines the recurrence as immutable instants rather than “09:00 New York.”

**CLAIM:** Precomputed UTC occurrences are acceptable only as a bounded, rebuildable cache.  
**EVIDENCE:** UTC is correct for execution and indexing after resolving an occurrence, but the source of truth must retain `{local start, TZID, recurrence rule, exceptions}`. Indefinite recurrences cannot be fully materialized.  
**CONFIDENCE:** 0.98  
**FALSIFIER:** Every series has a short fixed horizon and may legally ignore later time-zone-rule changes.

**CLAIM:** DST gaps and overlaps require an explicit resolution policy.  
**EVIDENCE:** Some local times do not exist or occur twice. RFC 5545 defines handling for such cases; a system must deliberately adopt and consistently apply compatible semantics. [RFC 5545 §3.3.5](https://www.rfc-editor.org/rfc/rfc5545.html#section-3.3.5)  
**CONFIDENCE:** 0.97  
**FALSIFIER:** Allowed schedules can never intersect any offset transition.

**VERDICT:** **CALL: Reject UTC-only storage; store the zoned local recurrence and derive UTC per occurrence, optionally caching a rolling horizon.** Biggest risk: a government changes time-zone rules after creation, leaving stored UTC instants at the wrong local hour. Cheapest discriminating test: create a future series, resolve its UTC cache, substitute a test tzdb with a changed future offset, and verify regenerated occurrences remain at 09:00 while the old UTC-only rows do not.
