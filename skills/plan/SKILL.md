---
name: plan
description: >
  Use to split an analyzed feature into small ordered tasks with dependencies and a done-definition
  each, written to tasks.json (and task cards and test-plan.md on full). Triggers on "plan {slug}",
  "break down {slug}", "tasks for {slug}", "/yrsdd:plan", "разбей на задачи".
argument-hint: "<slug>"
---

# yrsdd:plan

Shared rules: [`../_shared/levels.md`](../_shared/levels.md), [`../_shared/state.md`](../_shared/state.md).

## Gate

`brief.md` or `spec.md` exists and `state.md` stage is `plan` (or the user asks to re-plan, then
only unfinished tasks change). `full` also needs `design.md`. Missing → name the skill to run.

## Protocol

1. **Read** the criteria document, `design.md` if any, `CLAUDE.md`, and the code layout.
2. **Slice vertically.** Each task delivers one observable behaviour and fits a single
   test → code → review cycle (rule of thumb: at most ~half a day, one or two layers, one
   reviewable diff). Order: foundations first (types, schema, migration), then behaviour, then
   edges, then UI or wiring. Do not make "write all tests" or "refactor" tasks.
3. **Per task fields:** `id` (`T01…`), `title` (imperative), `deps`, `acs` (ids the task fully or
   partly satisfies), `dod` (the observable result that proves it is done), `files_hint`,
   `status: todo`, `rounds: 0`.
   Counts: `lite` 2–8 tasks, `mvp` 3–15, `full` as needed.
4. **Coverage check:** every `AC-xx` is claimed by at least one task. Every task claims at least
   one AC (except on `lite`, where `dod` references the brief's criteria). Dependencies form no
   cycle. Show any gap to the user and fix it before writing.
5. **Write** `tasks.json` (schema in `../_shared/state.md`). On `full` also:
   - `tasks/Txx.md` per task, self-contained: user story, the AC text copied verbatim with its
     source, the data or contract slice from `design.md`, edge cases, `dod`. An agent that reads
     only this card and the code must be able to do the task.
   - `test-plan.md`: table AC → test level (unit / integration / e2e) → task, plus the data and
     environment strategy.
6. **Confirm** the plan with the user: show the task table (id, title, deps, ACs). `AskUserQuestion`
   Approve / Change. Loop until approved.
7. **Update `state.md`:** `stage: implement`, `current_task: null`, log line. Handoff:
   `/yrsdd:implement <slug>`.

## Anti-patterns

- Tasks that cannot be reviewed alone ("implement the backend").
- A `dod` like "code written". It must be something a test or a person can observe.
- Plans that put all risk in the last task. Risky or uncertain work goes early.
