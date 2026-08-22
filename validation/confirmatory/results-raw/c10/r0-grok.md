**CALL: Reject.** Store local wall time + TZID + RRULE; expand to UTC at read/query time. Do not freeze UTC instants at create.

**CLAIM 1:** “Every Monday 09:00 America/New_York” is a wall-clock rule, not a bag of instants.  
**EVIDENCE:** Intent is 09:00 New York on each Monday. UTC of that wall time is 14:00 in EST and 13:00 in EDT. iCalendar (RFC 5545) stores `DTSTART;TZID=…` + `RRULE` for this reason.  
**CONFIDENCE:** 0.95  
**FALSIFIER:** Product spec says “fixed UTC, local display may drift.”

**CLAIM 2:** Converting with the *current* offset at create (the usual form of this proposal) shifts wall time by 1h across DST.  
**EVIDENCE:** A Monday created in January 2026 as 09:00 EST → 14:00 UTC still converts to 10:00 EDT in July. Spring-forward gaps / fall-back duplicates are also resolved once, with a TZDB that may not match the occurrence date.  
**CONFIDENCE:** 0.93  
**FALSIFIER:** Create-time expansion uses TZDB *per occurrence date*, and a July instance still renders 09:00 NY.

**CLAIM 3:** Even per-date create-time conversion goes stale when governments change rules.  
**EVIDENCE:** IANA TZDB updates several times a year. Frozen UTC keeps the old offset after “permanent DST” / offset-shift laws; attendees then see 08:00 or 10:00 local.  
**CONFIDENCE:** 0.88  
**FALSIFIER:** Recurrence window is short *and* you re-expand on every TZDB deploy.

**CLAIM 4:** Open-ended series cannot be fully materialized.  
**EVIDENCE:** “Every Monday, no until” is unbounded. A stored UTC list is a silent truncation plus a later recompute job that reintroduces TZ drift.  
**CONFIDENCE:** 0.90  
**FALSIFIER:** Product hard-caps a small finite count and never extends it.

**VERDICT:** Do not store create-time UTC for future local recurrences. **Biggest risk:** DST (then TZDB law changes) putting meetings at the wrong local hour. **Cheapest test:** Create “every Monday 09:00 America/New_York” spanning US spring-forward; assert every instance is 09:00 in that zone (not 08:00/10:00).
