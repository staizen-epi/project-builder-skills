---
name: sync-skills-via-git-diff
description: Git-diff check applies ONLY to syncing this repo to personal ~/.claude/skills — NOT to syncing into other working-directory projects
metadata:
  type: feedback
---

The git-diff reference for *what changed* applies **only** when syncing this repo out to the **personal** skills mirror at `~/.claude/skills/`. Ideally every commit corresponds to such a sync: the commit is the unit of change, so the skills touched since the last sync are exactly the ones to copy out.

**Boundary (important):** do **not** run a git-history check when syncing into some **other working-directory project** (e.g. `00_Workspace/<project>/.claude/skills/`). That's too much context to do per target. For those, just mirror the `package/skills/` folder directly — a plain folder-diff + `rsync` is enough; no need to reconcile against commit history. The git-check is the personal-mirror discipline, not a universal sync rule.

**Why:** the project (`package/skills/`, committed) is the source of truth; the **personal** mirror is the reusable copy kept in sync manually, so tying *its* syncs to commits keeps it tracking the source predictably. Other project directories are just consumers — copying the current source into them is the whole job; their own git state isn't ours to reconcile. See [[skills-under-test-in-package]].

**How to apply:**
- **→ personal `~/.claude/skills/` (git-check ON):** find what to sync via git — `git show --stat <commit>` or `git diff --name-only <last-synced>..HEAD -- package/skills/` — and copy out the skill folders those paths touch. `rsync -a --delete <skill>/ ~/.claude/skills/<skill>/` mirrors one folder exactly.
- **→ another project's `.claude/skills/` (git-check OFF):** just `rsync -a --delete-excluded --exclude='.DS_Store' package/skills/ <project>/.claude/skills/` (and `package/memory/` → `.claude/memory/`). No commit-history reconciliation.
- Either way: edits to existing SKILL.md are picked up live; brand-new skill folders need a Claude Code restart to be watched.
