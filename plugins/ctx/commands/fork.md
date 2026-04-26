---
name: fork
description: "Fork a subtask from current work for parallel development in another window"
argument-hint: "<subtask description>"
allowed-tools: ["Bash(mkdir:*)", "Bash(ls:*)", "Bash(date:*)", "Bash(basename:*)", "Bash(pwd:*)", "Bash(git:*)", "Read", "Write", "Glob", "Grep"]
---

# Fork Subtask

You are extracting a subtask from the current work so it can be done in a separate Claude Code window, in parallel with the current window.

Forks are stored in the same private cloud-backed repo as saves: `~/.claude/ctx-store/<project-slug>/`. They never live inside the project repo.

## Arguments

User provides a brief description of the subtask. Examples:
- `/ctx:fork config-service 加 timeout API 端点`
- `/ctx:fork 修复 login 页面的表单验证`
- `/ctx:fork 写 call-router 的单元测试`

If no argument, ask what subtask to fork.

## Step 1: Understand the Subtask

Determine:
1. **What needs to be done** — specific deliverable
2. **Why** — relation to parent task
3. **Scope boundary** — IN vs NOT IN
4. **Key files** — only those relevant to this subtask
5. **Dependencies** — what this needs from parent / what parent needs back
6. **Acceptance criteria**

## Step 2: Resolve Project Slug

```
basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
```

Sanitize to kebab-case lowercase.

## Step 3: Generate Filename

Format: `YYYY-MM-DD-fork-<keywords>.md`

- Date: `date +%Y-%m-%d`
- Keywords: 2–5 kebab-case words after `fork-`
- Example: `2026-04-26-fork-config-timeout-api.md`

## Step 4: Write Fork File

1. Ensure dir:
   ```
   mkdir -p ~/.claude/ctx-store/<project-slug>
   ```

2. Write to `~/.claude/ctx-store/<project-slug>/<filename>.md`:

```markdown
# [FORK] <One-line summary>

- **Forked from**: <parent task summary>
- **Forked at**: <datetime>
- **Project**: <project-slug>
- **Branch**: <suggested branch name>
- **Type**: fork

## Your Task

<Clear actionable description. Brief a colleague who has no conversation history.>

## Context You Need

<Only background needed for THIS subtask>
- Architecture decisions already made
- API contracts to conform to
- Existing patterns to follow

## Key Files

- `path/to/file.ts` — <why it matters>

## Out of Scope

- Do not modify <files handled by parent window>

## Acceptance Criteria

1. <verifiable criterion>
2. ...

## When Done

- File paths created/modified
- API contracts/interfaces defined
- Decisions affecting parent task
```

3. If filename collides, append `-2`, `-3`, etc.

## Step 5: Sync to Cloud

```
cd ~/.claude/ctx-store
git pull --rebase --autostash origin main
git add <project-slug>/<filename>.md
git commit -m "fork: <project-slug>/<filename>"
git push origin main
```

## Step 6: Confirm

Tell the user:
- Fork filename and path
- Pushed to `forriver63/ctx-store` (private)
- Suggested branch name
- One-line summary
- How to load: `/ctx:load` in another window
- Reminder: "When subtask is done, `/ctx:save` in that window to save the result"

## Critical Rules

- **Minimize context** — only what the subtask needs
- **Be explicit about boundaries** — prevent merge conflicts between windows
- **Define the interface** — if parent and fork need to connect, specify the contract
