# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

---

## Project: UEIntroProject

An Unreal Engine 5.8 game built on the First Person template, used to learn Unreal.
Gameplay logic is written in C++; levels, materials and Blueprint assets are authored
in the editor.

- `Source/UEIntroProject/` — C++ primary game module (`UEIntroProject`)
- `Content/` — binary `.uasset`/`.umap` assets, tracked via **Git LFS**
- `Config/` — project ini files

**Agents cannot edit `.uasset` or `.umap` files.** They are binary. Blueprint and asset
changes must be described as editor steps for the human to perform.

### Build

```bash
# Windows
"C:\Program Files\Epic Games\UE_5.8\Engine\Build\BatchFiles\Build.bat" \
  UEIntroProjectEditor Win64 Development \
  -Project="<abs path>\UEIntroProject.uproject" -WaitMutex
```

**The Unreal Editor must be closed before building.** If it is running, the build fails
with `Unable to build while Live Coding is active`. Check for `UnrealEditor` and
`LiveCodingConsole` processes and ask the human to close the editor — never force-kill
it, since unsaved level and Blueprint work is lost permanently.

### Machine setup requirements

Building this project requires, beyond the engine itself:

- **Unreal Engine 5.8** (the `.uproject` pins `EngineAssociation` to 5.8)
- **Git LFS** — without it, `Content/` clones as text pointer files and the project will not open
- **A C++ toolchain** — MSVC (Visual Studio or Build Tools) on Windows
- **.NET Framework 4.8 SDK** on Windows — the editor target pulls in `SwarmInterface`
  via `UnrealEd` and fails with `Could not find NetFxSDK install dir` without it.
  This is a separate component from the C++ toolchain and is easy to miss.

Build Tools alone (no full Visual Studio IDE) is sufficient. UBT will warn
`Unable to find Visual Studio SDK. Editor integration will be disabled` — this is benign.

---

## Working across multiple machines

Issue data lives in a **Dolt database under `.beads/dolt/`, which is gitignored** and does
NOT travel with the repo. The portable artifact is `.beads/issues.jsonl`, which IS
committed. Treat that file as the source of truth when moving between machines.

### Fresh clone on a new machine

```bash
git clone <repo-url>
cd UEIntroProject
git lfs install && git lfs pull   # required, or Content/ is just pointer files
bd bootstrap                      # create the local database
bd import                         # load issues from .beads/issues.jsonl
bd hooks install                  # wire git hooks (config is per-machine, not in git)
git config --local beads.role maintainer   # otherwise bd warns on every command
```

This sequence is verified: a fresh clone reproduces the full issue list with statuses
(open / in_progress / closed) intact.

### Every session

```bash
git pull --rebase
bd import        # pull issue changes into the local db -- NOT automatic
# ...work...
git commit       # pre-commit hook exports and stages .beads/issues.jsonl for you
git push
```

**Export is automatic.** A project-specific block in `.beads/hooks/pre-commit` (outside the
beads-managed markers) runs `bd export` and stages `.beads/issues.jsonl` on every commit,
so the committed issue list cannot drift from the local database. This is verified.

**Import is not confirmed automatic.** bd installs a `post-merge` hook that may import
after a pull, but that has not been tested here. Run `bd import` explicitly after pulling —
it uses upsert semantics, so running it when it was not needed is harmless.

bd has an `auto-export` config key that is accepted but has no observable effect — do not
rely on it.

**Overrides the bd-managed section below:** that block says to use `bd dolt push` and that
"no manual export/import is needed." That is not true for this project — there is no Dolt
remote configured. Use `bd export` / `bd import` against `.beads/issues.jsonl` instead.

### Things that do NOT travel between machines

Re-establish these on each machine; do not assume they are present:

- The Dolt database (`.beads/dolt/`) — rebuilt via `bd bootstrap` + `bd import`
- `bd remember` memories — they are stored in the database and are **not** included in
  `bd export`. Durable project knowledge belongs in this file, not in `bd remember`.
- `core.hooksPath` and all git hook wiring — per-machine, set by `bd hooks install`
- `.beads/.beads-credential-key` — machine-local secret, correctly gitignored
- Derived data: `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/`

### Who this project is for

The human is comfortable writing code but is new to game development and new to Unreal.
Explain engine concepts — actors, ticks, components, garbage collection, the reflection
macros — rather than language basics. The goal is learning Unreal, not shipping a product,
so prefer the approach that teaches the engine over the one that finishes fastest.

### Known bd gotcha

`bd init` auto-commits, and it stages `.beads/.beads-credential-key` — a real secret that
bd's own `.beads/.gitignore` does not exclude. It also appends a bare `*.db` rule to the
root `.gitignore`. Check both after running `bd init` in any new repo.

---

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work atomically
bd close <id>         # Complete work
bd dolt push          # Push beads data to remote
```

## Non-Interactive Shell Commands

**ALWAYS use non-interactive flags** with file operations to avoid hanging on confirmation prompts.

Shell commands like `cp`, `mv`, and `rm` may be aliased to include `-i` (interactive) mode on some systems, causing the agent to hang indefinitely waiting for y/n input.

**Use these forms instead:**
```bash
# Force overwrite without prompting
cp -f source dest           # NOT: cp source dest
mv -f source dest           # NOT: mv source dest
rm -f file                  # NOT: rm file

# For recursive operations
rm -rf directory            # NOT: rm -r directory
cp -rf source dest          # NOT: cp -r source dest
```

**Other commands that may prompt:**
- `scp` - use `-o BatchMode=yes` for non-interactive
- `ssh` - use `-o BatchMode=yes` to fail instead of prompting
- `apt-get` - use `-y` flag
- `brew` - use `HOMEBREW_NO_AUTO_UPDATE=1` env var

<!-- BEGIN BEADS INTEGRATION profile:full hash:d4f96305 -->
## Issue Tracking with bd (beads)

**IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Why bd?

- Dependency-aware: Track blockers and relationships between issues
- Git-friendly: Dolt-powered version control with native sync
- Agent-optimized: JSON output, ready work detection, discovered-from links
- Prevents duplicate tracking systems and confusion

### Quick Start

**Check for ready work:**

```bash
bd ready --json
```

**Create new issues:**

```bash
bd create "Issue title" --description="Detailed context" -t bug|feature|task -p 0-4 --json
bd create "Issue title" --description="What this issue is about" -p 1 --deps discovered-from:bd-123 --json
```

**Claim and update:**

```bash
bd update <id> --claim --json
bd update bd-42 --priority 1 --json
```

**Complete work:**

```bash
bd close bd-42 --reason "Completed" --json
```

### Issue Types

- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `chore` - Maintenance (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (default, nice-to-have)
- `3` - Low (polish, optimization)
- `4` - Backlog (future ideas)

### Workflow for AI Agents

1. **Check ready work**: `bd ready` shows unblocked issues
2. **Claim your task atomically**: `bd update <id> --claim`
3. **Work on it**: Implement, test, document
4. **Discover new work?** Create linked issue:
   - `bd create "Found bug" --description="Details about what was found" -p 1 --deps discovered-from:<parent-id>`
5. **Complete**: `bd close <id> --reason "Done"`

### Auto-Sync

bd automatically syncs via Dolt:

- Each write auto-commits to Dolt history
- Use `bd dolt push`/`bd dolt pull` for remote sync
- No manual export/import needed!

### Important Rules

- ✅ Use bd for ALL task tracking
- ✅ Always use `--json` flag for programmatic use
- ✅ Link discovered work with `discovered-from` dependencies
- ✅ Check `bd ready` before asking "what should I work on?"
- ❌ Do NOT create markdown TODO lists
- ❌ Do NOT use external issue trackers
- ❌ Do NOT duplicate tracking systems

For more details, see README.md and docs/QUICKSTART.md.

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd dolt push
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

<!-- END BEADS INTEGRATION -->
