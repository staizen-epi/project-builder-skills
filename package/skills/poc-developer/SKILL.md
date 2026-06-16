---
name: poc-developer
description: Builds a fast, throwaway proof-of-concept — a clickable, mock-data prototype that shows what the app could feel like — BEFORE the real foundation is scaffolded, and captures what it learned as hints for the scaffolder. Use this after an architecture exists (or even from a bare idea) when the user wants to "see it working quickly", "spike it", "throw together a prototype", "mock up the UI", "build a quick PoC/demo", or "show me what this could look like" before committing to the real build. It writes disposable code to a /poc folder and durable hints to specs/POC-NOTES.md — it never builds the real foundation (that's web-app-scaffolder) and never re-opens architecture (that's web-app-architect). If specs/DESIGN.md (or a visual reference like screenshots/Figma) exists, it styles the spike to those tokens loosely so the prototype looks on-brand — but it consumes the design system, it never authors it (that's ux-designer). If the user wants a production-ready Phase-0 skeleton rather than a quick glimpse, use web-app-scaffolder instead.
---

# PoC Developer

This skill builds a **disposable spike**: the fastest possible clickable prototype that makes an idea tangible, running entirely on **mock data**. Its job is to de-risk the build by showing what the app could feel like *before* anyone commits to scaffolding the real thing — and then to hand the scaffolder a short list of **hints** so the real foundation starts closer to reality.

It sits in the pipeline **between architecture and scaffolding**:

`PRD → ARCHITECTURE → **PoC (glimpse, mock data)** → scaffold (real foundation, informed by PoC hints) → build features`

Two outputs, with very different lifespans:

- **`/poc/` — throwaway code.** A mock-data prototype, clearly isolated and never wired into the real build. It exists to be *seen and then discarded*. It does not have to respect the architecture.
- **`specs/POC-NOTES.md` — durable hints.** The compact, lasting artifact: what the PoC validated, the UX/flow/component/data shapes that worked, and the pitfalls to avoid. **This is the interface the scaffolder consumes** — not the PoC source.

> **Why the seam is hard, and why it's cheaper.** The scaffolder reads `POC-NOTES.md`, **never the `/poc` code**. A small structured hints doc is cheap to consume and adapt; re-reading and salvaging a throwaway codebase (then ripping its shortcuts back out) is the expensive path in both tokens and risk. Keep the PoC disposable and let the *notes* carry forward.

## What this skill is NOT

- It does **not** scaffold the real foundation (no real auth, no real DB, no tenancy, no CI gate) — that's `web-app-scaffolder`.
- It does **not** decide or re-open architecture — that's `web-app-architect`.
- It does **not** use real data, real credentials, or real external calls. **Everything is mocked.** No secrets, no live APIs, no real database.
- It does **not** aim for production quality. Throwaway code is *allowed* to take shortcuts the real build never would.
- It does **not** author the design system. It *reads* `DESIGN.md` tokens (or a visual reference) to look on-brand and may note in `POC-NOTES.md` what styling felt right — but the durable visual language is `ux-designer`'s to write. A PoC and the design system are complementary halves of "see it before you build it": the spike makes the *flow* clickable, the design system makes the *visual language* real; they share the visual reference and `DESIGN.md` in either order.

If the user actually wants the production-ready skeleton, stop and point them at `web-app-scaffolder`.

## Precondition (soft)

Prefer an architecture, but allow a bare idea — this is a disposable glimpse, so it must not be gated behind heavy upstream docs.

1. **Read `specs/ARCHITECTURE.md` if present** — use it for scope, flows, and the intended stack/UX direction, so the PoC previews something close to the real thing. The PoC may still *diverge* from it for speed (e.g. ignore the real DB and use an in-memory mock) — that's fine and expected.
2. **Read `specs/PRD.md` if present** — use it for the persona, the core flows worth previewing, and what "looks impressive in a demo" actually means for this product.
3. **Read `specs/DESIGN.md` (and its tokens) or any visual reference if present — so the spike looks on-brand.** If `ux-designer` has already written a design system, pull its **tokens** (colour/type/spacing) and component look so the prototype previews the *real* visual language, not placeholder grey — this is what makes "see it before committing" actually convincing. If there's no `DESIGN.md` but the user supplies a **visual reference** (screenshots / a Figma link), follow it loosely for the look. Apply it lightly: this is still a throwaway spike, so approximate the styling rather than building the system for real (that's `ux-designer`'s job). If neither exists, use clean stack defaults and don't agonize over polish.
4. **Neither present →** you can still spike from the conversation. For anything non-trivial, mention that a PRD/architecture would normally come first (point at `product-requirements` / `web-app-architect`), but don't force it — a quick glimpse is a legitimate reason to skip ahead.

## Two modes

**Check for `/poc/` and `specs/POC-NOTES.md`.**

- **Absent → Bootstrap mode.** Pick the slice, build the mock-data prototype, write the hints. (Steps below.)
- **Present → Iterate mode.** Read `POC-NOTES.md` first. Extend or revise the existing spike (add a screen, try an interaction), keep it mock-only, and update `POC-NOTES.md` + its changelog. Don't let the notes drift from what the PoC actually shows.

---

## Bootstrap mode

### Step 0 — Read upstream (if present)

Pull from `ARCHITECTURE.md` / `PRD.md`: the **core user flow** worth previewing, the **persona** the demo speaks to, the **intended stack/UX direction** (so the PoC looks like a preview, not a detour), and any **non-goals** that bound the spike. Extract; don't re-ask what's already settled.

### Step 1 — Pick the slice (ask only what matters)

A PoC that tries to show everything shows nothing. **The one high-impact decision is: which single slice makes the idea click?** If the upstream docs or the conversation don't make this obvious, ask — it's the only thing worth blocking on:

- **The hero flow** — the one path that, when someone clicks through it, makes them *get* the product. Usually the core value loop (e.g. "capture an invoice → see the reminder schedule", not the settings page).

Assume sensible defaults for everything else and record them in the notes:

- **Fidelity** — default to **clickable UI with mock data** (front-end only, in-memory). Go lower (static mockup) or higher (a fake API layer) only if the user asks or the flow demands it.
- **Stack** — default to the architecture's intended front-end stack so the preview looks real; if there's no architecture, use a fast, familiar stack (e.g. Vite + React + Tailwind). Optimize for speed-to-clickable, not correctness.
- **Appearance** — if `DESIGN.md` tokens or a visual reference exist, style the spike to them (loosely) so it looks on-brand; otherwise use clean stack defaults. Don't build the design system here — approximate it.
- **Breadth** — one to three screens covering the hero flow. Resist adding secondary flows.

Bias hard toward the **smallest spike that demonstrates the value**. When unsure, build less.

### Step 2 — Build the spike (mock data, isolated, fast)

- **Isolate it.** All PoC code lives under `/poc/`. Never wire it into a real backend, real auth, or a real database.
- **Mock everything.** Hardcode realistic-looking sample data inline or in a `mockData` module. No network calls to real services; if you need an API shape, fake it in-memory. No secrets, ever.
- **Make it clickable.** The hero flow should actually run in a browser — buttons do something, state updates, the path completes. A static picture is a weaker glimpse than a working click-through; prefer the latter unless the user only wants a mockup.
- **Take shortcuts on purpose.** No validation hardening, no error-handling matrix, no tests, no accessibility pass unless trivial. This is a spike; note the shortcuts rather than fixing them.
- **Leave code hints inline.** Where the PoC discovers something the real build should know — a component that wants to be reused, a data shape that emerged, a third-party widget that worked — drop a short `// HINT(scaffolder): …` comment **and** record it in `POC-NOTES.md`. The comment is a courtesy; the notes file is the contract.

### Step 3 — Run it and show it

Get the spike running and confirm the hero flow clicks through end to end (boot it; click the path). A PoC that doesn't run isn't a PoC. Show the user how to launch it and what to click.

### Step 4 — Write the hints (the durable output)

Write `specs/POC-NOTES.md` using the template below. This is the artifact the scaffolder reads, so make it **actionable and compact** — hints, not a narrative. Capture what was *learned*, not a re-listing of the architecture. Then point the user at `web-app-scaffolder` for the real foundation.

---

## Output: specs/POC-NOTES.md

```markdown
# [Project Name] — PoC Notes

> Hints from a throwaway mock-data spike, for web-app-scaffolder.
> The /poc code is disposable; THESE NOTES are the durable interface.
> Reads: ARCHITECTURE.md (if present). Keep current; see Changelog.

**Status:** [Spike | Iterated] · **Last updated:** [date] · **PoC location:** /poc

## 1. What the PoC shows
The hero flow that was prototyped and what it demonstrates, in one short paragraph.

## 2. Validated
What the spike confirmed works / feels right — UX patterns, flows, interactions
worth keeping in the real build.

## 3. Build hints for the scaffolder
The actionable carry-forward. Be concrete:
- **UX/flow patterns** that landed (e.g. "inline edit beat a modal for the list").
- **Component shapes** worth reusing (named, with their rough responsibility).
- **Data shapes observed** — the mock entities/fields the UI actually needed
  (a hint for the real schema; NOT a schema decision — that's ARCHITECTURE.md).
- **Libraries/widgets that worked** (e.g. a date picker, a chart lib).

## 4. Shortcuts taken (do NOT carry forward)
Where the PoC cheated — mock data, skipped validation, no auth, no error states —
so the scaffolder knows what's intentionally missing and must be built for real.

## 5. Pitfalls & surprises
Anything that fought back or surprised you during the spike — a flow that was
clumsier than expected, a layout that didn't work — so the real build avoids it.

## 6. Divergences from ARCHITECTURE.md
Where the PoC intentionally ignored the architecture for speed (e.g. used
in-memory state instead of the real DB) and why. Flags nothing to change in the
architecture — just notes the gap so no one mistakes the PoC for the real shape.

## 7. Open questions surfaced
Product or design questions the spike raised. Route product-level ones back to
product-requirements; architectural ones to web-app-architect.

## 8. Changelog
Dated entries for each spike/iteration.
```

## Scope boundary (important)

- **Up:** don't redecide product scope (PRD) or architecture (ARCHITECTURE.md). Read them, preview them, and route surfaced questions back to their owner — don't resolve them here.
- **Down:** don't build the real foundation. No real auth, DB, tenancy, CI, or secrets. Hand that to `web-app-scaffolder` via the notes.
- **Sideways:** the `/poc` code is throwaway and stays isolated. The scaffolder reads `POC-NOTES.md`, not the PoC source — keep the hints self-sufficient so the code can be deleted without loss.

## Maintenance / iterate mode rules

- Read `POC-NOTES.md` before touching the spike; keep it the accurate record of what the PoC shows.
- On any change to the spike, update the relevant notes section **and** add a Changelog entry.
- Keep it mock-only. The moment the user wants real data, real auth, or a real DB, that's the scaffolder's job — hand off rather than growing the PoC into the real app.
- If the spike surfaces a product or architecture decision, route it to the owning skill; don't bake it silently into the PoC.

## Examples

**Example 1 — quick glimpse, architecture present**
Architecture exists for a freelancer invoice tracker. User: "Before we scaffold, throw together a quick PoC so I can see the reminder flow." The skill reads the architecture for the intended React stack and the hero flow, builds `/poc` with three mock-data screens (invoice list → invoice detail → reminder schedule preview), all hardcoded sample invoices, no backend. It runs, clicks through, and writes `specs/POC-NOTES.md`: validated the inline reminder-schedule preview, hints a reusable `ReminderTimeline` component and the `{invoice, dueDate, reminderOffsets[]}` shape the UI needed, flags that auth/DB/validation are all mocked, notes it diverged from the architecture by using in-memory state. Then points at `web-app-scaffolder`.

**Example 2 — declining to overbuild**
User: "Build me a working PoC with real login and a real database so I can start using it." The skill declines to make it real: a PoC is a throwaway mock-data glimpse by definition. It offers either a mock-data spike now (clickable, fake auth, sample data) *or*, if the user wants the real thing, to hand off to `web-app-scaffolder` for a genuine Phase-0 foundation — rather than quietly building production auth and a real DB under the PoC banner.

**Example 3 — bare idea, no upstream docs**
User: "I have an idea for a habit tracker — can you spike something so I can see it?" No PRD or architecture. The skill notes that a PRD/architecture would normally come first but a quick glimpse is a fine reason to skip ahead, picks the hero flow (mark a habit done → see the streak), builds a tiny mock-data `/poc`, and writes `POC-NOTES.md` with the validated streak interaction and a note that product/architecture decisions are still open (routed to `product-requirements` / `web-app-architect`).
