---
name: memory-cleanup
description: Run memory lifecycle — age, summarize, archive, and purge memories based on tiering policy
---

# Memory Cleanup (Lifecycle Management)

Run the memory lifecycle process to keep the active memory set efficient.

## Instructions

When invoked, do the following:

1. **Read the lifecycle config.** Read `.memory/lifecycle.json` to get the current tiering policy and check if test mode is enabled.

2. **Scan active tier.** List all memory files in `.memory/active/` with their dates (from frontmatter `date` field).

3. **Identify aging memories.** For each file, calculate age:
   - If **test_mode.enabled** is true: use minute-based thresholds from `test_mode`
   - If false: use day-based thresholds from `tiers`

4. **Process each tier transition:**

   ### Active → Warm (past active max age)
   For each aging active memory:
   a. Read the full file
   b. Write a **summary version** to `.memory/warm/<wing>/<room>/` with the same filename
      - Keep: date, wing, room, hall, tags from frontmatter
      - Add: `summarized_from: <original path>`, `summarized_at: <now>`
      - Body: Condense to key decision/outcome/rationale in 3-5 bullet points
      - Preserve any specific names, numbers, dates, or exact quotes that are critical
   c. Move the original to `.memory/archive/<wing>/<room>/`
   d. Update the wing index

   ### Warm → Cold (past warm max age)
   For each aging warm memory:
   a. Read the summary file
   b. Add a one-line entry to `.memory/cold/manifest.md`:
      ```
      | <date> | <wing> | <room> | <hall> | <one-line summary> | <archive path> |
      ```
   c. Delete the warm file (archive still has the original)

   ### Cold → Purge (past cold max age)
   For each aging cold entry:
   a. Remove the line from manifest.md
   b. The archive copy remains permanently (never auto-deleted)

5. **Report what was done.** Show a summary:
   - How many memories were in each tier before/after
   - What was moved, summarized, or purged
   - Current tier sizes

## Test Mode

To test the lifecycle quickly without waiting 30 days:

```bash
# Edit .memory/lifecycle.json and set:
# "test_mode": { "enabled": true, "active_max_minutes": 5, ... }
```

Then run `/memory-cleanup` and it will use minute-based thresholds instead of days.

## Rules

- NEVER delete original files from `.memory/archive/`. Archive is permanent.
- When summarizing, preserve exact quotes for decisions. "We chose Postgres" is better than "database was selected."
- Always show what was done — the user should be able to verify every transition.
- If `.memory/cold/manifest.md` doesn't exist, create it with a header row.
