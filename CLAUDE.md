# CLAUDE.md

**Read [AGENTS.md](./AGENTS.md).** It is the single source of truth for this project and
is kept current; this file exists only so that Claude-specific tooling finds it.

Everything relevant lives there: the build command, the requirement that the Unreal Editor
be closed before building, machine setup prerequisites, the multi-machine issue-sync
workflow, and what does and does not travel between machines.

## The two things most likely to waste your time

1. **Close the Unreal Editor before building.** Otherwise the build fails on the Live
   Coding lock. Ask the human to close it; never force-kill it.
2. **Issue sync is `bd export` / `bd import` against `.beads/issues.jsonl`**, not
   `bd dolt push`. There is no Dolt remote. The database itself is gitignored.

## Note on the user-level CLAUDE.md

A personal `~/CLAUDE.md` may also be loaded on some machines. Where the two disagree,
this repo's `AGENTS.md` wins — it is the one that travels with the code and reflects how
this project is actually configured.
