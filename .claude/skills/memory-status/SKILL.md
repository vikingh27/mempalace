---
name: memory-status
description: Show overview of the memory palace — tier sizes, wings, rooms, and recent activity
---

# Memory Status

Show the current state of the memory palace.

## Instructions

When invoked, do the following:

1. **Count files in each tier:**
   - `.memory/active/` — count all `.md` files (excluding `_index.md`)
   - `.memory/warm/` — count all `.md` files
   - `.memory/cold/manifest.md` — count rows
   - `.memory/archive/` — count all `.md` files
   - `.memory/diary/` — count diary entries

2. **List wings and rooms.** For each wing in `.memory/active/`:
   - List all rooms
   - Count memories per room
   - Show the most recent memory date

3. **Show lifecycle config.** Read `.memory/lifecycle.json` and display:
   - Current tier thresholds
   - Whether test mode is enabled
   - Hall categories

4. **Check for aging memories.** Flag any active memories that are approaching the tier threshold (within 20% of max age).

5. **Format as a clean summary:**

```
## Memory Palace Status

### Tiers
| Tier    | Files | Description                    |
|---------|-------|--------------------------------|
| Active  | 12    | Full verbatim, 0-30 days       |
| Warm    | 5     | Summarized, 30-60 days         |
| Cold    | 23    | Index only, 60-90 days         |
| Archive | 28    | Permanent deep storage          |
| Diary   | 4     | Session diary entries           |

### Active Wings
- **mempalace** (8 memories)
  - memory-philosophy: 5 memories (latest: 2026-04-08)
  - benchmarks: 3 memories (latest: 2026-04-08)

### Aging Soon
- 2 memories approaching warm threshold
```
