# `poc-developer`

> Builds a fast, **throwaway** proof-of-concept on mock data, and captures what it learned as durable hints.

[← Back to the overview](../../README.md) · **Stage:** optional spike — greenfield (after architecture) or brownfield (against a real app)

---

## What it does

Builds a **disposable spike** — the fastest possible clickable prototype that makes an idea tangible, running entirely on **mock data** — to de-risk by showing what something could feel like *before* anyone commits to building it for real. Then it hands the next stage a short list of **hints** so the real work starts closer to reality.

It auto-detects **two modes** from the repo state (and announces which, and why):

- **Greenfield** *(no real app code yet)* — spikes what the **whole app** could feel like, then hints the [scaffolder](web-app-scaffolder.md).
  `PRD → ARCHITECTURE → PoC (glimpse) → scaffold (informed by hints) → features`
- **Brownfield** *(a real app already exists)* — a **"quick response" spike** of one *targeted change*: it reads the real code as context to learn the integration seam, builds the change in `/poc` against a **mocked** version of that seam (never copying/importing/editing real code), then hands off to [`feature-spec`](feature-spec.md) → [`feature-developer`](feature-developer.md).
  `running app → PoC (one change, mocked seam) → feature-spec → feature-developer`

**The seam is load-bearing.** The next stage reads `POC-NOTES.md`, *never* the `/poc` code. A small structured hints doc is cheap to consume; salvaging a throwaway codebase (then ripping its shortcuts out) is the expensive path. In brownfield you *read* real code to learn the seam but never copy it into `/poc`.

## Sample prompts

- "Before we scaffold, throw together a quick PoC so I can see the reminder flow." *(greenfield)*
- "Mock up the UI for the dashboard — just enough to click through." *(greenfield)*
- "Show me what this could look like." / "Spike it." / "Build a quick PoC/demo." *(greenfield)*
- "Spike what a bulk-reminder action would feel like in the current app." *(brownfield)*
- "Quick-response PoC of inline editing on the invoice list before we commit." *(brownfield)*

> The PoC is throwaway and takes shortcuts on purpose. If you actually want the production-ready skeleton (greenfield) or the change built for real (brownfield), it hands off rather than making the PoC real.

## What it produces

| Output | Lifespan | Description |
|---|---|---|
| `/poc/` | **throwaway** | A mock-data, clickable prototype, isolated and never wired into the real build. Deletable. |
| `specs/POC-NOTES.md` | **durable** | The compact hints doc — the interface the next stage consumes. |

`POC-NOTES.md` carries: what the PoC shows · what it validated · build hints (UX patterns, component shapes, observed data shapes, libraries) · **the real integration seam (brownfield — named files/components/hooks/types)** · shortcuts taken (don't carry forward) · pitfalls · divergences from the architecture · open questions · changelog.

## Dependencies & consumers

**Reads (all soft — it's a disposable glimpse):**

| Input | For… |
|---|---|
| `specs/ARCHITECTURE.md` *(if present)* | scope, flows, intended stack/UX direction |
| `specs/PRD.md` *(if present)* | persona, the hero flow worth previewing |
| `specs/DESIGN.md` + `specs/design/` *(if present)* | links `tokens.css` (buildless) so the spike is on-brand |
| **real app code** *(brownfield only, as context)* | to learn the seam — never copied or imported |

**Consumed by — via `POC-NOTES.md` only:**

| Consumer | Mode | Uses the notes for… |
|---|---|---|
| [`web-app-scaffolder`](web-app-scaffolder.md) | greenfield | advisory hints (architecture wins on conflict) |
| [`feature-spec`](feature-spec.md) → [`feature-developer`](feature-developer.md) | brownfield | the validated change + the named real seam to wire into |
| [`ux-designer`](ux-designer.md) | either | crystallizes the *validated interaction patterns* into the durable system |

## Scope boundary

It does **not** scaffold the real foundation ([`web-app-scaffolder`](web-app-scaffolder.md)), build the real feature in brownfield ([`feature-developer`](feature-developer.md)), re-open architecture ([`web-app-architect`](web-app-architect.md)), or author the design system ([`ux-designer`](ux-designer.md)). Everything is mocked — no real data, credentials, external calls, and in brownfield no copying, importing, or editing real source. Distinct from `ux-designer`: the PoC previews *what using the product feels like* (clickable, throwaway); `ux-designer` previews *the visual language* (static, durable).
