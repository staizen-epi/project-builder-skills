---
name: web-app-scaffolder
description: Stands up the Phase-0 foundation of a web application from its ARCHITECTURE.md — repo structure, the chosen stack, auth, tenancy primitives, base schema, logging, and the CI gate — and proves it with the architecture's own Phase-0 acceptance criteria. Use this right after an ARCHITECTURE.md has been written and approved, or whenever the user says "scaffold the project", "bootstrap the repo", "set up Phase 0", "stand up the foundation", or "start building" a web app that already has an architecture. This skill builds ONLY the foundation, not features. If there is no specs/ARCHITECTURE.md, do not scaffold — use the web-app-architect skill to create one first. If a specs/POC-NOTES.md exists (from a prior proof-of-concept spike), read it as advisory hints — but the architecture always wins on conflict, and never adopt the throwaway /poc code. If a specs/DESIGN.md exists (from the ux-designer skill), adopt its portable bundle (specs/design/) — its tokens and base components — as the foundation's visual language (a decision, not a hint); and if a specs/DESIGN-BINDING.md exists (from the design-binder skill), adopt it too: wire the chosen stack adapter, apply the project theme override, and use its screen→component map for Phase-0 screens. The architecture still wins on how components are built.
---

# Web App Scaffolder

This skill turns an approved `ARCHITECTURE.md` into a running, verified Phase-0 skeleton. It is the downstream partner of `web-app-architect`: the architect decides *what* and *why*; this skill builds the foundation and proves it works. It deliberately stops at the foundation so features get built on solid ground, never the reverse.

The contract is simple: **the doc is the source of truth, and this skill obeys it.** It does not re-open architectural decisions, and it builds only what the doc's gates turned on. If the doc didn't gate in a queue, you don't install one. If it gated multi-tenancy off, you don't add tenant scoping.

## Precondition

**Read `specs/ARCHITECTURE.md` before doing anything** (all spec artifacts live under `specs/`).

- **Missing →** stop and tell the user the foundation needs an architecture first; point them to `web-app-architect`. Do not invent one here.
- **Present →** read it fully and extract the build set (below). If it is clearly incomplete (no stack, no gates resolved, no Phase-0 acceptance criteria), surface that and ask the user to complete it via the architect skill rather than guessing.

## Step 1 — Derive the build set from the doc

Extract these from `ARCHITECTURE.md` and write them down as the plan you'll execute. You are reading decisions, not making them.

- **Stack** (§3) — frontend, backend, database, jobs, testing, logging, plus any overrides.
- **Gates that are ON** (§2/§6) — tenancy, RBAC, PII/compliance handling, background jobs, scheduled work, cache, webhooks, metered-API caps, real-time, embed/SSO, audit logging.
- **Data model conventions + initial entities** (§5) — ID strategy, scoping column, PII handling, the entities to create as base schema.
- **Phase-0 acceptance criteria** (§8) — the observable conditions that define "foundation done." These are your finish line; do not declare success until they pass under a real run.

If the doc specifies a non-default stack (Next.js, Django, Rails, Go, etc.), scaffold in that ecosystem's idioms but keep the foundation checklist below identical — only the expression changes.

**The foundation doesn't exist yet, but this skill *derives* it from the architecture — it does not interview the user to invent decisions.** The "capture intent at creation" rule still applies, narrowly: where the foundation needs a choice the architecture genuinely leaves open or ambiguous (an unresolved gate, a missing Phase-0 criterion, a stack the doc names but doesn't pin a version/template for), **don't silently guess and bake it into running code — surface the gap and confirm it.** A wrong assumption set in the foundation is expensive to unwind. If the gap is an architectural decision rather than a build detail, route it back to `web-app-architect` to settle in `ARCHITECTURE.md` rather than resolving it here. Everything the doc *does* settle, you obey without re-asking.

**Also read `specs/POC-NOTES.md` if it exists — as advisory hints, not decisions.** A PoC may have been spiked between architecture and scaffolding. Its notes carry forward UX/flow patterns that worked, component shapes worth reusing, observed data shapes, libraries that landed, and pitfalls to avoid — so the foundation starts closer to reality. Two rules: **the architecture wins on every conflict** (the notes are advisory; if a hint contradicts a gate or convention in `ARCHITECTURE.md`, follow the doc), and **read only the notes, never the `/poc` code** — the spike is throwaway and its shortcuts (mock data, fake auth, no validation) must not leak into the real foundation. Ignore `/poc/` entirely when scaffolding.

**Also read `specs/DESIGN.md` (+ its bundle) and `specs/DESIGN-BINDING.md` if they exist — and adopt them as the foundation's visual language *for this project*.** Two artifacts make up the design layer: the **portable bundle** under `specs/design/` (`ux-designer`'s project-agnostic visual language — stack-neutral `tokens.json` + `tokens.css`, optional adapters, a buildless preview, `BUNDLE.md`) and, if present, the **binding** `specs/DESIGN-BINDING.md` (`design-binder`'s connecting artifact — the screen/flow→component map, this project's **theme override** under `specs/design/themes/`, and the **chosen stack adapter**). Unlike `POC-NOTES.md`, both are a **decision, not a hint**. When present, the scaffolder:
- **wires the chosen stack adapter** named by the binding (or, if no binding, links `tokens.css`/`tokens.json` directly or generates an adapter from `tokens.json`) — wiring *to* the bundle, not copying token values inline, so it stays exportable and in sync;
- **applies the project theme override** (`specs/design/themes/<project>.{css,json}`) on top of the bundle tokens, so the foundation renders in *this project's* brand, not the bundle's neutral default;
- **stands up the base component inventory** in their defined states and the layout/nav shell, using the binding's screen→component map to set up Phase-0 screens against the right components.

So every feature is built against one coherent visual language, themed for this project. The architecture still wins on *how* components are built (the stack/framework); DESIGN governs *how they look* and the binding governs *how this project uses them*. If there's a bundle but no binding, adopt the bundle with its neutral defaults. If there's no DESIGN.md at all, scaffold sensible stack-default styling and move on — don't invent a design system here (that's `ux-designer`) or a binding (that's `design-binder`).

## Step 2 — Scaffold in this order

Order matters: each step depends on the ones before it. Build the always-on foundation for every app; add a gated piece only when its gate is ON in the doc.

**Always-on foundation (every app):**
1. **Repo structure.** Initialize the project layout (frontend + backend; monorepo if the stack implies it). Add `.gitignore`, `README`, and an `.env.example` — never commit real secrets.
2. **Typed config + secret loading.** Centralize env access; fail fast on missing required vars. Secrets come from the environment / secret manager, never source or DB.
3. **Boundary validation wired in.** Install and wire the doc's validation library (e.g. Zod) into the request pipeline so every external input is schema-checked.
4. **Structured logging with redaction.** Configure the logger so secrets and PII never serialize to logs.
5. **Security headers + TLS-readiness.** Add the headers middleware (e.g. helmet) and assume TLS at the edge.
6. **Database + base schema.** Wire the DB connection and migrations. Create the initial entities from §5 following the stated conventions (ID strategy, PII hashing/tokenization).
7. **Auth.** Implement the doc's auth model end to end (login/logout/session or token).
8. **CI gate.** Set up lint, typecheck, the test runner, and dependency/secret scanning as a gate that blocks on red. Add a single smoke test now so the gate has something to run.
9. **Environment config.** Provide local/staging/production config paths so nothing is hardcoded to one environment.

**Gated foundation (only if the gate is ON):**
- **Design system →** if `specs/DESIGN.md` exists, adopt its portable bundle (`specs/design/`): wire the stack adapter named by `DESIGN-BINDING.md` (or, with no binding, `tokens.css`/`tokens.json` directly / one generated from `tokens.json`) into the front end, **apply the project theme override** (`specs/design/themes/<project>.*`) if a binding exists, and stand up its **base component inventory** in their defined states plus the layout/nav shell — using the binding's screen→component map for the Phase-0 screens. Wire *to* the bundle as the source of truth (don't copy token values inline) so it stays exportable and in sync; keep the buildless preview (`preview/index.html`) intact so the system stays visible. Conform to its accessibility baseline. The architecture wins on *how* components are built; DESIGN governs *how they look* and the binding *how this project uses them*. No DESIGN.md → use stack-default styling; don't invent a system (route to `ux-designer`) or a binding (route to `design-binder`).
- **Multi-tenant →** add the scoping column to every tenant-owned table, a query-scoping layer that injects `tenantId` automatically, Postgres RLS policies as defense-in-depth, **and** an automated cross-tenant access test that asserts a cross-tenant read *fails*.
- **RBAC →** create the role model and an authorization middleware; enforce it on mutating routes.
- **PII / compliance →** apply at-rest hashing/tokenization for the flagged fields and confirm redaction covers them; stub data export/delete if the regime requires it.
- **Background jobs →** stand up the queue + a worker process, and add one no-op repeatable job to prove the pipe end to end.
- **Scheduled work →** wire the scheduler (often the same runner) with one health-check job.
- **Cache →** add the cache client and a thin helper with an explicit invalidation convention. Skip entirely if the gate is off.
- **Webhooks →** add a signature-verification middleware skeleton and a dedupe store keyed on the provider's event id.
- **Metered APIs →** wrap the metered client in a spend-cap / circuit-breaker that trips before a runaway loop costs money.
- **Real-time →** stand up the WebSocket/SSE server with the doc's auth strategy.
- **Embed/SSO →** configure framing-safe responses and the SSO entry path.
- **Audit logging →** create the audit table and a write helper for sensitive/destructive actions.

> Reverse pressure is intentional: if you find yourself wanting to add something the doc didn't gate in, that's a signal to update the doc via the architect skill first — not to quietly scaffold it. Keep code and doc in step.

## Step 3 — Prove Phase 0

The output is not "files exist." It is **a foundation that demonstrably works.** Before declaring Phase 0 done:

1. **Run the app.** Confirm it boots and the auth path works (a user can log in).
2. **Run the gate.** Lint, typecheck, tests, and scans must pass green.
3. **Run the gated proofs that apply:**
   - Multi-tenant: the cross-tenant access test fails correctly (i.e. access is denied).
   - RBAC: a role check is enforced on a protected route.
   - Jobs: the no-op job runs and completes.
4. **Check against §8.** Every Phase-0 acceptance criterion in the doc passes under a real run. If any fails, fix it and re-run the whole check — do not partially pass.

Report what you built, what you proved, and any deviation from the doc (with the reason).

## Step 4 — Record it

Append a dated entry to the `ARCHITECTURE.md` Changelog noting Phase 0 was scaffolded and verified, and record any deviation or assumption you had to make. Keep the doc and the codebase in step from the very first commit.

## Stop condition

This skill builds and proves the **foundation only**. It does **not** implement Phase 1+ features. When Phase 0 is green, hand back to the user (or the feature-slice skill) to build the first vertical slice. Resist scope creep — a scaffolder that starts writing features is how the foundation ends up half-tested.

## Examples

**Example 1 — lean single-org app**
Doc: single organization, light RBAC, no jobs/cache/webhooks/metered APIs (the OKR-board shape). Scaffold: repo + config + validation + redacted logging + Postgres + base entities + password auth + a two-role authz check + CI gate. No Redis, no queue, no tenant scoping. Prove: app boots, login works, role check enforced, gate green, §8 criteria pass.

**Example 2 — multi-tenant SaaS**
Doc: multi-tenant (shared DB), RBAC, PII handling, background jobs, webhooks, scheduled health checks. Scaffold: the always-on foundation **plus** `tenantId` scoping + RLS + a cross-tenant test, role model + authz, token/PII encryption, a queue + worker with a no-op job, a webhook signature middleware, and a scheduled health-check job. Prove: cross-tenant read is denied, the job runs, the gate is green, §8 criteria pass — then stop, before any feature work.
