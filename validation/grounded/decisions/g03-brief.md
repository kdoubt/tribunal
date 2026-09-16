# Decision G03 - review a draft onboarding spec before it is committed for implementation

## Artifact(s)

Paths are relative to `./artifact/` (WhipNode monorepo checkout). Read anything
in the tree; nothing outside it is in bounds.

- `ONBOARDING_SPEC.md` (also at `docs/ONBOARDING_SPEC.md`) - the DRAFT
  implementation spec under review: a three-rung onboarding ladder (anonymous
  zero-click trial -> in-CLI claim -> `whipnode init` deliberate path) that
  consolidates two earlier design decisions. It is about to be committed for
  an implementing agent.
- The code the spec must be buildable against: `apps/worker/src/routes/`
  (`mcp.ts`, `capture.ts`, `auth-magic-link.ts`, `auth-verify.ts`,
  `api-keys.ts`, `mcp-devices.ts`), `apps/worker/src/middleware/`
  (`auth.ts`, `rate-limit.ts`), `apps/worker/src/scheduled/cleanup.ts`,
  `apps/worker/migrations/`, `packages/protocol/src/{constants,types}.ts`.

USER-FACT: the two prior decisions the spec encodes were (1) `whipnode init`
= device-code pairing wrapping the magic link, one Authorize click, one free
key + `mcp_devices` enrollment; (2) the zero-click trial ladder = anonymous
no-PII trial, wall-triggered claim via a non-bearer locator with possession
proof, in-place same-key conversion, rung 0 complements init. Three
parameters were left as owner calls: raw key vs device token to the CLI;
Turnstile day-one vs adaptive; trial quota 10/day vs 3/day.

## Question under review

Review the DRAFT SPEC for:
- **Q1 Fidelity.** Does it faithfully encode the two decisions above? Flag
  anything the spec adds that neither decision made, anything decided that the
  spec dropped, and any owner-call it silently resolves.
- **Q2 Buildability.** Is every referenced file, line, and mechanism real in
  the tree? Any step an implementer cannot execute as written? Any missing
  migration, endpoint, or edge case (for example: what happens on claim when
  the trial key was revoked mid-flow; init and claim sharing one pairing
  table - do their state machines compose)? Does the schema change the spec
  requires have a complete migration plan on this database engine?
- **Q3 Security holes the spec's own protocol introduces.** Walk the claim
  flow as an attacker: leaked claim locator plus stolen 0600 file; TTY-check
  bypass; race between poll and authorize; global-counter reset-time gaming.
- **Q4 Implementability traps.** Ambiguities an implementing agent would
  guess wrong.

## Decision criteria (owner-supplied)

Fidelity to the two decisions; buildable against the tree as it is; no new
bearer surface; every migration and cleanup task named explicitly.

## Constraints

- Cloudflare Worker + D1 (SQLite semantics). No new services.
- One implementing agent, no human review between spec and build.

## Output contract

Maximum 8 claims, each as:

```
CLAIM: <one sentence>
EVIDENCE: <file:line or verbatim span in ./artifact/, or USER-FACT, or ASSUMPTION, or SPECULATIVE (name it), or EXTERNAL with source>
CONFIDENCE: <0-1 probability, calibrated>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Then **VERDICT INPUT**: one line per question, and a PUBLISH VERDICT of
`blocker` | `ship-after-fixes` | `ship-as-is` with one sentence. Maximum
1200 words.
