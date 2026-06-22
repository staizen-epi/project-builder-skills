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

Not every feature earns a file, and not every feature fits in *one* file — both under- and over-documenting are failures. Pick the right **size** before writing:

- **Leave it in the PRD when** it's a simple, single-behaviour requirement that a PRD row already captures. Don't split a one-line requirement into a document. (The requirement still gets built — see the note below; "no spec" doesn't mean "no feature.")
- **Write one feature spec when** the feature has real depth — multiple user flows or states, non-obvious rules or limits, edge cases and failure modes — but is still *one* coherent thing.
- **Write a parent spec + child specs when** the feature is really several. If a single PRD requirement contains **≥3 independent sub-behaviours**, each with its own flow, states, and acceptance criteria (e.g. notifications = assignment + mention + comment triggers, each detected differently), don't cram them into one sprawling file. Write a thin **parent** spec that owns the shared surface and links to **child** specs, one per sub-behaviour. See "Splitting into sub-features" below.

If it's borderline between these, ask the user which size fits rather than defaulting.

### Right-sizing gate — bias to the cheaper path

The choice above isn't only about file size; it's about how much process a change earns, and the default should be **cheap**. Authoring and later re-reading a spec is real token cost, so don't spin one up unless the change needs it. Use an **objective** test, not a vibe — write a spec only if the change does **any** of:

- touches more than one module / spans more than one screen or flow,
- alters the data model or a shared contract (API shape, schema, types),
- adds a genuinely new user-facing flow (not a variant of an existing one),
- has non-obvious rules, limits, or multiple states/failure modes,
- introduces a new dependency or integration.

If **none** apply, it's a small, well-understood change: **don't write a spec** — leave it as its PRD row and let `feature-developer` build it straight from that row. (This is the same fast path the developer's spec-less escape hatch expects.)

> **Bias to cheap only where being wrong is also cheap.** Take the no-spec fast path for **reversible, single-module** changes, where a misjudgement costs one quick redo. For **data-model or cross-cutting** changes, write the spec even if it looks small — under-speccing those is expensive to unwind. The asymmetry is the rule: go cheap where the cost of error is cheap.

> **Escalation escape hatch.** Right-sizing is a fast judgement made up front, so it will sometimes wave through a change that turns out bigger mid-build. That's expected and recoverable: if `feature-developer` discovers a "small" change actually touches the data model or branches a flow, it **stops and routes back here** to write the spec properly — a one-time correction, not a silent improvisation. A misclassification escalating upward is the system working, not failing.

> **A spec-less requirement is still a real requirement.** Leaving a feature in the PRD means it has no *spec*, not that it won't be built — `feature-developer` builds it directly from the PRD row. Don't treat "too small for a spec" as "out of scope."

## Two modes

**Check for `specs/features/<feature>.md` for the feature in question.**

- **No spec → Bootstrap mode (interview).** Anchor to the PRD (Step 0), **interview to capture the feature's intended behaviour** (Step 1) — extract from the PRD/conversation, then draw out the flow, rules, and edge cases until no behaviour-shaping assumption is still askable — shape it (Step 2), write the file and update the PRD index (Step 3).
- **Spec exists → Consult & maintain mode.** Read it fully and treat it as the agreed feature behaviour. When the feature's requirements change, update the relevant section, add a Changelog entry, and keep the PRD index link accurate.

---

## Bootstrap mode

### Step 0 — Anchor to the PRD

Read `specs/PRD.md` and pin down what this feature inherits, so you expand the PRD rather than restate or contradict it:

- **Parent requirement IDs** — which PRD functional requirements (e.g. FR-02) this feature expands. These become the spec's anchor and its ID prefix.
- **Persona & user stories** — the user and the stories that touch this feature, so the detail serves a real need.
- **Constraints & non-goals** — anything in the PRD that bounds this feature (platform limits, cost, "not X"). Honour them.

**Don't contradict or outgrow the PRD silently.** If specifying the feature surfaces new top-level scope — a new user type, a new integration, a new non-goal to revise, **or a sibling feature the PRD never declared** (e.g. this feature depends on "comments" but there's no PRD row for it) — route that back through the `product-requirements` skill to add/update the PRD row, then continue here. The PRD stays the source of truth for *what exists*; this spec details *how it behaves*. (A missing dependency you merely *note* in §8 isn't enough if it's genuinely a new feature — that's new scope and belongs in the PRD.)

**Brownfield — locate the integration seam (read the real code).** If this feature is a *change to an existing app* rather than a greenfield addition, this skill owns finding where it plugs in — the job that used to be smuggled in via a PoC. Read the relevant **real** code as context (read-only; never edit it here) and identify the seam: the component(s) it sits next to, the data/types it consumes, the function/hook/endpoint it calls, the route/layout slot it lands in — by their **real names and shapes**. Record the seam in the spec's **§8 Dependencies** so `feature-developer` plugs it in without rediscovering it. This is a *hint to verify*, not a guarantee — the developer reconfirms the path before trusting it (same discipline as `REUSE.md`). If a `specs/POC-NOTES.md` from an opt-in brownfield spike already names a seam, treat it as a starting hint and verify it against the code; you still own the seam in the spec. (Greenfield features have no existing seam — skip this.)

### Step 1 — Interview to capture the feature's behaviour

The spec doesn't exist yet, so **enter interview mode** — this file becomes the source of truth the developer builds from, and an under-specified feature gets *invented* in code instead of decided here. Expand from the PRD and conversation first, then interview the user. For a single feature, three things define its behaviour — **if any are unclear, ask before writing:**

- **The primary flow** — the happy-path sequence the user goes through, start to finish.
- **The rules & limits** — validation, defaults, ordering, caps, permissions that govern it (e.g. "max 5 pinned items", "auto-saves after 600ms").
- **The states & failure behaviour** — empty, loading, error, success, and boundary states, and what happens when something goes wrong (a dependency is down, input is invalid).

**Interview, don't just collect:** walk the flow with the user and probe the seams they skip — "what happens if they hit submit twice?", "what's on screen before any data loads?", "what's the cap, and what does hitting it look like?" Edge cases are exactly the intent that's easiest to lose and most expensive to discover in code. **Keep asking until no behaviour-shaping rule or state is still resolvable by asking**, then default only the genuinely lower-impact detail (exact copy, pixel-level layout, secondary flows, telemetry), recording assumptions in Open Questions rather than blocking. When unsure, specify less and mark it TBD — an honest open question beats an invented requirement.

### Step 2 — Shape the spec

- **Detailed requirements** with IDs that **extend the parent** for traceability: if the PRD parent is FR-02, this spec uses FR-02.1, FR-02.2, … Each is atomic and testable; split anything with an "and". (In a child spec under a split feature, extend one level further — FR-03.1.x — see "Splitting into sub-features".)
- **Define every state explicitly** (empty / loading / error / success / boundary) and the behaviour in each.
- **Acceptance criteria** as observable checks — what a reviewer would do to confirm the feature is done.
- **Mark unresolved behaviour** as ⚠️ TBD inline and mirror it into Open Questions.
- **Defer design.** Note data or integration *needs* as dependencies, but leave schemas, endpoints, and stack to `ARCHITECTURE.md`.

### Step 3 — Write the spec and update the index

1. Create `specs/features/` if it doesn't exist. Name the file in kebab-case after the feature (e.g. `specs/features/jira-integration.md`).
2. Write the spec using the template below.
3. **Update the PRD's §7 Feature specs index**: add the entry linking the feature file to the requirement IDs it expands. This keeps parent↔child traceability intact.
4. Present it to the user and revise before it's relied on for build. Once approved, the natural next step is the `feature-developer` skill, which implements this feature from this spec.

---

## Splitting into sub-features

When the "should this get a spec?" gate lands on **parent + children** (a single PRD requirement holding ≥3 independent sub-behaviours), model it as a thin parent over child specs instead of one sprawling file:

1. **Layout.** Put the children in a folder named after the feature: `specs/features/<feature>/<sub-feature>.md`, and the parent at `specs/features/<feature>/_overview.md` (the `_` sorts it first). The parent is *thin* — it owns the shared surface (the common UI/center, shared rules, cross-cutting states) and **links to each child**; it does not restate child detail.
2. **IDs nest one more level.** The PRD parent is FR-0X; the parent spec groups by FR-0X.n (one per sub-feature); each child spec owns FR-0X.n.y. So notifications (FR-03) → assignment child owns FR-03.1.x, mention child FR-03.2.x, comment child FR-03.3.x. Keep them atomic and testable as usual.
3. **Each child is a normal feature spec** (same template) and **buildable on its own** — its own flow, states, acceptance criteria, dependencies. This is what lets `feature-developer` build the unblocked children now and defer a blocked one (e.g. a child that depends on a feature that doesn't exist yet) without blocking the whole feature.
4. **PRD §7 index** links to the parent (`_overview.md`) against the PRD requirement ID; the parent links down to the children. Don't list every child in the PRD — keep the index at one row per PRD requirement.
5. **Don't over-split.** Two sub-behaviours, or sub-behaviours that share most of their flow, stay one spec. Splitting is for genuinely independent sub-features, not for chopping a cohesive feature into fragments.

> **A blocked or out-of-scope sub-feature is normal.** If one child depends on a feature that doesn't exist (e.g. comment-notifications need a comments feature the PRD never declared), mark that child blocked, capture the missing sibling as a dependency, and — because it's *new product scope* — route it up via the `product-requirements` skill to add the PRD row (see Step 0). The other children proceed.

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
Observable conditions that define "done" — what a reviewer checks. Phrase each as
a black-box-checkable behaviour and tie it to the requirement id(s) it proves
(FR-0X.Y), so the qa skill can turn each criterion into an e2e assertion that
traces back to the requirement. Don't specify selectors or testIDs here — the
testID scheme is the architecture's (qa derives selectors from it); just keep the
criteria observable and id-linked.

## 8. Dependencies
Other features, data, or integrations this relies on or affects. Needs only —
no technical design.
[Brownfield only] **Integration seam** — the existing real files / components /
hooks / endpoints / types this change wires into, by name (a hint for
feature-developer to verify, not a design decision).

## 9. Open questions
Unresolved decisions with their impact and status.

## 10. Changelog
Dated entries for every change to this feature's requirements.
```

## Scope boundary (important)

This skill owns one feature's requirements in depth — not the product, not the design.

- **Up:** don't restate or redecide product-level scope; that's the PRD. Expand it and link back.
- **Down:** don't design the implementation; that's `ARCHITECTURE.md`. Capture data/integration *needs* as dependencies, not schemas or endpoints. (Brownfield: *naming an existing real seam* the change wires into — the components/hooks/endpoints already in the code — is **locating**, not designing; it's allowed. Deciding new schemas/endpoints/stack is not.)
- **Sideways:** keep distinct features in distinct files. If two features are deeply entangled, note the dependency and link — don't merge them into one sprawling spec. (Splitting *one* feature into a parent + children — see "Splitting into sub-features" — is the opposite move and is fine; that's still one feature, just sized to fit.)

## Maintenance mode rules

- Read the feature spec before changing the feature; treat it as the agreed behaviour.
- On any requirement change, update the section **and** add a Changelog entry, and confirm the PRD §7 index link is still accurate.
- If the build has drifted from the spec, surface it and reconcile.
- If a change to this feature implies a change to the product itself, update the PRD via the `product-requirements` skill — don't let a child spec quietly redefine the parent.
- When the `feature-developer` skill routes a gap back here (a behaviour the implementation needed but the spec didn't cover), resolve it in the spec — update the requirement, add a Changelog entry, keep the PRD §7 link accurate — then hand back. The spec leads the code; never let the implementation define behaviour the spec is silent on.

## Examples

**Example 1 — meaty feature, gets a spec**
PRD has `FR-01: AI Daily Brief` with a few rows. The user says "spec out the daily brief." It warrants depth (auto-generation, manual refresh, a fallback path, multiple states). The skill anchors to FR-01, then specifies: the generation flow, requirements FR-01.1…n, states (loading, generated, fallback, empty), rules (length, must cite issue keys), failure behaviour (API down → rule-based fallback with no visible error), and acceptance criteria. Writes `specs/features/ai-daily-brief.md` and adds it to the PRD §7 index linked to FR-01. No model names or API code — those are deferred to ARCHITECTURE.md.

**Example 2 — too small, no spec (but still gets built)**
PRD has `FR-09: A footer shows the app version.` The user asks to spec it. The skill notes this is a single, unambiguous behaviour already captured by the PRD row, and recommends leaving it there rather than creating a near-empty file — offering to add a clarifying acceptance line to the PRD instead. It's explicit that this is *not* descoping: `feature-developer` will build it straight from the PRD row, no spec file required.

**Example 3 — too big for one file, splits into children**
PRD has `FR-03: Members are notified of task changes relevant to them (assigned, mentioned, commented).` That's three independent sub-behaviours — assignment, @mention, and comment triggers each detect differently and have their own states — plus a shared notification center. The skill writes a thin parent `specs/features/notifications/_overview.md` (the center, shared rules, FR-03.1/2/3 grouping) linking to children `assignment.md` (FR-03.1.x), `mention.md` (FR-03.2.x), and `comment.md` (FR-03.3.x). The comment child depends on a comments feature the PRD never declared, so the skill routes that up to `product-requirements` to add a PRD row and marks the child blocked — the other two children proceed and are independently buildable. The PRD §7 index gets one row pointing at the parent. It does **not** merge all three into one sprawling `notifications.md`, nor split the cohesive task-board feature (Example 1) whose flows overlap.
