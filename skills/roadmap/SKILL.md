---
name: roadmap
description: >
  Use at the project level, before any feature work: decompose a whole product or epic into
  ordered steps, give each step a slug, size and dependencies, and keep them in docs/roadmap.md.
  Also adds, splits, reorders or re-sizes steps later. Triggers on "roadmap", "break the project
  into steps", "decompose", "/yrsdd:roadmap", "роадмап", "разбей проект на шаги".
argument-hint: "[idea text or path to a brief/ТЗ] | add <description> | split <slug> | show"
---

# yrsdd:roadmap

Project-wide skill: it works on the whole project, not on one step. Each step it creates is later
built on its own with `/yrsdd:start <slug>`. Shared rules: [`../_shared/levels.md`](../_shared/levels.md),
[`../_shared/state.md`](../_shared/state.md), [`../_shared/settings.md`](../_shared/settings.md).

## Output: `docs/roadmap.md`

Create `docs/` and `docs/features/` if they do not exist. Prose in `artifact_language`; headings,
column names, statuses and slugs in English.

```md
---
updated: 2026-11-01
---
# Roadmap

## Destination
One sentence: what the product is when all steps are shipped.

## Steps
| # | Step | Slug | Source | Size | Deps | Status |
|---|---|---|---|---|---|---|
| 1 | One or two sentences: what the user can do after this step | `project-skeleton` | brief.md §3 | S | – | idea |
| 2 | … | `first-mission` | tz.md §12 | L | 1 | idea |

## Open decisions
- <decision still needed>, owner, due.

## Execution order
Waves: steps with no unshipped dependencies, grouped. Steps in one wave may run in parallel.
```

- **Slug:** unique, kebab-case, 2–4 words, stable forever (it names the folder
  `docs/features/<slug>/` and the git branch). Never rename a slug after `start`.
- **Size** → suggested level: `XS`, `S` → `lite`; `M` → `mvp`; `L`, `XL` → `full`.
  The level is confirmed again at `/yrsdd:start`.
- **Status:** `idea` → `in-progress` (set by `start`) → `shipped` (set by `ship`); or `blocked`.
- **Source:** where in the input documents the step comes from (file and section).

## Protocol

1. **Gather input.** The argument (text or file path), `docs/idea-brief.md`, a ТЗ/PRD,
   `CLAUDE.md`, README, and the existing code if any. A first step is usually a skeleton when the
   repo is empty. Existing `docs/roadmap.md` → update mode (below), never overwrite.
2. **Ask only what changes the decomposition** (`AskUserQuestion`, at most 4 per round): the
   destination, what must work first (the riskiest or most valuable path), hard constraints
   (platform, deadline, data), what is explicitly out of scope.
3. **Decompose.** Each step is one shippable, user-visible increment that can be built through the
   full loop in one go (size `XL` means: split it). Order by dependencies and by risk, riskiest
   early. Do not put "refactoring" or "tests" as steps; do not plan far ahead in detail: later
   steps stay short and may be marked `idea` with an open decision.
4. **Dependencies:** list only real ones (step N needs the code or data of step M). Check there are
   no cycles. Compute waves.
5. **Confirm.** Show the steps table (not the whole file). `AskUserQuestion` Approve / Change per
   fork; loop until approved. Then write `docs/roadmap.md`.
6. **Handoff.** List the first available steps (no unshipped deps) and the command:
   `/yrsdd:start <slug>`.

## Update mode (`add`, `split`, `show`, or a plain re-run)

- `show`: print the table and the next available steps; change nothing.
- `add <description>`: append a step, ask for size and deps, derive the slug.
- `split <slug>`: only for steps with status `idea`; replace by 2+ steps, repoint deps. A step
  already `in-progress` is never split or renamed; ask the user to finish or abandon it.
- Re-sizing or re-ordering `idea` steps is allowed. Never edit the status of a step by hand
  except `blocked` / back to `idea` at the user's request; `start` and `ship` own the transitions.
- Always bump `updated` and keep a one-line note in `Open decisions` for anything unresolved.

## Anti-patterns

- A roadmap with dates or estimates in days. This is order and size, not a schedule.
- Steps that are layers ("backend", "UI"). Slice by user-visible behaviour.
- Detailed specs inside the roadmap. That is `analyze` for one step at a time.
