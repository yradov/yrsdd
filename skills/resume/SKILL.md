---
name: resume
description: >
  Use after a context reset, a new session or an interruption to continue a yrsdd feature from
  exactly where it stopped. Triggers on "resume {slug}", "continue", "where were we",
  "/yrsdd:resume", "продолжи", "на чём остановились". Reads state.md and tasks.json, reports the
  position, and runs the right next skill.
argument-hint: "[slug]"
---

# yrsdd:resume

Shared rules: [`../_shared/state.md`](../_shared/state.md) (this skill is its resume protocol made
executable).

## Protocol

1. **Find the feature.** Argument, else every `docs/features/*/state.md` whose `stage` is not
   `done`. None → "nothing to resume; start with `/yrsdd:start`". Several → ask which.
2. **Read** `state.md`, `tasks.json`, the tail of the log, `_log/` of the last done task, the
   latest `_review/` record, and `git status` / `git log -5`.
3. **Check consistency** and say what you find:
   - `git` has uncommitted changes while `task_phase` is `red`/`green`/`gate`: they belong to
     `current_task`; keep them.
   - `tasks.json` says `done` but there is no commit with `YR-Task: Txx`: warn.
   - Front matter and `tasks.json` disagree: `tasks.json` + git win; fix `state.md` and log it.
4. **Report in 5 lines or fewer** (language of the user): feature and level, stage, task
   `Txx` and phase and review round (if any), progress `done/total`, the next action.
5. **Continue** by running the skill for the stage: `analyze`, `design`, `plan`, `implement`,
   `review` or `ship`. Pass the slug. `implement` continues the saved phase.

Do not redo finished work and do not re-ask questions that `state.md` Decisions already answers.
