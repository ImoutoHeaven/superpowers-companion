---
name: start-summary
description: Use when an approved final recap artifact must become concrete spec and implementation plan documents for downstream engineers, especially when optional or ambiguous wording would leave product behavior, task scope, or verification open to interpretation.
---

# Start Summary

## Overview

Turn the last approved summary or design recap into two on-disk documents in two locked phases: converge the spec first, freeze it, re-read the frozen spec fresh, then derive and converge the implementation plan from that immutable spec. Do not reopen brainstorming. Do not leave optional wording about product behavior, scope, contracts, sequencing, or verification. Use one persistent reviewer session for the spec and one persistent reviewer session for the plan until each passes. The frozen plan is a derived coordination document, not a second point of truth equal to the frozen spec.

## When to Use

- The conversation already ended with an approved final recap artifact or equivalent design recap.
- Another engineer is blocked on written docs.
- The task is to produce a spec and a plan, not to implement code yet.
- Ambiguity, optionality, or implementer choice about product behavior, scope, sequencing, or verification would be harmful.

Do not use this skill when the design is still unsettled. That is a brainstorming problem, not a start-summary problem.

Do not use this skill when the discussion is settled but the recap is still ad hoc, loose, or not yet approved as a durable handoff artifact. Use `start-recap` first.

## Required Skills

- `writing-plans` is mandatory.
- Role-local reviewer skills for this workflow are:
  - spec reviewer: `review-start-summary-spec`
  - plan reviewer: `review-start-summary-plan`
- In this workflow, `writing-plans` remains the canonical source of truth for plan format and required sections. `start-summary` only adds orchestration-specific delta: approved recap as primary source, spec-first convergence, truth hierarchy, and the rule that code/function/test sections must stay at behavior/API/contract/assertion/verification level instead of drifting into gratuitous full implementation detail.
- If the recap is not yet written and approved as a deterministic handoff artifact, use `start-recap` first.
- Do not invoke `brainstorming` again just because the upstream material came from a design discussion. The final recap is already the approved design input.
- Do not use `review-spec-alignment` or `review-code-quality` in this workflow. Those are `start-action` implementation-review skills, not `start-summary` draft-convergence skills.

## Core Rules

1. The approved recap artifact is the primary input and starting truth. Use earlier conversation only as secondary context for supporting file references, history of resolved changes, or confirmation of what the recap already settled. Do not reconstruct product truth from raw chat when the approved recap already resolved it.
2. Write the spec first. Do not draft, outline, or derive any part of the plan until the spec reviewer returns `PASS`, the spec is frozen, and you have re-read the frozen spec fresh.
3. The spec reviewer is one fresh reviewer session reused with the same `task_id` until `PASS`.
4. Until the spec reviewer returns `PASS`, the spec is still a mutable draft. The spec reviewer must treat it as editable and not yet immutable.
5. The plan reviewer is one fresh reviewer session reused with the same `task_id` until `PASS`.
6. Every reviewer message must restate full context because reviewer memory can compact away.
7. Every review round must re-read the current spec or plan file fresh.
8. Remove all `optional` / `可选` / `可做可不做` wording about product behavior, scope, and data contracts in the spec, and about affected files, contract changes, sequencing, and verification in the plan. Replace it with a requirement, non-goal, or locked assumption.
9. Once the spec passes, it becomes immutable. Re-read the frozen spec yourself fresh before writing or fixing the plan.
10. The frozen spec is the product and behavior truth. The referenced repo or codebase is the executability truth. The plan is a derived coordination artifact and must never become a second point of truth.
11. If the plan conflicts with the frozen spec, the plan is wrong and must be revised. Do not reinterpret the spec to save the plan.
12. If later implementation proves that a frozen plan step is actually unexecutable against the real codebase, or executable only by drifting from the frozen spec, treat that as plan drift evidence. The downstream lead must investigate and adjudicate against the frozen spec plus actual codebase evidence instead of mechanically following the broken step.
13. Plan concreteness means no ambiguity about behavior, intended implementation surface, contract changes, sequencing, or verification. Name concrete file paths when must-read repo evidence already supports them. If repo evidence still leaves multiple plausible placements, name the relevant code area or boundary instead of inventing one brittle exact route.
14. When the plan uses code, function, or test snippets to satisfy `writing-plans`, keep them at behavior, API, contract, and assertion level: inputs, outputs, public types, required call ordering, required assertions, and verification targets. Do not turn them into copy-paste full function definitions, helper internals, unrelated implementation details, or one brittle local route unless the frozen spec or must-read repo evidence makes that detail necessary for executability.
15. In plan review, the ban on full detailed copy-paste transcript detail has priority over any broad reading of `no ambiguity`. Reviewers must remove coordination ambiguity without demanding transcript-level implementation detail.
16. The spec fixes product goals, scope, behavior, and acceptance criteria. The plan fixes intended implementation surface, API and data-contract decisions, sequencing, and verification. `writing-plans` fixes the plan's required structure and section types. Local code structure and helper detail belong to implementation and later review unless the frozen spec or must-read repo evidence forces them earlier.
17. Until the plan reviewer returns `PASS`, the plan is still a mutable draft.
18. Once the plan passes, it becomes immutable.
19. Final reporting must include the absolute spec path and the absolute plan path.
20. Every `start-summary` reviewer subagent must invoke its role-local reviewer skill before acting, and must re-invoke that same role-local skill after compaction if loaded-skill memory is lost.
21. The controller's pasted reviewer contract is required round context, not the reviewer's sole authority and not a substitute for the role-local reviewer skill.
22. Do not substitute `review-spec-alignment` or `review-code-quality` here. Those skills review implemented scopes in `start-action`, not mutable draft docs in `start-summary`.

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
3. Re-read the approved recap artifact first and treat it as the primary source. Re-read earlier conversation only as secondary context for supporting references, resolved-change history, or confirmation of what the approved recap already settled.
4. Build a context packet before drafting:
   - workspace or repo absolute path
   - approved recap absolute path
   - important file absolute paths that the docs must reference
   - optional grep keywords for reviewers
   - final recap highlights and focus points
   - brief note of the latest findings and what changed since them
   - resolved absolute spec path
   - resolved absolute plan path
5. Draft the spec first.
6. Spawn one fresh spec reviewer subagent and explicitly instruct it to invoke `review-start-summary-spec` before acting.
7. Keep using the same spec reviewer `task_id` until it returns `PASS`.
8. Freeze the spec only after `PASS`. Until then it remains an editable draft.
9. Re-read the frozen spec yourself fresh. Do not rely on memory from the drafting or review loop.
10. Only now derive the plan from the immutable spec. Any `TASK_GROUP`, `TASK`, `STEP`, or tentative task outline counts as plan drafting.
11. Use `writing-plans` as the canonical source for the plan's required format and sections. `TASK_GROUP` or `TASK_CHUNK` is a mandatory orchestration layer in this workflow, but it must wrap the underlying `writing-plans` task scaffold rather than replace or redefine it.
12. Make the plan concrete about the intended implementation surface, contract changes, sequencing, and verification. Name concrete file paths when must-read repo evidence already supports them, and otherwise name the relevant code area or boundary without inventing a brittle exact route. Do not freeze local code-level details beyond what the frozen spec and must-read repo evidence actually require.
13. Spawn one fresh plan reviewer subagent and explicitly instruct it to invoke `review-start-summary-plan` before acting.
14. Keep using the same plan reviewer `task_id` until it returns `PASS`.
15. Freeze the plan only after `PASS`.
16. Report both absolute paths.

Do not stop after the spec. Do not stop after the first review round. The workflow ends only after both documents converge.

## Spec Shape

The spec should be concrete enough that a downstream engineer does not need to ask what you meant about product behavior, scope, or data contracts. A good default structure is:

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

The plan must be executable by a little-context engineer. Remove ambiguity about behavior, intended implementation surface, contract changes, sequencing, and verification. Preserve the required structure and section types from `writing-plans`; this skill does not redefine them.

The only plan-shape delta this skill adds is:

- Keep code, function, and test snippets at behavior, public API, data-contract, assertion, and verification level.
- Comments, logic-flow bullets, API skeletons, contract tables, assertion skeletons, and verification matrices are valid plan representations when they lock behavior and verification intent without overfreezing local implementation internals.
- Do not let those sections drift into gratuitous full function definitions, full test definitions, helper internals, or unrelated implementation detail unless the frozen spec or must-read repo evidence makes that detail necessary for executability.
- Name concrete file paths when actual codebase evidence already supports them; otherwise name the relevant code area or boundary without freezing a brittle exact route.
- `TASK_GROUP` or `TASK_CHUNK` is mandatory in this workflow as the orchestration layer above the underlying `writing-plans` task scaffold. Do not replace, flatten, or redefine that scaffold here.

If the repo requires a stricter downstream execution workflow than the default `writing-plans` header names, state that requirement immediately after the preserved `writing-plans` header instead of silently rewriting the header contract.

## Truth Hierarchy

- The frozen spec is the product and behavior truth.
- The referenced repo or codebase is the executability truth.
- The frozen plan is a derived coordination document. It must never become a second point of truth equal to the spec.
- If a plan step conflicts with the frozen spec, the spec wins and the plan must be fixed.
- If later execution proves a frozen plan step impossible against the real codebase, or possible only by drifting from the frozen spec, that is plan drift evidence. The downstream lead must investigate and adjudicate against the frozen spec plus actual codebase evidence.
- If the downstream investigation confirms the plan step is wrong but the work can continue safely while respecting the frozen spec and actual codebase truth, execution may continue under the adjudicated understanding and the docs can be reconverged later.
- If the downstream investigation cannot safely resolve the drift without changing the frozen spec, or cannot establish a safe executable path, stop execution and reconverge the docs from the frozen spec before continuing.

## Spec Reviewer Prompt Contract

Every message to the spec reviewer must fully include:

- Workspace or repo absolute path.
- Approved recap absolute path.
- Important must-read file absolute paths.
- Optional grep keywords.
- Spec absolute path.
- Explicit instruction to invoke `review-start-summary-spec` before acting and to re-invoke it after compaction if loaded-skill memory is lost.
- Explicit statement that the pasted contract is current-round context only; it does not replace the reviewer's role-local skill.
- Explicit statement that the spec is still a mutable draft until the reviewer returns `PASS`; it is not immutable yet.
- Final summary or design recap highlights.
- A short note of previous findings and what changed since the last round.
- This exact review contract:
  1. The pasted contract is current-round context only. It does not replace your role-local reviewer skill.
  2. Re-read the spec file fresh on every round. Re-read the approved recap artifact fresh on round 1, and again after compaction, partial handoff, or whenever you suspect recap-to-spec drift. Other files do not need a fresh re-read every round.
  3. Treat the spec as a mutable draft until you return `PASS`. Do not assume it is already frozen.
  4. If any authoritative input is missing after compaction or broken handoff, return `CHANGES_REQUIRED` and use `SUGGEST_CHANGES` to request the missing authoritative inputs instead of guessing from prior notes.
  5. Find every ambiguous statement, especially `optional`, `可选`, `可做可不做`, and any wording that leaves product behavior, scope, or data contracts open to interpretation.
  6. Focus on implementability, correctness, macro-level soundness, internal consistency, and absence of ambiguity. Do not nitpick style.
  7. Do not force plan-level sequencing, file-routing decisions, literal test content, or local code-level choices into the spec. Keep the spec focused on product behavior, scope, data contracts, and acceptance criteria.
  8. Return all findings in one pass. Do not drip-feed one finding at a time.
  9. Return exactly `CHANGES_REQUIRED|PASS`, `SUGGEST_CHANGES`, and `RATIONALE`.

Recommended prompt skeleton:

```md
You are the spec reviewer. Invoke `review-start-summary-spec` before acting. If compacted and loaded-skill memory is lost, re-invoke `review-start-summary-spec` before resuming. Treat yourself as stateless.

Workspace: <absolute path>
Approved recap path: <absolute path>
Must-read files: <absolute paths>
Optional grep keywords: <keywords>
Spec path: <absolute path>
Pasted contract status: current-round context only, not a substitute for your role-local reviewer skill
Spec status: mutable draft until you return PASS
Final recap highlights: <bullets>
Previous findings and latest changes: <brief note>

Review requirements:
1. The pasted contract is current-round context only. It does not replace your role-local reviewer skill.
2. Re-read the spec file fresh on every round. Re-read the approved recap artifact fresh on round 1, and again after compaction, partial handoff, or whenever you suspect recap-to-spec drift. Other files do not need a fresh re-read every round.
3. Treat the spec as a mutable draft until you return PASS. Do not assume it is already frozen.
4. If any authoritative input is missing after compaction or broken handoff, return CHANGES_REQUIRED and use SUGGEST_CHANGES to request the missing authoritative inputs instead of guessing from prior notes.
5. Find every ambiguous statement, especially optional wording that leaves product behavior, scope, or data contracts open to interpretation.
6. Focus on implementability, correctness, macro-level soundness, internal consistency, and absence of ambiguity. Do not nitpick style.
7. Do not force plan-level sequencing, file-routing decisions, literal test content, or local code-level choices into the spec. Keep the spec focused on product behavior, scope, data contracts, and acceptance criteria.
8. Return all findings in one pass.
9. Return exactly:
   - CHANGES_REQUIRED|PASS
   - SUGGEST_CHANGES
   - RATIONALE
```

## Plan Reviewer Prompt Contract

Every message to the plan reviewer must fully include:

- Workspace or repo absolute path.
- Absolute path to the canonical `writing-plans` skill file.
- Important must-read file absolute paths.
- Optional grep keywords.
- Spec absolute path.
- Plan absolute path.
- Explicit instruction to invoke `review-start-summary-plan` before acting and to re-invoke it after compaction if loaded-skill memory is lost.
- Explicit statement that the pasted contract is current-round context only; it does not replace the reviewer's role-local skill.
- Explicit statement that the spec is immutable baseline and the plan is still a mutable draft until the reviewer returns `PASS`.
- Final summary or design recap highlights.
- A short note of previous findings and what changed since the last round.
- This exact review contract:
  1. The pasted contract is current-round context only. It does not replace your role-local reviewer skill.
  2. Re-read the spec, the plan, and the canonical `writing-plans` skill fresh on every round.
  3. Treat the spec as immutable baseline. Treat the plan as a mutable draft until you return `PASS`.
  4. If any authoritative input is missing after compaction or broken handoff, return `CHANGES_REQUIRED` and use `SUGGEST_CHANGES` to request the missing authoritative inputs instead of guessing from prior notes.
  5. Find every ambiguous statement, especially `optional`, `可选`, `可做可不做`, and any wording that leaves the intended implementation surface, contract changes, sequencing, or verification open to interpretation.
  6. Focus on reasonable task architecture, strict alignment with the spec, practical executability, and preventing the plan from becoming a second point of truth. Do not nitpick style.
  7. Treat `writing-plans` as the canonical source of truth for required plan format and section types. If the plan omits required `writing-plans` sections or silently rewrites that scaffold, return `CHANGES_REQUIRED`.
  8. Require concrete file paths when must-read repo evidence already supports them. If the repo still leaves multiple plausible placements, require the plan to name the relevant code area or boundary instead of freezing one brittle exact route.
  9. Judge code, function, and test sections by whether they lock API and contract behavior, inputs and outputs, required assertions, and verification intent. Judge command sections against `writing-plans` exact-command expectation; do not relax command exactness here.
  10. Treat comments, logic-flow bullets, API skeletons, contract tables, assertion skeletons, and verification matrices as valid plan representations when they lock behavior, interfaces, sequencing, assertions, and verification intent.
  11. If the plan includes gratuitous full function definitions, full test definitions, helper internals, or unrelated implementation detail that the frozen spec and must-read repo evidence do not require, return `CHANGES_REQUIRED`.
  12. Do not require the plan to freeze exact helper names, function bodies, helper internals, one exact file route, or transcript-level local wiring unless the frozen spec or must-read repo evidence makes that detail necessary for executability.
  13. The ban on full detailed copy-paste transcript plans has priority over any broad reading of `no ambiguity`. Remove coordination ambiguity, but preserve valid local implementation latitude.
  14. If a plan step depends on repo assumptions that are not supported by the must-read files, or would require spec drift if implemented literally, return `CHANGES_REQUIRED`; do not reinterpret the spec to save the plan. At `start-summary` stage this is a blocking draft defect that must be fixed before `PASS`; if the same defect survives into a frozen plan later, it becomes plan drift evidence for downstream adjudication.
  15. Return all findings in one pass. Do not drip-feed one finding at a time.
  16. Return exactly `CHANGES_REQUIRED|PASS`, `SUGGEST_CHANGES`, and `RATIONALE`.

Recommended prompt skeleton:

```md
You are the plan reviewer. Invoke `review-start-summary-plan` before acting. If compacted and loaded-skill memory is lost, re-invoke `review-start-summary-plan` before resuming. Treat yourself as stateless.

Workspace: <absolute path>
Canonical writing-plans path: <absolute path>
Must-read files: <absolute paths>
Optional grep keywords: <keywords>
Spec path: <absolute path>
Plan path: <absolute path>
Pasted contract status: current-round context only, not a substitute for your role-local reviewer skill
Spec status: immutable baseline
Plan status: mutable draft until you return PASS
Final recap highlights: <bullets>
Previous findings and latest changes: <brief note>

Review requirements:
1. The pasted contract is current-round context only. It does not replace your role-local reviewer skill.
2. Re-read the spec, the plan, and the canonical writing-plans skill fresh on every round.
3. Treat the spec as immutable baseline. Treat the plan as a mutable draft until you return PASS.
4. If any authoritative input is missing after compaction or broken handoff, return CHANGES_REQUIRED and use SUGGEST_CHANGES to request the missing authoritative inputs instead of guessing from prior notes.
5. Find every ambiguous statement, especially optional wording that leaves the intended implementation surface, contract changes, sequencing, or verification open to interpretation.
6. Focus on reasonable task architecture, strict alignment with the spec, practical executability, and preventing the plan from becoming a second point of truth. Do not nitpick style.
7. Treat writing-plans as the canonical source of truth for required plan format and section types. If the plan omits required writing-plans sections or silently rewrites that scaffold, return CHANGES_REQUIRED.
8. Require concrete file paths when must-read repo evidence already supports them. If the repo still leaves multiple plausible placements, require the plan to name the relevant code area or boundary instead of freezing one brittle exact route.
9. Judge code, function, and test sections by whether they lock API and contract behavior, inputs and outputs, required assertions, and verification intent. Judge command sections against writing-plans exact-command expectation; do not relax command exactness here.
10. Treat comments, logic-flow bullets, API skeletons, contract tables, assertion skeletons, and verification matrices as valid plan representations when they lock behavior, interfaces, sequencing, assertions, and verification intent.
11. If the plan includes gratuitous full function definitions, full test definitions, helper internals, or unrelated implementation detail that the frozen spec and must-read repo evidence do not require, return CHANGES_REQUIRED.
12. Do not require the plan to freeze exact helper names, function bodies, helper internals, one exact file route, or transcript-level local wiring unless the frozen spec or must-read repo evidence makes that detail necessary for executability.
13. The ban on full detailed copy-paste transcript plans has priority over any broad reading of no ambiguity. Remove coordination ambiguity, but preserve valid local implementation latitude.
14. If a plan step depends on repo assumptions that are not supported by the must-read files, or would require spec drift if implemented literally, return CHANGES_REQUIRED; do not reinterpret the spec to save the plan. At start-summary stage this is a blocking draft defect that must be fixed before PASS; if the same defect survives into a frozen plan later, it becomes plan drift evidence for downstream adjudication.
15. Return all findings in one pass.
16. Return exactly:
   - CHANGES_REQUIRED|PASS
   - SUGGEST_CHANGES
   - RATIONALE
```

## Convergence Rules

- If the reviewer returns stale findings, explicitly tell it to re-read the current file fresh and ignore superseded points.
- If the reviewer returns `CHANGES_REQUIRED` because authoritative inputs were missing after compaction or broken handoff, resend the missing authoritative inputs to that same reviewer session and continue the same document loop.
- Do not discard the reviewer and open a new session. Keep the same reviewer `task_id` for that document until convergence.
- The spec and plan reviewers must be different fresh sessions. Reusing the spec reviewer for the plan weakens separation.
- The spec-review loop must fully converge before any plan drafting or plan review begins.
- After the spec reviewer returns `PASS`, freeze the spec and re-read it fresh yourself before deriving the plan.
- The plan remains editable until the plan reviewer returns `PASS`; only then freeze it.
- If later execution proves a frozen plan step unexecutable or spec-drifting, treat that as plan drift evidence for downstream adjudication. Re-converge from the frozen spec only if that downstream investigation cannot establish a safe executable path that still respects the frozen spec and actual codebase truth.

## No-Ambiguity Scan

Before each review round, scan for wording like this in statements about product behavior, scope, or data contracts in the spec, and about the intended implementation surface, contract changes, sequencing, or verification in the plan:

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
| "The controller prompt already encodes the review procedure, so a reviewer skill is redundant" | No. The prompt carries current-round context. The reviewer must still invoke its role-local skill so the discipline survives compaction, partial handoff, and controller prompt drift. |
| "Because the controller restated everything after compaction, I can treat that pasted contract as the reviewer's live authority" | No. Restated context does not replace the role-local reviewer skill or the required fresh artifact review. |
| "There is no missing-context status token here, so the reviewer should guess from prior notes" | No. Return `CHANGES_REQUIRED` and use `SUGGEST_CHANGES` to request the missing authoritative inputs. |
| "If the role-local reviewer skill is unavailable in the environment, the reviewer should silently fall back to prompt-only review" | No. Surface the workflow mismatch explicitly instead of silently downgrading the review model. |
| "Optional wording is fine because the implementer is smart" | Smart implementers still diverge when behavior, scope, contracts, sequencing, or verification stay ambiguous. Remove that ambiguity. |
| "No optional wording means I should freeze exact helper names, code bodies, full test definitions, or one file route" | No. Remove coordination ambiguity without turning the plan into a code transcript. Preserve repo-local implementation latitude unless the frozen spec or repo evidence requires a specific route. |
| "No ambiguity means every plan step needs near-final code snippets" | No. Remove coordination ambiguity without collapsing the draft into transcript-level implementation detail. |
| "Comments, logic flow, or contract tables are ambiguous unless replaced with full code" | Not always. They are valid when they lock behavior, interfaces, sequencing, required assertions, and verification intent. |
| "I should keep asking for more detail until no local implementation choice remains" | No. Preserve valid local implementation latitude unless the frozen spec or must-read repo evidence forces one route. |
| "To avoid drift, I should remove code or test snippets from the plan entirely" | No. Respect `writing-plans` format. Keep snippets, but keep them at API, contract, assertion, and verification level instead of full implementation level. |
| "`writing-plans` asks for code and test snippets, so I should paste full function and test definitions" | No. Satisfy the format with snippets that lock behavior, inputs, outputs, public API, required assertions, and verification intent. Helpers and unrelated implementation detail belong to execution and review unless earlier truth forces them. |
| "A plan without inline code or test sections can still pass because we only care about contract level" | No. Required `writing-plans` sections must still be present. The override is semantic level, not section removal. |
| "`review-spec-alignment` or `review-code-quality` are close enough for `start-summary`" | No. Those are `start-action` implementation-review skills, not mutable draft convergence skills. |
| "I'll open a new reviewer session after each edit" | No. Reuse the same reviewer `task_id` until `PASS` so the reviewer can converge on the latest draft. |
| "I'll only fix the major findings and ignore the rest" | The reviewer must report all findings in one pass and you must resolve them before `PASS`. |
| "Once the plan passes, it becomes co-equal truth with the spec" | No. The plan is a derived coordination document. The frozen spec remains the product truth. |
| "If later codebase reality disproves a frozen plan step, implementation should still follow the plan" | No. Treat that as plan drift evidence. The downstream lead must adjudicate against the frozen spec and actual codebase truth. |

## Red Flags

- Reopening brainstorming instead of distilling the approved recap.
- Drafting any `TASK_GROUP`, `TASK`, `STEP`, or tentative task outline before the spec passes and freezes.
- Writing the plan before the spec passes.
- Using more than one spec reviewer session.
- Using more than one plan reviewer session.
- Letting the reviewer skip a fresh re-read of the current file.
- Skipping the fresh self re-read of the frozen spec before plan drafting.
- Letting the spec reviewer assume the spec is already immutable.
- Treating the controller's pasted review contract as the reviewer's only governing procedure.
- Reusing `review-spec-alignment` or `review-code-quality` inside `start-summary`.
- Inheriting a prior reviewer's "close to PASS" posture after compaction or partial handoff instead of re-establishing the evidence chain.
- Letting a reviewer silently degrade to prompt-only review because role-local skill invocation failed.
- Treating valid implementation latitude as unresolved review ambiguity.
- Editing the spec after declaring it immutable.
- Returning relative paths only.
- Leaving `optional` or similar wording in either final document.
- Removing code, function, or test sections that `writing-plans` requires instead of constraining their semantic level.
- Rejecting a spec-aligned plan only because it does not include literal code snippets, full test bodies, or frozen helper names.
- Converging the plan into copy-paste full function definitions, helper internals, or full test definitions that the frozen spec and repo evidence do not require.
- Repeating "still ambiguous" findings that disappear only when the plan is forced into transcript-level detail.
- Freezing local code-level choices that the frozen spec does not require and the must-read repo evidence does not force.
- Treating the frozen plan as a second point of truth equal to the frozen spec.
- Leaving no downstream rule for investigation and adjudication when real codebase analysis disproves a frozen plan step.

## Completion

The workflow is complete only when:

1. The spec reviewer returns `PASS`.
2. The spec is frozen.
3. The frozen spec is re-read fresh before plan drafting.
4. The plan reviewer returns `PASS`.
5. The plan is frozen.
6. You report the absolute spec path and absolute plan path.
