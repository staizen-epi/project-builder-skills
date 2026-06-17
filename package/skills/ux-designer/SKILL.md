---
name: ux-designer
description: Establishes and maintains the visual and interaction design system for a web app as a SELF-CONTAINED, PORTABLE BUNDLE under specs/design/ — specs/DESIGN.md (the contract) plus stack-neutral design tokens (tokens.json + CSS custom properties), optional stack adapters, and one zero-build static style-guide preview that all live together and can be exported and dropped into another project. Use this after a PRD exists and before features are built in earnest, whenever the user wants to "design the UI", "set up the design system", "define the look and feel", "pick colors/typography/spacing", "create a component library", "make it look polished/consistent", "establish the visual language", or "make a reusable/portable/exportable design system". It can also INFER a design system from an existing visual reference — one or more screenshots/mockups, or a Figma link/file — extracting the palette, type, spacing, components, and layout into the bundle; trigger it for "match this screenshot", "build a design system from this mockup", "use our Figma as the source of truth", "infer tokens from these images", or "make it look like <attached design>". It reads specs/PRD.md (the parent — persona and flows) and, if present, specs/POC-NOTES.md (validated interaction patterns), specs/ARCHITECTURE.md (the front-end stack), and any provided visual reference, and writes the durable design bundle the scaffolder adopts and every feature-developer slice conforms to — designed to work even when NOTHING is scaffolded yet (stack-neutral tokens + a buildless preview that opens in a browser with no app). It pairs with poc-developer so the look can be perceived before committing: the two share the same visual reference and the bundle in either order, so the PoC's clickable spike already looks on-brand. It owns the visual/interaction language only — not product scope (product-requirements), not feature behaviour (feature-spec), not the technical stack (web-app-architect), and not the throwaway product-feel spike (poc-developer). Requires specs/PRD.md as its parent. If there is no PRD, use product-requirements first.
---

# UX Designer

This skill establishes the **design system** and ships it as a **self-contained, portable bundle**: the durable visual and interaction language a product is built in — design tokens, the base component inventory and their states, layout and navigation patterns, and an accessibility baseline — packaged so it can be consumed *before anything is scaffolded* and **exported and dropped into another project** to share the same look. Every feature is built against one coherent visual language instead of each slice re-deciding colour, spacing, and component shape on its own.

It is, for *look and feel*, what `web-app-architect` is for *technical design*: a sibling owner. The architect owns the stack and how the app is built; this skill owns how it looks and how interactions behave — within that stack. **On conflict, the architecture wins** (it owns the component approach and front-end stack); DESIGN owns the visual/interaction language *within* the architecture's choices.

### The portable bundle

Everything the system needs to be *understood, rendered, and adopted* lives together under one folder — **`specs/design/`** — so it travels as a unit:

```
specs/
├── DESIGN.md                 # the contract (stays at specs/ root for discoverability by siblings)
└── design/                   # ← the portable bundle (copy this folder + DESIGN.md into any project)
    ├── tokens.json           # stack-NEUTRAL tokens — the single source of truth (W3C-style token shape)
    ├── tokens.css            # the same tokens as CSS custom properties (:root { --color-... }) — buildless, drop-in
    ├── adapters/             # OPTIONAL, generated FROM tokens.json for the chosen stack (gated; see Step 3)
    │   ├── tokens.ts         #   typed export for TS/JS consumers
    │   └── tailwind.tokens.js#   Tailwind theme extension
    ├── preview/              # the static style-guide preview — ZERO build, opens in a browser with no app
    │   ├── index.html        #   self-contained; links tokens.css; renders swatches, type, spacing, components
    │   └── assets/           #   any fonts/images the preview needs, vendored so it works offline/standalone
    └── BUNDLE.md             # manifest: what's here, how to view it, how to adopt it in a new project
```

- **`specs/DESIGN.md` — the contract.** Tokens, component inventory + states, layout/nav patterns, a11y baseline, theming, and the bundle manifest pointer. Stays at the `specs/` root because every sibling skill already looks for it there. The durable contract the scaffolder and every feature-developer read.
- **Stack-neutral tokens are the source of truth** — `tokens.json` (machine-readable, framework-agnostic) and `tokens.css` (CSS custom properties). Because they depend on **nothing** — no build, no app, no framework — they are consumable when nothing is scaffolded yet, and they are what makes the bundle portable. Any stack-specific file (`tokens.ts`, a Tailwind theme) is an **adapter generated from** the neutral source, never a parallel source of truth.
- **A zero-build static style-guide preview** (`preview/index.html`) — one self-contained page that renders the tokens (swatches, type scale, spacing) and the component inventory once, opening directly in a browser **without an app, build step, or dev server**. So the system is *visible* the moment it's written and inside any project it's dropped into. **Static, not a product flow** — that's the line that keeps it distinct from the PoC (see "What this skill is NOT").
- **`BUNDLE.md` — the manifest** — names every file, how to view the preview, and the two-step "adopt into a new project" recipe (copy the folder; point your stack at `tokens.css`/`tokens.json` or generate an adapter). This is what makes "drop it into another project" a recipe, not a guess.

**Portability rule:** the bundle must be self-contained — no path, import, or asset that reaches *outside* `specs/design/` (and `DESIGN.md`). If the preview needs a font or icon, vendor it into `preview/assets/`. Anyone can copy `DESIGN.md` + `specs/design/` into another repo and immediately view the style guide and consume the tokens, with no part of the originating app required.

**Stay project-agnostic (the seam with `design-binder`):** portability has a second half — the bundle must carry *no project specifics*. Define the visual *language* (semantic tokens, generic components and their states, layout patterns), not *this project's* wiring. Do **not** put the project's screens/flows, a screen→component map, the brand instance/theme override, or the chosen stack adapter *as a project decision* into the bundle — those are **`design-binder`'s** job (`DESIGN-BINDING.md` + `specs/design/themes/`). You read the PRD only for *persona and mood* to choose the language; you don't wire the language to the project. This is what lets the same bundle be inherited by many projects, each binding it differently. (You may still ship adapter *formats* generated from the tokens — see Step 3 — but *which* adapter the app uses is the binder's call.)

## Two ways the system originates

A design system can be **invented** (from the persona, mood, and good practice) or **inferred from an existing visual reference** the user already has. Both land in the same `DESIGN.md` + tokens + preview; the difference is where the values come from.

- **Invented** — no reference. You choose the palette, type, spacing, and components from the PRD's persona/mood and sensible defaults. (The original flow; Steps 0–4 below.)
- **Inferred** — the user supplies **one or more screenshots/mockups** and/or a **Figma link or file**. You *extract* the system from the reference rather than choosing it: palette, type scale, spacing rhythm, component shapes, and layout come from what's actually in the design. See "Inferring from a visual reference" below. Inference doesn't bypass the PRD precondition or the gating question — it's a richer Step 0/1 input, not a different skill.

## Working with poc-developer (perceive before committing)

The PoC and this skill are **complementary halves of "see it before you build it"**: the PoC makes the *flow* clickable; this skill makes the *visual language* real. They share two interfaces — the **visual reference** (screenshots/Figma) and **`DESIGN.md`** — and cooperate in **either order**:

- **Design-system first →** you extract/define the system into `DESIGN.md` + tokens, then the PoC builds its clickable spike *using those real tokens*, so the perceived prototype already looks on-brand instead of placeholder-grey.
- **PoC first →** the PoC spikes the feel (loosely following any reference), leaves `POC-NOTES.md`; you then crystallize the validated patterns + the reference into the durable system.
- **Either way, the visual reference is a shared upstream input** both skills may read. If a Figma/screenshot reference exists, point the PoC at the same reference (or at the bundle's tokens if they're written) so the two stay consistent.

Because the bundle is buildless, the PoC can adopt it **directly with no scaffolding**: a `/poc` spike just links `specs/design/tokens.css` (or imports `tokens.json`) and is instantly on-brand. That's the practical payoff of the portable bundle for `poc-developer` — the tokens are real and consumable before any app exists.

You still don't build the PoC's clickable product flow (that's `poc-developer`) and it doesn't write `DESIGN.md` (that's you). The seam stays clean; you just feed each other.

## What this skill is NOT

- It does **not** decide product scope or flows — that's `product-requirements` (the PRD). It reads the persona and flows; it doesn't invent them.
- It does **not** specify a single feature's behaviour, states, or rules — that's `feature-spec`. DESIGN defines the *shared* visual language and the *generic* component states (a button's loading state); a feature spec defines *that feature's* behaviour (when the button is disabled in this flow).
- It does **not** decide the tech stack or component framework — that's `web-app-architect`. It expresses the design *in* whatever stack the architecture chose.
- It does **not** build a clickable product prototype on mock data — that's `poc-developer`. The PoC previews *what using the product feels like*; this skill defines *the visual language* and previews it as a **static style guide** (a gallery rendered once), never a working product flow. They're complementary, not duplicative, and feed each other (see "Working with poc-developer") — but you write `DESIGN.md`, not the spike.
- It does **not** build features. The scaffolder wires the bundle's tokens + base components into the foundation; feature-developer builds slices that conform. This skill defines, packages, and previews the system, then hands off.
- It does **not** depend on the app existing. The bundle is produced to be consumable **pre-scaffold** and **in another project**; it never reaches into the originating app's stack or paths.
- It does **not** wire the design system to *this* project. The screen/flow→component map, the project's brand/theme instance, and the stack-adapter choice are `design-binder`'s job (`DESIGN-BINDING.md` + `specs/design/themes/`). Keeping those out of the bundle is exactly what makes it inheritable. You design the portable language; the binder contextualizes it.

If the user wants any of the above, point them at the owning skill rather than absorbing its job here.

## Precondition

**A design system needs a parent. Check for `specs/PRD.md` before doing anything.**

- **Missing →** a design system without a PRD has no persona, no flows, and no product to serve — it would be styling in a vacuum. Point the user at `product-requirements` to establish the product first. Only proceed without a PRD if the user explicitly wants a standalone style exploration and understands there's no product to trace it to.
- **Present →** read it (Step 0), then additionally read `specs/POC-NOTES.md` and `specs/ARCHITECTURE.md` if they exist, and any visual reference (screenshots / Figma link) the user has supplied.

## Does this product need a design system?

Not every app earns one — gating, not templates (see the suite's spine). Decide before building:

- **Skip it when** the product's visual polish isn't a real requirement: an internal admin panel, a CLI/dev tool, a throwaway-grade utility, or an app whose architecture already adopts a complete off-the-shelf design system (e.g. a full component kit used as-is). A near-empty DESIGN.md that just says "use the library defaults" is noise — record that decision in the PRD/architecture instead and move on.
- **Build one when** the product has a real user-facing surface where consistency and polish matter, multiple screens that must feel coherent, custom branding, or a component set the team will extend over many features.

If it's borderline, ask the user rather than defaulting to a full system. When unsure, prefer the lighter option — a system is cheaper to introduce later than to maintain unused.

## Two modes

**Check for `specs/DESIGN.md`.**

- **Absent → Bootstrap mode.** Read upstream (Step 0), gather the high-impact design direction (Step 1), define the system (Step 2), realize it as the portable bundle — neutral tokens, optional adapters, a buildless preview, and `BUNDLE.md` (Step 3) — write the doc and hand off (Step 4).
- **Present → Consult & maintain mode.** Read it fully and treat it as the agreed design language. When the design changes (a new token, a new component, a revised pattern), update the relevant section, update the source-of-truth tokens, regenerate adapters and the preview to match, add a Changelog entry, and never let the doc, the tokens, the adapters, and the rendered system drift apart — or break the bundle's portability.

---

## Bootstrap mode

### Step 0 — Read upstream

Pull what's already settled so you design *for this product*, not in the abstract:

- **From `specs/PRD.md` (the parent):** the persona (who uses this, in what context — affects density, tone, accessibility needs), the core flows (what screens the system must serve), the product's character and any brand cues, and constraints/non-goals that bound the design (platform, accessibility regime, "must look like X").
- **From `specs/POC-NOTES.md` (if present):** the **validated interaction patterns** — what felt right in the spike ("inline edit beat a modal", a timeline component that worked), component shapes that emerged, and pitfalls. This is the clean seam: the PoC validated interactions on throwaway code; you crystallize the keepers into the durable system. Read the *notes*, not the `/poc` code.
- **From `specs/ARCHITECTURE.md` (if present):** the **front-end stack and component approach** (e.g. React + Tailwind, a chosen primitive library like Radix/shadcn), so tokens and components are expressed in the real stack. The architecture wins on any conflict about *how* components are built; you decide *how they look*.
- **From any visual reference the user provides (if present):** screenshots/mockups and/or a Figma link or file. When a reference exists, the system is **inferred** from it (Step 1.5 below) rather than invented — the reference becomes the primary source for palette, type, spacing, components, and layout. The PRD still governs persona/flows/non-goals and the architecture still governs the stack; the reference governs the *look*.

Extract; don't re-ask what's settled. Don't restate the PRD or architecture in DESIGN — link to them.

### Step 1 — Gather the design direction (ask only what matters)

Most of a design system is defaulted from the persona, the stack, and good practice. **The high-impact decisions worth asking — only if undetermined by the PRD/PoC/conversation — are:**

- **Brand & mood** — the core colour(s) and the overall feel (e.g. "calm and minimal", "dense and data-heavy", "playful"). This anchors the whole palette and type choices. If the PRD or PoC settles it, don't ask.
- **Theming** — is dark mode (or multi-theme) a requirement, or light-only? It changes how tokens are structured, so it's worth knowing up front.
- **Accessibility floor** — the target (e.g. WCAG 2.1 AA) if the PRD's domain implies a regulated or broad-audience requirement; otherwise default to AA as a sane baseline.

Assume sensible defaults for everything else and record them in the doc:

- **Type & spacing scales** — default to a conventional modular type scale and an 8pt-based spacing scale unless the brand/density calls for otherwise.
- **Component primitives** — default to the architecture's chosen primitive library if it has one; otherwise a minimal hand-rolled set sized to the flows. Don't inventory components no flow needs.
- **Density & radius/elevation** — derive from the mood (data-heavy → denser, tighter radius; consumer → airier).

Bias toward the **smallest system that serves the flows**. Don't design tokens, components, or themes nothing in the PRD asks for — gating, not templates. When unsure, define less and note it.

### Step 1.5 — Inferring from a visual reference (only if one was provided)

When the user supplies screenshots/mockups or a Figma link, **extract** the system from the reference instead of inventing it. The goal is to turn an existing design into the same `DESIGN.md` + tokens + preview, faithfully.

**Read the reference in the cheapest faithful way — structured before pixels.** A design's *structure* (named colour/type styles, components, spacing variables) is far cheaper per unit of fidelity than a flat image, and more accurate — you read the exact value instead of eyeballing it.

- **Figma — prefer a structured read.** If a Figma MCP server or the Figma API (with a token) is available, pull the file's **styles/variables, components, and frame layout as structured data** and extract tokens directly from it (a named `Primary/500 = #2D6A4F` becomes your `primary` token verbatim; auto-layout gaps become the spacing scale; text styles become the type scale). This is the high-fidelity, low-cost path: you're *extracting* a system that already exists, not inferring one.
  - **No Figma access? Fall back to images.** Ask the user to export the key frames as screenshots and infer from those (below). State the trade plainly: screenshots cost more tokens and lose the exact values/names, so prefer the structured read whenever the connector exists. Don't silently default to the expensive path.
- **Screenshots/mockups — infer by vision, but bound the cost.** Read the provided images and extract: the **palette** (sample the recurring colours → map to semantic roles, don't just list hexes), the **type** (families, the apparent size/weight steps → a scale), the **spacing rhythm** (the repeating gaps/padding → a scale), the **component shapes** (button/input/card styling, radius, elevation), and the **layout** (shell, nav model, grid). A few representative frames beat a large dump — ask for the *key* screens (a primary screen, a form, a list/table) rather than every screen, to keep vision-token cost proportional to the value.

**In both cases:**
- **Normalize to semantic tokens, don't transcribe raw values.** The reference may use 11 near-identical greys; collapse them into a sane semantic set (`surface`, `surface-muted`, `border`, `text`, `text-muted`). A faithful *system* beats a pixel-perfect copy of an inconsistent design — note where you rationalized.
- **Respect the gate and the PRD.** A reference doesn't override the "does this need a design system?" question, the persona, or non-goals. If the reference implies components no flow needs, don't inventory them.
- **Record provenance.** Note in `DESIGN.md` §1/§7 that the system was inferred, from what (Figma file/frame names or the screenshots), and via which path (structured vs. vision) — so its origin is traceable and a later Figma change has a known source.
- **Flag conflicts, don't silently pick.** If the reference contradicts the PRD (e.g. a dark, dense design for a PRD that says "calm, airy, accessible to low-vision users") or fails the accessibility floor (contrast below AA), surface it as an Open Question rather than quietly following either — the user resolves it.

### Step 2 — Define the system

Decide and record, each sized to what the flows actually need:

- **Design tokens** — colour (semantic roles: surface/text/primary/danger/border…, not raw hex names scattered around), typography (family, the type scale, weights, line-heights), spacing scale, radius, elevation/shadow, and motion (durations/easing) if the product uses animation. Tokens are *semantic* so themes can re-map them.
- **Component inventory** — the base components the flows need (button, input, select, card, table, modal, toast, nav…), each with its **key states defined**: default / hover / focus / active / disabled / loading / error / empty where applicable. This generic state coverage is what feature slices inherit; a feature spec then says *when* each state applies in its flow.
- **Layout & navigation patterns** — the app shell, the navigation model (sidebar/topbar/tabs), responsive breakpoints, and the grid/spacing rhythm screens follow.
- **Accessibility baseline** — the checkable floor: contrast ratios met by the palette, visible focus on every interactive element, keyboard operability, motion-reduction respect, and semantic-structure expectations. Make it *verifiable*, not aspirational.
- **Theming** — if themed, how tokens re-map across light/dark (or brands), and the default theme.

Mark anything unresolved as ⚠️ TBD inline and mirror it into Open Questions. Defer feature-specific behaviour to `feature-spec` and stack/build decisions to `web-app-architect` — capture *needs*, not those decisions.

### Step 3 — Realize it as the portable bundle

Make the system real, visible, and **portable** — not just described. Build everything under `specs/design/` so it travels as a unit (layout in "The portable bundle" above). The discipline that keeps it portable: **stack-neutral source of truth, optional adapters, zero-build preview, self-contained.**

1. **Write the stack-neutral tokens first — the source of truth.**
   - `specs/design/tokens.json` — framework-agnostic, machine-readable (a W3C-design-tokens-style shape: each token has a value and, where useful, a `$type`). Semantic roles, the type/spacing/radius/elevation/motion scales, and any theme variants. This file depends on nothing and is what another project imports.
   - `specs/design/tokens.css` — the **same** tokens as CSS custom properties (`:root { --color-surface: …; } [data-theme="dark"] { … }`). This is the buildless drop-in: any HTML/app can `<link>` it with no toolchain, which is exactly why it works pre-scaffold and in any other project.
   - Keep these two in lockstep (the CSS is a serialization of the JSON). Everything else derives from them.

2. **Generate stack adapters only if the architecture calls for one (gated).** If `ARCHITECTURE.md` names a stack that wants typed or framework-native tokens, emit them **into `specs/design/adapters/`** *from* `tokens.json` — e.g. `tokens.ts` (typed export) or `tailwind.tokens.js` (theme extension). These are **generated artifacts, never a second source of truth**; note in `BUNDLE.md` that they regenerate from `tokens.json`. No stack decided yet, or the stack consumes CSS variables directly → **skip adapters** (the neutral tokens are enough). Don't manufacture adapters for stacks the project isn't using — gating, not templates.

3. **Build the zero-build static style-guide preview** at `specs/design/preview/index.html`:
   - A **single self-contained HTML page** that links `../tokens.css` and renders the tokens (colour swatches with their semantic names, the type scale, the spacing scale) and the component inventory once, each component shown in its defined states.
   - **No build step, no dev server, no framework** — it must open by double-clicking the file (or `open index.html`). Components are plain HTML+CSS styled by the tokens (not the app's component library) — this previews the *language*, and keeps the preview portable even if the real components are React/etc.
   - **Fonts: prefer a fallback stack; vendor only when necessary.** If the type token uses a font with a safe fallback (e.g. `Inter, system-ui, sans-serif`), the preview renders standalone with no font file — don't ship one (it just bloats an exportable bundle). **Only** vendor a font/icon into `preview/assets/` when the preview genuinely depends on one that has no acceptable system fallback. Either way it must render offline, in any project it's dropped into.
   - **Support an optional `?theme=<name>` override.** Add a tiny script so opening `preview/index.html?theme=<name>` also loads `../themes/<name>.css` over `tokens.css`. The bundle ships *no* theme (it's project-agnostic), but this hook lets `design-binder` preview a project's brand instance against the same style guide without forking the preview. Keep it a no-op when no `?theme=` is given.
   - **Static only** — it demonstrates the *visual language*; it is explicitly **not** a clickable product flow (that's the PoC's job). One page, one render, no product logic.

4. **Write `BUNDLE.md` — the manifest.** List every file in the bundle, "how to view" (open `preview/index.html`), and the **"adopt into a new project" recipe**: (1) copy `DESIGN.md` + `specs/design/` into the target repo; (2) link `tokens.css` / import `tokens.json` (or regenerate an adapter from it) and stand up the components from the inventory. This is what turns "portable" into a followable two-step.

5. **Open the preview and look at it.** Open `preview/index.html` directly (no server), confirm it renders standalone, and check the a11y baseline holds on it (contrast, visible focus). A design system you can't see isn't proven. Show the user how to view it.

6. **Verify portability before handing off.** Confirm nothing in the bundle references a path, import, or asset *outside* `specs/design/` (and `DESIGN.md`). The buildless preview opening on its own is the cheapest proof; if it needs the app to render, it isn't portable yet — fix it.

> Keep the tokens, the adapters, the preview, and `DESIGN.md` in lockstep from the first commit — all views of one system, all inside the bundle. If they drift, the contract is worthless.

### Step 4 — Write the doc and hand off

1. Write `specs/DESIGN.md` using the template below — the durable contract, pointing at the bundle (`specs/design/`): its tokens, adapters (if any), the preview, and `BUNDLE.md`.
2. Present it to the user and revise before it's relied on. The tokens and the buildless preview let them *see* it, not just read it.
3. Hand off. The bundle is deliberately **project-agnostic** (no flows, screens, or brand instance — see "Stay project-agnostic" below), so the natural next step is **`design-binder`**, which hard-wires the PRD + ARCHITECTURE to this bundle: it writes the screen/flow→component map, the project's theme-override file (under `specs/design/themes/`), and the chosen stack adapter — the *connecting* artifact the developers read alongside the bundle. After binding comes `web-app-scaffolder` (adopts the bundle + binding) and the build loop (every `feature-developer` slice conforms to both). Because the bundle is portable and buildless, `poc-developer` can also consume `tokens.css` directly with no scaffolding, and the bundle can be exported into another project as-is (per `BUNDLE.md`) — where a fresh `design-binder` run wires it to *that* project.

---

## Output: specs/DESIGN.md

```markdown
# [Project Name] — Design System

> The visual & interaction language for this product. Child of specs/PRD.md.
> Realized by the PORTABLE BUNDLE under specs/design/ (tokens, optional adapters,
> a zero-build preview, and BUNDLE.md) — see §7. The bundle is self-contained:
> consumable before anything is scaffolded and exportable into another project.
> Technical stack lives in ARCHITECTURE.md (it wins on build conflicts).
> Keep the doc, tokens, adapters, and preview in lockstep; see Changelog.

**Status:** [Draft | Active] · **Last updated:** [date]
**Bundle:** specs/design/ · **Tokens (source of truth):** specs/design/tokens.json + tokens.css
**Preview (buildless):** specs/design/preview/index.html · **Manifest:** specs/design/BUNDLE.md

## 1. Direction
The persona-anchored design intent in one short paragraph — the mood, who it
serves, the character. Trace to the PRD; don't restate it. If the system was
inferred from a reference, say so and name the source (see §7 provenance).

## 2. Design tokens
Semantic colour roles, typography (family + scale + weights), spacing scale,
radius, elevation, motion. The *names and values*; the files in §7 are the
machine-readable source. Tokens are semantic so themes can re-map them.

## 3. Component inventory
Each base component the flows need, with its key states
(default/hover/focus/disabled/loading/error/empty where applicable).
This generic state coverage is what feature slices inherit.

## 4. Layout & navigation
App shell, navigation model, responsive breakpoints, grid/spacing rhythm.

## 5. Accessibility baseline
The checkable floor: contrast targets met, visible focus, keyboard operability,
motion-reduction, semantic structure. Verifiable conditions, not aspirations.

## 6. Theming
If themed: how tokens re-map (light/dark/brands) and the default. Omit if
light-only — and say so.

## 7. Realization & portability (the bundle)
The system ships as a self-contained bundle under `specs/design/`:
- **Tokens (source of truth):** `tokens.json` (stack-neutral) + `tokens.css`
  (CSS custom properties, buildless). Everything derives from these.
- **Adapters (optional, generated from tokens.json):** e.g. `adapters/tokens.ts`,
  `adapters/tailwind.tokens.js` — only if the stack needs them; regenerable.
- **Preview (buildless):** `preview/index.html` — opens in a browser with no app,
  build, or server. How to view: open the file directly.
- **Manifest:** `BUNDLE.md` — file list + the "adopt into a new project" recipe.
The scaffolder adopts the bundle; `poc-developer` can consume it pre-scaffold;
it can be exported wholesale into another project. **Portability invariant:**
nothing in the bundle references anything outside `specs/design/` + this doc.

If inferred, record provenance: the source (Figma file/frame names or which
screenshots), the read path (structured Figma data vs. vision), and any values
rationalized from the reference — so the origin is traceable.

## 8. Open questions
Unresolved design decisions with impact and status (⚠️ TBD mirrors here).
Product-level questions route to product-requirements; stack questions to
web-app-architect.

## 9. Changelog
Dated entries for every change to the design system (and whether the tokens
and preview were updated to match).
```

## Scope boundary (important)

This skill owns the visual & interaction language — not the product, not feature behaviour, not the stack.

- **Up (product):** don't decide persona, scope, or flows — that's the PRD. Read them and design for them; route surfaced product questions back to `product-requirements`.
- **Up (technical):** don't decide the stack or component framework — that's `web-app-architect`. Express the design in the chosen stack; on conflict, the architecture wins. Route missing technical patterns there.
- **Sideways (feature behaviour):** don't specify a feature's states/rules/flow — that's `feature-spec`. Define the *generic* component states; let the feature spec say when each applies.
- **Sideways (product feel):** don't build a clickable mock-data prototype — that's `poc-developer`. Your preview is a *static* style guide, not a product flow. Crystallize the PoC's *validated* patterns into the durable system; don't re-spike them. You *cooperate* with the PoC (shared visual reference + `DESIGN.md`, either order) so the spike looks on-brand — feeding it, not absorbing it.
- **Down:** you own `DESIGN.md` and the portable bundle (`specs/design/`: tokens, adapters, the buildless preview, `BUNDLE.md`). The scaffolder adopts them; feature-developer conforms to them; `poc-developer` may consume them pre-scaffold; another project can import the bundle wholesale. Keep them all in lockstep and self-contained.

## Maintenance mode rules

- Read `DESIGN.md` (and glance at the bundle's tokens + preview) before changing the design; treat it as the agreed language.
- On any change, update the relevant section **and** the source-of-truth tokens (`tokens.json` + `tokens.css`) **and** regenerate any adapters **and** the preview to match, then add a Changelog entry. The doc, tokens, adapters, and rendered system must never drift apart — stale tokens get trusted and break slices.
- Preserve portability on every change: don't introduce a path, import, or asset that reaches outside `specs/design/`, and keep the preview buildless. If a change would break standalone rendering, fix it before committing. Re-open `preview/index.html` standalone as the cheap regression check.
- If a feature-developer (or scaffolder) routes a gap back here — a component or token a slice needed but the system didn't define — resolve it in the system (define the component/token, render it in the preview, update the doc), then hand back. The system leads the code; a slice never invents a token or component the system is silent on.
- If a change implies a product or stack change, route it to the owning skill (`product-requirements` / `web-app-architect`) — don't let the design system quietly redefine scope or the stack.

## Examples

**Example 1 — design system after a PoC, gets built**
PRD exists for a freelancer invoice tracker; a PoC was spiked and `POC-NOTES.md` records that an inline reminder timeline beat a modal and a calm, minimal feel landed well. The user says "set up the design system before we scaffold." The skill reads the PRD (persona: solo freelancers; flows: invoice list → detail → reminders) and the PoC notes, confirms light+dark is wanted, and defines: semantic colour tokens around a calm primary, an 8pt spacing scale, a modular type scale, a component inventory (button, input, card, table, the reminder-timeline pattern the PoC validated) each with states, a sidebar app shell, and a WCAG AA baseline. It writes the **portable bundle** under `specs/design/` — `tokens.json` + `tokens.css` (source of truth), a buildless `preview/index.html` rendering swatches, the type scale, and every component in its states, and `BUNDLE.md` — opens the preview standalone (no app exists yet), checks contrast and focus, and writes `specs/DESIGN.md` pointing at the bundle. Then it hands off to `web-app-scaffolder` (which adopts the tokens). Nothing is scaffolded yet, but the system is already viewable and the PoC could consume `tokens.css` as-is.

**Example 2 — declining to build a design system**
The product is an internal CLI-companion admin panel; the architecture already adopts a full off-the-shelf component kit used as-is, and the PRD says visual polish is explicitly a non-goal. The user asks to "set up a design system." The skill declines to manufacture one: a DESIGN.md that just restates the library's defaults would be noise. It recommends recording "use [kit] defaults; no custom design system" in the architecture instead, and offers to add only a tiny token override file (the brand colour) if even that is wanted — rather than inventorying components and themes no flow needs.

**Example 3 — routing a gap, not improvising**
While building a feature, `feature-developer` needs a "stepper" component the design system never defined. Per the seam, it routes the gap here rather than inventing one inline. The ux-designer skill defines the stepper (its states, tokens it uses), renders it in the style-guide preview, updates `DESIGN.md` §3 and the Changelog, and hands back — so the new component enters the *system* (and is reusable) instead of being a one-off baked into one slice.

**Example 4 — maintain mode, a token change**
The user says "the brand colour is changing to teal." The skill is in maintain mode: it updates the semantic primary token in `DESIGN.md` §2 **and** the source-of-truth tokens (`tokens.json` + `tokens.css`) **and** regenerates any adapter (`adapters/tokens.ts`) **and** re-renders the buildless preview, re-checks that contrast still meets the AA baseline against the new colour (adjusting the on-primary text token if needed), re-opens `preview/index.html` standalone as the portability check, and adds a Changelog entry noting all were updated in lockstep. It does not touch any feature code that correctly consumes the semantic token — that's the point of semantic tokens.

**Example 5 — inferred from Figma (structured read), with screenshot fallback**
User: "Use our Figma as the source of truth for the design system" + a Figma link. A Figma connector is available, so the skill does a **structured read** — pulling the file's colour/text variables, components, and frame auto-layout — and extracts tokens directly: `Brand/Primary` → the `primary` token verbatim, the text styles → the type scale, the auto-layout gaps → an 8pt spacing scale, the button/card components → the inventory with their states. It normalizes the file's 9 near-identical greys into 4 semantic surface/border/text roles (noting the rationalization), flags one Figma frame whose body text fails AA contrast as an Open Question rather than shipping it, writes `tokens.ts` + the static style guide, and records provenance in `DESIGN.md` §7 (file + frame names, "structured read"). *Contrast — no connector:* it instead asks the user to export the three key frames (dashboard, a form, a list) as screenshots, infers the same system by vision, and notes in §7 that this path is lossier (no exact names/values) and cost more — which is why the structured read is preferred when available.

**Example 6 — design-system-first, feeding the PoC (nothing scaffolded yet)**
There's a PRD and a Figma reference; the user says "I want to see this clickable before we build it." The skill runs **first**: it infers the system from Figma into `DESIGN.md` + the bundle (`tokens.json` + `tokens.css` + a buildless style guide). Because the bundle is portable and needs no app, `poc-developer`'s clickable spike just links `specs/design/tokens.css` and renders on-brand with **no scaffolding** — the user perceives the actual look-and-feel, not placeholder grey, before committing. The two stay consistent because they share the reference and the tokens; ux-designer wrote the durable bundle, the PoC wrote the throwaway spike — neither did the other's job.

**Example 7 — exporting the bundle into another project**
The team starts a second app that should share the first's look. The user says "use the same design system in this new repo." There's nothing to re-derive: the bundle is self-contained, so they follow `BUNDLE.md` — copy `DESIGN.md` + `specs/design/` into the new repo, open `preview/index.html` to confirm it renders standalone there, then link `tokens.css` (or regenerate the stack adapter from `tokens.json`) and stand up the inventory's components. The new project's `poc-developer`/`feature-developer` now consume the very same tokens. If the new repo's stack differs, only a fresh adapter is generated from the unchanged neutral `tokens.json` — the source of truth and the contract travel intact. This is the payoff of producing a portable bundle rather than tokens entangled with the originating app.
