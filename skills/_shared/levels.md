# Levels — what each level requires

> Reference only, not a skill. Every skill reads the feature's `level` from
> `docs/features/<slug>/state.md` and follows this table. The loop is identical on all levels:
> **analyze → (design) → plan → implement (RED → GREEN → GATE → REVIEW per task) → final review → ship.**
> Levels differ only in how deep each step goes and which documents it leaves on disk.

| Step | `lite` | `mvp` | `full` |
|---|---|---|---|
| When to choose | Small change, script, prototype, fix. Under ~1 day. | A real feature, one contributor, a few days. | Long-lived or risky feature, several modules, public contract, data migration. |
| **analyze** | `brief.md`: goal, 3–7 acceptance criteria, out of scope, risks. At most 3 questions. | `spec.md`: problem, user stories, acceptance criteria `AC-01…`, non-functional needs, out of scope, open questions. At most 6 questions. | `spec.md` as in mvp + an ambiguity pass by the reviewer agent (mode `spec`); every finding resolved or deferred with owner. |
| **design** | skipped | optional short `design.md` (only if the feature touches structure, data or a contract) | required `design.md`: components, data, contracts, error handling, risks, decisions (ADR section). |
| **plan** | `tasks.json`, 2–8 tasks, each with a done-definition. | `tasks.json` + each task has `acs`, `deps`, `dod`, `files_hint`. | mvp + `tasks/Txx.md` card per task (self-contained brief) + `test-plan.md` (AC → test level map). |
| **implement** | RED → GREEN → GATE → REVIEW per task. | same | same |
| **per-task record** | one line in the `state.md` log | `_log/Txx.md` handover note | `_log/Txx.md` handover note |
| **per-task review** | reviewer checks the diff against the task's `dod` and the brief. | reviewer checks the diff against the task's `acs` in `spec.md`. | same + reviewer re-reads `design.md` for contract fidelity. |
| **final review** (skill `review`) | skipped (per-task reviews are enough) | required, whole diff vs. `spec.md` | required, whole diff vs. spec + design + full AC trace, record in `_review/`. |
| **ship** | summary in `state.md` | `changelog.md` + PR text | `changelog.md` + PR text + docs sync check. |
| Required artifacts | `state.md`, `brief.md`, `tasks.json` | `state.md`, `spec.md`, `tasks.json`, `_log/`, `_review/` | everything above + `design.md`, `test-plan.md`, `tasks/` |

## Rules that hold on every level

1. Analysis happens before code. No `implement` without `brief.md` or `spec.md` and `tasks.json`.
2. Tests first. A task's test must be seen failing for the right reason before production code is written.
3. A task is `done` only after the gate is green **and** its review verdict is `PASS`.
4. Anything that must survive a context reset is written to disk (see `state.md` rules).
5. Raising the level mid-feature is allowed (`/yrsdd:start <slug> --level mvp`): add the missing artifacts, never rewrite finished ones. Lowering it is not allowed once code exists.
