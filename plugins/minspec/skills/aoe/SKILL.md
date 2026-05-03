---
name: aoe
description: Use Agent of Empires (aoe) for automated implementation of spec, decomposed or otherwise, in a git worktree
allowed-tools: Bash(aoe list *) Bash(aoe add *) Bash(aoe session *) Bash(aoe send *) Bash(git branch *) Bash(git worktree *)
---

You will receive a `<slug>`, a `<briefing>`, and optionally an `<agent>` from the caller. If not provided, derive:

- **slug**: kebab-case, max 40 chars, no special characters. Format: `<feature-name>` (e.g. `oauth-login`, `conflict-detection`). One session per feature, not per task.
- **briefing**: instruction to read the spec, implement all tasks using sub-agents, and follow the dependency graph.
- **agent**: `claude` (default) or `codex`. Use `codex` only when explicitly requested. Both are supported by aoe.

### 1. Check for existing session

```bash
aoe list --json
```

Scan the output for an entry whose `title` matches `<slug>`. If found, note the existing session and skip to step 3.

### 2. Create session

Build the `aoe add` command from these parts:

| Part | Value |
|------|-------|
| Base | `aoe add --yolo --trust-hooks` |
| Worktree (from main) | `-w <slug> -b` |
| Title | `-t "<slug>"` |
| Agent (non-default) | `-c codex` — omit entirely when using claude |

**If already inside a worktree** (current directory is not `main`), register the current directory without creating a new worktree:

```bash
aoe add . --yolo --trust-hooks -t "<slug>"                  # claude (default)
aoe add . --yolo --trust-hooks -t "<slug>" -c codex         # codex
```

**If on main**, let aoe create the worktree and a new branch in one step:

```bash
aoe add --yolo --trust-hooks -w <slug> -b -t "<slug>"               # claude (default)
aoe add --yolo --trust-hooks -w <slug> -b -t "<slug>" -c codex      # codex
```

- `-w <slug>` names both the worktree directory and the git branch.
- `-b` creates the branch; omit if the branch already exists.
- `-t "<slug>"` sets the session title used as the identifier in all subsequent commands.
- `--trust-hooks` suppresses the interactive trust prompt that codex shows on first run. Required for codex; harmless for claude.
- Do **not** use `--launch` — it tries to attach to a terminal and exits non-zero in non-interactive contexts, breaking any command chain.

If `aoe add` exits non-zero (session already exists), continue with the existing session title.

### 3. Start the session

```bash
aoe session start "<slug>"
```

This starts the agent process inside a tmux session. It must complete before sending any message.

### 4. Send bootstrapping message

Wait a moment for the agent to initialise, then send the briefing:

```bash
aoe send "<slug>" "<briefing>"
```

`<briefing>` must be a single shell-quoted string. Do not split it across multiple `send` calls. A good briefing looks like:

> Read `specs/<feature-name>/tasks.md`. The dependency graph is: Task 1, Task 2, Task 3 → Task 4 → Task 5. Start tasks 1, 2, and 3 immediately, each in its own sub-agent. As each completes, proceed to its dependents. Follow acceptance criteria and verification steps in the spec. Commit each task atomically. Open a PR when all tasks are complete. Work autonomously — do not ask for confirmation unless you hit a true blocker.

### 5. Stop and hand off

Print:

> Worktree registered as session **`<slug>`** (agent: `<agent>`). Switch to the aoe TUI (`aoe`) and start the session to continue with engineering review and implementation.

Do NOT proceed to:

- Read code in the worktree
- Produce or modify spec files
- Dispatch sub-agents
