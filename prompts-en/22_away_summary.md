# Away Summary (Session Recap)

**Observed in**: Claude Code internal architecture
**Function:** `buildAwaySummaryPrompt()`
**Model:** Small fast model (Haiku)

## Purpose

Generates a 1-3 sentence recap when a user returns to a session after stepping away. Displayed in a "while you were away" card in the UI.

## System Prompt

The system prompt is dynamically assembled with an optional session memory prefix:

```
[Session memory (broader context):
{memory content}]

The user stepped away and is coming back. Write exactly 1-3 short sentences. Start by stating the high-level task — what they are building or debugging, not implementation details. Next: the concrete next step. Skip status reports and commit recaps.
```

## Design Constraints

- **Brevity**: Exactly 1-3 short sentences
- **Task-first**: Start with what the user is building or debugging
- **Actionable**: Include the concrete next step
- **No noise**: Skip status reports and commit recaps
- **Implementation detail avoidance**: Focus on the high-level task, not code specifics

## Input Processing

- Uses only the last 30 messages (`RECENT_MESSAGE_WINDOW = 30`) to avoid "prompt too long" errors on large sessions
- Incorporates session memory content for broader context
- Disables thinking to keep responses fast
- Uses `skipCacheWrite: true` to avoid polluting the cache

## Error Handling

- Returns `null` on abort, empty transcript, or any error
- Silently absorbs `APIUserAbortError` exceptions
