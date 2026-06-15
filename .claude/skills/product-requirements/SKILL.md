---
name: product-requirements
description: Defines WHAT a product should be and WHY, producing a single specs/PRD.md (Product Requirements Document) that is the highest-level source of truth for a build. Use this at the very start of a new product or feature — before any architecture or code — whenever the user is figuring out what to build, scoping an MVP, writing requirements or user stories, or says "write a PRD", "spec this out", "what should this app do", or "let's build X" without an existing PRD. If specs/PRD.md already exists, use this skill to read it, follow it, and keep it current. This skill defines requirements only; it hands the technical "how" (architecture, data models, APIs) to the web-app-architect skill.
---

# Product Requirements

This skill turns a rough product idea into a concrete, reviewed `specs/PRD.md` — the requirements document that everything downstream depends on. It is the **front of the build pipeline**: requirements (`specs/PRD.md`) → architecture (`ARCHITECTURE.md`) → scaffold → features.

Its job is the *what* and the *why*: the problem, who it's for, what the product must and must not do, and how you'll know it's working. It deliberately does **not** decide the *how* — tech stack, system design, data models, and API shapes belong to the `web-app-architect` skill. Keeping that seam clean is what lets the skills compose instead of overwriting each other's work.

The single output is **`specs/PRD.md`**. It is the durable source of truth for product scope.

## Two modes

**Check for `specs/PRD.md` before doing anything else.**

- **No `specs/PRD.md` → Bootstrap mode.** Gather requirements (below), shape them, create the `specs/` folder if it doesn't exist, and write the first `specs/PRD.md`. Do not move on to architecture or code until the user has seen and approved it.
- **`specs/PRD.md` exists → Consult & maintain mode.** Read it fully and treat it as the agreed scope. When the user adds, drops, or changes a requirement, update the relevant section **and** append a dated Changelog entry in the same change. When a downstream skill (architect, scaffolder) needs product context, read it from here.

---

## Bootstrap mode

### Step 1 — Gather what the requirements need

The goal is a shared understanding of the product, not a filled-in form. Most products hinge on a handful of decisions; get those right and assume sensible defaults for the rest. Work in this order:

1. **Extract first.** Pull everything the conversation already states or implies and treat it as provisional.
2. **Ask only the high-impact items that are still unknown.** Five answers define the product; the rest only add detail. **If any of these five are undetermined, ask before generating — never invent them:**
   - **What & who** — what is the product in one sentence, and who is the primary user?
   - **Problem & success** — what problem does it solve, and what does "working well" look like?
   - **Must-have capabilities** — the small set of things that define a usable MVP.
   - **Non-goals** — what is explicitly out of scope? (As important as the goals; it's what stops scope creep.)
   - **Hard constraints & external systems** — platform (web/desktop/mobile), cost ceilings, offline/local-only, corporate/IT limits, and which existing tools or APIs it must work with.
3. **Assume sensible defaults for the rest** and record each assumption rather than blocking on it. A draft the user corrects beats a long interrogation.
4. **When unsure, scope smaller.** Prefer the tighter MVP and a richer Future Roadmap over a bloated must-have list. It is easier to add a requirement than to cut a half-built one.

Present the high-impact questions as one short block, with examples or defaults shown so the user can answer quickly. Explain briefly why you're asking when it isn't obvious.

**High-impact — ask if unknown:** what & who · problem & success · must-have capabilities · non-goals · hard constraints & external systems.

**Lower-impact — assume unless clearly relevant:** secondary user stories; detailed per-feature behaviour; non-functional needs (performance, persistence, privacy, accessibility); known limitations; open questions; roadmap horizon. Default to the simplest reasonable answer and note it.

### Step 2 — Shape the requirements

Turn the answers into structured, testable requirements following these conventions:

- **User stories** in the form: *As [persona], I want [capability] so that [benefit].* Group by area.
- **Functional requirements** that are atomic and testable, each with a stable ID (e.g. `FR-01`, with sub-items `FR-01-1`). Group by feature. One requirement per row — if it has an "and", split it.
- **Mark anything unresolved** inline (e.g. ⚠️ *TBD*) and add it to the Open Questions register rather than guessing a behaviour.
- **Non-functional requirements** only where they matter (performance budgets, persistence model, privacy, offline behaviour, cost). Don't pad.
- **Goals and non-goals both stated explicitly.**
- **Surface, but do not decide, downstream signals** the architect will need — data sensitivity, multi-user vs single-user, integrations, scheduled work — as requirements/constraints, not as technical choices. This warms up the handoff without crossing the seam.

### Step 3 — Write specs/PRD.md

Create `specs/` if it doesn't exist, then write `specs/PRD.md` using the template below. Fill the sections the requirements support; record assumptions in Open Questions. Present it to the user and revise before any architecture or code work begins.

---

## Output: specs/PRD.md

Produce the file in this structure. Keep it requirements-focused; defer the technical "how" to `ARCHITECTURE.md`.

```markdown
# [Product Name] — Product Requirements

> The highest-level source of truth for what this product is and why.
> Technical design lives in ARCHITECTURE.md. Keep current; see Changelog.

**Status:** [Draft | Active] · **Last updated:** [date]

## 1. Overview
One or two paragraphs: what the product is, who uses it, and the shape of it.

## 2. Problem statement
The problem being solved and the core question the product answers for its user.

## 3. Goals & non-goals
Goals: the outcomes the product is trying to achieve.
Non-goals: what it explicitly will NOT do (the scope fence).

## 4. User persona(s)
The primary user: role, tools, pain points, technical comfort, constraints.

## 5. User stories
Grouped "As [persona], I want [capability] so that [benefit]" statements.

## 6. Functional requirements
Per-feature tables of atomic, testable requirements with stable IDs.
Mark unresolved behaviour as ⚠️ TBD and mirror it into Open Questions.

## 7. Feature specs (index)
Index of detailed per-feature specs under `specs/features/`, authored by the
feature-spec skill. Each entry links a feature to the requirement IDs it
expands (e.g. `features/ai-daily-brief.md` → FR-01). Empty until features are
spec'd in depth.

## 8. Non-functional requirements
Only those that matter: performance, persistence, privacy, offline, cost,
accessibility.

## 9. Constraints & external systems
Platform, cost ceilings, environment limits, and the existing tools/APIs the
product must integrate with — stated as needs, not technical solutions.

## 10. Solution handoff
A short note pointing to ARCHITECTURE.md for tech stack, system design, data
models, and API design. List the signals the architect will need (tenancy,
data sensitivity, integrations, scheduled work) without deciding them here.

## 11. Open questions
A register of unresolved decisions with their impact and status.

## 12. Future roadmap
Near / medium / long-term items deliberately out of the current scope.

## 13. Changelog
Dated entries for every scope change.
```

## Scope boundary (important)

This skill owns requirements, not design. Do **not** put tech-stack choices, architecture diagrams, database schemas, or API endpoint specs in `specs/PRD.md` — those are the architect skill's job and live in `ARCHITECTURE.md`. If the user starts making technical decisions during requirements gathering, capture them as *constraints* ("must run locally", "must use the free tier of X") and note them for the architect, rather than designing the system here. One concept, one owner.

## Maintenance mode rules

- Read `specs/PRD.md` before adding or changing scope; treat it as the agreed product.
- When scope changes, update the relevant section **and** add a Changelog entry in the same change.
- If implementation has drifted from the PRD, surface it and reconcile — don't silently let the doc go stale.
- Keep the §7 Feature specs index current: when a feature spec is added, renamed, or removed under `specs/features/`, update the index and its requirement links. Detailed per-feature requirements are authored by the `feature-spec` skill, not here — the PRD indexes them.
- When `ARCHITECTURE.md` doesn't yet exist, the natural next step after an approved PRD is the `web-app-architect` skill, which should read this PRD as its input.

## Examples

**Example 1 — local single-user tool**
Input: "A personal dashboard that pulls my Jira tickets and calendar into one screen so I can plan my day, runs on my machine, no cloud."
High-impact resolved: solo user; problem = tool-switching at day start; must-haves = unified ticket+calendar view, daily plan; non-goals = not a Jira/Outlook replacement, not multi-user, not cloud; constraints = local-only, free tier, corporate IT limits, integrates Jira + calendar feed. The PRD captures personas, user stories, ID'd functional requirements, the local-only constraint, and a §10 handoff noting "single-user, local persistence, integrates Jira/calendar/AI" for the architect — but makes no stack or schema decisions.

**Example 2 — vague start**
Input: "I want to build a booking app."
Almost nothing high-impact is determined, so the skill asks the five high-impact questions (what & who, problem & success, must-haves, non-goals, constraints/integrations) before writing anything — and scopes the MVP tight, pushing speculative features to the roadmap.
