---
name: scan-local-claude-skills
description: On every prompt, check the project-local .claude/skills first and prefer a matching skill over an ad-hoc approach
metadata:
  type: feedback
---

On every prompt in this project, scan the project-local `.claude/skills/` first and consider whether any skill applies before responding. Prefer invoking a matching local skill over an ad-hoc approach.

**Why:** This project was set up from the spec-driven build-skills package (`package/skills/` copied into `.claude/skills/`). Those skills are the intended way to take work from idea → spec → architecture → scaffold → feature build, and they're easy to overlook if you don't look for them.

**How to apply:** At the start of handling each prompt, check `.claude/skills/` for an available skill that matches the request (e.g. writing a PRD, designing architecture, scaffolding, building a feature) and invoke it instead of improvising.
