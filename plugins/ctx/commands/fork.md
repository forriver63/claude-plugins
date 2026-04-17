---
name: fork
description: "Fork a subtask from current work for parallel development in another window"
argument-hint: "<subtask description>"
allowed-tools: ["Bash(mkdir:*)", "Bash(ls:*)", "Bash(date:*)", "Bash(git branch:*)", "Bash(git status:*)", "Read", "Write", "Glob", "Grep"]
---

# Fork Subtask

You are extracting a subtask from the current work so it can be done in a separate Claude Code window, in parallel with the current window.

## Arguments

The user provides a brief description of the subtask to fork. For example:
- `/ctx:fork config-service 加 timeout API 端点`
- `/ctx:fork 修复 login 页面的表单验证`
- `/ctx:fork 写 call-router 的单元测试`

If no argument is provided, ask the user what subtask they want to fork.

## Step 1: Understand the Subtask

From the user's description and the current conversation context, determine:

1. **What needs to be done** — The specific deliverable
2. **Why** — How this subtask relates to the parent task
3. **Scope boundary** — What is IN scope and what is NOT (the other window should not touch files outside its scope)
4. **Key files** — Only the files relevant to this subtask (not the full list from the parent task)
5. **Dependencies** — Does this subtask depend on anything from the parent task? Does the parent task need this subtask's output?
6. **Acceptance criteria** — How does the other window know it's done?

## Step 2: Generate Name

Create a descriptive filename. Rules:
- Prefix with `fork-` to distinguish from save contexts
- Use kebab-case, max 5 words after prefix
- Example: `fork-config-timeout-api`, `fork-login-form-validation`

## Step 3: Write Fork File

1. Create the contexts directory if it doesn't exist:
   ```
   mkdir -p .claude/contexts
   ```

2. Write to `.claude/contexts/<name>.md`:

```markdown
# [FORK] <One-line summary of the subtask>

- **Forked from**: <parent task summary>
- **Forked at**: <current datetime>
- **Branch**: <suggest a branch name, e.g., feat/timeout-api>
- **Type**: fork

## Your Task

<Clear, actionable description of what needs to be done. Write this as if briefing a colleague who just walked in — they don't have the conversation history.>

## Context You Need

<Only the background information required for THIS subtask. Don't dump the entire parent context. Include:>
- Relevant architecture decisions already made
- API contracts or interfaces this subtask must conform to
- Existing code patterns to follow

## Key Files

<Only files relevant to this subtask>
- `path/to/file.ts` — <why this file matters for the subtask>

## Out of Scope

<Explicitly state what this window should NOT touch>
- Do not modify <files being handled by the parent window>

## Acceptance Criteria

<How to know the subtask is complete>
1. <Specific, verifiable criterion>
2. <Another criterion>

## When Done

<What to report back to the parent window>
- The exact file paths created/modified
- Any API contracts or interfaces defined (so the parent can consume them)
- Any decisions made that affect the parent task
```

3. If a file with the same name exists, append a number.

## Step 4: Confirm

Tell the user:
- The fork filename
- A one-line summary of the subtask
- Suggested branch name
- How to load: `/ctx:load` in another window
- Remind: "When the subtask is done, use `/ctx:save` in that window to save the result"

## Critical Rules

- **Minimize context** — Only include what the subtask needs. Less is more. The other window should not be overwhelmed.
- **Be explicit about boundaries** — The "Out of Scope" section prevents two windows from editing the same files and creating merge conflicts.
- **Define the interface** — If the parent and fork need to connect (e.g., one builds an API, the other consumes it), specify the contract upfront.
