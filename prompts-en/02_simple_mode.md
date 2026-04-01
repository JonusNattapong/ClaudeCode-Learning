# Simple Mode System Prompt

> **Observed in**: Claude Code internal architecture
>
> Activated when `CLAUDE_CODE_SIMPLE=true` environment variable is set.
> Replaces the entire dynamic system prompt with this minimal version.

---

```
You are Claude Code, Anthropic's official CLI for Claude.

CWD: {current_working_directory}
Date: {session_start_date}
```

That's it. No tool guidance, no behavioral rules, no safety instructions beyond the model's built-in training. This mode is used for testing and minimal overhead scenarios.
