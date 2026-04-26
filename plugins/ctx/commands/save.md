---
name: save
description: "Save current working context for handoff to another window"
argument-hint: "[focus description]"
allowed-tools: ["Bash(mkdir:*)", "Bash(ls:*)", "Bash(date:*)", "Bash(basename:*)", "Bash(pwd:*)", "Bash(git:*)", "Read", "Write", "Glob", "Grep"]
---

# Save Working Context

You are saving the current working context so another Claude Code window can pick up where this one left off.

Contexts are stored in a private cloud-backed repo at `~/.claude/ctx-store/`, organized per project. Files NEVER live inside the project repo, so the main repo stays clean.

## Arguments

- `/ctx:save` — Save full context (everything in this conversation)
- `/ctx:save <focus>` — Save only the part related to the focus description.

## Step 1: Gather Context

If the user provided a focus description, **only collect information relevant to that focus**.

Collect:
1. **What is the user working on?** — One-sentence goal/task summary
2. **Current git branch** — `git branch --show-current`
3. **Modified files** — `git status --short`
4. **Key files involved** — files read/edited in this conversation
5. **Current progress** — what's done, rough %
6. **Blockers / open questions**
7. **Next steps** in order
8. **Active plan file** — check `.claude/plans/` and include path if relevant

## Step 2: Resolve Project Slug

Determine which project this context belongs to. Run:

```
basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
```

Sanitize to kebab-case (lowercase, replace spaces/underscores with `-`). This is the **project slug**.

## Step 3: Generate Filename

Format: `YYYY-MM-DD-<keywords>.md`

- Date: `date +%Y-%m-%d`
- Keywords: 2–5 kebab-case words describing the work (e.g., `call-router-webhook-timeout`, `whatsapp-retry-logic`)
- Be specific — no `bugfix`, `feature`, `update`
- Example: `2026-04-26-call-router-webhook-timeout.md`

## Step 4: Write Context File

1. Ensure storage dir exists:
   ```
   mkdir -p ~/.claude/ctx-store/<project-slug>
   ```

2. Write to `~/.claude/ctx-store/<project-slug>/<filename>.md`:

```markdown
# <One-line summary>

- **Saved**: <datetime>
- **Project**: <project-slug>
- **Branch**: <git branch>
- **Status**: <in-progress | blocked | ready-for-review>
- **Progress**: <X% — brief>

## What I Was Doing

<2-3 sentences>

## Changes So Far

<modified/created files, one-line each>

## Current State

<where work stopped, last thing done>

## Blockers / Open Questions

<or "None">

## Next Steps

1. ...
2. ...

## Key Files

<files the next window should read first>

## Plan File

<path to .claude/plans/*.md or "None">
```

3. If filename collides, append `-2`, `-3`, etc.

## Step 5: Sync to Cloud

```
cd ~/.claude/ctx-store
git pull --rebase --autostash origin main
git add <project-slug>/<filename>.md
git commit -m "save: <project-slug>/<filename>"
git push origin main
```

If push fails, tell the user — don't retry destructively.

## Step 6: Confirm

Tell the user:
- Saved path: `~/.claude/ctx-store/<project-slug>/<filename>.md`
- Pushed to `forriver63/ctx-store` (private)
- One-line summary
- How to load elsewhere: `/ctx:load`
