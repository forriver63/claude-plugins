---
name: load
description: "List and load saved contexts, or drop ones you no longer need"
argument-hint: "[keyword | --drop | --all]"
allowed-tools: ["Bash(ls:*)", "Bash(date:*)", "Bash(rm:*)", "Bash(wc:*)", "Bash(basename:*)", "Bash(pwd:*)", "Bash(git:*)", "Read", "Glob", "AskUserQuestion"]
---

# Load Working Context

You help the user find and load a saved context from the cloud-backed store at `~/.claude/ctx-store/`, scoped to the current project.

## Arguments

- `/ctx:load` — List contexts for the **current project**, let user pick
- `/ctx:load <keyword>` — Filter by keyword (filename or content) within current project
- `/ctx:load --all` — List contexts across **all projects**
- `/ctx:load --drop` — List and pick one to delete (also removes from cloud)

## Step 1: Sync from Cloud

Always pull first so this window sees saves made elsewhere:

```
cd ~/.claude/ctx-store && git pull --rebase --autostash origin main
```

If pull fails (no network, conflicts), warn the user and continue with local files.

## Step 2: Resolve Scope

Determine project slug:
```
basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
```

Look in `~/.claude/ctx-store/<slug>/*.md` (or all slugs if `--all`).

If empty / missing → tell user: "No saved contexts found for this project. Use `/ctx:save` or `/ctx:fork` to create one."

## Step 3: Display List

Sort by mtime (newest first):

```
Available contexts (project: <slug>):
─────────────────────────────────────────────────────────────
 #  Type     Date        Name                          Saved
─────────────────────────────────────────────────────────────
 1  handoff  2026-04-26  call-router-webhook-timeout   2h ago
 2  fork     2026-04-26  fork-config-timeout-api       3h ago
 3  handoff  2026-04-25  whatsapp-retry-logic          yesterday
─────────────────────────────────────────────────────────────
```

Rules:
- **Type**: "fork" if filename contains `-fork-` or starts with `fork-` after the date prefix, else "handoff"
- **Date**: parse `YYYY-MM-DD` prefix from filename
- **Name**: filename without date prefix and `.md` extension
- **Saved**: relative time of file mtime
- If `--all`: add a "Project" column showing the slug
- If keyword arg provided: filter

If `--drop`: ask which # to delete, then `rm` it AND commit+push the deletion.

## Step 4: User Selects

Use AskUserQuestion to pick by number or name.

## Step 5: Load Context

1. Read selected `.md`
2. Present clearly:

**Handoff**: "I've loaded the context. Here's where things left off:" → summarize what was being done, progress, next steps. Ask: "Should I continue from where this left off?"

**Fork**: "I've loaded a fork task. Here's what needs to be done:" → summarize task, files, acceptance criteria. Ask: "Should I start working on this?"

## Step 6: Orient

After user confirms, read the key files listed. The context describes the past — verify current code state before acting.

## Critical Rules

- **Always pull first** — keep local in sync with cloud
- **Never auto-load** — show list, let user choose
- **Context is a hint** — verify current state before making changes
- **For --drop**: after `rm`, run `git add -A && git commit -m "drop: ..." && git push` in `~/.claude/ctx-store`
