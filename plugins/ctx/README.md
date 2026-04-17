# ctx — Context Handoff Plugin for Claude Code

Save, fork, and load working context across Claude Code windows.

## Why

When you close a Claude Code window, the conversation context is lost. This plugin lets you:

- **Hand off** work to a new window (continue where you left off)
- **Fork** subtasks for parallel development (two windows, two tasks)
- **Load** saved contexts without remembering file names

## Install

```bash
/plugin install /Users/max/.claude/plugins/local/ctx
```

## Commands

### `/ctx:save`

Save your current working state for handoff. Auto-generates a descriptive filename.

```
/ctx:save
→ Saved: call-router-webhook-timeout.md
```

### `/ctx:fork <description>`

Extract a subtask for parallel development in another window. Only includes context relevant to the subtask.

```
/ctx:fork config-service 加 timeout API 端点
→ Saved: fork-config-timeout-api.md
```

### `/ctx:load`

List all saved contexts and pick one to load.

```
/ctx:load              # list all
/ctx:load webhook      # filter by keyword
/ctx:load --drop       # delete a context
```

## Storage

Contexts are stored in `.claude/contexts/` within each project directory. Different projects have separate contexts.

## Context Types

| Type | Prefix | Use Case |
|------|--------|----------|
| Handoff | (none) | Continue the same task in a new window |
| Fork | `fork-` | Parallel subtask in another window |
