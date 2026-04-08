---
name: memory-status
description: Show overview of the memory palace — tier sizes, wings, rooms, and recent activity
---

# Memory Status

Show the current state of both project and shared memory palaces.

## Memory Locations

- **Project memory**: `<repo>/.memory/` — current project only
- **Shared memory**: `~/.memory/` — cross-project, accessible everywhere

## Instructions

When invoked, do the following for **both** memory locations (project and shared). If either location doesn't exist, note it and skip gracefully.

1. **Count files in each tier** (for each location):
   - `active/` — count all `.md` files (excluding `_index.md`)
   - `warm/` — count all `.md` files
   - `cold/manifest.md` — count data rows
   - `archive/` — count all `.md` files
   - `diary/` — count diary entries (project memory only; shared has no diary)

2. **List wings and rooms.** For each wing in `active/`:
   - List all rooms
   - Count memories per room
   - Show the most recent memory date

3. **Show lifecycle config.** Read `lifecycle.json` from whichever location has it (prefer project, fall back to shared, or show both if they differ). Display:
   - Current tier thresholds
   - Whether test mode is enabled
   - Hall categories

4. **Check for aging memories.** Flag any active memories that are approaching the tier threshold (within 20% of max age).

5. **Format as a clean summary:**

```
## Memory Palace Status

### Project Memory (<repo-name>/.memory/)

#### Tiers
| Tier    | Files | Description                    |
|---------|-------|--------------------------------|
| Active  | 8     | Full verbatim, 0-30 days       |
| Warm    | 3     | Summarized, 30-60 days         |
| Cold    | 0     | Index only, 60-90 days         |
| Archive | 5     | Permanent deep storage          |
| Diary   | 2     | Session diary entries           |

#### Active Wings
- **api** (5 memories)
  - auth: 3 memories (latest: 2026-04-08)
  - schema: 2 memories (latest: 2026-04-07)

---

### Shared Memory (~/.memory/)

#### Tiers
| Tier    | Files | Description                    |
|---------|-------|--------------------------------|
| Active  | 4     | Full verbatim, 0-30 days       |
| Warm    | 2     | Summarized, 30-60 days         |
| Cold    | 1     | Index only, 60-90 days         |
| Archive | 3     | Permanent deep storage          |

#### Active Wings
- **design-system** (3 memories)
  - tokens: 2 memories (latest: 2026-04-08)
  - components: 1 memory (latest: 2026-04-06)
- **conventions** (1 memory)
  - git: 1 memory (latest: 2026-04-05)

### Aging Soon
- [project] 2 memories approaching warm threshold
- [shared] 1 memory approaching warm threshold
```
