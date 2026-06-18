# `feature-spec`

> Specifies **one feature** in depth — every flow, state, rule, and acceptance criterion. The first half of the build loop.

[← Back to the overview](../../README.md) · **Stage:** 4a — the build loop (spec it) · child of [`product-requirements`](product-requirements.md)

---

## What it does

Takes one feature and specifies it in depth, producing `specs/features/<feature>.md`. It is the **deep, narrow** layer of the spec family: the PRD says *a feature exists and roughly what it does*; a feature spec says *exactly how it behaves* — every flow, state, rule, edge case, and the conditions that mean it's done.

A feature spec is a **child of the PRD**: it expands specific `FR-0X` requirement IDs (becoming `FR-0X.Y`), traces back to them, and the PRD's §7 index links forward to it. Like its siblings it owns *requirements, not design* — schemas, endpoints, and stack are deferred to `ARCHITECTURE.md`.

It works in **two modes**: bootstrap (anchor to the PRD, gather the detail, write the file, update the PRD §7 index) and consult & maintain (a requirement change updates the section + changelog and keeps the index link accurate).

**It picks the right size first.** Not every feature earns a file:

- **Leave it in the PRD** when it's a single-behaviour requirement a PRD row already captures. *(A spec-less requirement is still real — [`feature-developer`](feature-developer.md) builds it straight from the row.)*
- **One feature spec** when the feature has real depth but is one coherent thing.
- **A parent + child specs** when a single PRD requirement holds ≥3 independent sub-behaviours (e.g. notifications = assignment + mention + comment triggers) — a thin parent owns the shared surface, children are each independently buildable.

## Sample prompts

- "Spec out the invoice-reminder feature in detail — all the states and edge cases."
- "Detailed requirements for X." / "Flesh out the X module."
- "What are all the states for X?"
- "The reminder timing changed to 3 and 7 days before due." *(maintain mode)*

> It asks about the three things that define a feature's behaviour if any are unclear — the primary flow · the rules & limits · the states & failure behaviour — and defaults lower-impact detail (copy, layout, telemetry).

## What it produces

| Output | Description |
|---|---|
| `specs/features/<feature>.md` | The feature's behaviour spec (or a parent `_overview.md` + child specs for a split feature). |
| updated `specs/PRD.md` §7 index | The entry linking the feature file to the requirement IDs it expands. |

Structured into: Summary · Parent requirements · User flow · **Detailed requirements (`FR-0X.Y` IDs)** · **States & edge cases** · Rules & constraints · **Acceptance criteria** (observable checks) · Dependencies (needs only, no design) · Open questions · Changelog.

## Dependencies & consumers

**Reads:**

| Input | For… |
|---|---|
| `specs/PRD.md` | **required parent** — the requirement IDs to expand, persona, constraints, non-goals |
| `specs/POC-NOTES.md` *(brownfield handoff)* | the validated change + the real seam to spec |

**Consumed by:**

| Consumer | Uses the spec for… |
|---|---|
| [`feature-developer`](feature-developer.md) | the source of truth for *what to build* and the acceptance criteria to prove |

**Routes back:** new top-level scope surfaced while speccing (a new user type, integration, or an **undeclared sibling feature** a dependency needs) routes *up* to [`product-requirements`](product-requirements.md) to add the PRD row — then continues here.

## Scope boundary

Owns **one feature's requirements in depth** — not product-level scope (that's [`product-requirements`](product-requirements.md); expand and link back), not the implementation design (that's [`web-app-architect`](web-app-architect.md); capture data/integration *needs* as dependencies). Keep distinct features in distinct files; splitting *one* feature into a parent + children is the allowed exception.
