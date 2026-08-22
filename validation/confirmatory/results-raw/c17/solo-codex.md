DECISION MEMO — C17

CALL

A

Use server-side rendering/server components with a framework such as Next.js App Router. Render marketing and read-heavy content on the server; use client components only at interactive boundaries.

Key tradeoffs and risks:

- Strong SEO, fast first content, and shareable metadata come naturally for marketing pages.
- Server components reduce shipped JavaScript and can access data close to the server, improving content-heavy views.
- The application and marketing site share routing, authentication, design systems, and deployment rather than becoming two architectures.
- The cost is higher conceptual complexity: server/client boundaries, caching, invalidation, streaming, hydration, and framework-specific behavior.
- CRUD-heavy screens can become awkward if the team forces server rendering onto interactions better handled with local client state.
- Next.js coupling and operational complexity are greater than with a conventional SPA plus API.
- Keep business logic behind explicit service boundaries; do not make server components the only reusable interface to the backend.

Single biggest risk:

The team misplaces server/client boundaries and creates a slow, fragile app through excessive round trips, cache inconsistencies, and accidental full-route refreshes during CRUD workflows.

Cheapest falsification test:

Build one representative vertical slice before committing: an authenticated dashboard containing a server-rendered list, filtering, pagination, an optimistic edit, validation failure, and permission-aware mutation. Test it under realistic latency and production caching.

The team chose wrong if this slice cannot achieve reliable cache consistency and responsive interactions—specifically, p75 interaction latency above 200 ms on a typical corporate connection—or requires pervasive client components and repeated cache workarounds. In that case, choose B for the application and host the SEO-sensitive marketing pages separately.
