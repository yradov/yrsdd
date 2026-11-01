---
name: review
description: >
  Use to run the independent whole-feature review after all tasks are done: the full diff against
  the spec, with an end-to-end trace of every acceptance criterion to code and a test. Required on
  mvp and full, skipped on lite. Triggers on "review {slug}", "/yrsdd:review". Verdict PASS or
  CHANGES REQUESTED; findings go back to implement.
argument-hint: "<slug>"
---

# yrsdd:review

Shared rules: [`../_shared/levels.md`](../_shared/levels.md), [`../_shared/state.md`](../_shared/state.md),
[`../_shared/settings.md`](../_shared/settings.md). Per-task reviews already happened inside
`implement`; this pass looks at the **whole change at once, with fresh eyes**.

## Gate

All tasks in `tasks.json` are `done` (or explicitly `blocked` and accepted). `lite` → "per-task
reviews are enough on lite; run `/yrsdd:ship`". No commits on the feature branch and no diff →
"nothing to review".

## Protocol

1. **Scope.** The diff from the branch point to `HEAD` (`git merge-base` with the default branch;
   else the `base` of the first task). Note the `YR-AC` trailers: the criteria the code *claims*.
2. **Dispatch** `yrsdd:reviewer` (`mode: code`, scope: whole feature, model `model_reviewer`,
   language `artifact_language`). It re-reads the spec, design and tasks itself.
   - `mvp`: criteria `AC-xx` vs. diff, plus quality.
   - `full`: also `design.md` contracts and decisions, `test-plan.md` levels, and the **complete
     trace** of every `AC-xx` of the spec (not only the claimed ones) to code and a test.
3. **Collect** findings; drop any without `file:line` and a criterion. A clean result is
   `REVIEW_CLEAN`.
4. **Resolve each finding with the user** (`AskUserQuestion`, group small ones):
   **Fix now** (becomes a follow-up task) / **Defer** (add to the spec's Open questions with owner
   and due; low severity only) / **Not an issue** (record why). A stage-1 finding can be fixed or
   explicitly de-scoped by changing the spec, never ignored.
5. **Record** `docs/features/<slug>/_review/review-<date>[-n].md`: scope (`git diff --stat`),
   gate result, findings table with the decision column, the criteria trace (full), checklist of
   what only a human can check (device, real data, performance numbers). Prose in
   `artifact_language`, literals in English.
6. **Verdict.**
   - No open findings → `PASS`: `stage: ship`.
   - Any "Fix now" → `CHANGES REQUESTED`: append tasks `T<next>` to `tasks.json` (`deps: []`,
     `acs`, `dod`, `source: review-<date>`; on `full` also cards), set `stage: implement`.
7. **Handoff.** `PASS` → `/clear`, `/yrsdd:ship <slug>`. `CHANGES REQUESTED` →
   `/yrsdd:implement <slug>` (no `/clear`: stay in context), then `/yrsdd:review <slug>` again on
   the changed surface.

## Anti-patterns

- Trusting the `YR-AC` trailers as the full criteria set.
- Shipping with an open stage-1 finding.
- Reviewing in the context that wrote the code.
