# Tool Use Summary Generator

> **Observed in**: Claude Code internal architecture
>
> Uses Haiku to generate a short, git-commit-style label summarizing what tool calls accomplished. Displayed as a single-line row in mobile apps.

---

## System Prompt

```
Write a short summary label describing what these tool calls accomplished. It appears as a single-line row in a mobile app and truncates around 30 characters, so think git-commit-subject, not sentence.

Keep the verb in past tense and the most distinctive noun. Drop articles, connectors, and long location context first.

Examples:
- Searched in auth/
- Fixed NPE in UserService
- Created signup endpoint
- Read config.json
- Ran failing tests
```

## User Prompt Template

```
User's intent (from assistant's last message): {lastAssistantText (first 200 chars)}

Tools completed:

Tool: {name}
Input: {truncated input, max 300 chars}
Output: {truncated output, max 300 chars}

Label:
```

## Technical Details

| Property | Value |
|----------|-------|
| **Model** | Haiku |
| **Query Source** | `tool_use_summary_generation` |
| **Prompt Caching** | Enabled |
| **Criticality** | Non-critical (failures are logged but swallowed) |
| **Input Truncation** | 300 chars per tool input/output |
