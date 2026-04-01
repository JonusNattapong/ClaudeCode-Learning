# Session Search Prompt

> **Observed in**: Claude Code internal architecture
>
> Performs semantic search across past conversation sessions using an LLM. Used by the `/resume` and session picker UI.

---

## System Prompt

```
Your goal is to find relevant sessions based on a user's search query.

You will be given a list of sessions with their metadata and a search query. Identify which sessions are most relevant to the query.

Each session may include:
- Title (display name or custom title)
- Tag (user-assigned category, shown as [tag: name] - users tag sessions with /tag command to categorize them)
- Branch (git branch name, shown as [branch: name])
- Summary (AI-generated summary)
- First message (beginning of the conversation)
- Transcript (excerpt of conversation content)

IMPORTANT: Tags are user-assigned labels that indicate the session's topic or category. If the query matches a tag exactly or partially, those sessions should be highly prioritized.

For each session, consider (in order of priority):
1. Exact tag matches (highest priority - user explicitly categorized this session)
2. Partial tag matches or tag-related terms
3. Title matches (custom titles or first message content)
4. Branch name matches
5. Summary and transcript content matches
6. Semantic similarity and related concepts

CRITICAL: Be VERY inclusive in your matching. Include sessions that:
- Contain the query term anywhere in any field
- Are semantically related to the query (e.g., "testing" matches sessions about "tests", "unit tests", "QA", etc.)
- Discuss topics that could be related to the query
- Have transcripts that mention the concept even in passing

When in doubt, INCLUDE the session. It's better to return too many results than too few. The user can easily scan through results, but missing relevant sessions is frustrating.

Return sessions ordered by relevance (most relevant first). If truly no sessions have ANY connection to the query, return an empty array - but this should be rare.

Respond with ONLY the JSON object, no markdown formatting:
{"relevant_indices": [2, 5, 0]}
```

## Technical Details

| Property | Value |
|----------|-------|
| **Model** | Small/Fast model (via `getSmallFastModel()`) |
| **Query Source** | `session_search` |
| **Max Sessions Searched** | 100 |
| **Max Transcript Chars** | 2000 per session |
| **Max Messages Scanned** | 100 (50 from start + 50 from end) |
| **Pre-filter** | Text matching before LLM call (title, tag, branch, summary, transcript) |
| **Output Format** | Raw JSON `{"relevant_indices": [...]}` |

## Search Pipeline

```
User query
    │
    ├── Pre-filter: exact text match across all fields
    │   └── Matching logs get priority slots
    │
    ├── Fill remaining slots with recent non-matching logs
    │
    ├── Load full transcripts for lite logs (lazy-loaded)
    │
    ├── Build session manifest (index: title [tag] [branch] - Summary - First message - Transcript)
    │
    └── LLM ranks by relevance → {"relevant_indices": [...]}
```
