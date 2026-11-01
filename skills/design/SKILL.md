---
name: design
description: >
  Use to design HOW a feature will be built, after analyze: components, data, contracts, error
  handling, risks and decisions, written to design.md. Required on full level, optional on mvp,
  skipped on lite. Triggers on "design {slug}", "architecture for {slug}", "/yrsdd:design".
argument-hint: "<slug>"
---

# yrsdd:design

Shared rules: [`../_shared/levels.md`](../_shared/levels.md), [`../_shared/state.md`](../_shared/state.md).

## Gate

`spec.md` exists and is approved (`state.md` stage is `design` or later). `lite` → say "design is
skipped on lite" and hand off to plan.

## Protocol

1. **Read** `spec.md`, `CLAUDE.md`, and map the code the feature touches: module boundaries,
   patterns already used, where similar features live, data and migration conventions, test
   conventions. Use the `Explore` agent for a large repo; do not guess.
2. **Draft `design.md`** in `artifact_language`, sections:
   - **Approach**: the chosen solution in a paragraph, and why.
   - **Alternatives considered**: 1–3, with the reason each lost.
   - **Components and responsibilities**: what is new, what changes, dependency direction.
   - **Data**: tables, fields, constraints, migration and rollback if any.
   - **Contracts**: public functions, endpoints or events, with error cases.
   - **Error and edge handling**: how each AC error case surfaces.
   - **Risks and mitigations**.
   - **Decisions (ADR)**: for each hard-to-reverse choice one entry: context, decision,
     consequences. Only decisions that cross modules or are expensive to undo.
   Reuse existing patterns and components; a new one needs a stated reason.
3. **Trace:** each `AC-xx` appears in at least one component or contract. An AC that fits nowhere
   means the design or the spec is incomplete: fix the document, do not hide it.
4. **Confirm** with the user: summarize approach, alternatives and decisions; `AskUserQuestion`
   Approve / Change for each real fork. Loop until approved.
5. **Update `state.md`:** `stage: plan`, log line, next action. Handoff: `/yrsdd:plan <slug>`.

## Anti-patterns

- Re-deciding what the repo already settled (framework, layout, naming).
- A design that is a restatement of the spec. It must name concrete modules, data and contracts.
