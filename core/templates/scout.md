# Scout - where would panels pay off in THIS project?

<!-- Adopter onboarding: paste this prompt into your own agentic CLI from
     your project's root, right after cloning Tribunal. The model
     reads YOUR project and returns project-specific panel use cases -
     so your first panel is a real one, not a toy. -->

You are helping me adopt Tribunal, an adversarial multi-model review
methodology (read its `core/METHODOLOGY.md`, especially "When to convene a
panel" and the stop rules, from: [path to your tribunal clone]).

You are READ-ONLY. Survey this project - code, docs, issues/TODOs, recent
history if available - and report:

1. **Panel-worthy decisions (max 5, ranked).** Upcoming or recurring
   decisions in THIS project that fit the convene criteria: irreversible or
   costly to roll back, ambiguous with incompatible plausible readings,
   hard-to-observe wrongness, or prone to one-model agreeable blind spots.
   For each: the decision, why it fits (cite the file/issue that evidences
   it), what the frozen brief's question would be, and what the panel's
   decision criteria should be.

2. **Not panel-worthy (max 3).** Things I might be tempted to panel that an
   oracle settles cheaply here - the test suite, the compiler, a schema, a
   primary document. Name the oracle. (Tribunal's first rule: run the
   oracle before convening a debate.)

3. **First-panel pick.** Which single item from list 1 to run first, chosen
   for: real stakes, artifacts small enough to review in one sitting, and a
   verdict I could act on this week. Draft its frozen brief using
   `core/templates/brief.md`.

4. **Lens recommendation.** Apply METHODOLOGY "Assigning lenses". A first
   panel is almost always 2-3 seats, so **default to raw heterogeneous
   seats on the same brief** and say so. Then name **at most three candidate
   review surfaces** for this artifact (e.g. correctness / security / DX) -
   as shared *decision criteria the whole panel addresses* (additive
   lenses), NOT as one-per-seat hats. Only recommend an *exclusive* slice
   (a seat prioritizing one surface) if the operator is already staffing
   N ≥ 4 with two heterogeneous seats per lens; otherwise fold the surfaces
   into the shared brief. Do not invent lenses to fill seats, and never put
   a single seat on an exclusive lens. Lenses are an incentive to look
   somewhere; they never substitute for model heterogeneity.

Ground every claim: cite the files that evidence each decision. If the
project gives you no panel-worthy decisions, say so plainly - do not invent
stakes; the methodology explicitly warns against convening panels for
ritual's sake.
