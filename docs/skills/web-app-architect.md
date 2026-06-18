# `web-app-architect`

> Designs **how** the app is built — the technical source of truth, sized to what the app actually needs.

[← Back to the overview](../../README.md) · **Stage:** 2 — design the build

---

## What it does

Turns a vague "let's build a web app" into a concrete, reviewed `specs/ARCHITECTURE.md`. It encodes the parts of good web architecture that hold true regardless of what the app *does* (the always-on invariants) while **gating** the parts that only apply to some apps — so a simple project never gets enterprise scaffolding it doesn't need.

It works in **two modes**:

- **No `ARCHITECTURE.md` → Bootstrap.** Reads the PRD first (Step 0), resolves the decision gates, generates the first architecture.
- **`ARCHITECTURE.md` exists → Consult & maintain.** Treats it as binding; updates it (and its changelog) whenever a change introduces a new architectural decision.

**Step 0 — read the PRD first.** If `specs/PRD.md` exists it derives project name, who logs in, roles, non-goals, constraints, integrations, data sensitivity, and scheduled work from it — so the questionnaire shrinks to only the genuine gaps. If `specs/QUALITY.md` exists it reads that too as the source of truth for non-functional *targets* and designs to them.

**The gates.** High-impact decisions split into *workload gates* (tenancy, data sensitivity, metered APIs, async/scheduled work — turn sections on/off) and *structural decisions* (FE/BE topology, codebase layout, monolith vs microservices, API protocol, database paradigm — a shape that's always chosen). It biases hard to the boring defaults: **modular monolith + single repo + REST + PostgreSQL**, deviating only with a recorded reason.

## Sample prompts

- "The PRD's done — design the architecture."
- "How should we build this? Set up the ARCHITECTURE.md."
- "Decide the tech stack and structure for this app."
- "Let's start building X." / "Scaffold a SaaS." / "Set up the backend." *(implied — still triggers it)*
- "We're adding a paid SMS API — update the architecture." *(maintain mode)*

> Because it reads the PRD, a single-user local tool won't get multi-tenancy bolted on, and a SaaS gets tenant isolation, RBAC, and the rest — automatically.

## What it produces

| Output | Description |
|---|---|
| `specs/ARCHITECTURE.md` | The durable source of truth for how the app is built. |

Structured into: Overview · **Decisions (§2 — structural + workload, with the why)** · Tech stack · System architecture · Data model conventions · **Security & resilience (§6 — invariants + gated controls)** · Environments & release · **Build phases (§8 — Phase 0 first, with acceptance criteria)** · Open questions & assumptions · Changelog.

It always includes the **invariants** (validate at every boundary, secrets in env, structured logging with redaction, TLS, tested backups, a CI gate, a staging step) and adds a gated section only when its condition is met.

## Dependencies & consumers

**Reads:**

| Input | For… |
|---|---|
| `specs/PRD.md` *(if present)* | product source of truth — project, users, auth, constraints, integrations |
| `specs/QUALITY.md` *(if present)* | non-functional **targets** to design to (the seam is bidirectional) |

**Consumed by:**

| Consumer | Uses the architecture for… |
|---|---|
| [`web-app-scaffolder`](web-app-scaffolder.md) | the build set — stack, gates, base schema, Phase-0 acceptance criteria |
| [`poc-developer`](poc-developer.md) | soft context — intended stack/UX direction (the PoC may diverge) |
| [`ux-designer`](ux-designer.md) | the front-end stack the design is expressed in |
| [`design-binder`](design-binder.md) | routes + front-end stack (which adapter to select) |
| [`feature-spec`](feature-spec.md) | defers technical design here (captures *needs*, not schemas) |
| [`feature-developer`](feature-developer.md) | how to build — conventions, invariants, gates, design principles |

**Routes back to it:** any skill that surfaces a *technical-design* gap (a pattern the architecture doesn't cover) routes it here. The seam with [`quality-requirements`](quality-requirements.md) is **bidirectional** — an infeasible NFR target routes back there to renegotiate, and the two docs must never drift on a shared number.

## Scope boundary

Owns the **technical how**, not product scope (that's [`product-requirements`](product-requirements.md)), feature behaviour ([`feature-spec`](feature-spec.md)), the visual language ([`ux-designer`](ux-designer.md)), or non-functional targets ([`quality-requirements`](quality-requirements.md)). On any design conflict, the architecture wins — but if a need contradicts the PRD, it surfaces and reconciles rather than diverging.
