CLAIM: A hybrid SSR/server-components architecture best matches mixed requirements: statically rendered or server-rendered marketing pages for SEO, plus interactive client components for dashboards and CRUD.
EVIDENCE: Next.js-style frameworks permit rendering boundaries rather than forcing every screen into one model. Marketing content can ship useful HTML with minimal JavaScript, while app routes hydrate only the controls needing browser state.
CONFIDENCE: High.
FALSIFIER: The framework cannot isolate client-side interactivity cleanly, causing most of the application to become hydrated client code anyway.

CLAIM: Server components can reduce browser bundles and simplify secure data access.
EVIDENCE: Data fetching and sensitive integration logic remain on the server; serialized results—not API credentials or unnecessary libraries—cross the network. A separate JSON API is still possible where external clients require it.
CONFIDENCE: High.
FALSIFIER: Mobile apps, partners, or multiple frontends require nearly every capability through a stable independent API.

CLAIM: A pure SPA creates avoidable complexity for this product mix.
EVIDENCE: The team would need a separate SEO rendering strategy for marketing pages while retaining client loading states, API orchestration, and duplicated authorization enforcement for the app.
CONFIDENCE: Medium-high.
FALSIFIER: Marketing already lives in a separate CMS-hosted site and the authenticated product has no indexing, first-load, or low-JavaScript requirements.

CLAIM: The main tradeoff is operational and architectural complexity, not capability.
EVIDENCE: Server/client boundaries, caching, invalidation, streaming, hydration, and runtime constraints can be harder to reason about than a conventional SPA/API split.
CONFIDENCE: High.
FALSIFIER: A representative feature is materially simpler to build, debug, deploy, and observe under the server-components model.

VERDICT: A — Biggest risk: the team mismanages server/client boundaries and caching, producing stale data, hydration bugs, and framework lock-in. Cheapest discriminating test: build one thin vertical slice—SEO marketing page plus authenticated CRUD dashboard with permissions, optimistic updates, and cache invalidation—and reject A if it requires pervasive client components or cannot reliably show fresh post-mutation data under production-like deployment.
