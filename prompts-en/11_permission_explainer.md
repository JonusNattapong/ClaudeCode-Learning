# Permission Explainer System Prompt

> **Observed in**: Claude Code internal architecture
>
> A lightweight side-query that explains what a tool/command does, why it's being run, and its risk level before the user approves it. Uses structured tool output for guaranteed JSON responses.

---

## System Prompt

```
Analyze shell commands and explain what they do, why you're running them, and potential risks.
```

## Structured Output Schema (Tool: `explain_command`)

```json
{
  "explanation": "What this command does (1-2 sentences)",
  "reasoning": "Why YOU are running this command. Start with 'I' - e.g. 'I need to check the file contents'",
  "risk": "What could go wrong, under 15 words",
  "riskLevel": "LOW | MEDIUM | HIGH"
}
```

### Risk Level Definitions

| Level | Description | Example |
|-------|-------------|---------|
| `LOW` | Safe dev workflows | `ls`, `cat`, `git status` |
| `MEDIUM` | Recoverable changes | File edits, `npm install` |
| `HIGH` | Dangerous/irreversible | `rm -rf`, `DROP TABLE`, force push |

## Runtime Behavior

- Uses the **main loop model** (not a separate Haiku model)
- Fires as a **side query** concurrent with the permission prompt
- Extracts last 3 assistant messages (up to 1000 chars) for "why" context
- Can be disabled by user config: `permissionExplainerEnabled: false`
