# skill: setup

## purpose
Run once per project. Creates `.agent-context/` — a shared knowledge folder that every other skill reads from. Without it, skills work in isolation. With it, they share project vocabulary, architecture decisions, and codebase structure.

## when_to_use
- First time using agent-skills in a project
- Starting work in an unfamiliar codebase
- After major architectural changes that make the old context stale

## when_not_to_use
- `.agent-context/` already exists and is current — just read it, don't overwrite
- Throwaway scripts or one-off tasks with no ongoing session

## input
```
project_root: string    # Root directory path
entry_points: list      # Known entry files (optional — skill will detect if empty)
```

## output
Creates the following files:
```
.agent-context/
├── context.md    # Project vocabulary, stack, decisions
└── graph.md      # Dependency map (populated by /code-map)
```

## instructions

```
You are setting up shared project context. Do the following in order:

1. Check if .agent-context/ already exists. If yes, read it and report what's there. Ask before overwriting.

2. Detect project type from files present:
   - package.json → Node/TypeScript
   - requirements.txt / pyproject.toml → Python
   - go.mod → Go
   - Cargo.toml → Rust
   - Look for main entry points: main.py, index.ts, app.py, server.js, cmd/

3. Create `.agent-context/context.md` using the exact headings/structure from `.agent-context-template/context.md`:
   - Fill `## Tech stack` and `## Entry points` from detection
   - Keep `## Domain vocabulary` table headers; fill only the cells you are confident about, otherwise leave them blank for the user to complete
   - Leave `## Architecture decisions NOT to re-litigate`, `## Common failure patterns in this codebase`, and `## External services and their quirks` empty (with placeholder bullet/table rows)

4. Run the equivalent of `/code-map` on the project and write results to `.agent-context/graph.md` using the same headings/sections as `.agent-context-template/graph.md`.

5. Print a summary of what was created and what the user should fill in manually.

Do not invent domain vocabulary — leave those rows empty for the user to complete.
Do not make architectural decisions — document what exists, not what should exist.
```

## constraints
- Max output tokens: 400 (summary only — files are written to disk)
- Must check for existing .agent-context/ before creating
- Must never overwrite existing files without explicit confirmation

## example

**Input:**
```
project_root: "/home/user/myapp"
entry_points: []
```

**Output (terminal):**
```
Created .agent-context/context.md
  ✓ Detected: Python / FastAPI / PostgreSQL
  ✓ Entry points found: main.py, app/routes/
  ⚠ Fill in: domain vocabulary, architectural decisions

Created .agent-context/graph.md
  ✓ 12 files mapped
  ✓ God nodes: app/config.py, app/db.py
  ✓ Clusters: auth, billing, notifications

Next: Open .agent-context/context.md and fill in the vocabulary table.
All other skills will read from these files automatically.
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
