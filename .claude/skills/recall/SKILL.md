---
name: recall
description: Search memory files to recall past decisions, discussions, and context
---

# Recall

Search the memory palace to find past decisions, discussions, insights, and context.

## Instructions

When invoked (e.g., `/recall why did we choose tiering?`), do the following:

1. **Parse the query.** Understand what the user is looking for — a decision, a person, a project detail, a past discussion.

2. **Search strategy — narrow before deep:**

   a. **Check indexes first.** Read `.memory/active/*/_index.md` files to identify which wings and rooms are likely relevant. This is cheap (one file per wing).

   b. **Grep for keywords.** Use the Grep tool to search across `.memory/active/` for key terms from the query. Also search `.memory/warm/` for summarized older memories.

   c. **Read matching files.** Read the top hits (up to 10 files). Understand the full context of each.

   d. **Check cold manifest.** If nothing found in active/warm, read `.memory/cold/manifest.md` for index entries that might point to archived content.

3. **Synthesize an answer.** Don't just dump file contents. Combine what you found into a coherent answer that:
   - Directly addresses the query
   - Cites which memory files the answer came from (path + date)
   - Includes relevant verbatim quotes where they add value
   - Notes if there are conflicting memories or if context has evolved

4. **Report gaps.** If you can't find what was asked, say so clearly. Suggest what might be missing and whether it should be saved.

## Search Priority

1. `.memory/active/` — full verbatim, most recent (search first)
2. `.memory/warm/` — summarized, 30-60 days old
3. `.memory/cold/manifest.md` — index entries, 60-90 days old
4. `.memory/diary/` — agent diary entries for session history

## Rules

- NEVER guess or fabricate if a memory isn't found. Say "I don't have a memory of that."
- Always cite the source file path and date.
- If the query is ambiguous, search broadly first, then ask the user to narrow down.
