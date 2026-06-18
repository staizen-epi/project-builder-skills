---
name: qa
description: Runs the QA gauntlet against a built web app — black-box test suites plus the non-functional checks — and owns the pass/fail gate before release. It tests the app as a BLACK BOX through its own public surface (UI + API), never reaching into source or the database. Use this after features are built and the foundation runs, whenever the user wants to "QA the app", "run the test gauntlet", "verify quality", "check the NFRs", "run the quality gate", "is this release-ready", "test the whole app end to end", "scan the API for vulnerabilities", "set up smoke tests", or "do a quality pass before we ship". It runs four black-box passes — e2e feature/functionality tests (always on), smoke-set flagging (always on), a bounded non-destructive API vulnerability scan (gated on the app exposing an API or handling sensitive data), and performance testing (gated on a perf NFR existing in specs/QUALITY.md) — and turns each specs/QUALITY.md NFR-0X "Verified by: qa" handoff into a runnable check. It requires the architecture's CI gate (lint/typecheck/unit) to be green first, then runs its passes and restarts the full gauntlet on any failure until green. It derives selectors from the testID scheme in specs/ARCHITECTURE.md (emitted by feature-developer) and stands the app up against the architecture's test-environment contract (seeded, resettable) for idempotent runs. It writes specs/QA-PLAN.md (the durable test contract — selector map, suite scope, smoke set, NFR→check map) and specs/QA-REPORT.md (the latest run's verdict). It owns running tests and the gate verdict only — not the targets (quality-requirements owns QUALITY.md), not feature behaviour (feature-spec), not the technical how (web-app-architect), not building the feature (feature-developer). A failing check that reveals a missing target routes to quality-requirements; a missing testID scheme or test-env contract routes to web-app-architect; a behaviour bug routes to feature-developer. Requires specs/ARCHITECTURE.md and a running, built app.
---

# QA

This skill is the **QA gauntlet**: it exercises a built web app as a **black box** and owns the **pass/fail gate** before release. It does for *verification* what `quality-requirements` does for *targets* and `feature-spec` does for *behaviour* — it proves the app actually works and meets its non-functional targets, observably, then enforces a gate.

Two principles define it:

1. **Black box, always.** Every test goes through the app's own public surface — the rendered UI and the published API. This skill never imports app source, reads internal modules, or touches the database directly. It tests what a user or an API client can observe. (It *reads* `specs/` docs and the test code it owns; it does not test through internal seams.)
2. **Test-centered, via a shared seam.** The app is built test-first: `feature-developer` emits stable `data-testid` attributes per the **testID scheme in `specs/ARCHITECTURE.md`**, derived from the feature specs' `FR-0X.Y` IDs. This skill **derives its selectors from that same scheme**, so e2e selectors are logical, stable, and trace straight back to the PRD + feature specs. The seam is load-bearing: qa does not invent selectors by guessing the DOM; it consumes the contract the developer emitted.

It runs **four black-box passes** (gated) and turns each `specs/QUALITY.md` `NFR-0X` target into a **runnable check**, then emits a single verdict and **restarts the full gauntlet on any failure until green**.

The two outputs:

- **`specs/QA-PLAN.md` — the durable test contract.** The selector map (testID scheme → the selectors used), the four suites' scope, the chosen smoke set, and the `NFR-0X → check` mapping. Read first, maintained after — the QA analogue of `REUSE.md`. It churns slowly.
- **`specs/QA-REPORT.md` — the latest run's verdict.** Pass/fail per check, the restart history, what's blocking the gate. It churns every run.

The test code itself lives **in the codebase**, per the architecture's testing stack (§3) — not in `specs/`.

## Preconditions

1. **`specs/ARCHITECTURE.md` must exist** and define two things this skill consumes:
   - the **testID scheme** (the `data-testid` convention), and
   - the **test-environment contract** (a runnable, *seeded*, *resettable* target — base URL + seed data + a reset hook).

   **Either missing →** route to `web-app-architect` to add it (see the seam below). Don't invent a scheme or scrape the DOM blind, and don't seed test data ad hoc — that makes runs non-idempotent. qa consumes the contract's seed and creates only *transient, per-test* state through the public surface; a *foundational* seed gap routes back to `web-app-architect` to widen the seed (see Step 3a).
2. **The app must be built and runnable.** There must be features to test and a way to stand the app up (the test-env contract). Nothing built yet → there's nothing to QA; point at the build loop (`feature-spec` → `feature-developer`).
3. **`specs/QUALITY.md` may or may not exist.** Present → each `NFR-0X` with a `Verified by: qa` handoff becomes a runnable check, and a perf NFR turns the perf pass on. Absent → run the always-on black-box passes only (e2e + smoke), and note that no NFR targets were available to verify.
4. **`specs/PRD.md` + `specs/features/` are the source of truth for *what* to test.** The e2e pass covers the PRD's flows and each feature spec's acceptance criteria — qa verifies stated behaviour; it does not invent new behaviour to test (a gap routes to `feature-spec`).

> Ignore `/poc/` entirely. It is throwaway mock-data code; never test it or treat its shortcuts as real behaviour.

## Two modes

**Check whether `specs/QA-PLAN.md` exists.**

- **No plan → Bootstrap (set up the gauntlet).** Read the inputs (Step 0), resolve which passes are gated on (Step 1), build the selector map from the testID scheme (Step 2), author the four suites + the NFR checks (Step 3), run the gauntlet to green (Step 4), write both docs (Step 5).
- **Plan exists → Maintain (re-run / extend).** Read `QA-PLAN.md`, re-derive selectors for any new/changed features, add checks for new acceptance criteria and new `NFR-0X` rows, run the full gauntlet, and refresh `QA-REPORT.md`. Keep the plan truthful — a stale selector map is worse than none.

---

## Bootstrap mode

### Step 0 — Read the inputs before writing a test

You are verifying decisions, not making them — so this skill **derives** the test contract from the architecture + specs + NFRs rather than interviewing the user to invent the criteria you test against. The "capture intent before creating" rule still binds, pointed upstream: where what "passing" means is genuinely undefined (a feature with no acceptance criteria, an `NFR-0X` with no measurable target, a `Verified by:` that names qa but states nothing checkable, a missing testID scheme), **don't invent the bar and quietly test to it** — that fabricates the very intent QA exists to verify. Route a missing/unmeasurable target back to `quality-requirements`, a missing acceptance criterion to `feature-spec`, and a missing testID scheme / test-env contract to `web-app-architect`, then build the check from the settled answer. The only thing you author yourself is *how* to check a stated criterion, never *what* the criterion is. Pull:

- **From `specs/ARCHITECTURE.md`:** the **testID scheme** (§Testability — the `data-testid` convention to derive selectors from), the **test-environment contract** (§Testability / §7 — base URL, seed data, reset hook), the **testing stack** (§3 — e.g. Playwright + Vitest, so suites use the project's tools), the **API surface & protocol** (§3/§4 — REST/GraphQL/gRPC, routes), and the **gated controls that imply security tests** (§6 — tenant isolation, RBAC, webhook verification → these become API-vuln checks).
- **From `specs/QUALITY.md` (if present):** every `NFR-0X` whose `Verified by:` names qa — each becomes a runnable check. Note especially the **security/tenant-isolation** NFRs (drive the API-vuln pass), the **performance** NFR (turns the perf pass on + sets its budget), the **accessibility** NFR (WCAG level → an axe + manual check folded into e2e), and **availability/observability** NFRs (checkable where the running app exposes them).
- **From `specs/PRD.md` + `specs/features/<feature>.md`:** the user flows and each spec's §7 **acceptance criteria** — the e2e pass proves these. These are the finish line for functionality.

### Step 1 — Resolve which passes are gated on

Gate the four passes like the rest of the suite — include only what the app needs. **e2e and smoke-flagging are the always-on core; API-vuln and perf gate on a trigger.**

| Pass | On when… | Off when… |
|---|---|---|
| **e2e (feature/functionality)** | **Always.** Every app has behaviour to prove. | Never off (if there are no features, there's nothing to QA — precondition). |
| **Smoke-set flagging** | **Always.** Even a tiny app has a critical path worth a fast check. | Never off. |
| **API vulnerability scan** | The app **exposes an API** (any REST/GraphQL/gRPC surface) **or** handles **sensitive data / multi-tenant data** (a security or isolation NFR exists, or PII handling is gated on in the architecture). | A pure static site with no API and no sensitive data. Record it OFF with the reason. |
| **Performance testing** | A **performance/latency/scale NFR exists in `specs/QUALITY.md`** (a budget to test against). | No perf NFR — there's no target to prove. Note it as a nice-to-have deferred until a perf NFR exists; don't invent a budget. |

Record each pass's on/off decision (and why) in `QA-PLAN.md`, the same way the architect records gate decisions. A pass that's off is omitted, not faked.

### Step 2 — Build the selector map from the testID scheme (the seam)

This is what makes the e2e suite robust and traceable instead of brittle DOM-scraping.

1. **Read the testID scheme** from `ARCHITECTURE.md` (§Testability). It defines the `data-testid` shape — function-named so each element is grabbable by *what it does*, e.g. `data-testid="<feature>-<function>[-<id>]"` → `invoices-download-button`, `invoices-quick-search-input`, `invoices-invoice-row-{id}`.
2. **Map each control/element the e2e suite needs to its function-named testID**, derived from the scheme — not from inspecting the rendered DOM. The selector for "the download button" is whatever the scheme produces for that *function* (`invoices-download-button`), because `feature-developer` tagged it exactly that. This is the payoff of the seam: you grab elements by name, not by parsing the page or writing deep XPaths.
3. **Verify against the running app.** Confirm the expected testID actually appears in the rendered output. If the app is missing a testID the scheme requires (the developer didn't emit it, or emitted a non-conforming one), that's a **seam break** → route it to `feature-developer` to emit the conforming testID (don't paper over it with a fragile CSS/text selector). If the *scheme itself* is missing or ambiguous, route to `web-app-architect`.
4. **Record the selector map in `QA-PLAN.md`** (element → testID → the `FR-0X.Y` it traces to), so selectors are reviewable and the trace to requirements is explicit.

> **Why testID, not CSS/text/XPath.** Text and layout change with copy edits and restyles; testIDs are a stable contract the developer commits to. Deriving selectors from the scheme (not the DOM) keeps the suite from breaking on cosmetic changes and keeps every selector traceable to a requirement. Fall back to a non-testID selector only for third-party UI you can't annotate, and record that exception in the plan.

### Step 3 — Author the gated passes + the NFR checks

Build the suites with the architecture's testing stack (§3), all against the running test-env target. Each is black-box.

**e2e — feature & functionality (always on).** Drive the app through the UI/API as a user would, asserting on testID-derived selectors. Cover, per feature spec: the **primary flow**, every **state** the spec lists (empty/loading/error/success/boundary), and each **§7 acceptance criterion** as an assertion. This is the functional finish line — a criterion that can't be proven through the public surface is a behaviour/spec gap → route to `feature-spec`. Fold the **accessibility** NFR in here (e.g. an axe pass on key screens + the manual checks the WCAG level requires) rather than as a separate pass.

**Smoke-set flagging (always on).** From the e2e cases, pick the **critical-path** set — the handful that, if green, gives high confidence the app is fundamentally working (e.g. login → dashboard, create → it appears in the list). **Tag them in the suite** (e.g. `@smoke`/test tag per the stack) **and** list the chosen set in `QA-PLAN.md` with the one-line reason each was chosen. The tag lives with the tests (so CI/`release-manager` can run just the smoke set); the plan explains the selection. Keep it small — a smoke set that runs everything isn't a smoke set.

**API vulnerability scan (gated).** **Bounded, non-destructive, against the project's own running API only.** No DoS, no destructive payloads, no targeting anything but this app's surface — consistent with the suite's authorization posture (this is authorized testing of the app under build). Cover, mapping each finding to a `specs/QUALITY.md` security NFR where one exists:
- **Broken authorization / IDOR / cross-tenant access** — can user/tenant A reach B's records by changing an id? (Directly proves a tenant-isolation NFR — the highest-value SaaS check. Needs ≥2 seeded tenants from the test-env contract.)
- **Authentication gaps** — unauthenticated access to protected routes; weak/missing session handling; missing rate-limit on auth endpoints.
- **Input validation / injection** — malformed and hostile inputs (SQLi/NoSQLi/command-injection probes) are rejected at the boundary, per the architecture's "validate at every boundary" invariant.
- **Security headers & transport** — expected headers present; TLS assumptions hold.
- **Secret / sensitive-data leakage** — no secrets, internal errors/stack traces, or PII in API responses (ties to the redaction invariant).
- **Rate-limiting presence** — abuse-prone endpoints are throttled (presence check, not a load attack).

Use the architecture's protocol (§3) to shape requests. Map each result to its NFR id in the report; a finding with no corresponding NFR but a real risk is surfaced and routed to `quality-requirements` (the *target* may be missing) rather than silently dropped.

**Performance testing (gated on a perf NFR).** Load/latency-test the path(s) the perf `NFR-0X` names, against its budget (e.g. "p95 API latency < 200 ms at 100 concurrent users"). Report measured vs. target. No perf NFR → this pass is off; don't invent a budget (the *target* is `quality-requirements`' to set).

**NFR → check mapping (the QUALITY.md handoff).** For every `NFR-0X` whose `Verified by:` names qa, build the runnable check it implies and record the `NFR-0X → check` row in `QA-PLAN.md`:

| NFR kind | qa check |
|---|---|
| Performance / latency budget | the perf pass above |
| Tenant isolation / authz | the cross-tenant/IDOR checks in the API-vuln pass |
| Accessibility (WCAG level) | axe + manual checks folded into e2e |
| Security baseline (headers, validation, secrets) | the API-vuln pass |
| Observability (logs/metrics/alerts) | assert the running app emits the required signals through its public/ops surface (where observable black-box) |
| Availability/SLA, DR (RPO/RTO) | note the method (uptime monitor / DR drill); these are often environment-level — record what's runnable here vs. what's an ops procedure |

A target that *can't* be verified black-box (e.g. a 99.9% monthly SLA needs a monitor over time, not a one-shot test) is recorded honestly in the report as "verified by ops monitoring, not in-gauntlet" rather than faked green.

### Step 3a — Get the test data the checks need (consume the seed; create through the surface; route the rest)

Tests need data in specific shapes — a record in a given state, two tenants for an IDOR probe, a paginated list, an expired item. Test data is **not qa's to own**: the architecture's **test-environment contract** (§Testability) owns the seed + reset hook, and qa consumes it. The question for every check is *where the data it needs comes from* — answer it with this order, not by reaching past the public surface:

1. **Use the seed.** If the test-env contract already seeds the shape a check needs, use it. This is the default and the cheapest path.
2. **Create it through the public surface.** If a check needs *transient, test-local* state — a record this one test creates, exercises, and the reset hook wipes — create it the black-box way: drive the app's own UI/API to set it up (same surface the test asserts on). This stays black-box (no DB writes, no source imports, no back-doors) and keeps the datum local to the test that needs it. Prefer this for one-off, per-test state.
3. **Route a *foundational* data need to `web-app-architect`.** When a check **foundationally** needs a wider seeded baseline than the contract provides — and surface-level creation is the wrong tool because the data is **structural, shared, or impractical to build per-test** — do **not** paper over it by scripting heavy setup through the UI on every run. That makes runs slow, brittle, and re-creates seed state ad hoc (the very non-idempotency the contract exists to prevent). Instead **route the gap to `web-app-architect` to widen the test-env contract's seed**, and record the demand in `QA-PLAN.md` (§ Test-data needs) so it's reviewable. This is a seam break like a missing testID — the contract is the owner, qa names the need.

   **Foundational (route up) vs. transient (create through surface):**
   - Route up when the data is a **precondition of the environment**, not of one test: ≥2 tenants/orgs for cross-tenant/IDOR checks, distinct **roles/permission fixtures** for RBAC checks, a **volume baseline** a perf NFR loads against (you can't build 10k rows through the UI each run), reference/lookup data multiple suites depend on, or any shape several checks share.
   - Create through the surface when the data is **one test's own scenario**: the single invoice this test downloads, the draft this test publishes, the comment this test edits — built, asserted, and reset within the test.
   - The smell that means *route up*: you're driving the UI/API mainly to manufacture starting state (not to exercise behaviour), the setup is duplicated across tests, or it's too slow/large to rebuild every run.

   qa **names the seed gap and routes it**; it does **not** edit the seed or the contract itself (that's `web-app-architect`), and it does **not** quietly script around a missing baseline. Until the contract is widened, record the affected check as blocked-on-seed in the report rather than faking the data.

### Step 4 — Run the gauntlet to green (the gate)

The verdict is observable, not "tests exist." Run in this order and **restart the full gauntlet on any failure until it passes clean** (the "QA gauntlet" discipline):

1. **CI gate first.** The architecture's own CI gate — lint, typecheck, unit/integration tests, dependency + secret scan (§6 invariant) — must be **green before** the black-box passes run. A red CI gate blocks the gauntlet; fix (or route the bug to `feature-developer`) and restart. This skill **complements** the CI gate; it does not replace it.
2. **Stand up the test-env target** per the architecture's contract: start the app, apply the seed, so data is deterministic. The **reset hook** runs between runs so the gauntlet is **idempotent** — the same starting state every time, which is what makes "restart until green" sound.
3. **Run the gated passes** (e2e + smoke + any API-vuln + any perf) and the NFR checks.
4. **On any failure:** identify the owner and route it (behaviour bug → `feature-developer`; missing/wrong selector or testID → `feature-developer` for the testID, `web-app-architect` for the scheme; infeasible/missing target → `quality-requirements`; missing test-env/contract **or a foundational seed gap** → `web-app-architect`). After the fix, **restart from step 1** — partial passes don't count. The gate is green only when a full clean run succeeds.
5. **Record the verdict** (and the restart history) in `QA-REPORT.md`.

> qa **owns the verdict** but **routes the fix.** It does not patch feature code to make a test pass (that's `feature-developer`), does not change a target to make an NFR pass (that's `quality-requirements`), and does not add the testID scheme/test-env itself (that's `web-app-architect`). It tests, reports, enforces, and routes.

### Step 5 — Write the two docs

Create `specs/` if needed.

1. **Write/update `specs/QA-PLAN.md`** (durable contract) from the template: the selector map, each pass's scope + on/off decision, the smoke set, and the `NFR-0X → check` mapping.
2. **Write/update `specs/QA-REPORT.md`** (latest verdict): pass/fail per check, restart history, what's blocking the gate (if anything), and anything routed back.
3. **Report** to the user: the gate verdict (green/blocked), which passes ran vs. gated off (and why), the smoke set chosen, NFR coverage (verified / can't-verify-black-box / no-target), and every gap routed back and to whom.

---

## Output: specs/QA-PLAN.md

The durable test contract — read first, maintained after. Keep it scannable; it's the QA analogue of `REUSE.md`.

```markdown
# [Product Name] — QA Plan (Test Contract)

> The durable contract for how this app is verified: the testID-derived selector
> map, the gated test passes, the smoke set, and how each NFR is checked. Test
> code lives in the codebase; this is the map to it. Maintained by qa — read
> first, update after. The latest run's verdict is in QA-REPORT.md.

**Last updated:** [date] · **Testing stack:** [from ARCHITECTURE.md §3]

## 1. Passes & gating
Which of the four passes are ON and why (mirror the gate decisions).
| Pass | On? | Reason |
|---|---|---|
| e2e (feature/functionality) | on | always |
| Smoke flagging | on | always |
| API vulnerability scan | on/off | [API exposed / sensitive data — or why off] |
| Performance | on/off | [perf NFR present — or no target yet] |

## 2. Selector map (testID scheme → selectors)
Derived from ARCHITECTURE.md §Testability. Function-named element → testID →
traced requirement. (Grabbed by function, not by DOM position.)
| Element (by function) | data-testid | Traces to |
|---|---|---|
| Download button (per invoice) | `invoices-download-button-{id}` | FR-03.1 |
| Quick-search input | `invoices-quick-search-input` | FR-03.2 |
| Invoice row | `invoices-invoice-row-{id}` | FR-03.2 |
(Note any non-testID fallback selector + why — e.g. third-party widget.)

## 3. e2e coverage
Per feature: the flows, states, and §7 acceptance criteria the suite asserts.
Point at the test file(s). One row per feature/spec.

## 4. Smoke set
The critical-path cases tagged `@smoke`, each with the one-line reason it's in
the set. (Consumed by CI / release-manager for a fast confidence check.)

## 5. API-vuln scope (if on)
The bounded, non-destructive checks run against the app's own API, each mapped
to its security NFR id where one exists.

## 6. NFR → check mapping
| NFR | Target (from QUALITY.md) | qa check | Verifiable black-box? |
|---|---|---|---|
| NFR-01 | p95 API < 200 ms @ 100 users | perf pass | yes |
| NFR-03 | no cross-tenant access | IDOR/cross-tenant in API-vuln | yes |
| NFR-02 | 99.9% uptime monthly | ops uptime monitor | no — ops procedure |

## 7. Test-data needs
How each check gets its data: from the seed, created through the surface, or a
foundational seed gap routed to web-app-architect. Source of truth for the seed
is the architecture's test-env contract; this records what qa consumes and what
it has asked the architect to widen.
| Need | Source | Status |
|---|---|---|
| ≥2 tenants (cross-tenant IDOR) | seed (test-env contract) | satisfied |
| Distinct admin/member roles (RBAC checks) | **seed gap → web-app-architect** | requested [date] |
| 10k invoices (perf load baseline) | **seed gap → web-app-architect** | requested [date] |
| Single invoice under test | created through UI/API, reset after | per-test |

## 8. Changelog
Dated entries: passes added/removed, selectors updated, checks added, seed needs requested.
```

## Output: specs/QA-REPORT.md

The latest run's verdict — churns every run; overwrite/append per run.

```markdown
# [Product Name] — QA Report (Latest Run)

> The verdict from the most recent QA gauntlet. The durable contract is in
> QA-PLAN.md. Maintained by qa.

**Run date:** [date] · **Verdict:** [GREEN | BLOCKED] · **Restarts:** [n]

## Gate summary
CI gate: [green/red]. Black-box passes run: [list]. Overall: [verdict].

## Results by check
| Check | Pass | Result / measurement | NFR | Routed to (if failed) |
|---|---|---|---|---|
| Login → dashboard (smoke) | ✅ | — | — | — |
| Cross-tenant access blocked | ✅ | A cannot read B | NFR-03 | — |
| p95 API latency | ❌ | 310 ms vs < 200 ms target | NFR-01 | feature-developer (slow query) |

## Blocking the gate
What must be fixed before this goes green, and who owns each.

## Not verifiable in-gauntlet
NFR targets that need ops/time-based verification (SLA monitor, DR drill),
recorded honestly rather than marked green.

## Restart history
Brief: run 1 failed on X → fixed by Y → run 2 green.
```

## Scope boundary (up / down / sideways — what this skill does NOT own)

- **Up (targets):** it does not set non-functional targets — that's `quality-requirements` (`QUALITY.md`). qa turns a target into a check and verifies it; a missing/infeasible target routes back there.
- **Up (behaviour):** it does not define what the app should do — that's the PRD / `feature-spec`. qa proves stated acceptance criteria; an untestable or missing behaviour routes to `feature-spec`.
- **Up (technical design):** it does not decide the stack, the testID scheme, or the test-env contract — that's `web-app-architect`. A missing scheme/contract routes back there. The **seed** is part of that contract: qa consumes it and creates *transient, per-test* state through the public surface, but a **foundational** seed gap (a wider baseline a check structurally needs — extra tenants, role fixtures, a perf volume) routes to `web-app-architect` to widen the seed; qa never edits the seed or scripts around it (see Step 3a).
- **Sideways (fixing):** it does not fix feature bugs or emit testIDs — that's `feature-developer`. qa routes the bug/seam-break and re-runs; it never patches feature code to force a green.
- **Down:** it owns the test code (in the codebase), `QA-PLAN.md`, and `QA-REPORT.md`, and the gate verdict. Keep them honest — never mark a check green it didn't actually prove.

## Maintenance mode rules

- Read `QA-PLAN.md` before re-running; re-derive selectors for new/changed features from the scheme (don't trust a stale map — verify against the running app).
- When a feature is added/changed, add its e2e coverage + acceptance assertions and update the selector map in the same pass.
- When a new `NFR-0X` appears (or a target changes), add/adjust its check and the `NFR → check` row.
- Keep the smoke set small and current — prune cases that no longer represent the critical path.
- Never weaken a check to make the gate pass; route the failure to its owner. A green report that hid a failure is the one unforgivable QA outcome.
- Keep both docs consistent with the codebase: a renamed testID or moved test file is reconciled in `QA-PLAN.md` in the same change.

## Examples

**Example 1 — multi-tenant SaaS, full gauntlet**
A scaffolded agency-management SaaS with features built, `QUALITY.md` present (tenant-isolation, p95 perf, WCAG AA, 99.9% SLA NFRs), `ARCHITECTURE.md` defining the testID scheme and a 2-tenant seeded test-env contract. qa derives the selector map from the scheme (each selector tracing to an `FR-0X.Y`), authors e2e covering every feature spec's acceptance criteria + states (with axe folded in for WCAG), tags the login→dashboard and create→list paths `@smoke`, runs the bounded API-vuln pass (cross-tenant IDOR using the two seeded tenants → proves the isolation NFR; headers, injection, secret-leakage), and a perf pass against the p95 budget. CI gate green first, reset hook gives a clean seed each run. The perf check fails (310 ms vs 200 ms); qa routes the slow query to `feature-developer`, which fixes it; qa **restarts the whole gauntlet** and gets a green run. It records the SLA NFR as "ops monitor, not in-gauntlet" rather than faking it. Writes `QA-PLAN.md` + a GREEN `QA-REPORT.md`.

**Example 2 — trivial internal tool, gates mostly off**
A small internal equipment-checkout tool, no `QUALITY.md` perf NFR, a tiny API but no sensitive/multi-tenant data. qa runs e2e (the checkout flow + its states + the PRD acceptance) and flags a one-case smoke set (check out → it shows as checked out). It turns the **API-vuln pass off** (no sensitive/multi-tenant data — recorded with the reason; still does the cheap header/secret-leak sanity check folded into e2e if an API exists) and the **perf pass off** (no perf NFR — noted as deferred until a target exists). The accessibility NFR (WCAG AA, the near-universal floor) is folded into e2e via axe. It does not manufacture a load test or a multi-tenant isolation test for an app that has neither — same discipline as the architect not bolting multi-tenancy onto a single-user app.

**Example 3 — routing a seam break instead of papering over it**
While building the selector map, qa finds the download button in the running app has no `data-testid` (the developer didn't emit it) — only an unstable class buried in nested markup. Rather than writing a brittle `.toolbar > div:nth-child(3) button` XPath, qa routes it to `feature-developer` to emit the conforming function-named `invoices-download-button-{id}` per the scheme, then grabs it by name. The e2e suite stays robust and readable; the seam is honoured, not worked around — and QA never had to parse the page.

**Example 4 — declining to invent a target**
User: "QA the app and make sure it's fast enough." There's no performance NFR in `QUALITY.md`. qa does **not** invent a latency budget to test against — "fast enough" isn't a number it owns. It runs the always-on passes, notes the perf pass is off for lack of a target, and routes the user to `quality-requirements` to set a measurable perf NFR (e.g. "p95 < 200 ms at N users"); once that exists, qa turns it into a perf pass. Setting the target isn't qa's job; proving it is.

**Example 5 — routing a foundational seed gap instead of scripting it through the UI**
The test-env contract seeds one tenant. qa needs to prove the tenant-isolation NFR (cross-tenant IDOR) and to RBAC-check admin-vs-member routes — both need data the seed doesn't have: a *second* tenant and two distinct role fixtures. qa does **not** script the UI to register a second org and invite a second-role user on every run (slow, brittle, and re-creating seed state ad hoc — the non-idempotency the contract exists to prevent). It distinguishes the two: the *single record under test* in each e2e case it creates through the surface and lets the reset hook wipe; but the second tenant and the role fixtures are **foundational environment preconditions several checks share**, so it routes them to `web-app-architect` to widen the seed and records both in `QA-PLAN.md` §7 (Test-data needs) as "seed gap → requested." Until the seed is widened, it marks the isolation and RBAC checks blocked-on-seed in the report rather than faking the data. Once the architect adds the second tenant + roles, qa runs the checks against the deterministic seed and the gate can go green.
