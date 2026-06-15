---
name: sync-skills-via-git-diff
description: When syncing skills to ~/.claude, use git diff to find changed skills — ideally every commit equates to a sync
metadata:
  type: feedback
---

When syncing the skills out to `~/.claude/skills/`, use git as the reference for *what changed* rather than diffing folders ad hoc. Ideally every commit corresponds to a sync: the commit is the unit of change, so the skills touched in a commit (or since the last sync) are exactly the ones to copy out.

**Why:** the project (`.claude/skills/`, committed) is the source of truth; personal (`~/.claude/skills/`) is the reusable mirror, kept in sync manually. Tying syncs to commits keeps the mirror tracking the source predictably instead of relying on folder-diffs of a possibly-dirty working tree.

**How to apply:** to find what to sync, look at git — e.g. `git show --stat <commit>` or `git diff --name-only <last-synced>..HEAD -- .claude/skills/` — and copy out the skill folders those paths touch. After a commit that changes skills, sync those skills. `rsync -a --delete <skill>/ ~/.claude/skills/<skill>/` mirrors one folder exactly. Edits to existing SKILL.md are picked up live; brand-new skill folders need a Claude Code restart to be watched.
