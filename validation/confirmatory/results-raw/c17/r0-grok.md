CLAIM: Public marketing/content and a logged-in CRUD app should share one router, design system, and auth cookie — not two stacks. SSR/RSC is the default that does both without a second site.

EVIDENCE: Marketing URLs need HTML for crawlers and LCP; dashboards need client state. App Router (or equivalent) is designed for that split: server pages at the edge of the graph, `"use client"` islands for tables, filters, editors. A SPA plus JSON API still needs a prerender/SSG layer for SEO, which is A in disguise plus glue.

CONFIDENCE: 0.75

FALSIFIER: Marketing is a handful of Webflow/Astro pages owned by a different team, and the product is 100% behind login with no public content.

CLAIM: Authenticated dashboards do not benefit from full-document SSR the way marketing pages do; most CRUD will be client components anyway.

EVIDENCE: Sessioned list/detail/edit flows are latency-bound on API + hydration/TTI, not on first HTML. Teams that “go RSC” often ship 70%+ `"use client"` and a server action/route-handler layer that is a JSON API with extra cache semantics.

CONFIDENCE: 0.7

FALSIFIER: The first ten app routes stay mostly server-fetched with no client boundary leakage and TTI beats a SPA shell + API on the same CRUD list.

CLAIM: B2B still needs a real HTTP API (SSO extras, partner integrations, later mobile). RSC-to-DB as the only data path delays that contract.

EVIDENCE: Integrations show up in B2B roadmaps even when v1 is “just the web app.” If RSC talks to the DB and a JSON API is added later, you maintain two access paths.

CONFIDENCE: 0.65

FALSIFIER: A year in, still no external consumer and server actions/loaders remain the only mutation/query surface.

CLAIM: Choosing B as the *product* renderer makes organic/marketing pages the afterthought they usually become.

EVIDENCE: SPA shells fail CWV/indexability unless someone owns prerender. The prompt names SEO as a requirement, not a nice-to-have.

CONFIDENCE: 0.8

FALSIFIER: Marketing already has a working static site with indexed URLs and good LCP, so the app can be a pure SPA.

VERDICT: A — Biggest risk: paying App Router cache/boundary tax while the real app becomes a client SPA *and* you still build a JSON API. Cheapest test: implement one marketing URL and one authenticated CRUD list; if the list is `"use client"` + an API hop and TTI is no better than a Vite SPA, A was the wrong default for the app (keep A only for marketing).
