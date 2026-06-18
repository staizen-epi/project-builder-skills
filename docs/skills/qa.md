# `qa`

> Runs the **QA gauntlet** against the built app as a **black box** and owns the pass/fail **gate** before release. The verification stage that closes the build loop.

[← Back to the overview](../../README.md) · **Stage:** 5 — verify & gate · consumes [`web-app-architect`](web-app-architect.md), [`quality-requirements`](quality-requirements.md), [`feature-developer`](feature-developer.md)

---

## What it does

Exercises a built web app through its **own public surface only** — the rendered UI and the published API — and enforces a quality **gate**. It never imports app source, reads internal modules, or touches the database; it tests what a user or an API client can observe. It does for *verification* what [`quality-requirements`](quality-requirements.md) does for *targets*: it proves the app actually works and meets its non-functional targets, observably, then blocks release until it's green.

It runs **four gated black-box passes** and turns each `specs/QUALITY.md` `NFR-0X` into a runnable check:

- **e2e (feature/functionality)** — *always on.* Drives each feature spec's primary flow, every state, and each §7 acceptance criterion (accessibility/axe folded in).
- **Smoke-set flagging** — *always on.* Picks the critical-path cases, **tags them `@smoke`** in the suite, and lists them in the plan (so CI / release can run just those).
- **API vulnerability scan** — *gated* on the app exposing an API or holding sensitive/multi-tenant data. **Bounded, non-destructive, own-app-only**: authz/IDOR/cross-tenant, authn gaps, injection, security headers, secret leakage, rate-limit presence. No DoS, no destructive payloads.
- **Performance testing** — *gated* on a performance `NFR-0X` existing in `QUALITY.md` (a budget to test against).

It is **test-centered via a shared seam**: the architect owns the **testID scheme**, [`feature-developer`](feature-developer.md) gives every interactive/meaningful element a **function-named** `data-testid` per it (`download-button`, `quick-search-input`, `invoice-row-{id}` — named by what it does), and `qa` **derives** its selectors from the same scheme — so it grabs each element by name rather than scraping the DOM or writing deep XPaths, selectors stay stable, and they still trace back to requirements.

It requires the architecture's **CI gate** (lint/typecheck/unit + secret scan) green **first**, then runs its passes and **restarts the whole gauntlet on any failure until green**. It works in **two modes**: bootstrap (set up the gauntlet, run to green, write both docs) and maintain (re-derive selectors for changed features, add checks for new criteria/NFRs, re-run, refresh the verdict).

## Sample prompts

- "QA the app and tell me if it's release-ready."
- "Run the test gauntlet." / "Verify quality before we ship."
- "Scan the API for vulnerabilities." / "Set up smoke tests."
- "Check that our NFRs are met."

> It owns the verdict but **routes the fix**: it never patches feature code or weakens a check to force a green.

## What it produces

| Output | Description |
|---|---|
| test code *(in the codebase)* | The four gated suites, built with the architecture's testing stack; smoke cases tagged `@smoke`. |
| `specs/QA-PLAN.md` | The **durable test contract** — testID-derived selector map, each pass's scope + gate decision, the smoke set, the `NFR→check` mapping. The QA analogue of `REUSE.md`. |
| `specs/QA-REPORT.md` | The **latest run's verdict** — pass/fail per check, restart history, what's blocking the gate, and NFRs that need ops/time-based verification. |

"Done" is observable: the CI gate is green, the app stands up against the seeded, **resettable** test environment (so runs are idempotent), every gated pass and NFR check passes under a real run, and a full clean gauntlet succeeds.

## Dependencies & consumers

**Reads:**

| Input | Treated as… | For… |
|---|---|---|
| `specs/ARCHITECTURE.md` | **required** | the testID scheme + the seeded/resettable test-env contract + testing stack + API surface |
| the running, built app | **required** | the black-box target to test |
| `specs/QUALITY.md` *(if present)* | the targets | each `NFR-0X` `Verified by: qa` → a runnable check; a perf NFR turns the perf pass on |
| `specs/PRD.md` + `specs/features/` | the behaviour | flows + §7 acceptance criteria the e2e pass proves |

> `/poc/` is ignored entirely — throwaway mock-data code is never tested or treated as real.

**Consumed by:** the gate verdict feeds release (a future `release-manager` promotes only on green); the `@smoke` tags let CI run a fast confidence check; `QA-PLAN.md` is the durable map maintained across runs.

**Routes back — never papers over:** a *behaviour bug* → [`feature-developer`](feature-developer.md); a *missing/non-conforming testID* → [`feature-developer`](feature-developer.md), or the *scheme/test-env contract itself* → [`web-app-architect`](web-app-architect.md); a *missing/infeasible target* → [`quality-requirements`](quality-requirements.md); an *untestable/missing behaviour* → [`feature-spec`](feature-spec.md).

## Scope boundary

Owns **running the tests, the test code, `QA-PLAN.md`/`QA-REPORT.md`, and the gate verdict** — not the non-functional *targets* ([`quality-requirements`](quality-requirements.md)), *what* the app should do ([`feature-spec`](feature-spec.md) / the PRD), the stack/testID scheme/test-env contract ([`web-app-architect`](web-app-architect.md)), or *fixing* the code ([`feature-developer`](feature-developer.md)). It tests, reports, enforces, and routes. The one unforgivable outcome is a green report that hid a failure — it never weakens a check to pass the gate.
