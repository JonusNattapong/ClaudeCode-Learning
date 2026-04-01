# Compact Service (Conversation Summarization)

**Observed in**: Claude Code internal architecture
**Variables:** `NO_TOOLS_PREAMBLE`, `BASE_COMPACT_PROMPT`, `PARTIAL_COMPACT_PROMPT`, `PARTIAL_COMPACT_UP_TO_PROMPT`
**Model:** Sonnet (via cache-sharing fork or streaming fallback)

## Purpose

Generates detailed conversation summaries when the context window approaches its limit. The compact service has three operating modes: full compaction, partial compaction of recent messages, and partial compaction of older messages.

## No-Tools Preamble

Injected as the first instruction to prevent the model from calling tools during summarization. This is critical because the cache-sharing fork path inherits the parent's full tool set.

```
CRITICAL: Respond with TEXT ONLY. Do NOT call any tools.

- Do NOT use Read, Bash, Grep, Glob, Edit, Write, or ANY other tool.
- You already have all the context you need in the conversation above.
- Tool calls will be REJECTED and will waste your only turn — you will fail the task.
- Your entire response must be plain text: an <analysis> block followed by a <summary> block.
```

## Base Compact Prompt (Full Summarization)

Used when the entire conversation is being summarized:

```
Your task is to create a detailed summary of the conversation so far, paying close
attention to the user's explicit requests and your previous actions.
This summary should be thorough in capturing technical details, code patterns, and
architectural decisions that would be essential for continuing development work
without losing context.
```

### Required Summary Sections

1. **Primary Request and Intent** — All explicit user requests and intents
2. **Key Technical Concepts** — Technologies, frameworks, patterns discussed
3. **Files and Code Sections** — Files examined/modified/created with full code snippets
4. **Errors and Fixes** — Errors encountered and resolution steps
5. **Problem Solving** — Solved problems and ongoing troubleshooting
6. **All User Messages** — Every non-tool-result user message (critical for tracking intent drift)
7. **Pending Tasks** — Outstanding work items
8. **Current Work** — Precise description of work in progress before summarization
9. **Optional Next Step** — Next step only if directly aligned with recent user requests

## Partial Compact Prompt (Recent Messages)

Used when only recent messages are being summarized while older context is retained:

```
Your task is to create a detailed summary of the RECENT portion of the conversation —
the messages that follow earlier retained context. The earlier messages are being kept
intact and do NOT need to be summarized. Focus your summary on what was discussed,
learned, and accomplished in the recent messages only.
```

## Partial Compact Up-To Prompt (Older Context)

Used when summarizing older messages to make room, with newer messages kept intact:

```
Your task is to create a detailed summary of this conversation. This summary will be
placed at the start of a continuing session; newer messages that build on this context
will follow after your summary (you do not see them here). Summarize thoroughly so
that someone reading only your summary and then the newer messages can fully understand
what happened and continue the work.
```

Replaces section 8 ("Current Work") with "Work Completed" and section 9 with "Context for Continuing Work."

## Analysis Block

All three variants include a `DETAILED_ANALYSIS_INSTRUCTION` that requires the model to wrap its reasoning in `<analysis>` tags before the `<summary>`. The analysis block is a drafting scratchpad stripped by `formatCompactSummary()` before the summary reaches context.

```
Before providing your final summary, wrap your analysis in <analysis> tags to organize
your thoughts and ensure you've covered all necessary points. In your analysis process:

1. Chronologically analyze each message and section of the conversation. For each
   section thoroughly identify:
   - The user's explicit requests and intents
   - Your approach to addressing the user's requests
   - Key decisions, technical concepts and code patterns
   - Specific details like: file names, full code snippets, function signatures, file edits
   - Errors that you ran into and how you fixed them
   - Pay special attention to specific user feedback
2. Double-check for technical accuracy and completeness.
```

## Custom Instructions Support

The compact prompts support user-provided summarization instructions via CLAUDE.md or hooks:

```
There may be additional summarization instructions provided in the included context.
If so, remember to follow these instructions when creating the above summary.
```

## Architecture Notes

- **Cache-sharing fork**: A forked process that inherits the parent's prompt cache. Requires the `NO_TOOLS_PREAMBLE` because it also inherits the full tool set.
- **Streaming fallback**: If the fork path fails (e.g., model attempts tool call with `maxTurns: 1`), falls through to a standard streaming query.
- **Proactive mode integration**: When proactive mode or Kairos is enabled, additional context about proactive tasks is appended.
