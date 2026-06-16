---
name: ux-designer
description: Establishes and maintains the visual and interaction design system for a web app, producing a single specs/DESIGN.md (design tokens, component inventory with states, layout/navigation patterns, and an accessibility baseline) plus the token files and one static style-guide preview page that realize it. Use this after a PRD exists and before features are built in earnest, whenever the user wants to "design the UI", "set up the design system", "define the look and feel", "pick colors/typography/spacing", "create a component library", "make it look polished/consistent", or "establish the visual language". It can also INFER a design system from an existing visual reference — one or more screenshots/mockups, or a Figma link/file — extracting the palette, type, spacing, components, and layout into the system; trigger it for "match this screenshot", "build a design system from this mockup", "use our Figma as the source of truth", "infer tokens from these images", or "make it look like <attached design>". It reads specs/PRD.md (the parent — persona and flows) and, if present, specs/POC-NOTES.md (validated interaction patterns), specs/ARCHITECTURE.md (the front-end stack), and any provided visual reference, and writes the durable design contract the scaffolder sets up and every feature-developer slice conforms to. It pairs with poc-developer so the look can be perceived before committing: the two share the same visual reference and DESIGN.md in either order, so the PoC's clickable spike already looks on-brand. It owns the visual/interaction language only — not product scope (product-requirements), not feature behaviour (feature-spec), not the technical stack (web-app-architect), and not the throwaway product-feel spike (poc-developer). Requires specs/PRD.md as its parent. If there is no PRD, use product-requirements first.
---

# UX Designer

This skill establishes the **design system**: the durable visual and interaction language a product is built in — design tokens, the base component inventory and their states, layout and navigation patterns, and an accessibility baseline. It produces one source-of-truth doc (`specs/DESIGN.md`) plus the artifacts that realize it (token files + a single static style-guide preview), so every feature is built against one coherent visual language instead of each slice re-deciding colour, spacing, and component shape on its own.

It is, for *look and feel*, what `web-app-architect` is for *technical design*: a sibling owner. The architect owns the stack and how the app is built; this skill owns how it looks and how interactions behave — within that stack. **On conflict, the architecture wins** (it owns the component approach and front-end stack); DESIGN owns the visual/interaction language *within* the architecture's choices.

Three outputs, one durable contract:

- **`specs/DESIGN.md` — the design-system spec.** Tokens, component inventory + states, layout/nav patterns, a11y baseline, theming. The durable contract the scaffolder and every feature-developer read.
- **Token files** — the machine-readable tokens (e.g. CSS custom properties / a `tokens.{ts,json,css}`) the scaffolder wires in. The doc's tokens, made real and adoptable.
- **A single static style-guide preview** — one page that renders the tokens (swatches, type scale, spacing) and the component inventory once, so the system is *visible*. **Static, not a product flow** — that's the line that keeps it distinct from the PoC (see "What this skill is NOT").

## Two ways the system originates

A design system can be **invented** (from the persona, mood, and good practice) or **inferred from an existing visual reference** the user already has. Both land in the same `DESIGN.md` + tokens + preview; the difference is where the values come from.

- **Invented** — no reference. You choose the palette, type, spacing, and components from the PRD's persona/mood and sensible defaults. (The original flow; Steps 0–4 below.)
- **Inferred** — the user supplies **one or more screenshots/mockups** and/or a **Figma link or file**. You *extract* the system from the reference rather than choosing it: palette, type scale, spacing rhythm, component shapes, and layout come from what's actually in the design. See "Inferring from a visual reference" below. Inference doesn't bypass the PRD precondition or the gating question — it's a richer Step 0/1 input, not a different skill.

## Working with poc-developer (perceive before committing)

The PoC and this skill are **complementary halves of "see it before you build it"**: the PoC makes the *flow* clickable; this skill makes the *visual language* real. They share two interfaces — the **visual reference** (screenshots/Figma) and **`DESIGN.md`** — and cooperate in **either order**:

- **Design-system first →** you extract/define the system into `DESIGN.md` + tokens, then the PoC builds its clickable spike *using those real tokens*, so the perceived prototype already looks on-brand instead of placeholder-grey.
- **PoC first →** the PoC spikes the feel (loosely following any reference), leaves `POC-NOTES.md`; you then crystallize the validated patterns + the reference into the durable system.
- **Either way, the visual reference is a shared upstream input** both skills may read. If a Figma/screenshot reference exists, point the PoC at the same reference (or at `DESIGN.md`'s tokens if they're written) so the two stay consistent.

You still don't build the PoC's clickable product flow (that's `poc-developer`) and it doesn't write `DESIGN.md` (that's you). The seam stays clean; you just feed each other.

## What this skill is NOT

- It does **not** decide product scope or flows — that's `product-requirements` (the PRD). It reads the persona and flows; it doesn't invent them.
- It does **not** specify a single feature's behaviour, states, or rules — that's `feature-spec`. DESIGN defines the *shared* visual language and the *generic* component states (a button's loading state); a feature spec defines *that feature's* behaviour (when the button is disabled in this flow).
- It does **not** decide the tech stack or component framework — that's `web-app-architect`. It expresses the design *in* whatever stack the architecture chose.
- It does **not** build a clickable product prototype on mock data — that's `poc-developer`. The PoC previews *what using the product feels like*; this skill defines *the visual language* and previews it as a **static style guide** (a gallery rendered once), never a working product flow. They're complementary, not duplicative, and feed each other (see "Working with poc-developer") — but you write `DESIGN.md`, not the spike.
- It does **not** build features. The scaffolder wires the tokens + base components into the foundation; feature-developer builds slices that conform. This skill defines and previews the system, then hands off.

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

- **Absent → Bootstrap mode.** Read upstream (Step 0), gather the high-impact design direction (Step 1), define the system (Step 2), realize it as tokens + a static preview (Step 3), write the doc and hand off (Step 4).
- **Present → Consult & maintain mode.** Read it fully and treat it as the agreed design language. When the design changes (a new token, a new component, a revised pattern), update the relevant section, update the token files and the preview to match, add a Changelog entry, and never let the doc, the tokens, and the rendered system drift apart.

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

### Step 3 — Realize it: tokens + a static preview

Make the system real and visible, not just described:

- **Write the token files** in the architecture's stack idiom (CSS custom properties, a `tokens.ts`/`tokens.json`, a Tailwind theme extension — whatever the architecture implies). These are what the scaffolder adopts; keep them the single source the doc points at.
- **Build one static style-guide preview page** that renders the tokens (colour swatches with their semantic names, the type scale, the spacing scale) and the component inventory once, each component shown in its defined states. **Static only** — it demonstrates the *visual language*; it is explicitly **not** a clickable product flow (that's the PoC's job). One page, one render, no product logic.
- **Run it and look at it.** Boot the preview, confirm it renders, and check the a11y baseline holds on it (contrast, visible focus). A design system you can't see isn't proven. Show the user how to view it.

> Keep the tokens, the preview, and `DESIGN.md` in lockstep from the first commit — three views of one system. If they drift, the contract is worthless.

### Step 4 — Write the doc and hand off

1. Write `specs/DESIGN.md` using the template below — the durable contract, pointing at the token files and the preview page.
2. Present it to the user and revise before it's relied on. The token files and preview let them *see* it, not just read it.
3. Hand off: the natural next steps are `web-app-scaffolder` (which wires the tokens + base components into the foundation) and then the build loop, where every `feature-developer` slice conforms to this system.

---

## Output: specs/DESIGN.md

```markdown
# [Project Name] — Design System

> The visual & interaction language for this product. Child of specs/PRD.md.
> Realized by the token files and the static style-guide preview (see §7).
> Technical stack lives in ARCHITECTURE.md (it wins on build conflicts).
> Keep the doc, tokens, and preview in lockstep; see Changelog.

**Status:** [Draft | Active] · **Last updated:** [date]
**Tokens:** [path to token file(s)] · **Preview:** [path to the style-guide page]

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

## 7. Realization & provenance
Where the system lives in code: the token file path(s) and the static
style-guide preview page (how to view it). The scaffolder adopts these.
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
- **Down:** you own `DESIGN.md`, the token files, and the static preview. The scaffolder adopts them; feature-developer conforms to them. Keep the three in lockstep.

## Maintenance mode rules

- Read `DESIGN.md` (and glance at the token files + preview) before changing the design; treat it as the agreed language.
- On any change, update the relevant section **and** the token files **and** the preview to match, then add a Changelog entry. The doc, tokens, and rendered system must never drift apart — stale tokens get trusted and break slices.
- If a feature-developer (or scaffolder) routes a gap back here — a component or token a slice needed but the system didn't define — resolve it in the system (define the component/token, render it in the preview, update the doc), then hand back. The system leads the code; a slice never invents a token or component the system is silent on.
- If a change implies a product or stack change, route it to the owning skill (`product-requirements` / `web-app-architect`) — don't let the design system quietly redefine scope or the stack.

## Examples

**Example 1 — design system after a PoC, gets built**
PRD exists for a freelancer invoice tracker; a PoC was spiked and `POC-NOTES.md` records that an inline reminder timeline beat a modal and a calm, minimal feel landed well. The user says "set up the design system before we scaffold." The skill reads the PRD (persona: solo freelancers; flows: invoice list → detail → reminders) and the PoC notes, confirms light+dark is wanted, and defines: semantic colour tokens around a calm primary, an 8pt spacing scale, a modular type scale, a component inventory (button, input, card, table, the reminder-timeline pattern the PoC validated) each with states, a sidebar app shell, and a WCAG AA baseline. It writes `tokens.ts` + a static `/design/styleguide` preview page rendering swatches, the type scale, and every component in its states, boots it, checks contrast and focus, and writes `specs/DESIGN.md` pointing at both. Then it hands off to `web-app-scaffolder`.

**Example 2 — declining to build a design system**
The product is an internal CLI-companion admin panel; the architecture already adopts a full off-the-shelf component kit used as-is, and the PRD says visual polish is explicitly a non-goal. The user asks to "set up a design system." The skill declines to manufacture one: a DESIGN.md that just restates the library's defaults would be noise. It recommends recording "use [kit] defaults; no custom design system" in the architecture instead, and offers to add only a tiny token override file (the brand colour) if even that is wanted — rather than inventorying components and themes no flow needs.

**Example 3 — routing a gap, not improvising**
While building a feature, `feature-developer` needs a "stepper" component the design system never defined. Per the seam, it routes the gap here rather than inventing one inline. The ux-designer skill defines the stepper (its states, tokens it uses), renders it in the style-guide preview, updates `DESIGN.md` §3 and the Changelog, and hands back — so the new component enters the *system* (and is reusable) instead of being a one-off baked into one slice.

**Example 4 — maintain mode, a token change**
The user says "the brand colour is changing to teal." The skill is in maintain mode: it updates the semantic primary token in `DESIGN.md` §2 **and** the token file **and** re-renders the style-guide preview, re-checks that contrast still meets the AA baseline against the new colour (adjusting the on-primary text token if needed), and adds a Changelog entry noting all three were updated in lockstep. It does not touch any feature code that correctly consumes the semantic token — that's the point of semantic tokens.

**Example 5 — inferred from Figma (structured read), with screenshot fallback**
User: "Use our Figma as the source of truth for the design system" + a Figma link. A Figma connector is available, so the skill does a **structured read** — pulling the file's colour/text variables, components, and frame auto-layout — and extracts tokens directly: `Brand/Primary` → the `primary` token verbatim, the text styles → the type scale, the auto-layout gaps → an 8pt spacing scale, the button/card components → the inventory with their states. It normalizes the file's 9 near-identical greys into 4 semantic surface/border/text roles (noting the rationalization), flags one Figma frame whose body text fails AA contrast as an Open Question rather than shipping it, writes `tokens.ts` + the static style guide, and records provenance in `DESIGN.md` §7 (file + frame names, "structured read"). *Contrast — no connector:* it instead asks the user to export the three key frames (dashboard, a form, a list) as screenshots, infers the same system by vision, and notes in §7 that this path is lossier (no exact names/values) and cost more — which is why the structured read is preferred when available.

**Example 6 — design-system-first, feeding the PoC**
There's a PRD and a Figma reference; the user says "I want to see this clickable before we build it." The skill runs **first**: it infers the system from Figma into `DESIGN.md` + `tokens.ts` + a style guide, then hands the PoC the *real tokens* (and the same Figma reference) so `poc-developer`'s clickable spike renders on-brand — the user perceives the actual look-and-feel, not placeholder grey, before committing. The two stay consistent because they share the reference and the tokens; ux-designer wrote the durable `DESIGN.md`, the PoC wrote the throwaway spike — neither did the other's job.
