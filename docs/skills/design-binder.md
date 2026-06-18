# `design-binder`

> The **connecting artifact** — hard-wires the portable design bundle to *this* project's PRD and architecture.

[← Back to the overview](../../README.md) · **Stage:** optional design layer (after [`ux-designer`](ux-designer.md), before the scaffolder)

---

## What it does

Takes the **portable, project-agnostic** design bundle that [`ux-designer`](ux-designer.md) ships (`specs/DESIGN.md` + `specs/design/`) and **hard-wires it to *this* project** by reading the PRD and the architecture. The bundle is the *language*, inheritable into any project; the binding is *how this project speaks it*.

This is the second half of the deliberate, load-bearing split:

- **[`ux-designer`](ux-designer.md) → the portable artifact.** No project specifics — that's what makes it exportable.
- **`design-binder` → the connecting artifact.** *All* the project specifics: which components/tokens realize which screen, this project's brand/theme instance, and which stack adapter the architecture consumes.

Keeping these in two files means one bundle can be inherited by many projects, each binding it differently — and the developer skills read **both**.

It works in **two modes**: bootstrap (build the map + theme + adapter selection, write the doc) and consult & maintain (a new screen, brand tweak, or stack change updates the binding, the theme file, and the adapter pointer in lockstep).

## Sample prompts

- "Wire the design system up to this project." / "Connect the design system to our app."
- "Map our screens to the design system's components."
- "Set our brand colour/theme on the design system."
- "Pick which design adapter the stack uses." / "Contextualize the design system."

> Gated like the design system: nothing to bind if no bundle exists (use [`ux-designer`](ux-designer.md) first), and a trivial single-screen app may not need a map. It **never** edits the portable bundle — that stays exportable.

## What it produces

| Output | Description |
|---|---|
| `specs/DESIGN-BINDING.md` | The connecting contract: the screen/flow → component map, the chosen stack adapter, and a pointer to the theme file. Read *alongside* `DESIGN.md`. |
| `specs/design/themes/<project>.css` (and/or `.json`) | The **brand instance** — this project's concrete values layered *over* the bundle's semantic tokens. The one file the binder writes inside the bundle, only under `themes/`. |

The theme is an **override layer**, never merged into `tokens.json`/`tokens.css` (those stay the generic source of truth). Exporting the bundle elsewhere drops the `themes/` instance; a fresh binder run writes a new one.

`DESIGN-BINDING.md` is structured into: Binding intent · Theme override (the brand instance, contrast confirmed) · Stack adapter · **Screen/flow → component map** (per screen: route · layout shell · components + states · token roles · source PRD flow + architecture route) · Open questions · Changelog.

## Dependencies & consumers

**Reads:**

| Input | For… |
|---|---|
| `specs/DESIGN.md` + `specs/design/` | **required** — the available components/tokens/adapters to map and theme |
| `specs/PRD.md` | **required** — the flows/screens to map, and the brand the project wants |
| `specs/ARCHITECTURE.md` *(if present)* | routes/screens to reconcile, and the **front-end stack** (decides the adapter) |

**Consumed by:**

| Consumer | Uses the binding for… |
|---|---|
| [`web-app-scaffolder`](web-app-scaffolder.md) | wires the chosen adapter, applies the project theme, uses the map for Phase-0 screens |
| [`feature-developer`](feature-developer.md) | builds each screen from the screen→component map, in this project's theme |

**Routes back:** a *missing component/token* is a **bundle gap → [`ux-designer`](ux-designer.md)** (not invented in the map); a *missing flow* → [`product-requirements`](product-requirements.md); a *stack with no adapter* → `ux-designer` (generate it) and/or [`web-app-architect`](web-app-architect.md) (confirm the stack).

## Scope boundary

Owns the **project-specific binding** only — not the visual language ([`ux-designer`](ux-designer.md); a missing component/token routes back there, and it never edits the bundle's `tokens.*`/components/preview), product scope ([`product-requirements`](product-requirements.md)), feature behaviour ([`feature-spec`](feature-spec.md)), or the stack ([`web-app-architect`](web-app-architect.md); it *selects among* adapters, it doesn't choose the stack).
