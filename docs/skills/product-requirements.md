# `product-requirements`

> Defines **what** a product should be and **why** — the highest-level source of truth for the whole build.

[← Back to the overview](../../README.md) · **Stage:** 1 — define the product

---

## What it does

Turns a rough product idea into a concrete, reviewed `specs/PRD.md` (Product Requirements Document). It owns the *what* and the *why*: the problem, who it's for, what the product must and must not do, and how you'll know it's working. It deliberately does **not** decide the *how* — tech stack, system design, data models, and API shapes are the [`web-app-architect`](web-app-architect.md)'s job.

It works in **two modes**:

- **No `specs/PRD.md` → Bootstrap (interview).** The PRD doesn't exist yet, so it *interviews* you to capture intent — leading with the high-impact questions, then following up on your answers until no PRD-shaping assumption is still worth asking about — then writes the first PRD with the best data captured at that point.
- **`specs/PRD.md` exists → Consult & maintain.** Reads it as agreed scope; when you add, drop, or change a requirement it updates the relevant section *and* appends a dated Changelog entry. (No interview here — it touches only what changed.)

On first creation it leads with the five high-impact questions that genuinely define a product — **what & who · problem & success · must-have capabilities · non-goals · hard constraints & external systems** — then draws out the rest (probing vague capabilities, confirming non-goals, surfacing the user or integration you didn't mention) before defaulting only the genuinely lower-impact remainder and recording each assumption. When unsure, it scopes *smaller* (tighter MVP, richer roadmap).

## Sample prompts

- "I want to build a tool that lets freelancers track invoices and send reminders. Help me write a PRD."
- "Let's spec out a new product: a reading tracker that syncs my Goodreads shelf."
- "Define the requirements for an internal dashboard my team will use to track weekly OKRs."
- "I want to build a booking app." *(vague start — it will interview you, starting from the five high-impact questions and following up until the product is clear)*
- "Add a requirement to the PRD: let users export invoices to CSV." *(maintain mode)*

> On first creation it interviews you before writing — answer the questions (and the follow-ups), or reply "defaults" to let it assume sensible ones and note them. Once the PRD exists it stops interviewing and just maintains it.

## What it produces

| Output | Description |
|---|---|
| `specs/PRD.md` | The single durable source of truth for product scope. |

The PRD is structured into: Overview · Problem statement · Goals & non-goals · Persona(s) · User stories · Functional requirements (stable `FR-0X` IDs) · **Feature specs index (§7)** · Non-functional requirements · Constraints & external systems · **Solution handoff (§10)** · Open questions · Future roadmap · Changelog.

Two parts are load-bearing for the pipeline:
- **§7 Feature specs index** — links each `FR-0X` to its detailed spec under `specs/features/` (filled in by [`feature-spec`](feature-spec.md)).
- **§10 Solution handoff** — surfaces the signals the architect needs (tenancy, data sensitivity, integrations, scheduled work) *without deciding them*.

## Dependencies & consumers

**Reads:** the conversation (it's the front of the pipeline — no upstream artifact).

**Consumed by — nearly everything downstream depends on the PRD:**

| Consumer | Uses the PRD for… |
|---|---|
| [`web-app-architect`](web-app-architect.md) | derives project name, who logs in, auth, tenancy, integrations, constraints |
| [`quality-requirements`](quality-requirements.md) | required parent — app type, data sensitivity, scale, compliance signals |
| [`feature-spec`](feature-spec.md) | required parent — the requirement IDs each feature spec expands |
| [`ux-designer`](ux-designer.md) | required parent — persona and mood (only) |
| [`design-binder`](design-binder.md) | flows/screens to map, the brand the project wants |
| [`poc-developer`](poc-developer.md) | the hero flow worth previewing, the persona |
| [`feature-developer`](feature-developer.md) | the escape hatch: builds deliberately spec-less single-behaviour PRD rows directly |

**Routes back to it:** any downstream skill that surfaces *new product scope* (a new user type, integration, or undeclared sibling feature) routes it here rather than diverging silently.

## Scope boundary

Owns **requirements, not design.** Tech-stack choices, architecture diagrams, schemas, and API specs belong to [`web-app-architect`](web-app-architect.md). If the user makes technical decisions during requirements gathering, it captures them as *constraints* ("must run locally", "must use the free tier of X") rather than designing the system. One concept, one owner.
