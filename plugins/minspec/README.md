# /feature workflow — install bundle

A single-entry-point feature workflow for Claude Code: discovery → engineering → implementation handoff, auto-routed by depth.

## Why one entry point

You run `/feature` and nothing else. The orchestrator handles the rest:

- Auto-detects depth (quick vs deep) from your description
- Runs `grill-me` skill in main context
- Saves a compressed `intent.md` (the bridge artifact between discovery and engineering)
- Delegates spec authoring to a subagent so codebase exploration doesn't bloat your main context
- Prints `claude --worktree` commands for parallelisable mini-specs at the handoff

You don't need to remember which skill runs when.

## Files in this bundle

| Plugin file                | Path                          |
| -------------------------- | ----------------------------- |
| `feature.md`               | `commands/feature.md`         |
| `spec-author.md`           | `agents/spec-author.md`       |
| `save-intent-SKILL.md`     | `skills/save-intent/SKILL.md` |
| `intent-template.md`       | `templates/intent.md`         |
| `requirements-template.md` | `templates/requirements.md`   |
| `design-template.md`       | `templates/design.md`         |
| `tasks-template.md`        | `templates/tasks.md`          |

## How the flow works

1. **`/feature [description]`** — orchestrator detects depth, derives a feature name, creates `specs/<feature>/`.
2. **Discovery** — your existing `grill-me` skill runs in main session. Stops when shared understanding is reached.
3. **Save intent** — `save-intent` skill writes `specs/<feature>/intent.md`. Compressed, structured, 30-80 lines.
4. **Gate 1** — proceed / ship as-is / revise / stop. For quick work, "ship as-is" is the default suggestion (skips engineering).
5. **Engineering** (if proceeding) — `spec-author` subagent reads intent.md, explores codebase, writes `requirements.md`, `design.md`, `tasks.md`. Heavy work isolated in subagent context — your main session stays lean.
6. **Gate 2** — proceed / revise / stop.
7. **Handoff** — orchestrator prints the dependency graph and `claude --worktree <name>` commands for the parallelisable mini-specs, with a one-line briefing each. You spawn them in separate terminals.

## Variations

- **Quick path:** auto-detected quick → Gate 1 → ship as-is → handoff with `intent.md` only as briefing. ~3 minutes start to terminal-spawn.
- **Deep path:** auto-detected deep → Gate 1 → engineering → Gate 2 → handoff. ~15-30 minutes depending on grilling depth.
- **Resume path:** `/feature` with no args lists in-progress features. Pick one to continue from whatever phase it's in.

## Iterating

The orchestrator prompt is in `feature.md`. The compression rules are in `save-intent-SKILL.md`. The decomposition rules are in `spec-author.md`. Edit them in place — they're just markdown. Reload by starting a new Claude Code session.

If the auto-detection misclassifies your work too often, edit the heuristics in `feature.md` Phase 0. If decompositions are coming back too coarse or too fine, tune the heuristics in `spec-author.md`. If intents are too verbose or too thin, tune the compression rules in `save-intent-SKILL.md`.
