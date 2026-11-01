---
name: analyze
description: >
  Use to analyze a task before any code: turn the raw idea into brief.md (lite) or spec.md (mvp,
  full) with testable acceptance criteria. Triggers on "analyze {slug}", "write the spec",
  "/yrsdd:analyze". Asks only the questions that change what gets built, writes the document in
  the configured artifact language, and on full level runs a reviewer ambiguity pass.
argument-hint: "<slug>"
---

# yrsdd:analyze

Shared rules: [`../_shared/levels.md`](../_shared/levels.md), [`../_shared/state.md`](../_shared/state.md),
[`../_shared/settings.md`](../_shared/settings.md).

## Gate

`docs/features/<slug>/state.md` must exist, otherwise say "run `/yrsdd:start` first". Follow the
resume protocol. If `stage` is already past `analyze`, ask before rewriting anything.

## Protocol

1. **Read** the original task text (first log line of `state.md`), `CLAUDE.md`, and the code the
   task touches (explore enough to ask informed questions; use `Explore` agent for big repos).
2. **Question budget:** `lite` 3, `mvp` 6, `full` as many as needed but grouped, max 4 per round.
   Ask with `AskUserQuestion`, recommended option first. Ask only what changes the build: scope
   edges, behaviour on error and empty input, data and compatibility, non-functional limits.
   For anything the user does not decide, write the default you chose in the document, marked
   `Assumption:`. Never silently pick between two real interpretations; present both.
3. **Write the document** in `artifact_language`:
   - `lite` → `brief.md`: Goal (2–3 sentences) · Acceptance criteria (3–7, each one a testable
     sentence, ids `AC-01…`) · Out of scope · Risks and assumptions.
   - `mvp` / `full` → `spec.md`: Context and problem · User stories (`US-01…`) · Acceptance
     criteria (`AC-01…`; each states trigger, action, observable result; include at least one
     error case and one boundary case per story) · Non-functional requirements (only measurable
     ones) · Out of scope · Open questions (owner + due).
   A criterion that cannot be turned into a test is rewritten until it can.
4. **Full level only: ambiguity pass.** Dispatch the `reviewer` agent (`subagent_type:
   "yrsdd:reviewer"`, `mode: spec`, model from `model_reviewer`) on `spec.md`. For each finding ask
   the user: **Resolve** (tighten the spec now) / **Defer** (Open questions with owner and due) /
   **Not an issue** (record why). No finding stays dangling.
5. **Show the document** summary to the user (not the whole file): the criteria list and the
   assumptions. Ask for confirmation with `AskUserQuestion` (Approve / Change). Loop until
   approved.
6. **Update `state.md`:** `stage: design` for `full`, `stage: plan` otherwise (for `mvp`, mention
   `design` is optional). Log line. `Next action` set.
7. **Handoff.** Files to look at: the document. Next: `/yrsdd:design <slug>` (full) or
   `/yrsdd:plan <slug>`.

## Anti-patterns

- Interviewing the user about things the code already answers. Read the code first.
- Acceptance criteria like "works well" or "is fast". Make them measurable or drop them.
- Writing the solution into the spec. The spec says what, `design.md` says how.
