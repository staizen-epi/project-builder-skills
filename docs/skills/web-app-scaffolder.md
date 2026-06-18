# `web-app-scaffolder`

> Stands up the **Phase-0 foundation** from the architecture — and proves it works.

[← Back to the overview](../../README.md) · **Stage:** 3 — stand up the foundation

---

## What it does

Turns an approved `specs/ARCHITECTURE.md` into a running, verified Phase-0 skeleton: repo structure, the chosen stack, auth, tenancy primitives, base schema, logging, and the CI gate. It is the downstream partner of [`web-app-architect`](web-app-architect.md): the architect decides *what* and *why*; this skill builds the foundation and proves it works, then stops — so features get built on solid ground.

**The contract is simple: the doc is the source of truth, and this skill obeys it.** It does not re-open architectural decisions and builds only what the doc's gates turned on. If the doc didn't gate in a queue, it doesn't install one. If it gated multi-tenancy off, it doesn't add tenant scoping.

It scaffolds in dependency order — the **always-on foundation** for every app (repo, typed config + secret loading, boundary validation, redacted logging, security headers, DB + base schema, auth, CI gate, environment config) — then adds a **gated piece only when its gate is ON** (multi-tenancy + isolation test, RBAC, PII handling, jobs, scheduler, cache, webhooks, spend caps, real-time, embed/SSO, audit logging).

## Sample prompts

- "Scaffold the project and stand up Phase 0."
- "Bootstrap the repo from the architecture."
- "Set up the foundation and prove it works."
- "Start building" *(a web app that already has an architecture)*

> It stops once the foundation boots, the CI gate is green, and the Phase-0 checks pass — then hands back to you to build features. A scaffolder that starts writing features is how the foundation ends up half-tested.

## What it produces

| Output | Description |
|---|---|
| your codebase | The proven Phase-0 foundation — repo, stack, auth, gated primitives, CI gate. |
| `ARCHITECTURE.md` changelog entry | A dated note that Phase 0 was scaffolded and verified, plus any deviation/assumption. |

"Done" is not "files exist" — it is **a foundation that demonstrably works**: the app boots and login works, the gate is green, the gated proofs pass (e.g. a cross-tenant read is *denied*, the no-op job runs), and every Phase-0 acceptance criterion in §8 passes under a real run.

## Dependencies & consumers

**Reads:**

| Input | Treated as… | For… |
|---|---|---|
| `specs/ARCHITECTURE.md` | **required, authoritative** | the build set — stack, gates, base schema, Phase-0 acceptance criteria |
| `specs/POC-NOTES.md` *(if present)* | **advisory hints** | UX/flow patterns, component shapes, data shapes — architecture wins on conflict; never adopt `/poc` code |
| `specs/DESIGN.md` + `specs/design/` *(if present)* | **a decision** | adopts the bundle's tokens + base components as the visual language |
| `specs/DESIGN-BINDING.md` *(if present)* | **a decision** | wires the chosen adapter, applies the project theme, uses the screen→component map for Phase-0 screens |

> Note the difference: `POC-NOTES.md` is a *hint* (architecture wins); the design bundle and binding are *decisions* (adopted). `/poc/` code is ignored entirely.

**Consumed by:**

| Consumer | Depends on the foundation for… |
|---|---|
| [`feature-developer`](feature-developer.md) | features build on a scaffolded foundation, not bare ground |

## Scope boundary

Builds and proves the **foundation only.** It does not implement Phase 1+ features (that's [`feature-developer`](feature-developer.md)), invent a design system (route to [`ux-designer`](ux-designer.md)) or a binding (route to [`design-binder`](design-binder.md)), or re-open architecture (route to [`web-app-architect`](web-app-architect.md)). Reverse pressure is intentional: wanting to add something the doc didn't gate in is a signal to update the doc via the architect first.
