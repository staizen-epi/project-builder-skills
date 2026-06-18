# `ux-designer`

> Establishes the **portable, project-agnostic** design system — a self-contained bundle that renders with no build and exports into any project.

[← Back to the overview](../../README.md) · **Stage:** optional design layer (after the PoC, before the scaffolder) · pairs with [`design-binder`](design-binder.md)

---

## What it does

Establishes the **design system** and ships it as a **self-contained, portable bundle**: the durable visual and interaction language — design tokens, the base component inventory and their states, layout/navigation patterns, and an accessibility baseline — packaged so it can be consumed *before anything is scaffolded* and **dropped into another project** to share the same look.

The bundle is deliberately **project-agnostic** — no flows, no screens, no brand instance — which is exactly what makes it inheritable. It reads the PRD only for *persona and mood*; wiring it to a specific project is [`design-binder`](design-binder.md)'s job. This two-skill split is the heart of the design layer: **one exportable artifact (the bundle), one connecting artifact (the binding).**

It can **invent** a system (from persona + mood + good practice) or **infer** one from an existing visual reference — screenshots/mockups or a **Figma** link/file (preferring a structured read of styles/variables/components over eyeballing pixels). Both land in the same bundle.

It works in **two modes**: bootstrap (define the system, realize the bundle, write the doc) and consult & maintain (a token/component change updates the doc, the source-of-truth tokens, any adapters, and the preview *in lockstep*).

It is **gated** — an internal admin panel, a CLI/dev tool, or an app that adopts a complete off-the-shelf kit doesn't earn one, and it will say so rather than manufacturing noise.

## Sample prompts

- "Set up the design system before we scaffold." / "Define the look and feel — colours, typography, spacing, components."
- "Make it look polished and consistent — establish the visual language."
- "Make a portable/exportable design system we can reuse across projects."
- "Match this screenshot." / "Build a design system from this mockup." / "Use our Figma as the source of truth." *(inferred)*

> Distinct from the [PoC](poc-developer.md): the PoC previews *using the product* (clickable, throwaway); this previews *the visual language* as a **static** style guide (durable). They cooperate — the PoC can link the bundle's tokens to look on-brand.

## What it produces

| Output | Description |
|---|---|
| `specs/DESIGN.md` | The contract — tokens, component inventory + states, layout/nav, a11y baseline, theming, provenance. Stays at the `specs/` root for sibling discoverability. |
| `specs/design/tokens.json` | Stack-neutral tokens — the **single source of truth** (W3C-style shape). |
| `specs/design/tokens.css` | The same tokens as CSS custom properties — the **buildless drop-in**. |
| `specs/design/adapters/` | *Optional, gated* — typed/framework-native tokens generated *from* `tokens.json`. |
| `specs/design/preview/index.html` | A **zero-build** static style guide — opens in a browser with no app, build, or server. |
| `specs/design/BUNDLE.md` | The manifest + the "adopt into a new project" recipe. |

**Portability invariant:** nothing in the bundle references a path, import, or asset *outside* `specs/design/` (and `DESIGN.md`). Copy the folder into any repo and it renders standalone.

## Dependencies & consumers

**Reads:**

| Input | For… |
|---|---|
| `specs/PRD.md` | **required parent** — persona and mood *only* (no flows/screens) |
| `specs/POC-NOTES.md` *(if present)* | validated interaction patterns to crystallize |
| `specs/ARCHITECTURE.md` *(if present)* | the front-end stack (so adapters target it) |
| a visual reference *(if provided)* | screenshots / Figma — the system is *inferred* from it |

**Consumed by:**

| Consumer | Uses the bundle for… |
|---|---|
| [`design-binder`](design-binder.md) | the palette of components/tokens to **map** and **theme** (writes the project theme under `specs/design/themes/`) |
| [`web-app-scaffolder`](web-app-scaffolder.md) | adopts the bundle (tokens + base components) as the foundation's visual language |
| [`feature-developer`](feature-developer.md) | every slice conforms to its tokens/components/a11y baseline |
| [`poc-developer`](poc-developer.md) | links `tokens.css` directly (buildless) so the spike is on-brand |

**Routes back to it:** any *missing component or token* (from `design-binder`, `feature-developer`, or the scaffolder) routes here to be added to the system + preview — never invented inline.

## Scope boundary

Owns the **visual & interaction language** only — not product scope ([`product-requirements`](product-requirements.md)), feature behaviour ([`feature-spec`](feature-spec.md)), the stack ([`web-app-architect`](web-app-architect.md), which wins on *how* components are built), the clickable product-feel spike ([`poc-developer`](poc-developer.md)), or the project wiring ([`design-binder`](design-binder.md)). Keeping project specifics *out* of the bundle is exactly what makes it inheritable.
