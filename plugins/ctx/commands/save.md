---
name: save
description: "Save current working context for handoff to another window"
argument-hint: "[focus description]"
allowed-tools: ["Bash(mkdir:*)", "Bash(ls:*)", "Bash(date:*)", "Bash(git branch:*)", "Bash(git status:*)", "Bash(git diff:*)", "Read", "Write", "Glob", "Grep"]
---

# Save Working Context

You are saving the current working context so another Claude Code window can pick up where this one left off.

## Arguments

- `/ctx:save` — Save full context (everything in this conversation)
- `/ctx:save <focus>` — Save only the part related to the focus description. Ignore conversation content unrelated to the focus.

Examples:
- `/ctx:save` → full context
- `/ctx:save call-router webhook 部分` → only webhook-related work
- `/ctx:save 只保存 schema 修改的进度` → only schema changes

## Step 1: Gather Context

If the user provided a focus description, **only collect information relevant to that focus**. Skip anything unrelated.

If no focus was provided, collect everything.

Collect the following information by reading the conversation history and running commands:

1. **What is the user working on?** — Summarize the goal/task in one sentence
2. **Current git branch** — Run `git branch --show-current`
3. **Modified files** — Run `git status --short` to see uncommitted changes
4. **Key files involved** — Which files has the user been reading/editing in this conversation?
5. **Current progress** — What has been done so far? What percentage complete?
6. **Blockers or open questions** — Is anything stuck or unclear?
7. **Next steps** — What should be done next, in order?
8. **Active plan file** — Check if there's a plan file in `.claude/plans/` that's relevant. If so, read it and include its path.

## Step 2: Generate a Descriptive Name

Create a short, descriptive filename from the work being done. Rules:
- Use kebab-case, max 5 words
- Format: `<module>-<action>-<detail>` (e.g., `call-router-webhook-timeout`, `assistant-api-voice-config`, `whatsapp-retry-logic`)
- Must be specific enough that the user can recognize it in a list without opening the file
- Do NOT use generic names like `bugfix`, `feature`, `update`
- Do NOT use dates or random strings

## Step 3: Write Context File

1. Create the contexts directory if it doesn't exist:
   ```
   mkdir -p .claude/contexts
   ```

2. Write the context file to `.claude/contexts/<name>.md` with this format:

```markdown
# <One-line summary of the task>

- **Saved**: <current datetime>
- **Branch**: <git branch>
- **Status**: <in-progress | blocked | ready-for-review>
- **Progress**: <X% — brief description of what's done>

## What I Was Doing

<2-3 sentences explaining the task, why it matters, and the approach taken>

## Changes So Far

<List of modified/created files with one-line description of each change>

## Current State

<Where exactly did work stop? What was the last thing done?>

## Blockers / Open Questions

<Any unresolved issues, or "None">

## Next Steps

1. <Specific next action>
2. <Next action after that>
3. ...

## Key Files

<List of files that the next window should read first to get oriented>

## Plan File

<Path to .claude/plans/*.md if one exists, or "None">
```

3. If a file with the same name already exists, append a number: `<name>-2.md`, `<name>-3.md`

## Step 4: Confirm

Tell the user:
- The filename that was saved
- A one-line summary of what was captured
- How to load it: `/ctx:load` in another window
