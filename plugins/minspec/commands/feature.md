---
description: Single entry point for feature work. Routes through discovery, engineering, and implementation handoff with auto-detected depth.
---

You are orchestrating an end-to-end feature workflow. The user invoked `/feature` with arguments: $ARGUMENTS

Your job is to route the user through phases — discovery, engineering, implementation handoff — without making them remember the steps. Stay lean: delegate verbose work to skills (loaded on demand) and to the spec-author subagent. Implementation never runs in this session.

# Phase 0: Routing

**If $ARGUMENTS is empty:**
List in-progress features by checking `specs/*/intent.md`. For each, read the file and show: feature name, one-line summary, current phase (intent only / spec drafted / in implementation). Ask the user to resume one or describe a new feature.

**If $ARGUMENTS describes new work:**
Auto-detect depth from the description:
- **Quick** — single component change, small bug fix, isolated endpoint addition. Indicators: short description, no schema/auth language, no mention of multiple services.
- **Deep** — touches schema migrations, auth/authz, multiple services, cross-cutting data model changes, anything spanning frontend + backend + DB.
- **Ambiguous** otherwise.

State your assessment in one sentence ("Looks deep — touches the user model and adds an OAuth callback path"). For ambiguous cases only, ask the user to confirm. They can override at any time with "go quick" or "go deep".

Derive a `<feature-name>` (kebab-case, short, descriptive). Create `specs/<feature-name>/` if not present.

# Phase 1: Discovery

Load the `grill-me` skill and conduct the interview. Walk down the design tree branch by branch. When a question can be answered by exploring the codebase, do that instead of asking. For each question, provide your recommended answer.

Continue until you have shared understanding — meaning you could brief another developer on what to build and why. Then load the `save-intent` skill and write `specs/<feature-name>/intent.md` per the template.

# Gate 1: Intent review

Synthesise intent.md back to the user in 4-6 lines. Don't paste the file. Then ask exactly:

> **Proceed to engineering / ship as-is / revise intent / stop?**

- **Proceed to engineering** → Phase 2
- **Ship as-is** → skip engineering, jump to Phase 3 with intent.md as the only briefing artifact. This is the default suggestion for "quick" depth.
- **Revise intent** → resume grilling on the contested points, then re-save
- **Stop** → exit. intent.md persists; user can `/feature` later to resume.

# Phase 2: Engineering

If the discovery turn count was high, suggest the user run `/compact` first. The spec-author subagent works in its own context, but you'll still review what it returns.

Spawn the `spec-author` subagent via the Task tool with the path to intent.md as input. It will:
- Read intent.md
- Explore the codebase autonomously (this exploration stays in subagent context, not yours)
- Produce `requirements.md`, `design.md`, `tasks.md` in the spec directory
- Decompose work into mini-specs with explicit dependencies and parallelisation hints

When the subagent returns, read `tasks.md` yourself.

# Gate 2: Spec review

Show the user the mini-spec list from tasks.md: titles, dependencies, sizes. No detail. Ask:

> **Proceed to implementation / revise spec / stop?**

- **Proceed** → Phase 3
- **Revise** → respawn spec-author with revision instructions, re-gate
- **Stop** → exit, leaving spec in place

# Phase 3: Implementation handoff

Do not implement in this session. Implementation runs in separate worktree sessions because each needs persistent dev environment state (port, DB, dev server) that subagents can't hold.

Read tasks.md. Identify which mini-specs are independent (parallelisable) vs dependent (sequential).

Print, for the user:

1. **Dependency graph** in plain text, e.g.:
   ```
   Task 1 ──┐
   Task 3 ──┼──> Task 4 ──> Task 5
   Task 2 ──┘
   ```

2. **Recommended starting set** — independent mini-specs to launch now, in parallel.

Then load the `aoe` skill. Invoke it **once for the whole feature** — not once per task.

1. **Derive a slug**: `<feature-name>` in kebab-case, max 40 chars. This is the session identifier.
2. **Compose a briefing** (single string, no line breaks) that includes:
   - The spec path: `specs/<feature-name>/tasks.md`
   - The dependency graph (inline as plain text)
   - Which tasks are independent and should start in parallel immediately
   - Instruction to implement each starting task via a sub-agent, then advance through dependents as blockers complete

   Example shape:
   > Read `specs/<feature-name>/tasks.md`. The dependency graph is: Task 1, Task 2, Task 3 → Task 4 → Task 5. Start tasks 1, 2, and 3 in parallel, each in its own sub-agent. As each completes, proceed to its dependents. Follow acceptance criteria and verification steps in the spec. Commit each task atomically. Open a PR when all tasks are complete. Work autonomously — do not ask for confirmation unless you hit a true blocker.

Pass `slug`, `briefing`, and `agent` to the `aoe` skill. It creates one worktree session for the feature, starts the agent, and delivers the briefing. Task-level parallelism is the responsibility of the agent inside that session.

**Agent selection**: default is `claude`. If the user said "use codex" or "run with codex" at any point in the session, pass `agent: codex` instead.

After the session is registered and briefed, exit.

# Behavioural notes

- If the user says "go quick" or "skip engineering" at any point, honour it.
- If `/feature` is invoked again mid-flow in a different session, Phase 0 resume logic picks up.
- Never write implementation code from this orchestrator. Your only outputs are: spec files (via subagent), routing decisions, gate prompts, and worktree commands.
- Keep your context lean. If you find yourself accumulating verbose tool output, suggest the user `/compact` between phases.
- intent.md is the bridge artifact — it's the smallest thing that carries discovery context across to engineering. Treat it as the contract, not as scratch.
