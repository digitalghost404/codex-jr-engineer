# Codex Jr. Engineer

A Claude Code skill that automatically dispatches mechanical tasks to OpenAI's Codex CLI in the background, while Claude Code handles complex work directly. Think of it as giving Claude Code a junior engineer that works in parallel — you never switch tools, never manage a second workflow, and never wait idle during rate limits.

## The Problem

If you're on Claude Code's Max plan, you've hit rate limits during heavy sessions. Every task — whether it's designing an API or adding struct tags — burns the same Opus token budget. Mechanical work (writing tests, adding comments, refactoring to a pattern) eats through your quota just as fast as the complex architecture work that actually needs Opus-level reasoning.

Meanwhile, OpenAI's Codex CLI sits on a completely separate token pool. It's good at self-contained, well-specified tasks. But using it means switching terminals, writing prompts manually, tracking what you dispatched, reviewing output, and integrating changes — all of which breaks your flow.

## The Solution

This skill makes Codex invisible infrastructure. During plan execution, Claude Code automatically:

1. **Classifies** each task as mechanical (Codex-eligible) or complex (Claude Code-only)
2. **Dispatches** mechanical tasks to Codex via background bash — immediately, in parallel with complex work
3. **Reviews** every piece of Codex output against a 5-point checklist before it touches your codebase
4. **Integrates** passing work or discards and redoes failed work directly
5. **Recovers** across sessions via a lightweight state file

You talk to Claude Code. Codex runs in the background. Nothing unreviewed enters your codebase.

## How It Works

### Task Classification

When executing a plan, the skill evaluates each task:

**Codex-eligible (all must be true):**
- Self-contained in one file or module
- No network access needed (Codex runs sandboxed)
- Fully described by the plan — no conversation context required
- Clear completion criteria
- Target files aren't being edited by Claude Code or another Codex task

**Claude Code-only (any triggers this):**
- Depends on an incomplete task's output
- Needs interactive debugging or iteration
- Requires MCP servers, web search, or live environment access
- Touches multiple interconnected files
- Contains design decisions not specified in the plan

### Dispatch

Codex tasks are dispatched immediately — not after complex tasks finish. The skill runs up to 3 concurrent Codex tasks, never overlapping file targets.

Each dispatch includes a self-contained prompt with exact file paths, project conventions, acceptance criteria, and constraints. Claude Code writes a `.codex-dispatch.json` state file to track what's in flight.

### Review

When Codex completes a task, Claude Code runs a 5-step review:

1. **Read output** — verify expected files were produced
2. **Spec compliance** — matches the plan's acceptance criteria
3. **Convention check** — naming, patterns, error handling match the project
4. **Integration fit** — no conflicts with parallel changes
5. **Run tests** — project test suite or relevant subset

Output either fully passes and gets integrated, or gets fully discarded. No partial integration, ever.

### Session Recovery

If a session dies mid-execution, the state file persists. Next session, Claude Code picks up where it left off — reviewing completed Codex output, re-dispatching stalled tasks, or handling them directly.

## Installation

Copy `SKILL.md` to your Claude Code personal skills directory:

```bash
mkdir -p ~/.claude/skills/codex-overflow
cp SKILL.md ~/.claude/skills/codex-overflow/SKILL.md
```

That's it. The skill is global — it works on every project, no per-project setup needed.

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI or desktop)
- [Codex CLI](https://github.com/openai/codex) installed and on your PATH
- An existing plan-based workflow (the skill activates during plan execution)

## What You'll See

During a typical session:

```
> Using codex-overflow to dispatch mechanical tasks to Codex.
> Tasks 2, 3, 5 are Codex-eligible. Dispatching while working on others directly.

> Dispatched to Codex: unit tests for pkg/config/. Continuing with Task 1.
> Dispatched to Codex: godoc comments for pkg/api/. Continuing with Task 1.

  [Claude Code works on Task 1 — complex architecture work]

> Codex completed: unit tests for pkg/config/. Reviewed — clean pass. Tests passing. Integrated.
> Codex completed: godoc comments for pkg/api/. Reviewed — 1 minor fix (missing return doc). Integrated.

  [Claude Code continues with Task 4]
```

You never switch windows. You never manage Codex. You just work.

## When It Doesn't Activate

- Codex CLI not installed (graceful skip, all tasks run in Claude Code)
- No plan being executed (ad-hoc work, brainstorming)
- All tasks are complex (nothing to dispatch)

The skill degrades gracefully — your workflow never breaks, it just runs without the parallel boost.

## License

MIT
