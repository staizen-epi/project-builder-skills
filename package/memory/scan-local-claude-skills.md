---
name: scan-local-claude-skills
description: Always scan for local .claude skills on every prompt in the inspo-wallpaper project
metadata: 
  node_type: memory
  type: feedback
  originSessionId: bb8d6ee6-dfd7-44fe-a9d7-44a785bc368d
---

On every prompt in this project directory, scan for local `.claude` skills (e.g. `.claude/skills/`) and consider whether any apply before responding.

**Why:** The user wants project-local skills to be reliably discovered and used rather than overlooked.

**How to apply:** At the start of handling each prompt in this project, check the project's `.claude` directory for available skills and prefer invoking a matching one over an ad-hoc approach.
