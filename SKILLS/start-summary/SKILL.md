---
name: start-summary
description: Use when a final conversation summary or design recap must become concrete spec and implementation plan documents for downstream engineers, especially when optional or ambiguous wording would leave implementers too much freedom.
---

# Start Summary

## Overview

Turn the last approved summary or design recap into two on-disk documents in two locked phases: converge the spec first, freeze it, re-read the frozen spec fresh, then derive and converge the implementation plan from that immutable spec. Do not reopen brainstorming. Do not leave optional wording. Use one persistent reviewer session for the spec and one persistent reviewer session for the plan until each passes.

## When to Use

- The conversation already ended with a final summary or design recap.
- Another engineer is blocked on written docs.
- The task is to produce a spec and a plan, not to implement code yet.
- Ambiguity, optionality, or "implementer choice" would be harmful.

Do not use this skill when the design is still unsettled. That is a brainstorming problem, not a start-summary problem.

## Required Skills

- `writing-plans` is mandatory.
- Do not invoke `brainstorming` again just because the upstream material came from a design discussion. The final recap is already the approved design input.

## Core Rules

1. The final recap is the starting truth. If earlier messages conflict with it, resolve the conflict explicitly in the spec instead of leaving it vague.
2. Write the spec first. Do not draft, outline, or derive any part of the plan until the spec reviewer returns `PASS`, the spec is frozen, and you have re-read the frozen spec fresh.
3. The spec reviewer is one fresh reviewer session reused with the same `task_id` until `PASS`.
4. Until the spec reviewer returns `PASS`, the spec is still a mutable draft. The spec reviewer must treat it as editable and not yet immutable.
5. The plan reviewer is one fresh reviewer session reused with the same `task_id` until `PASS`.
6. Every reviewer message must restate full context because reviewer memory can compact away.
7. Every review round must re-read the current spec or plan file fresh.
8. Remove all `optional` / `可选` / `可做可不做` wording. Replace it with a requirement, non-goal, or locked assumption.
9. Once the spec passes, it becomes immutable. Re-read the frozen spec yourself fresh before writing or fixing the plan.
10. Until the plan reviewer returns `PASS`, the plan is still a mutable draft.
11. Once the plan passes, it becomes immutable.
12. Final reporting must include the absolute spec path and the absolute plan path.

## Default Paths

- Respect user-provided paths first.
- Else respect repo doc conventions.
- Else use:
  - spec: `<repo>/docs/superpowers/specs/YYYY-MM-DD-<slug>-design.md`
  - plan: `<repo>/docs/superpowers/plans/YYYY-MM-DD-<slug>.md`

Always report the absolute resolved paths, not just the repo-relative paths.

## Workflow

1. Announce: `I'm using start-summary to turn the final recap into a spec and implementation plan.`
2. Invoke `writing-plans`.
3. Re-read the full conversation, with extra focus on the final summary or design recap.
4. Build a context packet before drafting:
   - workspace or repo absolute path
   - important file absolute paths that the docs must reference
   - optional grep keywords for reviewers
   - final recap highlights and focus points
   - brief note of the latest findings and what changed since them
   - resolved absolute spec path
   - resolved absolute plan path
5. Draft the spec first.
6. Spawn one fresh spec reviewer subagent.
7. Keep using the same spec reviewer `task_id` until it returns `PASS`.
8. Freeze the spec only after `PASS`. Until then it remains an editable draft.
9. Re-read the frozen spec yourself fresh. Do not rely on memory from the drafting or review loop.
10. Only now derive the plan from the immutable spec. Any `TASK_GROUP`, `TASK`, `STEP`, or tentative task outline counts as plan drafting.
11. Structure the plan as `TASK_GROUP (CHUNK) -> TASKS -> STEPS`.
12. Spawn one fresh plan reviewer subagent.
13. Keep using the same plan reviewer `task_id` until it returns `PASS`.
14. Freeze the plan only after `PASS`.
15. Report both absolute paths.

Do not stop after the spec. Do not stop after the first review round. The workflow ends only after both documents converge.

## Spec Shape

The spec should be concrete enough that a downstream engineer does not need to ask what you meant. A good default structure is:

```md
# <Feature Name> Design Spec

## Goal
## Non-Goals
## Scope
## Actors And Entry Points
## Functional Behavior
## State And Data Contracts
## Error Handling And Edge Cases
## Acceptance Criteria
## Locked Assumptions
```

Every section must prefer deterministic language over fuzzy language.

## Plan Shape

The plan must be executable by a little-context engineer. It must not leave room for discretionary implementation choices.

```md
# <Feature Name> Implementation Plan

## TASK_GROUP: <chunk>

### TASK: <single deliverable>

- STEP 1: <single action>
- STEP 2: <single action>
```

Each chunk may contain multiple tasks. Each task must have its own steps. Do not collapse chunk, task, and step into one flat list.

## Spec Reviewer Prompt Contract

Every message to the spec reviewer must fully include:

- Workspace or repo absolute path.
- Important must-read file absolute paths.
- Optional grep keywords.
- Spec absolute path.
- Explicit statement that the spec is still a mutable draft until the reviewer returns `PASS`; it is not immutable yet.
- Final summary or design recap highlights.
- A short note of previous findings and what changed since the last round.
- This exact review contract:
  1. Re-read the spec file fresh on every round. Other files do not need a fresh re-read every round.
  2. Treat the spec as a mutable draft until you return `PASS`. Do not assume it is already frozen.
  3. Find every ambiguous statement, especially `optional`, `可选`, `可做可不做`, and any wording that leaves implementation freedom.
  4. Focus on implementability, correctness, macro-level soundness, internal consistency, and absence of ambiguity. Do not nitpick style.
  5. Return all findings in one pass. Do not drip-feed one finding at a time.
  6. Return exactly `CHANGES_REQUIRED|PASS`, `SUGGEST_CHANGES`, and `RATIONALE`.

Recommended prompt skeleton:

```md
You are the spec reviewer. Treat yourself as stateless.

Workspace: <absolute path>
Must-read files: <absolute paths>
Optional grep keywords: <keywords>
Spec path: <absolute path>
Spec status: mutable draft until you return PASS
Final recap highlights: <bullets>
Previous findings and latest changes: <brief note>

Review requirements:
1. Re-read the spec file fresh on every round. Other files do not need a fresh re-read every round.
2. Treat the spec as a mutable draft until you return PASS. Do not assume it is already frozen.
3. Find every ambiguous statement, especially optional wording that leaves implementation freedom.
4. Focus on implementability, correctness, macro-level soundness, internal consistency, and absence of ambiguity. Do not nitpick style.
5. Return all findings in one pass.
6. Return exactly:
   - CHANGES_REQUIRED|PASS
   - SUGGEST_CHANGES
   - RATIONALE
```

## Plan Reviewer Prompt Contract

Every message to the plan reviewer must fully include:

- Workspace or repo absolute path.
- Important must-read file absolute paths.
- Optional grep keywords.
- Spec absolute path.
- Plan absolute path.
- Explicit statement that the spec is immutable baseline and the plan is still a mutable draft until the reviewer returns `PASS`.
- Final summary or design recap highlights.
- A short note of previous findings and what changed since the last round.
- This exact review contract:
  1. Re-read the spec and the plan files fresh on every round. Other files do not need a fresh re-read every round.
  2. Treat the spec as immutable baseline. Treat the plan as a mutable draft until you return `PASS`.
  3. Find every ambiguous statement, especially `optional`, `可选`, `可做可不做`, and any wording that leaves implementation freedom.
  4. Focus on reasonable task architecture, strict alignment with the spec, and practical executability. Do not nitpick style.
  5. Return all findings in one pass. Do not drip-feed one finding at a time.
  6. Return exactly `CHANGES_REQUIRED|PASS`, `SUGGEST_CHANGES`, and `RATIONALE`.

Recommended prompt skeleton:

```md
You are the plan reviewer. Treat yourself as stateless.

Workspace: <absolute path>
Must-read files: <absolute paths>
Optional grep keywords: <keywords>
Spec path: <absolute path>
Plan path: <absolute path>
Spec status: immutable baseline
Plan status: mutable draft until you return PASS
Final recap highlights: <bullets>
Previous findings and latest changes: <brief note>

Review requirements:
1. Re-read the spec and the plan files fresh on every round. Other files do not need a fresh re-read every round.
2. Treat the spec as immutable baseline. Treat the plan as a mutable draft until you return PASS.
3. Find every ambiguous statement, especially optional wording that leaves implementation freedom.
4. Focus on reasonable task architecture, strict alignment with the spec, and practical executability. Do not nitpick style.
5. Return all findings in one pass.
6. Return exactly:
   - CHANGES_REQUIRED|PASS
   - SUGGEST_CHANGES
   - RATIONALE
```

## Convergence Rules

- If the reviewer returns stale findings, explicitly tell it to re-read the current file fresh and ignore superseded points.
- Do not discard the reviewer and open a new session. Keep the same reviewer `task_id` for that document until convergence.
- The spec and plan reviewers must be different fresh sessions. Reusing the spec reviewer for the plan weakens separation.
- The spec-review loop must fully converge before any plan drafting or plan review begins.
- After the spec reviewer returns `PASS`, freeze the spec and re-read it fresh yourself before deriving the plan.
- The plan remains editable until the plan reviewer returns `PASS`; only then freeze it.

## No-Ambiguity Scan

Before each review round, scan for wording like:

- `optional`
- `optionally`
- `if needed`
- `as needed`
- `should`
- `could`
- `may`
- `might`
- `where appropriate`
- `nice to have`
- `etc.`
- `可选`
- `视情况`
- `尽量`
- `可以`
- `可做可不做`

For each hit, choose one of three outcomes:

1. Turn it into a hard requirement.
2. Turn it into a hard non-goal.
3. Turn it into a locked assumption.

Do not carry the ambiguous wording into the final docs.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Parallel reviewers are faster" | This workflow requires one persistent reviewer session per document so feedback converges instead of forking. |
| "Self-review is enough" | No. The contract requires a dedicated spec reviewer and a dedicated plan reviewer. |
| "I can sketch rough TASK_GROUPS while the spec review is running" | No. A tentative task outline is still plan derivation. Do not draft any part of the plan before the spec passes, freezes, and is re-read fresh. |
| "The spec can keep evolving while I write the plan" | No. The plan must be written from an immutable spec. |
| "I already know the spec from drafting it, so I can skip the post-freeze re-read" | No. Re-read the frozen spec fresh before deriving the plan so the plan matches the final immutable text, not your memory. |
| "The spec reviewer will understand that the spec is still a draft" | No. Tell the spec reviewer explicitly that the spec is mutable until `PASS`; do not rely on inference. |
| "Optional wording is fine because the implementer is smart" | Smart implementers still diverge when the spec leaves freedom. Remove the freedom. |
| "I'll open a new reviewer session after each edit" | No. Reuse the same reviewer `task_id` until `PASS` so the reviewer can converge on the latest draft. |
| "I'll only fix the major findings and ignore the rest" | The reviewer must report all findings in one pass and you must resolve them before `PASS`. |

## Red Flags

- Reopening brainstorming instead of distilling the approved recap.
- Drafting any `TASK_GROUP`, `TASK`, `STEP`, or tentative task outline before the spec passes and freezes.
- Writing the plan before the spec passes.
- Using more than one spec reviewer session.
- Using more than one plan reviewer session.
- Letting the reviewer skip a fresh re-read of the current file.
- Skipping the fresh self re-read of the frozen spec before plan drafting.
- Letting the spec reviewer assume the spec is already immutable.
- Editing the spec after declaring it immutable.
- Returning relative paths only.
- Leaving `optional` or similar wording in either final document.

## Completion

The workflow is complete only when:

1. The spec reviewer returns `PASS`.
2. The spec is frozen.
3. The frozen spec is re-read fresh before plan drafting.
4. The plan reviewer returns `PASS`.
5. The plan is frozen.
6. You report the absolute spec path and absolute plan path.
