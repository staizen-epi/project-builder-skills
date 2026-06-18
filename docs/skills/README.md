# Skill reference

One page per skill — how it works, sample prompts, what it produces, and who depends on it.

[← Back to the main overview](../../README.md)

| Skill | Stage | One line |
|---|---|---|
| [`product-requirements`](product-requirements.md) | 1 · define the product | What & why → `specs/PRD.md`, the highest-level source of truth |
| [`web-app-architect`](web-app-architect.md) | 2 · design the build | How it's built → `specs/ARCHITECTURE.md`, gated to what the app needs |
| [`quality-requirements`](quality-requirements.md) | *(optional)* alongside the architect | Non-functional targets (NFRs + GDPR) → `specs/QUALITY.md` |
| [`poc-developer`](poc-developer.md) | *(optional)* spike | A throwaway mock-data glimpse → `/poc/` + `specs/POC-NOTES.md` |
| [`ux-designer`](ux-designer.md) | *(optional)* design layer | A **portable** design system → `specs/DESIGN.md` + `specs/design/` |
| [`design-binder`](design-binder.md) | *(optional)* design layer | Wires the bundle to **this** project → `specs/DESIGN-BINDING.md` + theme |
| [`web-app-scaffolder`](web-app-scaffolder.md) | 3 · stand up the foundation | Phase-0 foundation, proven → your codebase |
| [`feature-spec`](feature-spec.md) | 4a · build loop (spec) | One feature in depth → `specs/features/<feature>.md` |
| [`feature-developer`](feature-developer.md) | 4b · build loop (build) | A tested vertical slice → code + tests + `specs/REUSE.md` |

**Pipeline:** PRD → architecture → *(optional)* quality → *(optional)* PoC → *(optional)* design system → design binding → scaffold → build loop (spec → build, repeat).
