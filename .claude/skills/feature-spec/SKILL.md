---
name: feature-spec
description: Authors a detailed requirements spec for a single feature or module, expanding one part of the PRD into depth — the user flow, sub-requirements, states and edge cases, rules, and acceptance criteria — and writes it to specs/features/<feature>.md. Use this mid-project when going deep on one feature, after a PRD exists: "spec out the X feature", "detailed requirements for X", "flesh out the X module", "what are all the states for X". This is the child layer beneath specs/PRD.md; it requires a PRD as its parent and defers technical design to ARCHITECTURE.md. For the high-level product (what the whole thing is), use the product-requirements skill instead.
---

# Feature Spec

This skill takes one feature and specifies it in depth, producing `specs/features/<feature>.md`. It is the **deep, narrow** layer of the spec family: the PRD says *a feature exists and roughly what it does*; a feature spec says *exactly how it behaves* — every flow, state, rule, edge case, and the conditions that mean it's done.

It sits below the PRD in the pipeline: `specs/PRD.md` (the whole product) → `specs/features/*.md` (each complex feature) → `ARCHITECTURE.md` (the how) → build. A feature spec is a **child of the PRD**: it expands specific PRD requirement IDs and traces back to them, and the PRD's §7 index links forward to it.

Like its siblings, it owns requirements, not design — it never decides tech stack, schemas, or API shapes. Those live in `ARCHITECTURE.md`.

## Precondition

**A feature spec needs a parent. Check for `specs/PRD.md` before doing anything.**

- **Missing →** a feature spec without a PRD is an orphan. Suggest the `product-requirements` skill to establish the product first. Only proceed without a PRD if the user explicitly wants a standalone feature note and understands there's no parent to trace to.
- **Present →** read it and locate the feature (see Step 0).

## Should this feature get its own spec?

Not every feature earns a file — over-documenting is its own failure. Apply this gate before writing:

- **Write a feature spec when** the feature has real depth: multiple user flows or states, non-obvious rules or limits, edge cases and failure modes, or enough ambiguity that the team would otherwise guess.
- **Leave it in the PRD when** it's a simple, single-behaviour requirement that a PRD row already captures. Don't split a one-line requirement into a document.

If it's borderline, ask the user whether the feature is meaty enough to warrant its own spec rather than defaulting to creating one.

## Two modes

**Check for `specs/features/<feature>.md` for the feature in question.**

- **No spec → Bootstrap mode.** Anchor to the PRD (Step 0), gather the feature detail (Step 1), shape it (Step 2), write the file and update the PRD index (Step 3).
- **Spec exists → Consult & maintain mode.** Read it fully and treat it as the agreed feature behaviour. When the feature's requirements change, update the relevant section, add a Changelog entry, and keep the PRD index link accurate.

---

## Bootstrap mode

### Step 0 — Anchor to the PRD

Read `specs/PRD.md` and pin down what this feature inherits, so you expand the PRD rather than restate or contradict it:

- **Parent requirement IDs** — which PRD functional requirements (e.g. FR-02) this feature expands. These become the spec's anchor and its ID prefix.
- **Persona & user stories** — the user and the stories that touch this feature, so the detail serves a real need.
- **Constraints & non-goals** — anything in the PRD that bounds this feature (platform limits, cost, "not X"). Honour them.

**Don't contradict or outgrow the PRD silently.** If specifying the feature surfaces new top-level scope (a new user type, a new integration, a new non-goal to revise), route that back through the `product-requirements` skill to update the PRD — then continue here. The PRD stays the source of truth for *what exists*; this spec details *how it behaves*.

### Step 1 — Gather the feature detail

Expand from the PRD and conversation first; ask only the high-impact unknowns. For a single feature, three things define its behaviour — **if any are unclear, ask before writing:**

- **The primary flow** — the happy-path sequence the user goes through, start to finish.
- **The rules & limits** — validation, defaults, ordering, caps, permissions that govern it (e.g. "max 5 pinned items", "auto-saves after 600ms").
- **The states & failure behaviour** — empty, loading, error, success, and boundary states, and what happens when something goes wrong (a dependency is down, input is invalid).

Assume sensible defaults for lower-impact detail (exact copy, pixel-level layout, secondary flows, telemetry) and record assumptions in Open Questions rather than blocking. When unsure, specify less and mark it TBD — an honest open question beats an invented requirement.

### Step 2 — Shape the spec

- **Detailed requirements** with IDs that **extend the parent** for traceability: if the PRD parent is FR-02, this spec uses FR-02.1, FR-02.2, … Each is atomic and testable; split anything with an "and".
- **Define every state explicitly** (empty / loading / error / success / boundary) and the behaviour in each.
- **Acceptance criteria** as observable checks — what a reviewer would do to confirm the feature is done.
- **Mark unresolved behaviour** as ⚠️ TBD inline and mirror it into Open Questions.
- **Defer design.** Note data or integration *needs* as dependencies, but leave schemas, endpoints, and stack to `ARCHITECTURE.md`.

### Step 3 — Write the spec and update the index

1. Create `specs/features/` if it doesn't exist. Name the file in kebab-case after the feature (e.g. `specs/features/jira-integration.md`).
2. Write the spec using the template below.
3. **Update the PRD's §7 Feature specs index**: add the entry linking the feature file to the requirement IDs it expands. This keeps parent↔child traceability intact.
4. Present it to the user and revise before it's relied on for build.

---

## Output: specs/features/<feature>.md

```markdown
# <Feature Name> — Feature Spec

> Child of specs/PRD.md · Expands: FR-0X[, FR-0Y]
> Technical design lives in ARCHITECTURE.md. Keep current; see Changelog.

**Status:** [Draft | Active] · **Last updated:** [date]

## 1. Summary
What this feature is and the user need it serves, in the product's context.
One short paragraph.

## 2. Parent requirements
The PRD requirement IDs this spec expands, and a one-line restatement of each
so the spec is readable on its own.

## 3. User flow
The primary happy-path flow, step by step. Note any secondary flows.

## 4. Detailed requirements
Atomic, testable requirements with IDs extending the parent (FR-0X.1, …),
grouped logically. Mark unresolved behaviour as ⚠️ TBD.

## 5. States & edge cases
Empty / loading / error / success / boundary states and the behaviour in each,
plus failure modes (dependency down, invalid input, limit reached).

## 6. Rules & constraints
Validation, limits, defaults, ordering, and permissions specific to this
feature. Honour the PRD's constraints and non-goals.

## 7. Acceptance criteria
Observable conditions that define "done" — what a reviewer checks.

## 8. Dependencies
Other features, data, or integrations this relies on or affects. Needs only —
no technical design.

## 9. Open questions
Unresolved decisions with their impact and status.

## 10. Changelog
Dated entries for every change to this feature's requirements.
```

## Scope boundary (important)

This skill owns one feature's requirements in depth — not the product, not the design.

- **Up:** don't restate or redecide product-level scope; that's the PRD. Expand it and link back.
- **Down:** don't design the implementation; that's `ARCHITECTURE.md`. Capture data/integration *needs* as dependencies, not schemas or endpoints.
- **Sideways:** keep each feature in its own file. If two features are deeply entangled, note the dependency and link — don't merge them into one sprawling spec.

## Maintenance mode rules

- Read the feature spec before changing the feature; treat it as the agreed behaviour.
- On any requirement change, update the section **and** add a Changelog entry, and confirm the PRD §7 index link is still accurate.
- If the build has drifted from the spec, surface it and reconcile.
- If a change to this feature implies a change to the product itself, update the PRD via the `product-requirements` skill — don't let a child spec quietly redefine the parent.

## Examples

**Example 1 — meaty feature, gets a spec**
PRD has `FR-01: AI Daily Brief` with a few rows. The user says "spec out the daily brief." It warrants depth (auto-generation, manual refresh, a fallback path, multiple states). The skill anchors to FR-01, then specifies: the generation flow, requirements FR-01.1…n, states (loading, generated, fallback, empty), rules (length, must cite issue keys), failure behaviour (API down → rule-based fallback with no visible error), and acceptance criteria. Writes `specs/features/ai-daily-brief.md` and adds it to the PRD §7 index linked to FR-01. No model names or API code — those are deferred to ARCHITECTURE.md.

**Example 2 — too small, no spec**
PRD has `FR-09: A footer shows the app version.` The user asks to spec it. The skill notes this is a single, unambiguous behaviour already captured by the PRD row, and recommends leaving it there rather than creating a near-empty file — offering to add a clarifying acceptance line to the PRD instead.
