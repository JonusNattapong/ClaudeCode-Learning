# Session Title Generator

**Observed in**: Claude Code internal architecture
**Variable:** `SESSION_TITLE_PROMPT`
**Model:** Haiku (via `queryHaiku`)
**Output Format:** JSON (`{ "title": "..." }`)

## Purpose

Generates concise, sentence-case session titles (3-7 words) from conversation content. Acts as the single source of truth for AI-generated session titles across all surfaces (SDK, CCR remote sessions, REPL bridge).

## System Prompt

```
Generate a concise, sentence-case title (3-7 words) that captures the main topic or goal of this coding session. The title should be clear enough that the user recognizes the session in a list. Use sentence case: capitalize only the first word and proper nouns.

Return JSON with a single "title" field.

Good examples:
{"title": "Fix login button on mobile"}
{"title": "Add OAuth authentication"}
{"title": "Debug failing CI tests"}
{"title": "Refactor API client error handling"}

Bad (too vague): {"title": "Code changes"}
Bad (too long): {"title": "Investigate and fix the issue where the login button does not respond on mobile devices"}
Bad (wrong case): {"title": "Fix Login Button On Mobile"}
```

## Input Processing

The `extractConversationText()` function flattens message arrays into a single text string:
- Skips meta and non-human messages
- Tail-slices to the last 1000 characters so recent context wins
- Filters by `msg.origin.kind === 'human'`

## Integration Points

- **SDK print path**: Non-interactive session title generation
- **CCR remote sessions**: Interactive session title via `useRemoteSession`
- **REPL bridge**: Fire-and-forget title upgrade after 3 user messages
