# `feature-developer`

> Implements **one feature** as a tested vertical slice — built to spec, within the architecture, reusing what exists. The second half of the build loop.

[← Back to the overview](../../README.md) · **Stage:** 4b — the build loop (build it) · partner of [`feature-spec`](feature-spec.md)

---

## What it does

Implements **one feature** as a working, tested **vertical slice** — the end-to-end path from UI (or API surface) down through the data layer — built to its spec and consistent with the architecture. It is the downstream partner of [`feature-spec`](feature-spec.md): the spec says *how the feature must behave*; this skill makes it real and proves it does.

It develops from **pre-conceived context** instead of rediscovering the codebase each time, via three sources of truth and one registry it owns:

- **`specs/features/<feature>.md` — what to build** (behaviour, states, rules, acceptance criteria). Built *to*, not redecided.
- **`specs/ARCHITECTURE.md` — how to build it** (stack, conventions, invariants, the gates that are ON). Built *within*, never violated.
- **`specs/DESIGN.md` + `specs/DESIGN-BINDING.md` — what it must look like** (if present): the bundle's tokens/components and this project's screen→component map + theme.
- **`specs/REUSE.md` — what already exists to reuse.** Read it **first** (reuse before building), update it **after** (so the next feature is faster).

It works in **two modes**: bootstrap (read all inputs → reuse-scan → build the slice within the architecture → prove it against the spec → register reusables) and maintain (a focused edit; if the change implies new behaviour, the spec changes *first*).

A feature can be **built partially** when part of it is blocked — it builds the unblocked vertical slice and defers the blocked part (without building the missing dependency to unblock it).

## Sample prompts

- "Build the invoice-reminder feature from its spec."
- "Implement X." / "Develop the X module." / "Code up X from its spec."
- "Let's build the next feature."
- "Change how reminders are sent." *(maintain — but the spec changes first)*

> If there's no spec and the behaviour is genuinely under-specified, it routes to [`feature-spec`](feature-spec.md). But a *deliberately spec-less* single-behaviour PRD row (e.g. "a footer shows the app version") it builds straight from the row — looping back would only bounce.

## What it produces

| Output | Description |
|---|---|
| code + tests | The vertical slice — every state the spec defines, honouring every invariant and ON gate, with tests for the happy path + edge/failure cases. |
| `specs/REUSE.md` | The reuse registry — new reusables registered with path + one-line purpose; drifted lines reconciled. |
| changelog updates | The feature spec's changelog (implemented; criteria verified); `ARCHITECTURE.md` only if a decision actually changed. |

"Done" is observable: the primary flow runs end to end, every §7 acceptance criterion passes under a real run, every state behaves as specified, and the gate (lint, typecheck, tests, isolation/authz where touched) is green.

## Dependencies & consumers

**Reads:**

| Input | Treated as… | For… |
|---|---|---|
| `specs/features/<feature>.md` | **required** | the behaviour to build and the criteria to prove |
| `specs/ARCHITECTURE.md` | **required** | conventions, invariants, the gates that are ON |
| `specs/REUSE.md` *(if present)* | the index | reuse before building; verify a path before trusting it |
| `specs/DESIGN.md` + bundle *(if present)* | the visual language | tokens/components/a11y baseline to build *with* |
| `specs/DESIGN-BINDING.md` *(if present)* | this project's wiring | the screen→component map + theme |

> `/poc/` is ignored entirely — its learnings already reached the build via the scaffolder.

**Consumed by:** the next loop iteration reads `REUSE.md`; otherwise this is the end of the pipeline — it produces the running feature.

**Routes back — never improvises scope:** a *behaviour* gap → [`feature-spec`](feature-spec.md); a *technical-design* gap → [`web-app-architect`](web-app-architect.md); a *missing component/token* → [`ux-designer`](ux-designer.md); a *missing/wrong screen mapping or theme value* → [`design-binder`](design-binder.md).

## Scope boundary

Owns the **implementation, its tests, and `REUSE.md`** — not *what* the feature does ([`feature-spec`](feature-spec.md)), the stack/conventions/gates ([`web-app-architect`](web-app-architect.md)), the visual language ([`ux-designer`](ux-designer.md)), the screen map/theme ([`design-binder`](design-binder.md)), or the foundation/other features. One feature, one vertical slice. A change that alters behaviour means the **spec changes first**, then the code — never the reverse.
