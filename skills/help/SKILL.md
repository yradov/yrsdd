---
name: help
description: >
  Use to see all yrsdd commands, agents and the order they are used in, plus where the current
  project stands. Triggers on "help", "what commands are there", "/yrsdd:help", "помощь",
  "какие есть команды".
argument-hint: "[command name]"
---

# yrsdd:help

Read-only. Never creates or changes files. Reply in the language of the user's message.

## Protocol

1. **With an argument** (`/yrsdd:help implement`): read `../<name>/SKILL.md` and explain that one
   command in a few lines: what it does, what it needs, what it writes, what comes next.
   Unknown name → show the table below.
2. **Without an argument**, print:

   **Project-wide (no slug)**

   | Command | What it does |
   |---|---|
   | `/yrsdd:roadmap` | decompose the project into steps with slugs, sizes, deps → `docs/roadmap.md` |
   | `/yrsdd:config` | document language, models, review round limit, commands |
   | `/yrsdd:resume [slug]` | continue an unfinished step after a reset |
   | `/yrsdd:help [command]` | this list |

   **Per step (use the slug from the roadmap), in order**

   | # | Command | What it does |
   |---|---|---|
   | 1 | `/yrsdd:start <slug>` | open the step, choose level lite / mvp / full |
   | 2 | `/yrsdd:analyze <slug>` | `brief.md` (lite) or `spec.md` with acceptance criteria |
   | 3 | `/yrsdd:design <slug>` | `design.md` (full; optional on mvp; skipped on lite) |
   | 4 | `/yrsdd:plan <slug>` | `tasks.json` (+ task cards, test plan on full) |
   | 5 | `/yrsdd:implement <slug> [--task Txx]` | per task: failing test → code → gate → review; rework on findings |
   | 6 | `/yrsdd:review <slug>` | whole-step review (mvp, full) |
   | 7 | `/yrsdd:ship <slug>` | gate, verification, changelog, PR text |

   **Agents** (dispatched by the skills; can be called by hand):
   `yrsdd:test-author` (writes the failing test), `yrsdd:implementer` (code to green, rework),
   `yrsdd:reviewer` (PASS / CHANGES REQUESTED; mode `spec` for ambiguities),
   `yrsdd:debugger` (root cause of a stuck failure, never edits). Models: `/yrsdd:config`.

3. **Where you are.** If `docs/roadmap.md` exists, list steps with status `in-progress` and the
   steps that can start now (deps all `shipped`). Else list `docs/features/*/state.md` with
   `stage` not `done`. For each in-progress step print: slug, level, stage, current task and
   phase, and the exact next command (from `state.md` → `Next action`).
   Nothing found → "no work yet: `/yrsdd:roadmap` for a whole project or `/yrsdd:start
   "description"` for one task".
