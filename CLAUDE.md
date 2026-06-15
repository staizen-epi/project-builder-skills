# CLAUDE.md — Spec-Driven Build Skills (project context)

Context for continuing this project in Claude Code. This repo is a **suite of Claude Code skills** that take a web app from idea → running foundation, spec-first. When extending it (new skills, edits, fixes), follow the conventions below so the set stays coherent.

`README.md` is the **user-facing** usage guide (how to talk to the skills). This file is the **builder/maintainer** context (how the skills are designed and how to add more). Keep that separation.

## What's here

Five skills, each a folder with a `SKILL.md`. They chain by reading/writing shared files:

| Skill | Stage | Reads | Writes |
|---|---|---|---|
| `product-requirements` | Define the product | conversation | `specs/PRD.md` |
| `feature-spec` | Detail one feature | `specs/PRD.md` | `specs/features/<feature>.md` |
| `web-app-architect` | Design the build | `specs/PRD.md` | `specs/ARCHITECTURE.md` |
| `poc-developer` | Spike a throwaway glimpse | `specs/ARCHITECTURE.md` (soft) | `/poc/` (throwaway) + `specs/POC-NOTES.md` |
| `web-app-scaffolder` | Stand up Phase 0 | `specs/ARCHITECTURE.md` + `specs/POC-NOTES.md` (hints) | the codebase |

Pipeline: **PRD → feature specs → architecture → (optional PoC) → scaffold → build**. The PoC is an optional, disposable step between architecture and scaffold: it shows what the app could feel like on mock data, and leaves `specs/POC-NOTES.md` as hints so the scaffolder starts closer to reality. The `/poc/` code itself is throwaway — the scaffolder reads only the notes, never the spike code. All durable artifacts live under one `specs/` folder in the target project.

```
specs/
├── PRD.md                  # what & why (highest level)
├── features/<feature>.md   # how each complex feature behaves (children of the PRD)
├── ARCHITECTURE.md         # how it's built
└── POC-NOTES.md            # hints from a throwaway spike, for the scaffolder (optional)
/poc/                       # disposable mock-data prototype (not under specs/; deletable)
```

## Design principles (the spine — apply to every skill, existing and new)

1. **Gating, not templates.** A skill includes only what the app needs. Patterns (multi-tenancy, queues, caching, even a feature's own file) are switched on by explicit conditions and omitted otherwise. Cheaper to add later than to rip out. When unsure, default to the simpler/off option.
2. **One concept, one owner; clean seams.** Product scope → PRD. Feature behaviour → feature specs. Technical design → architecture. Disposable proof-of-concept → the PoC (`/poc/` code + `POC-NOTES.md` hints). Implementation → (future) developer skill. No skill redoes another's job; they **link and trace**, never duplicate. If work surfaces a decision that belongs upstream, route it back to the owning skill rather than diverging silently.
3. **Ask only what matters, then assume.** Each bootstrap uses an **impact-tiered** gather: a few high-impact questions asked only if undetermined (hard "ask before generating" threshold), everything else defaulted with the assumption recorded. No long forms.
4. **Read upstream first (Step 0).** A skill reads the previous stage's artifact before asking anything, so it never re-asks what's already settled (e.g. the architect derives project name, auth, tenancy, integrations from the PRD).
5. **Prove it.** "Done" means observable, not "files exist." Outputs carry acceptance criteria / checkable conditions; the scaffolder must boot, pass the CI gate, and meet the architecture's Phase-0 criteria.
6. **Traceability.** Stable IDs and parent/child links. PRD requirements are `FR-0X`; feature specs extend them as `FR-0X.Y`; the PRD §7 index links to each feature spec. `⚠️ TBD` inline mirrors into an Open Questions register. Every doc has a Changelog.
7. **Two modes.** Each skill checks for its output file: absent → **bootstrap**; present → **consult & maintain** (update the relevant section + Changelog, never let code and doc drift).
8. **Disposable work hands off via a cheap interface, not via its output.** Where a stage produces throwaway artifacts (the PoC's `/poc/` spike), the *next* stage consumes a small, durable hints doc (`POC-NOTES.md`) — never the throwaway code itself. This keeps the seam clean and is the token/quota-efficient choice: re-reading and salvaging a spike's source (then ripping its shortcuts back out) is the expensive path. The advisory doc never overrides an upstream owner's doc — on conflict, the authoritative spec (e.g. `ARCHITECTURE.md`) wins.

## SKILL.md authoring conventions

- **Frontmatter:** `name` (kebab-case, matches the folder) + a **deliberately pushy `description`**. Descriptions under-trigger by default, so state *what it does*, *when to use it*, concrete trigger phrases, how it disambiguates from sibling skills, and its precondition (which file must exist). Include examples — they measurably improve activation.
- **Length:** keep `SKILL.md` under ~500 lines. Move depth into a `resources/` subfolder referenced situationally ("when X, consult resources/y.md"). Only `name` + `description` load until the skill triggers.
- **Standard sections we use:** precondition / two-modes; Step 0 (read upstream); impact-tiered gather (high-impact vs lower-impact); a decision-gate table or gating rule; a verifiable output template; a scope boundary ("up/down/sideways — what this skill does NOT own"); maintenance rules; examples — including at least one example of the skill **declining** to over-do its job (e.g. not speccing a trivial feature, not bolting on multi-tenancy).
- **Shape:** structured input → checkable output; compose via shared files under `specs/`.

## How we develop/improve skills (the loop)

1. Read the skill-creator conventions (`/mnt/skills/examples/skill-creator/SKILL.md` in the authoring environment) before writing a `SKILL.md`.
2. Draft the skill, then **write 1–2 test prompts and dry-run them** by following the SKILL.md as if you were the agent. Pick prompts that pull behaviour in opposite directions (a gates-heavy case and a gates-light/ambiguous case).
3. Note the rough edges and **patch the SKILL.md**. This loop already surfaced and fixed: an explicit ask-vs-assume threshold, handling of "light"/borderline gates, an audit-logging gate, leading the questionnaire with high-impact gates + biasing to the simpler default, and — when adding `poc-developer` — making the throwaway/durable seam explicit so the scaffolder reads only `POC-NOTES.md` (not the spike code) and the architecture wins on conflict.
4. After any change to a description or gate, re-run the relevant test prompt as a regression check.

## Roadmap — planned skills

These extend the pipeline past scaffold into implementation, quality, acceptance, and release. Build each on the same spine (read the upstream artifact, gate, prove it, two modes, clean seams). Rough suggested order: developer → QA → UAT → release.

- **`feature-developer`** — implements one feature from its `specs/features/<feature>.md` + `specs/ARCHITECTURE.md`, as a vertical slice that honours the architecture's invariants/gates and the spec's acceptance criteria. Reads: the feature spec + architecture. Writes: code + tests. Seam: builds to spec; if the spec is wrong, route back to `feature-spec`, don't improvise scope.
- **`qa`** — the "QA gauntlet": authors/runs the quality gate against the architecture's invariants and gates (lint, typecheck, tests, security/secret scan, tenant-isolation tests where applicable), restarting the full pass on any failure until green. Reads: architecture + codebase. Writes: a test/quality report; enforces the gate.
- **`uat`** — turns acceptance criteria from the PRD and feature specs into a user-runnable UAT checklist, and records sign-off before release. Reads: PRD + feature specs (their acceptance criteria). Writes: a UAT checklist + sign-off record. Seam: validates against stated criteria; doesn't invent new ones (gaps go back to the PRD/feature spec).
- **`release-manager`** — promotes through environments (local → staging → production) only when the QA gate is green; manages versioning, changelog aggregation, and the deploy flow. Reads: architecture's environments/release section + QA status. Writes: release notes / version bumps; performs the promotion. Mind the action-safety boundary — destructive or irreversible deploy steps need explicit human confirmation.

## Open items / TODOs

- **Plugin packaging:** bundle all **five** skills into one installable Claude Code plugin (add `.claude-plugin/plugin.json`; optionally a marketplace entry) so they install/update as a unit instead of loose folders. Don't forget `poc-developer` when packaging.
- **Install location:** *(resolved)* the skills live both in the project (`.claude/skills/`, committed) and in personal (`~/.claude/skills/`, reusable everywhere); the two are kept in sync manually (project is the source of truth — copy changed/new skills out to personal). They stack, so both being present is fine. New skill folders need a Claude Code restart to be watched; edits to existing `SKILL.md` are picked up live.
- **feature-spec presentation:** README orders it before architecture (dependency order). In practice features are often spec'd lazily after scaffolding — consider noting that. Minor.
- **PoC convention reminder:** the `/poc/` spike is throwaway and lives *outside* `specs/`; only `specs/POC-NOTES.md` is durable. When the (future) `feature-developer`/`qa` skills land, they should also ignore `/poc/` and never treat its shortcuts as real.
- Re-run regression dry-runs after any description/gate edits.

## Continuing in Claude Code

Open this repo in Claude Code and just describe the next step, e.g.:
- "Build the `feature-developer` skill following the conventions in CLAUDE.md."
- "Improve the architect's gate table — add a real-time/websocket example."
- "Package all the skills as a single plugin."
- "Dry-run the feature-spec skill against an example PRD and find rough edges."

Claude will have this file as context and should keep new work consistent with the principles and authoring conventions above.
