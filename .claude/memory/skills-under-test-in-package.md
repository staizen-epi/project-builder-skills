---
name: skills-under-test-in-package
description: In this repo the skills being developed/tested live in package/skills, not .claude/skills
metadata:
  type: project
---

In this maintainer repo, the skills under development and test are in `package/skills/<skill>/SKILL.md` — that is the source of truth. When the user asks to test, dry-run, edit, or improve a skill here, read and modify the copy under `package/skills/`, not any installed copy.

**Why:** the skills were moved into `package/` so the whole set can be exported wholesale into other projects (copy `package/skills/` → a target project's `.claude/skills/`). The repo root holds maintainer context (`CLAUDE.md`, `README.md`); the exportable payload lives under `package/`. See [[sync-skills-via-git-diff]] for how changed skills get mirrored out.

**How to apply:** to test an updated skill in this repo, point at `package/skills/<skill>/SKILL.md`. Don't expect a top-level `.claude/skills/` here — there isn't one; the skills live in `package/skills/`.
