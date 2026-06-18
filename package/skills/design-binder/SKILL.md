---
name: design-binder
description: Binds the PORTABLE design system (the ux-designer bundle under specs/design/) to THIS specific project — the connecting artifact that turns an inheritable, project-agnostic design system into one wired to this app's PRD and ARCHITECTURE. It writes specs/DESIGN-BINDING.md (a screen/flow→component map saying which bundle components, tokens, and layout realize each PRD flow and architecture route; plus the chosen stack adapter the architecture consumes) and a project theme-override token file under specs/design/themes/ (the brand instance, layered on the bundle without polluting it). Use this AFTER a design bundle exists (specs/DESIGN.md + specs/design/) and a PRD exists, whenever the user wants to "apply the design system to this project", "wire up the design system", "map screens to components", "set the project's brand/theme on the design system", "pick which design adapter the stack uses", "contextualize the design system", or "connect the design system to our app". It reads specs/PRD.md (flows/screens), specs/ARCHITECTURE.md (routes + the front-end stack), and the bundle (the available components/tokens), and writes the connecting contract the scaffolder and every feature-developer read ALONGSIDE the bundle. It owns the project-specific binding only — not the portable visual language itself (that's ux-designer; a missing component/token routes back there), not product scope (product-requirements), not feature behaviour (feature-spec), not the technical stack (web-app-architect). Requires specs/DESIGN.md (the bundle) and specs/PRD.md. If there is no design bundle, use ux-designer first; if there is no PRD, use product-requirements first.
---

# Design Binder

This skill produces the **connecting artifact**: it takes the **portable, project-agnostic design bundle** that `ux-designer` ships (`specs/DESIGN.md` + `specs/design/`) and **hard-wires it to *this* project** by reading the PRD and the architecture. The bundle is the *language*, inheritable into any project; the binding is *how this project speaks it*.

The split is deliberate and load-bearing for the whole suite's design layer:

- **`ux-designer` → the portable artifact.** A self-contained bundle with **no** project specifics (no flows, no screens, no brand instance, no stack choice) — that's what makes it exportable and inheritable.
- **`design-binder` → the connecting artifact.** **All** the project specifics that wire the bundle to this app: which components/tokens realize which screen, this project's brand/theme instance, and which stack adapter the architecture consumes.

Keeping these in two files means one design bundle can be inherited by many projects, each binding it differently — and the developer skills read **both**: the bundle for the visual language, the binding for how this project uses it.

## Two outputs, one connecting contract

- **`specs/DESIGN-BINDING.md` — the binding contract.** The screen/flow→component map (each PRD flow + architecture route → the bundle components, tokens, and layout shell that realize it), the chosen **stack adapter** the architecture consumes, and the pointer to this project's theme file. The durable doc the scaffolder and every feature-developer read alongside `DESIGN.md`.
- **A project theme-override file** under `specs/design/themes/<project>.css` (and/or `.json`) — the **brand instance**: this project's concrete values layered *over* the bundle's semantic tokens (e.g. the bundle defines `--color-primary` as a role; this project sets it to the brand teal). It overrides, never edits, the bundle — so the bundle stays exportable and generic. (This is the *one* file the binder writes inside `specs/design/`, and only under `themes/`; everything else in the bundle stays `ux-designer`'s.)

> **Why the theme lives under the bundle but is owned here.** Placing the override at `specs/design/themes/` keeps it travelling *with* the bundle when convenient, but it is **project-specific output of this skill**, layered on top — never merged into `tokens.json`/`tokens.css` (those stay the generic source of truth). Exporting the bundle to another project drops the `themes/` instance and a fresh binder run writes a new one.

## What this skill is NOT

- It does **not** define or change the visual language — that's `ux-designer`. It maps and themes *existing* bundle pieces; if a screen needs a component or token the bundle doesn't have, that's a **bundle gap → route to `ux-designer`** (which adds it to the system + preview), not something to invent here.
- It does **not** decide product scope or flows — that's `product-requirements`. It reads the PRD's flows/screens; it doesn't invent them. A missing flow routes back to the PRD.
- It does **not** specify a feature's behaviour, states, or rules — that's `feature-spec`. It says *which* component renders a screen and *in this project's theme*; the feature spec says *when* each state applies.
- It does **not** decide the tech stack or component framework — that's `web-app-architect`. It *selects among* the bundle's adapters to match the architecture's stack; it doesn't choose the stack. A stack the bundle has no adapter for routes to `ux-designer` (generate the adapter) and/or `web-app-architect` (confirm the stack).
- It does **not** build features or scaffold. The scaffolder adopts the binding; feature-developer conforms to it. This skill writes the connecting contract, then hands off.

If the user wants any of the above, point them at the owning skill rather than absorbing its job here.

## Precondition

**The binding connects two things — it needs both.**

- **`specs/DESIGN.md` + `specs/design/` (the bundle) must exist.** There's nothing to bind without a portable design system. Missing → use `ux-designer` first.
- **`specs/PRD.md` must exist.** The binding maps the PRD's flows/screens to the bundle; without a PRD there are no flows to wire. Missing → use `product-requirements` first.
- **`specs/ARCHITECTURE.md` is read if present** — for the routes/screens and, crucially, the front-end **stack** (which decides the adapter). If absent, you can still map flows→components and write the theme, but record the adapter choice as ⚠️ TBD until the stack is decided.

## Does this project need a binding?

Gating, not templates (the suite's spine). The binding exists *because* the bundle is generic; if the bundle was skipped, so is this.

- **Skip it when** there's no design bundle (nothing to bind — `ux-designer` was rightly skipped for an admin panel / dev tool / off-the-shelf-kit app), **or** the app is a single trivial screen where a one-line "use the bundle defaults everywhere, brand colour = X" note in the architecture is enough. A binding doc that just says "render everything with bundle defaults" is noise.
- **Build one when** the bundle exists *and* the project has multiple screens/flows that need a clear map to components, a real brand instance distinct from the bundle's neutral defaults, or a stack that needs a specific adapter wired. That's the normal case once `ux-designer` has run.

If it's borderline, prefer the lighter option and note it — a binding is cheap to add later.

## Two modes

**Check for `specs/DESIGN-BINDING.md`.**

- **Absent → Bootstrap mode (interview where it's user-owned).** Read upstream (Step 0) — most of the binding *derives* from the bundle + PRD + architecture, so the interview is narrow: it focuses on the genuinely user-owned unknown, the **brand instance (theme)**, which the bundle can't supply. Confirm the high-impact binding decisions (Step 1) — interview on the brand until no theme-shaping value is still askable, and confirm (don't silently assume) the adapter and coverage inferences — build the map + theme + adapter selection (Step 2), realize the theme file and verify (Step 3), write the doc and hand off (Step 4).
- **Present → Consult & maintain mode.** Read it fully and treat it as the agreed binding. When the project changes (a new screen/flow, a brand tweak, a stack change), update the relevant section, update the theme file and the adapter pointer to match, add a Changelog entry, and never let the binding, the theme, and the project drift apart.

---

## Bootstrap mode

### Step 0 — Read upstream

Pull what's already settled so you *connect* rather than re-decide:

- **From the bundle (`specs/DESIGN.md` + `specs/design/`):** the available **components** (the inventory + their states), the **semantic tokens** (the roles you'll theme, e.g. `--color-primary`, `--surface`, the type/spacing scales), the **layout/nav patterns** it offers, the **adapters** present under `adapters/` (or the note that the stack consumes `tokens.css` directly), and the a11y baseline. This is your palette of bindable pieces — you map *to* it, you don't add to it.
- **From `specs/PRD.md` (the parent):** the **flows and the screens they imply** (invoice list → detail → reminders…), the persona/density cues, and the **brand** the project actually wants (the concrete colour(s), the feel) — this drives the theme override. The bundle gave you the *role* `primary`; the PRD/brand gives you its *value* here.
- **From `specs/ARCHITECTURE.md` (if present):** the **routes/screens** (to reconcile with the PRD's flows into a concrete screen list) and the **front-end stack** (React+Tailwind, plain CSS, Vue…) — which decides **which bundle adapter** this project consumes.

Extract; don't re-ask what's settled. Don't restate the bundle's component definitions or the PRD's flows in the binding — *link* to them and map between them.

### Step 1 — Confirm the high-impact binding decisions (interview on the brand)

The binding doesn't exist yet, but most of it is *derivable* from the bundle + PRD + architecture — so unlike the upstream artifact skills, the interview here is **narrow and targeted at what only the user can supply: the brand instance.** Everything else you derive and *confirm* rather than interrogate. **The decisions worth resolving — only if undetermined by the PRD/architecture/bundle — are:**

- **The brand instance (theme).** The concrete value(s) for the bundle's semantic roles in *this* project — at minimum the primary/brand colour, and dark-mode brand values if the bundle is themed. If the PRD states the brand, don't ask; just record it. **If it doesn't, interview** — this is the project-specific intent the bundle deliberately can't carry, so draw it out (the brand colour, whether dark mode is in play, any secondary/accent the flows imply) rather than defaulting to the bundle's neutral and producing an unbranded app. (Everything genuinely *not* overridden inherits the bundle's neutral default — that's the point.)
- **The stack adapter.** Which bundle adapter the architecture's stack consumes (`adapters/tailwind.tokens.js`, `adapters/tokens.ts`, or `tokens.css` directly). If `ARCHITECTURE.md` names the stack, derive it — only ask if the stack is genuinely undecided or the bundle has no matching adapter (then it's a gap to route, not a question to default).
- **Coverage scope.** Whether to map *every* screen now or just the Phase-0/hero screens, deferring the rest. Default to **the screens that exist or are imminent** (don't map flows the PRD only gestures at); note deferred screens as ⚠️ TBD.

Assume sensible defaults for everything else and record them:

- **Screen→component mapping** — default each PRD flow's screens to the obvious bundle components (a list flow → the table/card + the app-shell layout; a form flow → input/select/button) and only call out the non-obvious bindings. Don't enumerate trivial mappings exhaustively.
- **Layout/nav** — default to the bundle's offered shell (sidebar/topbar) unless the architecture or PRD implies otherwise.

Bias toward the **smallest binding that wires the real screens**. Don't map screens no flow needs, and don't override tokens the project is happy to inherit — gating, not templates.

### Step 2 — Build the map, the theme, and the adapter selection

Decide and record, each sized to what the project actually needs:

- **Screen/flow → component map.** For each screen the project has (reconcile PRD flows with architecture routes into one list), record: the **route/path**, the bundle **layout shell** it uses, the bundle **components** that compose it (by their inventory names + the states that matter here), and any **token roles** it leans on. This is what a `feature-developer` reads to build a screen *from the system* instead of re-choosing components. Where a screen needs something the bundle lacks, mark it ⚠️ TBD and **route the gap to `ux-designer`** — don't invent a component in the map.
- **Project theme override (the brand instance).** Map the project's concrete brand values onto the bundle's semantic roles — `--color-primary: <brand>`, dark-mode values if themed, any project-specific radius/density tweak the brand calls for. **Override only what differs** from the bundle defaults; inheriting the rest is correct. Verify each overridden colour still meets the bundle's a11y/contrast baseline (re-check on-primary text, etc.) — surface a failure as an Open Question, don't ship below the floor.
- **Stack adapter selection.** Name the single adapter this project consumes (from the bundle's `adapters/`, or `tokens.css` directly) and how it's wired (the scaffolder will do the actual wiring; you record the *decision*). If the bundle has no adapter for the architecture's stack, that's a gap → route to `ux-designer` to generate it from `tokens.json` (and confirm the stack with `web-app-architect` if it's in question).

Mark anything unresolved as ⚠️ TBD inline and mirror it into Open Questions. Defer the visual *language* to `ux-designer` and the stack itself to `web-app-architect` — capture *bindings and the brand instance*, not those decisions.

### Step 3 — Realize the theme and verify against the preview

Make the binding real and checkable, not just described:

- **Write the theme-override file** at `specs/design/themes/<project>.{css,json}` — the concrete brand values layered on the bundle's tokens (CSS: a scoped block or `:root` override that *follows* the bundle's `tokens.css`; JSON: a partial that the adapter merges over `tokens.json`). Keep it an **override layer**, never a copy of the bundle's tokens — only the values that differ.
- **Preview the themed system.** The cheapest proof: open the bundle's buildless preview with your theme applied via its override hook — `preview/index.html?theme=<project>` (the bundle's preview supports an optional `?theme=` param that loads `../themes/<name>.css` over `tokens.css`). Confirm the brand reads correctly and contrast holds. If you're working with an older bundle whose preview lacks that hook, make a one-line themed copy of the preview that links both `tokens.css` and your theme file — or route to `ux-designer` to add the hook — rather than editing the bundle's own preview. You're verifying the *theme*, not rebuilding the style guide; the bundle still owns the preview.
- **Sanity-check the map.** Confirm every screen in the map names components that actually exist in the bundle inventory (no dangling references) and every overridden role exists in the bundle's tokens. A binding that points at a component the bundle doesn't have is a stale line that will get trusted — fix it (or route the gap) before handing off.

> Keep the binding doc, the theme file, and the bundle in lockstep — but only the binding doc and `themes/` are *yours*. Never edit the bundle's `tokens.*`, components, or preview to make the binding work; route bundle changes to `ux-designer`.

### Step 4 — Write the doc and hand off

1. Write `specs/DESIGN-BINDING.md` using the template below — the connecting contract, pointing at the bundle (the language) and the theme file (the brand instance).
2. Present it to the user and revise before it's relied on. The themed preview lets them *see* the project's instance of the system, not just read the map.
3. Hand off: the natural next step is `web-app-scaffolder` (which adopts the bundle **and** this binding — wiring the chosen adapter, applying the theme override, standing up the shell + mapped components), then the build loop, where every `feature-developer` slice builds its screen from the map, in this project's theme.

---

## Output: specs/DESIGN-BINDING.md

```markdown
# [Project Name] — Design Binding

> Connects the PORTABLE design bundle (specs/DESIGN.md + specs/design/) to THIS
> project. The bundle is the visual language (project-agnostic, owned by
> ux-designer); this doc is how this project uses it. Read ALONGSIDE DESIGN.md.
> Bundle changes route to ux-designer; product/stack changes to their owners.
> Keep this doc + the theme file in lockstep with the project; see Changelog.

**Status:** [Draft | Active] · **Last updated:** [date]
**Bundle:** specs/DESIGN.md (specs/design/) · **Theme:** specs/design/themes/[name].css
**Stack adapter:** [specs/design/adapters/<file> | tokens.css direct]

## 1. Binding intent
One short paragraph: which design bundle this binds, this project's brand in a
line, and the front-end stack it's wired into. Trace to PRD + ARCHITECTURE;
don't restate them.

## 2. Theme override (the brand instance)
The bundle's semantic roles → this project's concrete values (primary, dark-mode
values if themed, any project tweak). Only what differs from bundle defaults;
the rest is inherited. The file in the header is the machine-readable source.
Contrast against the a11y baseline confirmed: [yes/notes].

## 3. Stack adapter
Which bundle adapter this project consumes and how the scaffolder wires it.
(If the bundle lacks an adapter for the stack → Open Question, routed to
ux-designer to generate from tokens.json.)

## 4. Screen / flow → component map
Per screen: route/path · bundle layout shell · bundle components (by inventory
name + states that matter here) · token roles it leans on · source flow (PRD
FR-0X) and architecture route. This is what feature-developer reads to build a
screen from the system. Mark screens needing a missing component ⚠️ TBD and
route the gap to ux-designer.

## 5. Open questions
Unresolved binding decisions with impact and status (⚠️ TBD mirrors here).
Bundle gaps route to ux-designer; product gaps to product-requirements; stack
gaps to web-app-architect — note where each routed.

## 6. Changelog
Dated entries for every change to the binding (and whether the theme file and
adapter pointer were updated to match).
```

## Scope boundary (important)

This skill owns the **project-specific binding** — not the visual language, the product, the feature behaviour, or the stack.

- **Up (visual language):** don't define or change tokens, components, or the design language — that's `ux-designer`. Map and theme *existing* bundle pieces; a missing component/token is a bundle gap → route to `ux-designer`. Never edit the bundle's `tokens.*`/components/preview.
- **Up (product):** don't decide persona, scope, or flows — that's `product-requirements`. Map the PRD's flows to components; a missing flow routes back to the PRD.
- **Up (technical):** don't decide the stack — that's `web-app-architect`. *Select* the matching bundle adapter; route a stack with no adapter to `ux-designer` (generate it) and/or confirm the stack with `web-app-architect`.
- **Sideways (feature behaviour):** don't specify a feature's states/rules/flow — that's `feature-spec`. Say which component renders a screen, in this theme; let the feature spec say when each state applies.
- **Down:** you own `DESIGN-BINDING.md` and the theme-override file under `specs/design/themes/`. The scaffolder adopts both (with the bundle); feature-developer conforms. Keep them in lockstep with the project; never let the binding point at a bundle piece that doesn't exist.

## Maintenance mode rules

- Read `DESIGN-BINDING.md` (and glance at the theme file + the bundle inventory) before changing the binding; treat it as the agreed wiring.
- On any change, update the relevant section **and** the theme file **and** the adapter pointer to match, then add a Changelog entry. The doc, the theme, and the project must never drift apart — a stale screen→component line gets trusted and breaks a slice.
- If a feature-developer (or scaffolder) routes a gap back here — a screen with no mapping, a token role with no project value — resolve it in the binding (map the screen, set the override), update the doc + theme + Changelog, then hand back.
- If a gap is really a **bundle** gap (a component/token the system itself lacks), route it to `ux-designer`, not into the binding. If it's a product or stack change, route to the owning skill. The binding never quietly redefines the language, the scope, or the stack.
- Never edit the bundle to make the binding work. If the bundle changes (ux-designer adds a component, renames a token), reconcile the binding to it in the same change — the bundle leads, the binding follows.

## Examples

**Example 1 — binding a portable bundle to a project (the normal case)**
A portable bundle exists (calm, minimal, light+dark, components: button/input/card/table/reminder-timeline) and a PRD for a freelancer invoice tracker. The user says "wire the design system up to this project." The skill reads the bundle (the available components/tokens), the PRD (flows: invoice list → detail → reminders; brand: the user wants a teal primary), and the architecture (React + Tailwind). It writes `specs/design/themes/invoicetracker.css` overriding `--color-primary` to the brand teal (re-checking on-primary contrast holds AA), selects `adapters/tailwind.tokens.js` as the stack adapter, and writes `DESIGN-BINDING.md` mapping each screen — `/invoices` → app-shell + table + card in their list/empty/loading states; `/invoices/:id` → detail layout + the reminder-timeline; `/invoices/new` → the form components — each traced to its PRD flow. It previews the bundle style guide with the teal theme applied, confirms it reads right, and hands off to `web-app-scaffolder`. The bundle is untouched and still exportable; this project is now fully wired.

**Example 2 — declining to build a binding**
The product is an internal admin panel; `ux-designer` was rightly skipped (it adopts an off-the-shelf kit as-is) so there's no `specs/design/` bundle. The user asks to "bind the design system." The skill declines: there's nothing portable to bind. It explains the binding only exists to connect a generic bundle to a project, points at the architecture's "use [kit] defaults" note as already sufficient, and offers nothing more — rather than manufacturing a map of components the project gets from the kit.

**Example 3 — routing a bundle gap, not inventing it**
While mapping the project's screens, a "settings" screen needs a **tabs** component the bundle never defined. Per the seam, the binder does **not** invent tabs in the map. It marks that screen ⚠️ TBD, records an Open Question, and **routes the gap to `ux-designer`** (which adds tabs to the bundle inventory + the style-guide preview). Once the bundle has tabs, the binder maps the settings screen to it and clears the TBD. The new component entered the *portable system* (reusable, exportable) instead of being a one-off baked into one project's binding.

**Example 4 — maintain mode, a brand change**
The user says "the brand is changing from teal to indigo." The skill is in maintain mode: it updates the project theme file (`themes/invoicetracker.css`) primary value to indigo **and** re-checks contrast against the bundle's a11y baseline (adjusting on-primary text if needed) **and** updates `DESIGN-BINDING.md` §2 and the Changelog. It does **not** touch the bundle's `tokens.*` (those stay the generic source of truth) and does **not** touch feature code that consumes the semantic token — the override flows through. The bundle remains exportable to other projects, still teal-neutral; only *this* project's instance moved to indigo.

**Example 5 — same bundle, second project**
A second app inherits the first's portable bundle (copied per `BUNDLE.md`). There's no binding to reuse — the binding is project-specific. The user runs `design-binder` in the new repo: same bundle, new PRD (a different persona, different flows), new brand (a warm orange), possibly a different stack (Vue → `tokens.css` direct). The skill writes a *fresh* `DESIGN-BINDING.md` + `themes/<newproject>.css` for this project, mapping its own screens to the same shared components. Two projects, one inherited design language, two bindings — exactly the split this skill exists for.
