---
date: "2026-04-09T12:00:00"
wing: mempalace
room: hybrid-memory
hall: decisions
tags: [hybrid, cross-project, shared-memory, scope]
source: session
scope: project
---

## Decision: Hybrid memory — project-scoped + shared cross-project

Implemented dual memory locations:
- **Project memory** (`<repo>/.memory/`) — project-specific decisions, travels with the repo
- **Shared memory** (`~/.memory/`) — cross-project decisions, accessible from any repo

### How scope is determined at save time:
- Shared if memory applies across projects: design systems, conventions, infrastructure, preferences
- Project if specific to current codebase: API design, database schema, feature decisions
- Heuristic: wing name matching cross-cutting concerns defaults to shared; wing matching repo name defaults to project
- If unclear, skill asks the user

### What changed in each skill:
- `/save-memory` — added scope field (project/shared), saves to the right location
- `/recall` — searches both locations, labels results as [project] or [shared]
- `/memory-status` — shows both locations with separate tier counts
- `/memory-cleanup` — runs lifecycle on both independently

### Global setup:
- Skills copied to `~/.claude/skills/` (available in every project)
- `~/.claude/CLAUDE.md` created (protocol loads every session)
- Memory hook added to `~/.claude/settings.json` (fires everywhere)
- `~/.memory/` created with full tier structure + lifecycle.json

### End-to-end test results (6/6 passed):
- Shared memory visible from any project
- Project memory stays scoped to its repo
- No cross-project leakage
- Merged recall with source labels works
