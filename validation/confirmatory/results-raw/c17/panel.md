## Decision memo (independent cross-vendor panel)

**Recommendation: A** - both independent seats, in isolation, chose A.

### Seat 1 position
VERDICT: A — Biggest risk: the team mismanages server/client boundaries and caching, producing stale data, hydration bugs, and framework lock-in. Cheapest discriminating test: build one thin vertical slice—SEO marketing page plus authenticated CRUD dashboard with permissions, optimistic updates, and cache invalidation—and reject A if it requires pervasive client components or cannot reliably show fresh post-mutation data under production-like deployment.

### Seat 2 position
VERDICT: A — Biggest risk: paying App Router cache/boundary tax while the real app becomes a client SPA *and* you still build a JSON API. Cheapest test: implement one marketing URL and one authenticated CRUD list; if the list is `"use client"` + an API hop and TTI is no better than a Vite SPA, A was the wrong default for the app (keep A only for marketing).
