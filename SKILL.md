---
name: resilient-continuation
description: >
  Resilient auto-resume agent for long-running, multi-step tasks.
  Use when: (1) executing large or multi-stage tasks, (2) user complains about agent freezing, resetting, or losing progress,
  (3) user asks for auto-continue, auto-resume, recovery after stop/continue/crash,
  (4) user wants work to resume from last checkpoint after pressing Continue/Resume,
  (5) any coding, file, or multi-tool task that may exceed response limits or timeout.
  NOT for: simple one-shot questions, single-tool calls, or trivial edits.
---

# Resilient Continuation / Auto-Resume Agent

Устойчивое выполнение длинных задач с автопродолжением и восстановлением после сбоев.

## Core Rules

### Before Starting
1. Compose a brief plan (bullet list, not essay).
2. Break the task into numbered micro-steps.
3. Create **CHECKPOINT 0** with initial state.

### Checkpoint Format
After every meaningful action, record:

```
[CHECKPOINT N]
- Goal: <overall goal>
- Current step: <N of M>
- Done: <completed steps>
- Last success: <last successful action>
- Changed: <files/artifacts modified>
- Created: <new files/artifacts>
- Errors: <any issues>
- Next: <immediate next step>
- Resume hint: <how to safely continue from here>
```

### On Interruption
If any of these occur: freeze, truncation, length limit, stop, reload, crash, timeout, tool error, window close, Continue/Resume button — then:

1. **Do NOT restart from scratch.**
2. Recover from last CHECKPOINT.
3. Begin response with: `Восстановил состояние. Продолжаю с шага: [N]`
4. Execute only remaining/incomplete work.
5. Do not repeat already-completed steps.

### Retry Strategy (max 3 attempts)
1. Retry the exact failed sub-step.
2. Same sub-step in smaller chunks.
3. Alternative approach for the same sub-step.
Update CHECKPOINT after each attempt.

### User Commands
| Command | Action |
|---|---|
| `продолжай` / `continue` / `resume` | Resume from last CHECKPOINT |
| `перезапусти последний шаг` | Retry only the last failed sub-step |
| `собери состояние` / `что сделано?` | Show status summary |

### Never
- Re-request already-known data.
- Rewrite entire output after a minor failure.
- Duplicate completed work.
- Roll back successful steps without cause.

### Always
- Preserve maximum context.
- Record where you stopped.
- Show the concrete next step.
- Split long work into iterations.

## Extended: Coding & File Tasks

### Session State
Maintain after each stage:
- `TASK_LIST` — all planned steps
- `DONE` — completed
- `IN_PROGRESS` — current
- `BLOCKERS` — issues
- `NEXT_STEP` — immediate next action

### File Operations
**Before** changing files: note which files, why, what's verified.
**After** changing files: note which changed, brief diff summary, remaining work.

### On Coding Session Break
1. Restore SESSION STATE first.
2. Then continue execution.

### On Hung Command
Do not replay the entire chain. Retry only the current narrow step.

### Partial Results
Use existing partial results as foundation — never discard and restart.

## Anti-Repeat Mode
After any interruption, if work is partially done:
1. Verify completion of each step.
2. Mark verified steps in CHECKPOINT.
3. Continue from first incomplete step.

## Safe Recovery Mode
When uncertain:
1. Do NOT restart the whole process.
2. Assess if continuation is possible.
3. If yes → continue.
4. If no → restore only the minimum necessary part.
