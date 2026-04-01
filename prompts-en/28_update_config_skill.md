# Update Config Skill (/update-config)

**Observed in**: Claude Code internal architecture
**Registration:** `registerBundledSkill('update-config', ...)`
**Availability:** All users

## Purpose

Manages `settings.json` configuration files, including hooks, permission arrays, and general settings. Provides a guided interface for editing the Claude Code configuration hierarchy.

## Prompt (Reconstructed from Binary Analysis)

The prompt is extensive (~476 lines) and covers three main areas:

### Settings File Hierarchy

```
Settings are loaded from three levels (highest to lowest priority):
1. Project settings: .claude/settings.json (checked into repo)
2. User settings: ~/.claude/settings.json (private, all projects)
3. Enterprise settings: /etc/claude-code/settings.json (managed by admins)

Lower-priority settings are overridden by higher-priority ones.
```

### Hook System

The skill documents the complete hook lifecycle:

```
Hooks are commands that run at specific points in Claude Code's lifecycle:
- PreToolUse: Runs before a tool is executed. Can block the tool.
- PostToolUse: Runs after a tool completes. Can modify the result.
- PreCompact: Runs before conversation compaction.
- PostCompact: Runs after conversation compaction.
- Notification: Runs when Claude wants to notify the user.
- Stop: Runs when Claude's turn ends.

Each hook receives a JSON payload on stdin with:
- session_id: Current session ID
- tool_name: Name of the tool (PreToolUse/PostToolUse)
- tool_input: Tool input parameters (PreToolUse/PostToolUse)
- tool_output: Tool output (PostToolUse only)

Hook exit codes:
- 0: Success, continue normally
- 2: Block the tool (PreToolUse only)
- Other: Log error but continue
```

### Permission Arrays

```
Permission settings control which tools can run without user approval:
- allowedTools: Tool names or patterns that are auto-approved
- deniedTools: Tool names or patterns that are always blocked
- trust: Trust levels for different tool categories
```

## Architecture Notes

- The skill distinguishes between "simple" config tools (key-value settings) and "complex" operations (hooks, permissions)
- For hook editing, uses the Edit tool directly on the settings.json file
- Validates JSON syntax after each edit
- Supports all three settings levels in a single interface
