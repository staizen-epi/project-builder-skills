---
name: web-app-architect
description: Establishes and maintains the technical architecture for any web application, producing a single ARCHITECTURE.md that is the source of truth for the build. Use this at the very start of ANY new web app, whenever the project has no ARCHITECTURE.md yet, or whenever the user is deciding on a tech stack, multi-tenancy, data model conventions, auth, background jobs, caching, deployment, or "how should this app be structured." If a specs/PRD.md exists, read it first and treat it as the product source of truth, deriving project name, users, auth needs, constraints, and integrations from it instead of re-asking. If an ARCHITECTURE.md already exists, use this skill to read it, follow it, and keep it current. Trigger this even when the user only implies architectural setup (e.g. "let's start building X", "scaffold a SaaS", "set up the backend") rather than saying the word "architecture".
---

# Web App Architect

This skill turns a vague "let's build a web app" into a concrete, reviewed `ARCHITECTURE.md` that the rest of the build follows. It encodes the parts of good web architecture that hold true regardless of what the app *does*, while gating the parts that only apply to some apps so a simple project never gets enterprise scaffolding it doesn't need.

The single output of this skill is **`specs/ARCHITECTURE.md`** (the spec family lives together under `specs/`). It is the durable source of truth. Everything else (code, schema, deploy config) should be derivable from it and consistent with it. When a `specs/PRD.md` exists, this skill reads it first as the product source of truth and shapes the architecture to fit it, rather than re-deriving product facts.

## Two modes

**Check for `specs/ARCHITECTURE.md` before doing anything else.**

- **No `ARCHITECTURE.md` → Bootstrap mode.** Gather what the gates need (below), resolve the decision gates, then generate the first `ARCHITECTURE.md` using the output template. Do not start writing app code until the user has seen and approved the architecture.
- **`ARCHITECTURE.md` exists → Consult & maintain mode.** Read it fully and treat it as binding. Build in line with it. When a change introduces a new architectural decision (a new dependency, a new external integration, a tenancy or auth change), update `ARCHITECTURE.md` in the same change and append to its changelog. Never let the code and the doc silently diverge.

---

## Bootstrap mode

### Step 0 — Read the PRD first (if present)

Look for `specs/PRD.md` before asking the user anything. If it exists, it is the product source of truth — read it fully and extract the facts it already settles, so the Step 1 questionnaire only covers genuine gaps. **Don't re-ask what the PRD answers.**

Map PRD content to architecture inputs:

- **Project name** ← the PRD title. Use it; don't ask.
- **Overview & persona** ← seed §1 Overview, and infer **who logs in**: a single-user or "personal/local" tool implies single-tenant with minimal or no auth; "teams", "organizations", or "clients of clients" implies multi-tenant.
- **Roles in the persona / user stories** ← whether RBAC is needed and what the roles are (e.g. admin vs member). If the PRD describes one user doing everything, don't invent roles.
- **Non-goals** ← hard "off" signals. "Not multi-user", "no accounts", "no cloud" switch the matching gates **off**, not on.
- **Constraints & external systems (PRD §8)** ← platform and deployment (local-only, offline, desktop), cost ceilings (free-tier requirements), and which integrations exist. Paid/metered APIs → the spend-cap gate; inbound integrations → the webhook gate.
- **Non-functional requirements** ← data sensitivity (PII / payment / health → the data gate), the persistence model, and any performance or privacy needs.
- **User stories / roadmap mentioning** reminders, digests, syncs, or scheduled jobs ← the async/scheduled-work gate.
- **PRD Open Questions with architectural impact** ← carry them into the architecture's §9 rather than silently deciding them.

Two rules:
1. **Don't re-ask what the PRD settles.** After reading it, the Step 1 questionnaire should usually shrink to only the high-impact gates the PRD genuinely leaves open — often just confirming an inference (e.g. "the PRD reads as single-user and local; confirming there's no login requirement").
2. **Don't override the PRD silently.** If an architectural need seems to contradict the PRD (e.g. the PRD says single-user but a requested feature implies multi-tenant), surface the conflict and reconcile — update the PRD via the `product-requirements` skill rather than quietly diverging.

If there is **no** `specs/PRD.md`, proceed from the conversation as usual. For a non-trivial product with no PRD, it's worth suggesting the `product-requirements` skill first — but the architect can still work directly from the conversation when the user prefers.

### Step 1 — Gather what the gates need

Your goal is enough information to resolve the decision gates in Step 2 — not to fill out a form. Long forms are friction; most apps only hinge on a handful of decisions. Work in this order:

1. **Extract first.** Pull every answer the PRD (Step 0) and the current conversation already imply, and treat it as provisional.
2. **Ask only the high-impact gates that are still unknown.** Four answers reshape the whole architecture; the rest only tune it. **If any of these four are undetermined, ask before generating — never silently guess them:**
   - **Tenancy** — one user / one organization / multi-tenant (many independent orgs)?
   - **Data sensitivity** — personal data? payment data? regulated data (health, etc.)?
   - **Paid/metered external APIs** — could a runaway loop cost real money?
   - **Async or scheduled work** — anything that shouldn't block a request, or runs on a schedule?
3. **Assume sensible defaults for everything else** and record each assumption in the Decisions section rather than blocking on it. Generating a draft from stated defaults and letting the user correct it beats gating the whole output behind a long questionnaire.
4. **When genuinely unsure, default to the simpler gate (off).** It is far cheaper to add scaffolding later than to rip out multi-tenancy, a queue, or a cache the app never needed. Do not let a default nudge an app toward heavier architecture than its description warrants (e.g. most "booking apps" are single-business, not marketplaces).

Present the high-impact questions as one short block with defaults shown, so the user can reply "defaults" and move on. Explain briefly why you're asking when it isn't obvious.

**High-impact — ask if unknown:** tenancy · data sensitivity · metered APIs · async/scheduled work.

**Lower-impact — assume unless clearly relevant:** project name & app type; the specific roles inside RBAC; real-time needs; caching; inbound webhooks; embed/SSO; stack overrides; scale & ops maturity. Default each to the simplest viable choice, the recommended stack below, and a light-but-safe ops setup (one staging step + a CI gate).

### Step 2 — Resolve the decision gates

Map answers to what goes in the architecture. **Invariants are always included. Gated sections are included only if their condition is met.** This is the whole point — don't bolt multi-tenancy or a job queue onto an app that doesn't need it.

| If the answer is… | Include in ARCHITECTURE.md |
|---|---|
| Multi-tenant | Tenant-scoping rule on every query + (for shared DB) row-level security + cross-tenant isolation tests |
| Multiple roles | RBAC model and an authorization layer |
| Stores PII / regulated data | Data-handling section: hashing/tokenization at rest, redaction in logs, retention rules, the relevant compliance notes |
| Background jobs | Job queue + worker + retry-with-backoff policy |
| Scheduled work | Scheduled-jobs ("watchdog") layer + health checks + escalation-on-repeated-failure |
| Metered external APIs | Spend caps / circuit breakers + cost alerting |
| Inbound webhooks | Webhook signature verification + payload validation |
| Read-heavy / repetitive reads | Cache layer (e.g. Redis) with an explicit invalidation strategy |
| Embedded / SSO | Embed-aware frontend (framing-safe, token/SSO entry) |
| Real-time needs | WebSocket/SSE layer + connection/auth strategy |
| Multiple users acting on shared data, or destructive/sensitive actions | Audit log of sensitive actions (who did what, when). Otherwise lightweight `updatedBy`/`updatedAt` stamps suffice — don't build an audit table for a single-user app |

**Gates aren't all-or-nothing.** When a gate is warranted but light, include it in minimal form and note in Decisions that it can grow: RBAC as just two roles (admin/member), caching named as a deferred optimization rather than built up front, a job runner with a single worker. Pick the minimal form when a gate is borderline. But don't invent a "light" version of a gate the app doesn't need at all — that is simply the gate being **off**, and it should be omitted, not shrunk.

### Step 3 — Generate ARCHITECTURE.md

Use the template in "Output: ARCHITECTURE.md". Fill the always-on sections, add only the gated sections that apply, and record the resolved decisions and any assumptions in the Decisions section so future readers know *why* the architecture looks the way it does. Present it to the user and revise before any app code is written.

---

## The always-on invariants

These belong in every web app's architecture regardless of answers, because skipping them is how projects acquire security holes, surprise bills, and unrecoverable data loss. State them in `ARCHITECTURE.md` even when they feel obvious.

- **Validate at every boundary.** All external input (HTTP bodies, params, webhook payloads) is schema-validated before use. Reject malformed input early.
- **Secrets live in the environment / a secret manager** — never in source, never in the database, never in logs.
- **Structured logging with redaction.** Logs are structured, and secrets and personal data are redacted so they never appear in plaintext. A log leak should never be a data breach.
- **TLS everywhere**, with sane security headers.
- **Backups with a tested restore** for any persistent datastore. An untested backup is not a backup.
- **A CI gate before merge/deploy:** lint, typecheck, tests, dependency/secret scanning. Treat a red gate as blocking.
- **At least a staging step before production.** Nothing ships straight to prod unreviewed.

## Default stack (recommend, don't impose)

Offer this as the default and let the user override any layer. It's a boring, typed, well-supported spine that fits most data-driven web apps.

- **Frontend:** React + Vite + TypeScript, Tailwind, a component library (e.g. shadcn/ui), TanStack Query for server state, React Router.
- **Backend:** Node + TypeScript, Fastify (or Express), Zod for validation, Prisma as ORM.
- **Data:** PostgreSQL primary. Add Redis only if caching or queues are gated in. Add `pgvector` only if semantic search / embeddings are needed.
- **Jobs (if gated in):** a Redis-backed queue such as BullMQ for workers, retries, and repeatable scheduled jobs.
- **Testing:** Vitest (unit/integration) + Playwright (E2E).
- **Logging:** pino with redaction configured.

If the user picks a different stack (Next.js, Django, Rails, Go, etc.), keep the *invariants and gated patterns the same* and just express them in that ecosystem's idioms.

## Build methodology (carry into ARCHITECTURE.md)

The architecture should prescribe *how* the build proceeds, not just its shape:

- **Phase 0 before features.** Scaffold the project, auth, tenancy primitives, and the CI gate *first*. Features build on a foundation, not the reverse.
- **Vertical slices, MVP-first.** Ship one end-to-end path working before broadening. Order phases so each delivers something demonstrable.
- **Acceptance criteria per phase.** Each phase states what "done" means in observable terms.
- **A QA gate before production.** If any check fails, fix it and re-run the full gate before deploying.

---

## Output: ARCHITECTURE.md

ALWAYS produce the file in this structure. Include every always-on section; include a gated section only when its condition was met (delete the others rather than leaving them empty).

```markdown
# [Project Name] — Architecture

> Source of truth for how this app is built. Keep current; see Changelog.

## 1. Overview
One paragraph: what the app is and who uses it (drawn from the PRD if present).

## 2. Decisions
The resolved answers that shaped this architecture (tenancy, roles, data
sensitivity, workloads, external deps, stack). Record the *why*, briefly.

## 3. Tech stack
Frontend, backend, database, jobs, testing, logging — with any overrides noted.

## 4. System architecture
The layers that apply: edge/ingress → application → data, plus any gated
layers (jobs, cache, real-time, integrations). A small diagram is welcome.

## 5. Data model conventions
Naming, ID strategy, how PII is stored, audit logging, and (if multi-tenant)
the scoping column present on every tenant-owned table. Then the initial entities.

## 6. Security & resilience
The always-on invariants, plus every gated control that applies
(tenant isolation, RBAC, webhook verification, spend caps, redaction, etc.).
Each item states the requirement, not just the intent.

## 7. Environments & release
Local → staging → production, the CI/QA gate, and the deploy flow.

## 8. Build phases
Phase 0 (foundation) first, then vertical slices. Each phase lists its
acceptance criteria.

## 9. Open questions & assumptions
What still needs the owner's input (credentials, compliance specifics,
hosting target) and what was assumed in the meantime. Include any
architecturally-relevant open questions carried over from the PRD.

## 10. Changelog
Dated entries for every architectural decision change.
```

## Maintenance mode rules

- Read `ARCHITECTURE.md` before building; build to match it.
- When a change alters an architectural decision, update the relevant section **and** add a Changelog entry in the same change.
- If the code already drifts from the doc, surface it to the user and reconcile — don't quietly pick one.
- Revisit the decision gates when the app's nature changes (e.g. it becomes multi-tenant, or starts calling a paid API); add the now-applicable gated sections.
- Keep the architecture consistent with `specs/PRD.md`. If the PRD's scope changes materially (new users, new integrations, new data), revisit the affected gates and update the architecture to match.

## Examples

**Example 1 — minimal app, gates mostly off**
Input: "Build me a personal recipe tracker, just for me."
Resolution: no auth or single-user, no PII regime, no jobs, no metered APIs, no webhooks. The generated ARCHITECTURE.md keeps the invariants (validation, backups, CI gate, logging) and the default stack, but omits tenancy, RBAC, queues, cache, spend caps, and webhook verification. Phase 0 is light.

**Example 2 — multi-tenant SaaS, many gates on**
Input: "A SaaS where agencies manage clients and we send automated messages via a paid API."
Resolution: multi-tenant (shared DB + row scoping + isolation tests), RBAC, PII handling, background jobs + retries, scheduled health checks, spend caps on the messaging API, webhook verification for delivery callbacks. The generated ARCHITECTURE.md includes all of those gated sections and a Phase 0 that stands up tenancy and the CI gate before any feature.
