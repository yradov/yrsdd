# State — how work survives a context reset

> Reference only, not a skill. All state lives in files under `docs/features/<slug>/`. A new
> session (or an agent after `/clear`) must be able to continue by reading these files alone.

## Project layout

```
docs/
├─ roadmap.md              # project level: steps, slugs, sizes, deps, status (skill roadmap)
└─ features/
   └─ <slug>/              # one folder per roadmap step; <slug> comes from roadmap.md
      ├─ state.md  tasks.json  brief.md | spec.md  (design.md  test-plan.md  tasks/)
      ├─ _log/  _review/  changelog.md
```

Skills create `docs/` and `docs/features/` when they are missing. Everything an agent needs for a
step is inside its folder; agents never need another step's folder except to read a shipped
dependency. The roadmap is optional: without it a slug is simply chosen at `start`.

## `state.md` (required, every level)

```md
---
slug: <slug>
level: lite | mvp | full
stage: analyze | design | plan | implement | review | ship | done
current_task: T03          # null when no task is in flight
task_phase: red | green | gate | review | rework   # null when current_task is null
review_round: 0            # review rounds spent on current_task
updated: 2026-11-01
---

# <feature title>

## Next action
`/yrsdd:implement <slug>` — one line: the exact command to run next and why.

## Decisions
- <date> <decision> — <reason>. Keep short; long decisions go to `design.md`.

## Log
- <date> <stage/task>: <what happened, one line>. Append-only, newest last.
```

Rules:
- `state.md` is rewritten at **every phase change** (not only at task end). It is the single place
  that says where the work stopped.
- The front matter is the machine-readable truth. The body is for people and for the next agent.
- Never delete log lines. Corrections are new lines.

## `tasks.json` (required, every level)

```json
{
  "slug": "<slug>",
  "level": "lite",
  "tasks": [
    {
      "id": "T01",
      "title": "Short imperative title",
      "deps": [],
      "acs": ["AC-01"],
      "dod": "Observable result that proves the task is done.",
      "files_hint": ["src/..."],
      "status": "todo",
      "rounds": 0,
      "file": "docs/features/<slug>/tasks/T01.md"
    }
  ]
}
```

- `status`: `todo` → `in_progress` → `done`; or `blocked` (with `blocked_reason`).
- `acs` is empty only in `lite` (the brief's criteria are referenced in `dod`).
- `file` exists only on `full`.
- `rounds`: total review rounds the task has used (for the report).

## `_log/Txx.md` (mvp and full) — the handover note between tasks

Written when a task reaches `done`. Short, factual, for the next agent that knows nothing:

```md
# T03 — <title>
- Result: done | blocked
- Files changed: <list>
- Tests added: <list, with the command that runs them>
- Gate: tests <n passed>, lint <ok>, types <ok>
- Review: PASS after <n> round(s); findings fixed: <short list>
- Notes for the next task: <anything not obvious from the diff: decisions, traps, TODOs>
- Commit: <hash>
```

## Resume protocol (what every skill does first)

1. Find the feature: the `<slug>` argument, or the only folder in `docs/features/` whose `state.md`
   has `stage` other than `done`. Several candidates → ask which.
2. Read `state.md`, then `tasks.json`, then the artifacts of the current stage.
3. Trust the files over the conversation. If the conversation disagrees with `state.md`, follow
   `state.md` and say so.
4. If `task_phase` is set, continue **that phase** of `current_task` (do not restart the task):
   - `red` → check whether a failing test already exists; reuse it.
   - `green` → run the tests, continue from the current failure.
   - `gate` → re-run the gate.
   - `review` → dispatch the reviewer again (reviews are cheap and clean-context).
   - `rework` → read the last findings in `_log/Txx.md` or `state.md`, fix them.

## Handoff block (last output of every skill)

```md
## ✅ <skill> — <slug>

**What I did**
- <1–3 bullets>

**Files to look at**
- `docs/features/<slug>/<file>` — <what to check>

**Run next**
1. `/clear` (optional but recommended: the next step re-reads everything from disk)
2. `/yrsdd:<next> <slug>`
```

When the next step is a rework, say "no `/clear`" and why.
