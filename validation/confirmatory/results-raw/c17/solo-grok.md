DECISION MEMO — C17 Rendering model

CALL
A

Use SSR / React Server Components (Next.js App Router or equivalent) as the default. Marketing and content pages render on the server. Dashboards and CRUD are Client Components at the leaf, talking to server actions or route handlers. One app, one router, one deploy.

Tradeoffs
A gives first-byte HTML, shareable URLs, and indexable marketing/content without a second site. Data fetching colocation cuts the BFF/JSON ceremony for read-heavy screens. The cost is a harder mental model: client boundaries, serialization, cache tags, and request waterfalls. Highly interactive CRUD still needs client state, optimistic updates, and often a JSON mutation path — so you do not escape SPA work; you contain it.

B is simpler for form-heavy dashboards and cleaner for a separate API (mobile, partners). It fails the stated SEO/content requirement unless you add prerender or a second frontend — at which point you rebuilt A with worse cohesion.

Biggest risk
The team marks page roots `'use client'`, keeps a JSON API for mutations, and still fights RSC caching. Result: SPA complexity plus server-runtime coupling, with no SEO or TTFB win.

Falsification test (cheap)
In two weeks, ship (1) one marketing/content URL and (2) one authenticated CRUD flow. Measure:

- Marketing LCP / HTML in first response (curl the URL; view-source must contain the primary copy, not an empty shell).
- % of authenticated routes whose page/layout is a Client Component at the root.
- Click → visible mutation on the CRUD screen (p95, local, warm).

You chose wrong if the marketing document is an empty shell **or** >70% of app routes are client-rooted **and** mutations already go through a separate JSON API. Then split: static/SSR marketing + SPA app. Until those numbers show up, stay on A.
