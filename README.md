# Spec-Driven Build Skills for Claude Code

A chained set of Claude Code skills that take a web app from a rough idea to a running, verified foundation — **spec-first**. Each skill writes a file; the next one reads it. You never repeat yourself, and the architecture you get is sized to what the app actually needs instead of a one-size-fits-all template.

```
  IDEA
   │
   ▼
┌─────────────────────┐   writes    specs/PRD.md            ← what & why
│ product-requirements│ ──────────▶ (the whole product)
└─────────────────────┘
   │
   ▼
┌─────────────────────┐   writes    specs/ARCHITECTURE.md   ← how it's built
│  web-app-architect  │ ──────────▶ (stack, gates, phases)
└─────────────────────┘
   │
   ▼
┌─────────────────────┐   writes    /poc/ + POC-NOTES.md    ← a quick glimpse
│    poc-developer    │ ──────────▶ (throwaway, mock data)     (optional)
└─────────────────────┘   the spike is disposable; the notes
   │                      become hints for the scaffolder
   ▼
┌─────────────────────┐   writes    specs/DESIGN.md         ← the look & feel
│     ux-designer     │ ──────────▶ + tokens + style guide     (optional)
└─────────────────────┘   the design system the foundation
   │                      adopts and every feature conforms to
   ▼
┌─────────────────────┐   writes    your codebase           ← Phase-0 foundation, proven
│  web-app-scaffolder │ ──────────▶ (scaffolded + verified)
└─────────────────────┘   reads POC-NOTES.md as hints,
   │                      adopts DESIGN.md tokens + components
   │
   ▼
 ┌── BUILD FEATURES (loop, one feature at a time) ────────────────┐
 │                                                                │
 │  ┌──────────────────┐  writes  specs/features/*.md  ← how a    │
 │  │   feature-spec   │ ───────▶ (deep, per-feature)    feature  │
 │  └──────────────────┘                                 behaves  │
 │           │                                                    │
 │           ▼                                                    │
 │  ┌───────────────────┐  writes  code + tests         ← builds  │
 │  │ feature-developer │ ──────▶ (+ specs/REUSE.md)      to spec │
 │  └───────────────────┘  reuse-first, within ARCHITECTURE       │
 │                                                                │
 └────────────────────────────────────────────────────────────────┘
```

## The seven skills

| Skill | Stage | Reads | Writes |
|---|---|---|---|
| `product-requirements` | Define the product | the conversation | `specs/PRD.md` |
| `web-app-architect` | Design the build | `specs/PRD.md` | `specs/ARCHITECTURE.md` |
| `poc-developer` | Spike a quick glimpse *(optional)* | `specs/ARCHITECTURE.md` | `/poc/` (throwaway) + `specs/POC-NOTES.md` |
| `ux-designer` | Design the look & feel *(optional)* | `specs/PRD.md` + `specs/POC-NOTES.md` + `specs/ARCHITECTURE.md` | `specs/DESIGN.md` + tokens + style guide |
| `web-app-scaffolder` | Stand up the foundation | `specs/ARCHITECTURE.md` + `specs/POC-NOTES.md` + `specs/DESIGN.md` | your codebase |
| `feature-spec` | Detail one feature | `specs/PRD.md` + `specs/ARCHITECTURE.md` | `specs/features/<feature>.md` |
| `feature-developer` | Build one feature | `specs/features/<feature>.md` + `specs/ARCHITECTURE.md` + `specs/REUSE.md` + `specs/DESIGN.md` | code + tests + `specs/REUSE.md` |

They run in that order: **PRD → architecture → (optional PoC) → (optional design system) → scaffold**, then a per-feature **build loop**. The PoC is an optional, disposable spike — a clickable mock-data prototype to see what the app could feel like before committing — and it leaves `specs/POC-NOTES.md` as hints the scaffolder reads so the real foundation starts closer to reality. The design system is also optional: `ux-designer` establishes the durable visual & interaction language (`specs/DESIGN.md` + token files + a static style-guide preview) that the scaffolder adopts and every feature conforms to. The build loop is the `feature-spec` → `feature-developer` pair, run once per feature: the writer details how a feature behaves (and keeps the PRD's feature index pointing at it), the developer implements that behaviour as a tested vertical slice — reusing code from a registry it maintains (`specs/REUSE.md`) so it builds from pre-conceived context, staying within the architecture's rules and conforming to the design system if one exists. Everything durable they produce lives together under one `specs/` folder.

## Installation

Each skill is a folder containing a `SKILL.md`. Drop them into one of Claude Code's skill locations:

```
.claude/skills/                 # project-level: committed with the repo, shared with your team
├── product-requirements/SKILL.md
├── web-app-architect/SKILL.md
├── poc-developer/SKILL.md
├── ux-designer/SKILL.md
├── web-app-scaffolder/SKILL.md
├── feature-spec/SKILL.md
└── feature-developer/SKILL.md
```

- **Project** (`.claude/skills/` in your repo) — travels with the codebase; good when you want the team to share them.
- **Personal** (`~/.claude/skills/`) — available across all your projects; good for general reuse. The two locations stack, so you can use both.

If the `.claude/skills/` directory didn't exist when your session started, restart Claude Code once so it gets watched. Edits to an existing `SKILL.md` are picked up live.

## How it works

You don't invoke these skills manually. Claude Code reads each skill's description and routes to the right one based on what you ask and which spec files already exist. So you just talk naturally:

- Which skill fires is decided by **intent + file state** — e.g. `web-app-architect` activates when you ask about building/structure *and* there's no `specs/ARCHITECTURE.md` yet; once it exists, the same skill switches to maintaining it.
- Each skill **reads the previous stage's output**, so you don't restate the product when you move to architecture, or restate the architecture when you scaffold.
- Each stage **pauses for your approval** before the next one runs. Review the `specs/` file, correct it, then continue.

You can also jump in anywhere: if you already have a PRD, start at architecture; if you already have an architecture, start at scaffold.

---

## Using it, stage by stage

### 1. Define the product → `specs/PRD.md`

Start here when you have an idea but no `specs/PRD.md`. The skill asks a short set of high-impact questions (what & who, problem & success, must-haves, non-goals, constraints), then writes the PRD.

**Example prompts:**
- "I want to build a tool that lets freelancers track invoices and send reminders. Help me write a PRD."
- "Let's spec out a new product: a reading tracker that syncs my Goodreads shelf."
- "Define the requirements for an internal dashboard my team will use to track weekly OKRs."

> It will ask a few questions before writing — answer them, or reply "defaults" to let it assume sensible ones and note them.

### 2. Design the build → `specs/ARCHITECTURE.md`

Run this when the PRD is approved. It reads the PRD first — pulling the project name, who logs in, integrations, and constraints — so it only asks what the PRD leaves open, then decides the stack and which architectural patterns the app actually needs (and switches off the ones it doesn't).

**Example prompts:**
- "The PRD's done — design the architecture."
- "How should we build this? Set up the ARCHITECTURE.md."
- "Decide the tech stack and structure for this app."

> Because it reads the PRD, a single-user local tool won't get multi-tenancy bolted on, and a SaaS will get tenant isolation, RBAC, and the rest — automatically.

### 2½. (Optional) Spike a quick glimpse → `/poc/` + `specs/POC-NOTES.md`

Run this when you want to *see* the idea working before committing to the real build. It throws together a clickable, **mock-data** prototype of the one flow that makes the product click — no real auth, no real database, no live APIs — so you get a feel for it fast. The spike lands in a disposable `/poc/` folder; what lasts is `specs/POC-NOTES.md`, a short list of hints (what worked, component shapes, observed data shapes, pitfalls) that the scaffolder reads next.

**Example prompts:**
- "Before we scaffold, throw together a quick PoC so I can see the reminder flow."
- "Mock up the UI for the dashboard — just enough to click through."
- "Spike a prototype of the core flow with fake data."

> It's optional and disposable. The PoC code is throwaway and takes shortcuts on purpose; the scaffolder reads only the *notes*, never the spike code, and the architecture always wins on any conflict. Skip it entirely if you'd rather go straight to the real foundation.

### 2¾. (Optional) Design the look & feel → `specs/DESIGN.md` + tokens + style guide

Run this when the product has a real user-facing surface and you want one coherent visual language instead of each feature re-deciding colour, spacing, and component shape on its own. It reads the PRD (persona and flows), the PoC notes if present (the interaction patterns that were validated), and the architecture (the front-end stack), then establishes a **design system**: semantic design tokens (colour/type/spacing/radius/motion), a base component inventory with their states, layout/navigation patterns, and an accessibility baseline. It writes `specs/DESIGN.md` (the durable contract) plus the **token files** and one **static style-guide preview page** so you can *see* the system, not just read it.

**Example prompts:**
- "Set up the design system before we scaffold."
- "Define the look and feel — colours, typography, spacing, components."
- "Make it look polished and consistent — establish the visual language."

> It's optional and gated: an internal admin panel, a dev tool, or an app that adopts an off-the-shelf component kit as-is doesn't need one, and the skill will say so rather than manufacturing noise. The preview is a *static* style guide (a gallery rendered once) — distinct from the PoC, which previews what *using the product* feels like. The architecture wins on *how* components are built; the design system governs *how they look*.

### 3. Stand up the foundation → your codebase

Run this once the architecture is approved. It reads `specs/ARCHITECTURE.md` (plus `specs/POC-NOTES.md` if a PoC was spiked, as advisory hints, and `specs/DESIGN.md` if a design system was established, adopting its tokens and base components), scaffolds exactly the foundation the doc calls for (repo, stack, auth, the gated pieces, CI gate), and proves it against the architecture's own Phase-0 acceptance criteria. It builds the foundation only — not features.

**Example prompts:**
- "Scaffold the project and stand up Phase 0."
- "Bootstrap the repo from the architecture."
- "Set up the foundation and prove it works."

> It stops once the foundation boots, the CI gate is green, and the Phase-0 checks pass — then hands back to you to build features.

### 4. Build features, one at a time → spec then code *(the loop)*

With the foundation standing, you build features one by one, as a pair of steps per feature: **spec it, then build it.** Simple features stay as rows in the PRD; only complex ones earn their own spec. This is the repeating loop you'll spend most of the build in: **spec a feature → build it → repeat.**

**Spec it** — `feature-spec` writes `specs/features/<feature>.md`:
- "Spec out the invoice-reminder feature in detail — all the states and edge cases."
- "Flesh out the Goodreads sync module: what happens on conflicts, failures, first run?"

> It reads both the PRD and the architecture, traces back to the PRD requirement it expands (e.g. FR-03 → FR-03.1, FR-03.2…), and adds itself to the PRD's feature index.

**Build it** — `feature-developer` implements that spec as a tested vertical slice:
- "Build the invoice-reminder feature from its spec."
- "Implement the Goodreads sync module."

> It reads the feature spec (what to build) and the architecture (how to build — honouring its invariants, gates, and conventions), conforms to the design system if `specs/DESIGN.md` exists (its tokens, components, and a11y baseline), and proves the slice against the spec's own acceptance criteria. It also keeps `specs/REUSE.md`, a registry of reusable code it reads *before* building and updates *after* — so it reuses what exists instead of rediscovering the codebase each feature, and gets faster as the project grows. If the spec is missing a behaviour it needs, it routes back to `feature-spec`; if it needs a component or token the design system doesn't define, it routes back to `ux-designer` — rather than improvising either.

---

## End-to-end example

A short walkthrough of one project moving through the stages:

1. **You:** "I want to build a tool for freelancers to track invoices and send payment reminders. Write a PRD."
   **→** A few questions, then `specs/PRD.md` (persona, goals/non-goals, functional requirements, the "sends reminders" + "connects to a payment provider" constraints).

2. **You:** "Now design the architecture."
   **→** It reads the PRD, sees single-user + a payment integration + scheduled reminders, and writes `specs/ARCHITECTURE.md` with a job/scheduler layer and webhook verification gated **on**, multi-tenancy gated **off**, plus a phased build plan.

3. **You:** "Before we build it for real, spike a quick PoC of the reminder flow." *(optional)*
   **→** It builds a throwaway `/poc/` with three mock-data screens you can click through, then writes `specs/POC-NOTES.md` — validated patterns, a reusable `ReminderTimeline` component, the data shape the UI needed, and a note that auth/DB are mocked. You get a feel for it without committing to anything.

4. **You:** "Set up the design system before we scaffold." *(optional)*
   **→** It reads the PRD (persona, flows) and the PoC notes (the inline timeline that landed), and writes `specs/DESIGN.md` plus `tokens.ts` and a static style-guide page: semantic colour/type/spacing tokens, a component inventory with states, a sidebar shell, and a WCAG AA baseline. You open the style guide and *see* the system before any feature is built.

5. **You:** "Scaffold Phase 0."
   **→** It reads the architecture (and the PoC notes as hints, and adopts the design tokens + base components) and stands up the repo, stack, auth, the scheduler, the CI gate — then runs the Phase-0 checks until green and stops. The `/poc/` code is ignored; only the notes carry forward.

6. **You:** "Spec out the reminder feature — all the states and timing rules."
   **→** `specs/features/payment-reminders.md`, expanding the PRD's reminder requirement into flows, schedule rules, and failure behaviour; the PRD's feature index links to it.

7. **You:** "Now build the reminder feature."
   **→** `feature-developer` reads that spec (what) and the architecture (how), checks `specs/REUSE.md` and reuses the `ReminderTimeline` the PoC notes seeded, builds the slice with the design system's tokens and components — scheduler hook-up, all the states, tests — within the architecture's gates, proves it against the spec's acceptance criteria, and registers the new reusable bits it created. Then you loop back to spec the next feature.

From there you stay in the build loop on a proven foundation: spec a feature, build it, repeat — the reuse registry making each pass a little faster than the last.

## What you end up with

```
your-project/
├── specs/
│   ├── PRD.md                       # the product: what & why
│   ├── features/
│   │   └── payment-reminders.md     # one feature, in depth
│   ├── ARCHITECTURE.md              # the build: how
│   ├── POC-NOTES.md                 # hints from the spike (if a PoC was run)
│   ├── DESIGN.md                    # the design system (if one was established)
│   └── REUSE.md                     # registry of reusable code (grows as you build)
├── poc/ … (throwaway mock-data spike — deletable)
└── src/ … (scaffolded foundation + features, built to spec)
```

## Maintaining the specs

The skills don't just write once — each switches to a **maintain mode** when its file already exists:

- "Add a requirement to the PRD: let users export invoices to CSV." → updates `specs/PRD.md` and its changelog.
- "We're adding a paid SMS API — update the architecture." → revisits the affected gates in `specs/ARCHITECTURE.md`.
- "The reminder timing changed to 3 and 7 days before due." → updates the feature spec and keeps the PRD index link accurate.
- "Change how reminders are sent." → the spec changes first (behaviour), then `feature-developer` updates the code to match — never the other way round.

Keep the specs current and they stay the source of truth the whole build derives from.

## Design principles (why it behaves this way)

- **Gating, not templates.** Each skill includes only what the app needs. Patterns (multi-tenancy, queues, caching, a feature's own file) are switched on by explicit conditions and left off otherwise — it's cheaper to add later than to rip out.
- **One concept, one owner.** Product scope lives in the PRD, feature behaviour in feature specs, technical design in the architecture. No skill redoes another's job; they link instead of duplicating.
- **Ask only what matters, then assume.** Each stage asks a few high-impact questions and defaults the rest, recording assumptions rather than blocking on a long form.
- **Prove it.** The foundation isn't "done" because files exist — it's done when it boots and passes the architecture's acceptance criteria.
