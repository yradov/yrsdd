# yrsdd — spec-driven development in three levels

A Claude Code plugin. One loop on every level: **analyze → (design) → plan → implement
(failing test → code → gate → independent review, per task) → final review → ship.**
State lives in files under `docs/features/<slug>/`, so work resumes after `/clear` or a new session.

## Levels

| | lite | mvp | full |
|---|---|---|---|
| Analysis | `brief.md` | `spec.md` (AC) | `spec.md` + reviewer ambiguity pass |
| Design | – | optional | `design.md` with ADR and alternatives |
| Plan | `tasks.json` | `tasks.json` | + task cards, `test-plan.md` |
| Per-task record | state log | `_log/Txx.md` | `_log/Txx.md` |
| Per-task review | yes | yes | yes (+ design contracts) |
| Final review | – | whole diff vs. spec | + full AC trace, record in `_review/` |
| Ship | summary | changelog + PR text | + docs sync |

Details: `skills/_shared/levels.md`.

## Workflow

Whole project → steps → each step on its own:

1. `/yrsdd:roadmap` once per project: decompose the idea into steps, each with a slug, size and
   dependencies, written to `docs/roadmap.md`.
2. For every step, in dependency order: `/yrsdd:start <slug>` → `analyze` → (`design`) → `plan` →
   `implement` → `review` → `ship`, all with the same `<slug>`. All documents of the step live in
   `docs/features/<slug>/`. `start` marks the step `in-progress`, `ship` marks it `shipped`.
3. The roadmap is optional: `/yrsdd:start "short description"` works for a one-off task.

## Commands

Project-wide (no slug):

| Command | Purpose |
|---|---|
| `/yrsdd:roadmap` | decompose the project, add/split steps, show the next available ones |
| `/yrsdd:config` | document language, models, limits |
| `/yrsdd:resume` | find the unfinished step and continue it |
| `/yrsdd:help [command]` | list commands, agents and where the project stands |

Per step (take the slug from `docs/roadmap.md`):

| Command | Purpose |
|---|---|
| `/yrsdd:start <slug>` | open the step, choose the level |
| `/yrsdd:analyze <slug>` | brief / spec |
| `/yrsdd:design <slug>` | design.md (full; optional on mvp) |
| `/yrsdd:plan <slug>` | tasks.json (+ cards, test plan) |
| `/yrsdd:implement <slug>` | TDD loop with per-task review and rework |
| `/yrsdd:review <slug>` | whole-step review (mvp, full) |
| `/yrsdd:ship <slug>` | gate, verify, changelog, PR text |
| `/yrsdd:resume <slug>` | continue this step after a reset |

Agents: `test-author` (sonnet), `implementer` (sonnet), `reviewer` (opus, read-only),
`debugger` (opus, read-only). Models are configurable per project.

## Who does what

Skills run in your main session and drive the work; agents are workers with their own clean
context that the skills dispatch.

| Step | Skill (main session) | Agents it dispatches |
|---|---|---|
| roadmap | decomposes the project | – |
| start | creates the step folder, level, branch | – |
| analyze | interviews you, writes brief/spec | `reviewer` (mode `spec`, full only) |
| design | explores code, writes design.md | – |
| plan | writes tasks.json, cards, test plan | – |
| implement | orchestrates the loop per task | `test-author` → `implementer` → `reviewer` (`code`), rework back to `implementer` |
| review | whole-step review, resolves findings with you | `reviewer` (`code`, whole step) |
| ship | gate, verification, changelog, PR text | – |

| Agent | Role | Model | Tools |
|---|---|---|---|
| `yrsdd:test-author` | writes the failing test (RED), never production code | sonnet | read, write, bash |
| `yrsdd:implementer` | minimal code to green, refactor, gate; applies review findings | sonnet | read, write, bash |
| `yrsdd:reviewer` | independent review, verdict `PASS` / `CHANGES REQUESTED`; mode `spec` hunts ambiguities | opus | read-only, bash |
| `yrsdd:debugger` | finds the root cause of a stuck failing test or a bug (reproduce, trace, ranked hypotheses, fix plan); never edits | opus | read-only, bash |

`debugger` is dispatched by `implement` on the second failed GREEN attempt (after one plain
retry), before a stronger model is tried. Its verdict decides the next move: `CODE` / `ENVIRONMENT`
→ the implementer retries with the root cause; `TEST` → back to `test-author`; `CRITERION` /
`UNKNOWN` → the question goes to you. You can also call it by hand on any failure:
`Use the yrsdd:debugger agent on the failing test <name>, test command "<cmd>", feature <slug>.`

## Running an agent or a skill by hand

- Skills: type the command, for example `/yrsdd:implement first-mission --task T03`.
- Agents: ask in plain words or mention them with `@` (`/agents` lists them):
  `Use the yrsdd:reviewer agent, mode code, feature first-mission, scope task T03, language ru.`
  `Use the yrsdd:test-author agent for task T05 of first-mission, test command "npm test".`
  An agent knows only what is in its prompt plus the files it reads, so give it the feature
  folder `docs/features/<slug>/`, the task id, the mode or scope, and the language.
  A hand-run agent does not update `state.md` or `tasks.json`; record the result yourself or
  re-run the skill.

## Install

Try without installing (development):

```
claude --plugin-dir "D:\__PROGRAMMING__\___PROJECTS___\_my_sdd_plugin"
```

Install into your Claude Code so every project can use it:

```
/plugin marketplace add D:\__PROGRAMMING__\___PROJECTS___\_my_sdd_plugin
/plugin install yrsdd@yrsdd
```

After editing the plugin files run `/plugin marketplace update yrsdd` (or restart) to pick up changes.
To share it, push this folder to a Git repository and use `/plugin marketplace add <owner>/<repo>`.

## Project settings

`/yrsdd:start` creates `.claude/yrsdd.md` on first use (language of documents, default level,
review round limit, models, commands). Plugin instructions are always English; documents follow
`artifact_language`.
