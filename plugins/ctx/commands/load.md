---
name: load
description: "List and load saved contexts, or drop ones you no longer need"
argument-hint: "[keyword | --drop]"
allowed-tools: ["Bash(ls:*)", "Bash(date:*)", "Bash(rm:*)", "Bash(wc:*)", "Read", "Glob", "AskUserQuestion"]
---

# Load Working Context

You help the user find and load a saved context from `.claude/contexts/`.

## Arguments

- `/ctx:load` — List all available contexts, let user pick one
- `/ctx:load <keyword>` — Filter contexts by keyword (matches filename or content)
- `/ctx:load --drop` — List contexts and let user pick one to delete

## Step 1: Find Contexts

1. Look for context files in `.claude/contexts/*.md`
2. If the directory doesn't exist or is empty, tell the user: "No saved contexts found. Use `/ctx:save` or `/ctx:fork` to create one."

## Step 2: Display List

Show all contexts in a table format, sorted by modification time (newest first):

```
Available contexts:
─────────────────────────────────────────────────
 #  Type     Name                          Saved
─────────────────────────────────────────────────
 1  handoff  call-router-webhook-timeout   2 hours ago
 2  fork     fork-config-timeout-api       3 hours ago
 3  fork     fork-frontend-timeout-field   3 hours ago
 4  handoff  whatsapp-retry-logic          yesterday
─────────────────────────────────────────────────
```

Rules for the display:
- **Type**: "fork" if filename starts with `fork-`, otherwise "handoff"
- **Name**: filename without `.md` extension
- **Saved**: relative time (e.g., "2 hours ago", "yesterday", "3 days ago")
- If a keyword argument was provided, only show matching contexts

If `--drop` flag was provided, ask the user which number to delete, then remove the file and confirm.

## Step 3: User Selects

Ask the user to pick a context by number or name. Use AskUserQuestion for this.

## Step 4: Load Context

1. Read the selected `.md` file
2. Present the content to the user in a clear format
3. Based on the context type:

**For handoff contexts:**
Say: "I've loaded the context. Here's where things left off:" then summarize:
- What was being done
- Current progress
- What needs to happen next
- Ask: "Should I continue from where this left off?"

**For fork contexts:**
Say: "I've loaded a fork task. Here's what needs to be done:" then summarize:
- The specific task
- Key files to work with
- Acceptance criteria
- Ask: "Should I start working on this?"

## Step 5: Orient

After the user confirms, read the key files listed in the context to get oriented with the actual code state. The context file describes what WAS happening — the code may have changed since then. Always verify before acting.

## Critical Rules

- **Never auto-load** — Always show the list first, let the user choose
- **Verify freshness** — After loading, check if the branch still exists and files are still relevant
- **Context is a hint, not truth** — The saved context describes a past state. Always read actual files before making changes.
