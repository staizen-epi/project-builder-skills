---
name: poc-developer
description: Builds a fast, throwaway proof-of-concept — a clickable, mock-data prototype — and captures what it learned as durable hints. It works in two auto-detected modes. GREENFIELD (no real app code yet) spikes what the whole app could feel like before scaffolding, and hints the scaffolder. BROWNFIELD (a real app already exists) is a "quick response" spike that proves one targeted change against the current app: it reads the real code as context to learn the integration seam, then builds the change in /poc against a MOCKED version of that seam (never copying or importing real code), and hands off to feature-spec → feature-developer. Use it when the user wants to "see it working quickly", "spike it", "mock up the UI", "build a quick PoC/demo", "show me what this could look like", or — against an existing app — "spike this change", "what would X feel like in the real app", "quick response PoC of X". It writes disposable code to /poc and durable hints to specs/POC-NOTES.md. It never builds the real foundation (web-app-scaffolder), never re-opens architecture (web-app-architect), never authors the design system (ux-designer), and never wires the spike into the real app. If the user wants a production-ready skeleton or the real feature built, hand off rather than making the PoC real.
---

# PoC Developer

This skill builds a **disposable spike**: the fastest possible clickable prototype that makes an idea tangible, running entirely on **mock data**. Its job is to de-risk by showing what something could feel like *before* anyone commits to building it for real — and then to hand the next stage a short list of **hints** so the real work starts closer to reality.

It runs in **two auto-detected modes** depending on whether a real app already exists:

- **Greenfield** — *no real app code yet.* Spike what the **whole app** could feel like, then hint the scaffolder.
  `PRD → ARCHITECTURE → **PoC (glimpse, mock data)** → scaffold (informed by PoC hints) → build features`
- **Brownfield** — *a real app already exists.* A **"quick response" spike**: prove **one targeted change** against the current app, then hand off to the build loop.
  `… running app → **PoC (quick-response spike of one change, mock data)** → feature-spec → feature-developer (build it for real)`

In **both** modes there are two outputs with very different lifespans:

- **`/poc/` — throwaway code.** A mock-data prototype, clearly isolated, **never wired into the real build**. It exists to be *seen and then discarded*. It does not have to respect the architecture.
- **`specs/POC-NOTES.md` — durable hints.** The compact, lasting artifact: what the PoC validated and the carry-forward for the next stage. **This is the interface the next stage consumes** — not the PoC source.

> **Why the seam is hard, and why it's cheaper.** The next stage reads `POC-NOTES.md`, **never the `/poc` code**. A small structured hints doc is cheap to consume and adapt; re-reading and salvaging a throwaway codebase (then ripping its shortcuts back out) is the expensive path in tokens and risk. In brownfield this is the load-bearing rule: you **read** the real code to *learn the seam*, but you **never copy it into `/poc`** and never import the real app from the spike — the notes carry the real seam names forward, the code stays disposable.

## What this skill is NOT

- It does **not** scaffold the real foundation (no real auth, no real DB, no tenancy, no CI gate) — that's `web-app-scaffolder`.
- It does **not** build the real feature in brownfield — that's `feature-developer` (after `feature-spec`). The PoC proves the *idea* of the change; it does not ship it.
- It does **not** decide or re-open architecture — that's `web-app-architect`.
- It does **not** author the design system. It *reads* `DESIGN.md` tokens (or a visual reference) to look on-brand; the durable visual language is `ux-designer`'s.
- It does **not** use real data, real credentials, or real external calls, and in brownfield it does **not** copy, import, or modify the real app. **Everything is mocked.** No secrets, no live APIs, no real database, no edits to real source.
- It does **not** aim for production quality. Throwaway code is *allowed* to take shortcuts the real build never would.

If the user actually wants the production-ready skeleton (greenfield) or the change built for real (brownfield), stop and hand off rather than making the PoC real.

## Precondition (soft)

This is a disposable glimpse, so it must not be gated behind heavy upstream docs.

1. **Read `specs/ARCHITECTURE.md` if present** — for scope, flows, and the intended stack/UX direction, so the spike previews something close to the real thing. The PoC may still *diverge* for speed (e.g. an in-memory mock instead of the real DB) — that's fine and expected.
2. **Read `specs/PRD.md` if present** — for the persona, the flows worth previewing, and what "looks impressive in a demo" means here.
3. **Read `specs/DESIGN.md` and consume its portable bundle (`specs/design/`) if present** — so the spike looks on-brand. The bundle is **buildless**: link `specs/design/tokens.css` (or import `tokens.json`) directly — no scaffolding, build step, or app needed — and the spike is instantly on-brand instead of placeholder grey. If a `specs/DESIGN-BINDING.md` + project theme (`specs/design/themes/<project>.css`) exist, also link the theme so the spike matches *this project's* brand, not the bundle's neutral default. (No bundle but a visual reference? Pull colour/type/spacing from that instead.) Apply it loosely — this is still a throwaway spike; don't build the system for real (that's `ux-designer`'s) or the binding (that's `design-binder`'s).
4. **Neither present →** you can still spike from the conversation. For anything non-trivial, mention that a PRD/architecture would normally come first (point at `product-requirements` / `web-app-architect`), but don't force it.

## Mode detection (do this first, then announce)

Before building, decide which mode you're in from the **repo state**, and say which mode and why in one line.

- **Brownfield** if a real, scaffolded app is present: a real source tree (e.g. `src/`, `app/`) with actual feature code, a package manifest with real dependencies, and signs the foundation was stood up (a built `ARCHITECTURE.md` Phase-0, CI config, etc.). The user is asking to spike a *change to something that exists*.
- **Greenfield** if there's no real app code yet — only `specs/`, maybe a prior `/poc/`, no real source tree or only a bare manifest. The user is asking to spike *the app itself* before it's built.
- **Ambiguous?** Treat a thin or template-only repo as greenfield; only treat it as brownfield when there is genuine **feature** code to plug into. A freshly-scaffolded foundation with *no features yet* counts as greenfield for this purpose — the spike is still "what should this feel like," hinting the features to come, not a change to existing behaviour. If you truly can't tell, ask the one question — greenfield-glimpse or brownfield-quick-response — and proceed.

State it plainly, e.g. *"Real app detected — running in brownfield 'quick response' mode: I'll spike just the {X} change against a mock of its real seam, then hand off to feature-spec/feature-developer."*

## Two run-states (per mode)

**Check for `/poc/` and `specs/POC-NOTES.md`.**

- **Absent → Bootstrap.** Pick the slice — **interview enough to nail *which* slice and what "it works" looks like** before building (a spike aimed at the wrong slice wastes the whole effort) — build the mock-data prototype, write the hints. The interview is deliberately narrow here: the slice/change is the one thing worth blocking on; everything else is defaulted (the PoC is throwaway, so an imperfect default is cheap to redo).
- **Present → Iterate.** Read `POC-NOTES.md` first. Extend or revise the spike (add a screen, try an interaction), keep it mock-only, update `POC-NOTES.md` + its changelog. Don't let the notes drift from what the PoC actually shows. (A brownfield iterate may add a second targeted change; keep each disposable.)

---

## Bootstrap — Greenfield (no real app yet)

### G0 — Read upstream (if present)
Pull from `ARCHITECTURE.md` / `PRD.md`: the **core user flow** worth previewing, the **persona**, the **intended stack/UX direction**, and any **non-goals** bounding the spike. Extract; don't re-ask what's settled.

### G1 — Pick the slice (interview on the one thing that matters)
A PoC that tries to show everything shows nothing. **The one high-impact decision: which single slice makes the idea click?** If upstream/conversation don't make it obvious, **interview** — it's the only thing worth blocking on, so draw it out: which moment of the product is the user actually trying to feel, and what would make them say "yes, that's it"? Usually the **hero flow**: the core value loop (e.g. "capture an invoice → see the reminder schedule", not the settings page). Pin that before building; default everything else.

Default everything else and record it in the notes:
- **Fidelity** — clickable UI with mock data (front-end only, in-memory). Lower (static mockup) or higher (a fake API layer) only if asked or the flow demands it.
- **Stack** — the architecture's intended front-end stack; if none, a fast familiar stack (e.g. Vite + React + Tailwind).
- **Appearance** — if the design bundle exists, link `specs/design/tokens.css` (buildless, no scaffolding needed) so the spike is on-brand; else style to a visual reference loosely, or clean defaults.
- **Breadth** — one to three screens covering the hero flow. Resist secondary flows.

Bias hard toward the **smallest spike that demonstrates the value**.

### G2 — Build the spike (mock data, isolated, fast)
- **Isolate it.** All PoC code under `/poc/`. Never wire it into a real backend, auth, or DB.
- **Mock everything.** Hardcode realistic sample data inline or in a `mockData` module. No real network calls; fake API shapes in-memory. No secrets.
- **Make it clickable.** The hero flow runs in a browser — buttons do something, state updates, the path completes.
- **Take shortcuts on purpose.** No validation hardening, no error matrix, no tests, no a11y pass unless trivial. Note shortcuts rather than fixing them.
- **Leave code hints inline.** Drop `// HINT(scaffolder): …` where the PoC discovers something the real build should know, **and** record it in `POC-NOTES.md`.

### G3 — Run it and show it
Boot it; confirm the hero flow clicks through end to end. Show the user how to launch it and what to click.

### G4 — Write the hints
Write `specs/POC-NOTES.md` (template below) in **greenfield framing** — reader is `web-app-scaffolder`. Then point the user there.

---

## Bootstrap — Brownfield ("quick response" spike against the real app)

The user wants to see one **targeted change** working without committing to building it for real yet. Prove it in `/poc` against a **mock of the real seam** — informed by the real code, but never copying or touching it.

### B0 — Read upstream + locate the seam
- Read `specs/ARCHITECTURE.md` / `PRD.md` / `DESIGN.md` as in the precondition.
- **Read the relevant *real* code as context.** Find the surface the change plugs into and learn its **real names and shapes**: the component(s) it sits next to, the data/types it consumes, the function/hook/endpoint it would call, the routing or layout slot it lands in. You are reading to *understand the seam*, not to lift code.
- Identify the **integration seam**: the single boundary where the new change meets the existing app (a prop, a hook, an API call, a route). This is the thing you will mock, and the thing the notes will name.

### B1 — Pick the change (interview on the one thing that matters)
**The one high-impact decision: what single change is being proven, and what does "it works" look like?** If it isn't already crisp, **interview** until it is — what behaviour changes, against which seam, and what observable outcome counts as proof — because a brownfield spike aimed at a fuzzy "it works" proves nothing and hands off a fuzzy seam. Keep it to one targeted behaviour against one seam. If the change is sprawling, narrow it to the slice that makes the idea click — and note the rest as out of scope.

Default everything else:
- **Strategy — read-as-context, mock the seam (default).** Build the change in `/poc` against a mocked version of the real seam (mock the hook/endpoint/props using the *real names and shapes* you learned). Do **not** copy real components into `/poc`; re-create only the thin surface needed to host the change.
- **Fallback — thin fake shell.** If the seam is too entangled to mock cleanly (deeply coupled state, no clear boundary), build a thin fake facade of just that surface and make **only the change** interactive. Note in the seam doc *why* you fell back.
- **Never — copy real code into `/poc`.** Declining this is correct (it breaks the disposable handoff and drifts instantly). If the user explicitly insists on running against real code, that's no longer a PoC — hand off to `feature-developer` (after `feature-spec`) instead.
- **Stack/appearance** — match the real app's stack and consume the design bundle's tokens (`specs/design/tokens.css`) so the spike looks slotted-in.
- **Breadth** — the one change and the minimum surrounding mock to make it clickable.

### B2 — Build the spike (mock the seam, isolated, fast)
- **Isolate it.** All PoC code under `/poc/`. **Never import the real app, never edit real source, never call real services.**
- **Mock the seam with real shapes.** Stand up an in-memory mock of the exact boundary (same prop/type/endpoint names you read), seeded with realistic sample data — so the change behaves as if plugged in.
- **Make the change clickable.** The targeted behaviour actually runs: the new interaction completes against the mocked seam.
- **Take shortcuts on purpose**, and **leave `// HINT(feature-developer): …` comments** at the seam (the real file/component/type it should wire into), mirrored into `POC-NOTES.md`.

### B3 — Run it and show it
Boot the spike; confirm the targeted change clicks through against the mock. Show the user how to launch it and what to click.

### B4 — Write the hints
Write `specs/POC-NOTES.md` (template below) in **brownfield framing** — reader is **`feature-spec` → `feature-developer`**. The carry-forward is the validated behaviour **and the real seam it plugs into** (named files/components/types). Then point the user at `feature-spec` to capture the change as a spec, which `feature-developer` then builds for real.

---

## Output: specs/POC-NOTES.md

The header and §3/§7 adapt to the mode. Keep it **actionable and compact** — hints, not narrative.

```markdown
# [Project Name] — PoC Notes

> Hints from a throwaway mock-data spike.
> The /poc code is disposable; THESE NOTES are the durable interface.
> Mode: [Greenfield → web-app-scaffolder | Brownfield → feature-spec → feature-developer]
> Keep current; see Changelog.

**Status:** [Spike | Iterated] · **Mode:** [Greenfield | Brownfield] · **Last updated:** [date] · **PoC location:** /poc

## 1. What the PoC shows
Greenfield: the hero flow prototyped and what it demonstrates.
Brownfield: the single targeted change proven, and what "it works" looked like.

## 2. Validated
What the spike confirmed works / feels right — UX patterns, flows, interactions
worth keeping in the real build.

## 3. Build hints for the next stage
The actionable carry-forward. Be concrete:
- **UX/flow patterns** that landed (e.g. "inline edit beat a modal").
- **Component shapes** worth reusing (named, with rough responsibility).
- **Data shapes observed** — mock entities/fields the UI actually needed
  (a hint for the schema; NOT a schema decision — that's ARCHITECTURE.md).
- **Libraries/widgets that worked.**
- **[Brownfield] The real integration seam** — the exact existing files /
  components / hooks / endpoints / types the change must wire into, by name,
  so feature-developer plugs it in without rediscovering. Note the chosen
  strategy (mocked seam vs thin-shell fallback) and why.

## 4. Shortcuts taken (do NOT carry forward)
Where the PoC cheated — mock data, skipped validation, no auth, no error states,
mocked seam — so the next stage knows what's intentionally missing.

## 5. Pitfalls & surprises
Anything that fought back or surprised you during the spike.

## 6. Divergences from ARCHITECTURE.md
Where the PoC intentionally ignored the architecture for speed and why.
Flags nothing to change in the architecture — just notes the gap.

## 7. Open questions surfaced
Product or design questions the spike raised. Route product-level ones to
product-requirements; architectural ones to web-app-architect; behaviour-spec
gaps (brownfield) to feature-spec.

## 8. Changelog
Dated entries for each spike/iteration.
```

## Scope boundary (important)

- **Up:** don't redecide product scope (PRD) or architecture (ARCHITECTURE.md). Read, preview, route surfaced questions back to their owner.
- **Down:** don't build the real thing. Greenfield: no real auth/DB/tenancy/CI/secrets — hand to `web-app-scaffolder` via the notes. Brownfield: don't build the real feature and don't touch real source — hand to `feature-spec` → `feature-developer` via the notes.
- **Sideways:** `/poc` is throwaway and stays isolated. The next stage reads `POC-NOTES.md`, not the PoC source. In brownfield, read real code for *context* only — never copy, import, or modify it.

## Maintenance / iterate mode rules

- Read `POC-NOTES.md` before touching the spike; keep it the accurate record of what the PoC shows, including its mode.
- On any change, update the relevant notes section **and** add a Changelog entry.
- Keep it mock-only. The moment the user wants real data/auth/DB (greenfield) or the change built for real (brownfield), hand off — don't grow the PoC into the real app.
- If the spike surfaces a product/architecture/behaviour decision, route it to the owning skill; don't bake it silently into the PoC.

## Examples

**Example 1 — greenfield glimpse, architecture present.** Architecture exists for a freelancer invoice tracker. User: "Before we scaffold, throw together a quick PoC so I can see the reminder flow." No real app code → **greenfield**. The skill reads the architecture for the React stack and hero flow, builds `/poc` with three mock-data screens (invoice list → detail → reminder schedule), all hardcoded, no backend. It runs, clicks through, and writes `POC-NOTES.md` (mode: greenfield → scaffolder): validated the inline reminder-schedule preview, hints a reusable `ReminderTimeline` and the `{invoice, dueDate, reminderOffsets[]}` shape, flags auth/DB/validation as mocked. Points at `web-app-scaffolder`.

**Example 2 — brownfield "quick response", real app present.** A scaffolded invoice app already runs. User: "Spike what a bulk-reminder action on the invoice list would feel like in the real app." Real source tree → **brownfield**. The skill reads the real `InvoiceList` component and the `useInvoices` hook to learn the seam (the list's row shape, the `sendReminder(invoiceId)` call), then builds `/poc` with a mocked `useInvoices` (same shapes, sample data) and a working bulk-select → "Send reminders" interaction — **without importing or editing the real app**. It runs, clicks through, and writes `POC-NOTES.md` (mode: brownfield → feature-spec/feature-developer): validated bulk-select UX, names the real seam (`InvoiceList`, `useInvoices`, `sendReminder`) the change must wire into, notes the seam was mocked. Points at `feature-spec` to capture the change, then `feature-developer` to build it.

**Example 3 — declining to copy real code.** Brownfield app present. User: "Just copy the real checkout flow into the PoC so the demo is the actual thing." The skill declines: copying real code into `/poc` breaks the disposable handoff, drifts instantly, and drags in the dependency graph. It offers either a proper quick-response spike (mock the checkout seam with real shapes, make only the new bit interactive) *or*, if they want the actual thing wired in, to hand off to `feature-spec` → `feature-developer` to build it for real — rather than smuggling production code under the PoC banner.

**Example 4 — bare idea, no upstream docs.** User: "I have an idea for a habit tracker — can you spike something so I can see it?" No PRD/architecture, no real app → **greenfield**. The skill notes a PRD/architecture would normally come first but a quick glimpse is a fine reason to skip ahead, picks the hero flow (mark a habit done → see the streak), builds a tiny mock-data `/poc`, and writes `POC-NOTES.md` (greenfield) with the validated streak interaction and product/architecture questions routed to `product-requirements` / `web-app-architect`.
