---
name: codex-overflow
description: Use when executing implementation plans and Codex CLI is installed — dispatches mechanical tasks to Codex in background while handling complex tasks directly
---

# Codex Overflow

Dispatch mechanical plan tasks to Codex CLI in the background while you handle complex tasks directly. You are the lead — classify, dispatch, review, integrate. User never switches tools.

**Announce:** "Using codex-overflow to dispatch mechanical tasks to Codex."

## Activation

During plan execution, at classification phase before execution begins.

**Check availability:** `command -v codex` — if not found, announce "Codex CLI not available" and skip.

**Session start:** if `.codex-dispatch.json` exists in project root, run Session Recovery first.

## Classify Tasks

**Codex-eligible (ALL true):** self-contained in one file/module, no network needed, fully described by plan, clear criteria, target files not in-flight.

**Claude Code-only (ANY true):** depends on incomplete task, needs debugging/iteration, needs MCP/web/live env, multi-file interconnected, unspecified design decisions, file overlap with in-flight Codex task.

**Announce:** "Tasks [N, M, P] are Codex-eligible. Dispatching while working on others directly."

## Dispatch

**Dispatch immediately — parallel with complex tasks, not after.** Start mechanical work the moment you begin the plan.

1. **State file** — write `.codex-dispatch.json` in project root (add to `.gitignore`):
```json
{"dispatched": [{"id": "task-N", "plan_task": "title", "prompt": "full prompt", "target_files": ["path"], "status": "dispatched", "dispatched_at": "ISO-8601"}]}
```
Statuses: `"dispatched"` (running), `"completed"`, `"failed"`. Only add entries when actually dispatching — do not pre-populate queued tasks.

2. **Self-contained prompt** — task description, file paths, conventions, acceptance criteria, constraints ("do not modify files outside X", "follow patterns in Y")

3. **Background bash:** `cd /project && codex --approval-mode full-auto "prompt"` with `run_in_background: true`

4. **Notify:** "Dispatched to Codex: [summary]. Continuing with [next task]."

**Rules:** max 3 concurrent, never overlap file targets, dispatch blockers first.

## Review & Integration

When background bash returns:

1. Read output — verify files produced
2. Spec compliance — matches plan criteria
3. Convention check — naming/patterns match project
4. Integration fit — no conflicts with your parallel changes
5. Run tests

**Decision:** Pass → integrate, status `"completed"`. Minor issues → fix inline, integrate. Fail → discard, status `"failed"`, redo directly.

**Hard rule:** no partial integration. Fully passes or fully discarded.

## Session Recovery

If `.codex-dispatch.json` exists on session start:
- `"dispatched"` + files modified: run review checklist
- `"dispatched"` + files unchanged: ask user — re-dispatch or do directly?
- `"completed"`/`"failed"`: skip

## Red Flags — STOP

- Dispatching without checking file overlap
- Integrating without running tests
- More than 3 concurrent Codex tasks
- Codex prompt references "conversation context" (it has none)
- Target files actively being edited by you
- Doing complex tasks first, mechanical "later" — dispatch NOW
- Vague Codex prompt missing paths/conventions/criteria
