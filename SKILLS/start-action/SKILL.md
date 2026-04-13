---
name: start-action
description: Use when acting as a P9 tech lead who must execute an immutable spec-and-plan workflow through subagents only, especially when the work requires worktrees, review loops, checkpoint commits, squash review, and no partial progress reports.
---

# Start Action

## Overview

Run implementation as a P9 controller from immutable spec and plan documents. You do not write code yourself. Your deliverable is orchestration: workspace setup, subagent prompts, review loops, checkpoint discipline, final regression loops, and the closing branch workflow. The immutable plan is a derived coordination document, not a second source of truth over the spec or the real codebase. During execution, P9 must also maintain one worktree-local drift adjudication ledger as the durable execution-stage overlay artifact for already-adjudicated plan drift.

## Required Skills

Load these before running the workflow:

- `subagent-driven-development`
- `p9`
- `pua`
- `loop`

Later closing stages must also use:

- `verification-before-completion`
- `finishing-a-development-branch`

Role-local subagent skills for this workflow are:

- implementer: `execute-implementation-scope`
- spec reviewer: `review-spec-alignment`
- code reviewer: `review-code-quality`
- adjudication investigator: `investigate-plan-drift`

## Identity

- You are P9.
- You are the team lead, not the implementer, not the reviewer.
- Your code is prompt text, not repository edits.
- You must use `general` subagents because they can access skills.
- The four role-local skills above are for subagents only. Do not invoke them as the main controller.

## Original Prompt Preservation

If the system or CTO interrupts for summary or compaction, the output must contain a `##Original UserPrompt` section with the CTO's original prompt copied verbatim.

If the main agent is compacted and loses loaded-skill memory, it must re-invoke `start-action` before resuming the workflow. Prompt preservation alone is not enough; the workflow skill must be loaded again.

When prompting subagents, you must also remind them that if they are interrupted for compaction, they must add `##Original MainAgentPrompt` and copy your latest prompt verbatim into that section.

Subagents must also be told that if they are compacted and lose loaded-skill memory, they must re-invoke only the workflow skill(s) required for their current role before resuming. Subagents in this workflow do not re-invoke `start-action`.

## Core Rules

1. Do not write code yourself.
2. Do not do partial upward reporting. Finish the whole workflow unless you reach an explicit exception state.
3. Read and care about the spec and the plan in full. These are the immutable baseline.
4. Do not let subagents reinterpret immutable spec or plan documents.
5. Every subagent prompt must contain full context because subagent memory can compact away.
6. Every reviewer prompt must explicitly forbid file modification.
7. Every reviewer may run read-only verification commands and probe scripts, but may not modify repo files.
8. If a task enters a second implementation or review round, all later rounds for that task must explicitly invoke `pua` and `systematic-debugging` in the subagent prompt.
9. No implementer commits before spec review and code review both converge for that task granularity.
10. After both reviews converge, require the implementer to create the checkpoint commit before you advance.
11. The immutable spec is the product and behavior truth. The real repo or worktree is the executability truth. The immutable plan is a derived coordination artifact, not a co-equal point of truth.
12. The drift adjudication ledger is a durable execution-stage overlay artifact for this workflow. It never overrides the immutable spec or actual repo or worktree truth.
13. The drift adjudication ledger does override literal reuse of any frozen plan step already adjudicated stale for this workflow until a later adjudication supersedes that entry.
14. If a frozen plan step conflicts with the immutable spec, the spec wins.
15. If a frozen plan step is proven unexecutable as written against the repo or worktree, or executable only by drifting from the immutable spec, that is plan drift evidence.
16. Do not make the code worse, less correct, or spec-drifting just to preserve alignment with a wrong plan step.
17. When plan drift evidence appears, P9 must pause the affected scope, dispatch fresh investigation subagents, and adjudicate against the immutable spec plus actual repo or worktree evidence.
18. If the investigation shows the plan step is wrong but the work can continue safely while respecting the immutable spec and actual repo or worktree truth, P9 must append a durable ledger entry before execution resumes.
19. If the investigation shows the immutable spec itself must change before work can safely continue, stop the workflow and raise a `SPEC_RECONVERGENCE_REQUIRED` exception report.
20. If the investigation cannot establish a safe executable path or binding execution overlay from current authoritative evidence, stop the workflow and raise an `ADJUDICATION_INSUFFICIENT_EVIDENCE` exception report.
21. Do not silently reinterpret the plan and do not keep iterating the same unsatisfiable review loop.
22. Every start-action subagent must invoke its role-local skill before acting, and must re-invoke that same role-local skill after compaction if loaded-skill memory is lost.

## Workflow Skeleton

### 0. Worktree Scaffold

Before any implementation task:

1. Check whether `<repo>/.worktrees/` exists.
2. If missing, create it.
3. Check whether `.gitignore` ignores `.worktrees/`.
4. If `.worktrees/` is not ignored, add it on the current mainline branch and create the required chore commit.
5. Create the task worktree under:
   - single repo: `<repo>/.worktrees/<branch-name>/`
   - multi repo: `<each-repo>/.worktrees/<branch-name>/`
6. Inside the worktree, create `docs/superpowers/specs/`, `docs/superpowers/plans/`, and `docs/superpowers/drift-adjudication/`.
7. Copy the immutable spec and plan docs into the corresponding worktree docs paths.
8. Resolve one drift adjudication ledger path under `docs/superpowers/drift-adjudication/`, reusing the immutable plan basename when possible.
9. Create the ledger file before any implementation or review subagent runs. Seed it with the immutable spec path, immutable plan path, and an empty `## Active Adjudications Summary` section.
10. Install project dependencies inside the worktree.
11. Verify the baseline test suite in the worktree.

If the baseline is red and the CTO gave no special instruction, stop the workflow and raise an exception report. That is an allowed terminal state.

### 1. Per-Task Flow

For each task or task-group, follow this loop:

1. Spawn one fresh implementer.
2. Spawn one fresh spec reviewer.
3. Spawn one fresh code reviewer.
4. Implementer returns one of `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, `BLOCKED`, or `PLAN_DRIFT_EVIDENCE`.
5. If the implementer returns `NEEDS_CONTEXT`, provide the missing context and continue the same scope with the same implementer session.
6. If the implementer returns `BLOCKED`, decide whether the blocker is resolvable without changing the immutable spec or adjudicating plan drift. If yes, provide the needed direction and continue the same scope. If no, stop the current scope and raise an exception report.
7. If the implementer returns `PLAN_DRIFT_EVIDENCE`, pause the affected scope and run a P9-controlled adjudication loop before deciding whether execution can continue.
8. If the implementer returns `DONE` or `DONE_WITH_CONCERNS`, send the exact completion claims and any concerns to both reviewers.
9. Spec reviewer checks immutable spec alignment first, then whether the frozen plan still functions as a valid derived guide for that same scope.
10. Code reviewer checks correctness, regressions, quality, and whether actual repo or worktree evidence proves the frozen plan step stale or unexecutable for that same scope.
11. If either reviewer returns `NEEDS_CONTEXT`, resend the missing authoritative inputs to that same reviewer session and repeat the same review round.
12. If either reviewer produces plan drift evidence, pause the affected scope and run a P9-controlled adjudication loop before deciding whether execution can continue.
13. In the adjudication loop, spawn fresh read-only investigation subagents to compare the immutable spec, the frozen plan, the drift adjudication ledger, and actual repo or worktree evidence. They may read files and run read-only verification commands, but may not modify files.
14. If the adjudication concludes `EXECUTABLE_AS_WRITTEN` or `PLAN_DRIFT_CONTINUE_SAFE`, append a ledger entry and update the active summary before resuming. Continue the scope under the effective execution contract recorded there.
15. If the adjudication concludes `SPEC_RECONVERGENCE_REQUIRED`, append the ledger entry, stop the current scope, and raise a `SPEC_RECONVERGENCE_REQUIRED` exception report.
16. If the adjudication concludes `INSUFFICIENT_EVIDENCE`, append the ledger entry, stop the current scope, and raise an `ADJUDICATION_INSUFFICIENT_EVIDENCE` exception report.
17. If either reviewer returns blocking findings that do not require adjudication or invalidation, send all findings back to the same implementer.
18. Repeat until both reviewers converge.
19. Only then require the implementer to create the checkpoint commit.
20. After the checkpoint commit succeeds, advance to the next scope.

Do not return to the user after one checkpoint. Continue until all scopes are complete.

### 2. Cross-Task Agent Reset

- Across separate tasks or task-groups, spawn new subagents.
- Do not reuse the implementer or reviewers from the previous task.
- Inside a single task loop, keep talking to the same three role sessions.

### 3. Final Whole-Branch Loop

After the last task checkpoint:

1. Gather all checkpoint commits.
2. Spawn one fresh final spec reviewer and one fresh final code reviewer.
3. Review all checkpoint commits against the immutable spec first, then against the frozen plan only as a derived coordination artifact.
4. If either reviewer produces plan drift evidence, run a P9-controlled adjudication loop before deciding whether execution can continue.
5. If the adjudication concludes `SPEC_RECONVERGENCE_REQUIRED` or `INSUFFICIENT_EVIDENCE`, append the ledger entry and stop with the matching exception report.
6. If the adjudication concludes `EXECUTABLE_AS_WRITTEN` or `PLAN_DRIFT_CONTINUE_SAFE`, continue.
7. If both pass, continue.
8. If either returns `CHANGES_REQUIRED`, spawn one fresh final implementer.
9. Send all mandatory and optional findings to the final implementer.
10. Re-run the two final reviewers until convergence.
11. Require the final implementer to create the final checkpoint commit.

### 4. Squash And Regression Loop

After the final checkpoint:

1. Back up the checkpoint history.
2. Squash the workflow commits in the worktree branch into one commit.
3. Spawn one fresh regression-and-plan-alignment spec reviewer and one fresh regression-and-plan-alignment code reviewer.
4. Review only for immutable spec alignment, whether the frozen plan still remains a valid derived guide, regression risk, split-brain behavior, undocumented features or bugs, race conditions, and correctness.
5. Do not allow these reviewers to nitpick.
6. If either reviewer produces plan drift evidence, run a P9-controlled adjudication loop before deciding whether execution can continue.
7. If the adjudication concludes `SPEC_RECONVERGENCE_REQUIRED` or `INSUFFICIENT_EVIDENCE`, append the ledger entry and stop with the matching exception report.
8. If the adjudication concludes `EXECUTABLE_AS_WRITTEN` or `PLAN_DRIFT_CONTINUE_SAFE`, continue.
9. If both pass one-shot, the implementation phase ends happily.
10. If either returns `CHANGES_REQUIRED`, spawn one fresh regression-fix implementer.
11. Send all mandatory and optional findings to that implementer.
12. Re-run the two regression reviewers until convergence.
13. Require the regression-fix implementer to create the regression-fix checkpoint commit.
14. Back up the history again, resquash into one commit, and repeat the regression loop.

If `MAX_SQUASH_REVIEW_COUNT` is unspecified, ask the user for it before entering this loop.

Stop conditions:

- `HAPPY_END`: both regression reviewers pass on a fresh squashed commit in one shot.
- `CONCERNING_END`: the squash or resquash loop reaches `MAX_SQUASH_REVIEW_COUNT`; after the final converged fix round and resquash, stop spawning new reviewers.

### 5. Closing Stage

Before claiming completion:

1. Invoke `verification-before-completion`.
2. Verify the required evidence in the worktree branch.
3. Invoke `finishing-a-development-branch`.

## Drift Adjudication Ledger

Create exactly one worktree-local ledger per `start-action` workflow under `docs/superpowers/drift-adjudication/`. Reuse the immutable plan basename when possible, for example:

- immutable plan: `/repo/.worktrees/<branch>/docs/superpowers/plans/2026-04-13-data-sync.md`
- drift ledger: `/repo/.worktrees/<branch>/docs/superpowers/drift-adjudication/2026-04-13-data-sync.md`

This ledger is the durable execution-stage overlay artifact. It records already-adjudicated plan drift and the effective execution contract later agents must follow.

Only P9/controller may create, append, supersede, close, summarize, or otherwise mutate the drift adjudication ledger. Subagents may read it, cite `DRIFT_ID`, report evidence, and request supersession, but may not edit the ledger file directly.

Ledger rules:

- Keep a short `## Active Adjudications Summary` table at the top listing `DRIFT_ID`, `STATUS`, `SCOPE`, `P9_FINAL_ADJUDICATION`, and the one-line effective execution contract.
- Append one entry per adjudication. Every entry must include:
  - `DRIFT_ID`
  - `STATUS: ACTIVE|SUPERSEDED|CLOSED`
  - `SUPERSEDES_DRIFT_ID: NONE|<DRIFT_ID>`
  - `SUPERSEDED_BY_DRIFT_ID: NONE|<DRIFT_ID>`
  - `SCOPE`
  - `TRIGGER`
  - `FROZEN_PLAN_STEP`
  - `GOVERNING_SPEC_REQUIREMENT`
  - `REPO_OR_WORKTREE_EVIDENCE`
  - `INVESTIGATOR_JUDGMENTS`
  - `P9_FINAL_ADJUDICATION`
  - `EFFECTIVE_EXECUTION_CONTRACT`
  - `SAFE_TO_CONTINUE: YES|NO`
  - `REOPEN_ONLY_IF`
- `EXECUTABLE_AS_WRITTEN` entries may be marked `CLOSED`.
- `PLAN_DRIFT_CONTINUE_SAFE` entries stay `ACTIVE` until superseded or the workflow finishes.
- `SPEC_RECONVERGENCE_REQUIRED` and `INSUFFICIENT_EVIDENCE` entries must still be written before raising the exception report.
- When P9 supersedes an existing entry, append a new entry that names the prior one in `SUPERSEDES_DRIFT_ID`, then update the prior entry's `SUPERSEDED_BY_DRIFT_ID` and `STATUS: SUPERSEDED`.
- Later subagents must treat relevant non-superseded ledger entries as binding workflow state unless they can prove materially new spec or repo evidence that requires supersession.
- Do not mutate the frozen plan to hide drift. The ledger is the overlay artifact; the frozen plan stays frozen.

## Task Granularity

The workflow depends on explicit granularity choices.

- Implementer granularity: `${Chunk|Task}`
- Reviewer granularity: `${Chunk|Task}`

If the user leaves this ambiguous, ask which granularity to use before execution.

Supported combinations:

1. Implementer `Task`, reviewer `TaskGroup`.
2. Implementer `Task`, reviewer `Task`.
3. Implementer `TaskGroup`, reviewer `TaskGroup`.

Unsupported combination:

- Implementer `TaskGroup`, reviewer `Task`.

Checkpoint commit granularity follows reviewer granularity.

## Subagent Prompt Contract

Every subagent prompt must include:

- worktree absolute path
- immutable spec absolute path
- immutable plan absolute path
- drift adjudication ledger absolute path
- current progress state
- assigned scope text verbatim
- explicit reminder to read spec, plan, and drift adjudication ledger in full before working or reviewing
- explicit reminder that the immutable spec is the product truth, the repo or worktree is the executability truth, the drift adjudication ledger is the durable execution-stage overlay artifact, and a proven-unexecutable frozen plan step must be escalated rather than mechanically followed
- explicit reminder that plan drift evidence triggers P9 investigation and adjudication, not automatic termination
- explicit reminder that relevant non-superseded drift-ledger entries are binding workflow state unless superseded by materially new evidence
- explicit reminder that only P9/controller may mutate the drift adjudication ledger; subagents may request supersession but may not edit the ledger directly
- explicit instruction to invoke the role-local skill for the assigned role before acting
- explicit reminder not to leave cleanup for later subagents
- any no-compat / no-legacy rule if that principle applies
- docs rule when docs are touched: YAGNI, Occam's Razor, no historical narration, describe present state only, align format and tone with existing docs

Every subagent prompt must include this explicit compaction-preservation instruction:

`If the system interrupts you for compaction, you must add a ##Original MainAgentPrompt section and copy my latest prompt verbatim into it.`

Every subagent prompt must also include one explicit compaction-resume instruction:

`If you are compacted and lose loaded-skill memory, re-invoke only the workflow skill(s) required for your current role before resuming. Do not re-invoke start-action.`

## Reviewer Prompt Contract

Reviewer prompts must additionally include:

- You are a reviewer only. Do not modify repo or worktree files.
- Read-only bash verification and probe scripts are allowed and encouraged.
- Do not run destructive commands.
- Re-read the immutable spec, the frozen plan, and the drift adjudication ledger before reviewing the current scope.
- Review immutable spec alignment first. Treat the frozen plan as a derived guide, not co-equal truth.
- If a relevant non-superseded drift-ledger entry already adjudicates the same frozen-plan mismatch and no materially new evidence disproves it, review against the effective execution contract recorded there instead of re-raising the same drift finding.
- If any authoritative review artifact is missing after compaction, broken handoff, or prompt loss, return `NEEDS_CONTEXT` instead of guessing from partial notes or repo wandering.
- If the drift adjudication ledger is missing after compaction, broken handoff, or prompt loss, return `NEEDS_CONTEXT` instead of silently re-litigating previously adjudicated drift.
- If the current implementation is spec-correct but a frozen plan step is proven unexecutable or spec-drifting, return plan drift evidence with evidence instead of requesting mechanical re-alignment to the broken step.
- If you believe a relevant non-superseded ledger entry is now wrong, return plan drift evidence with explicit supersession evidence instead of silently ignoring the ledger.
- Return all findings in one pass.

For second and later rounds on the same scope, reviewer prompts must explicitly tell the reviewer to invoke `pua` and `systematic-debugging`.

## Adjudication Loop

When plan drift evidence appears:

1. P9 pauses the affected scope. Do not let the implementer keep making speculative changes while the drift question is unresolved.
2. Spawn fresh read-only investigation subagents. At minimum, use:
   - one investigation subagent focused on immutable spec plus actual repo or worktree evidence
   - one investigation subagent focused on whether the frozen plan step is still executable as written
3. Investigation subagents may read files and run read-only verification commands, but may not modify repo or worktree files.
4. Ask both subagents to invoke `investigate-plan-drift` and return one concrete judgment with evidence, not vague risk language.
5. P9 adjudicates using this priority order:
   - immutable spec truth
   - actual repo or worktree executability truth
   - relevant non-superseded drift adjudication ledger entries as already-adjudicated workflow state
   - frozen plan as derived coordination
6. If the adjudication result is `EXECUTABLE_AS_WRITTEN`, append a `CLOSED` ledger entry that records the false-positive drift claim, then continue using the frozen plan as written.
7. If the adjudication result is `PLAN_DRIFT_CONTINUE_SAFE`, append an `ACTIVE` ledger entry that records the stale plan step, the governing spec requirement, and the effective execution contract, then continue under that recorded understanding.
8. If the adjudication result is `SPEC_RECONVERGENCE_REQUIRED`, append the ledger entry, then raise `SPEC_RECONVERGENCE_REQUIRED`.
9. If the adjudication result is `INSUFFICIENT_EVIDENCE`, append the ledger entry, then raise `ADJUDICATION_INSUFFICIENT_EVIDENCE`.
10. Later subagent prompts must include the ledger path and any relevant `DRIFT_ID` values. Do not rely on prompt paraphrase alone.

## Implementer Prompt Contract

Implementer prompts must additionally include:

- You own this scope end to end. Do not expect later agents to finish your cleanup.
- Read the drift adjudication ledger before touching code. If a relevant non-superseded entry already defines the safe route, implement against that effective execution contract instead of re-litigating the same stale plan step.
- Do not edit the drift adjudication ledger directly. If you think it must change, return evidence and the relevant `DRIFT_ID` for P9 to adjudicate.
- Before declaring done, compare your diff against the checkpoint baseline and verify alignment with the task requirements.
- Pass your exact completion claims back so the lead can forward them to both reviewers.
- Do not create a commit until both reviewers converge.
- If you prove a frozen plan step is unexecutable or spec-drifting, stop and report evidence for P9 adjudication. Do not silently substitute a new route and do not degrade spec compliance just to preserve literal plan alignment.
- If materially new evidence disproves a relevant non-superseded drift-ledger entry, return plan drift evidence with explicit supersession evidence. Do not silently overrule the ledger.

For second and later rounds on the same scope, implementer prompts must explicitly tell the implementer to invoke `pua` and `systematic-debugging`.

## Frontend Reminder

If a subtask is frontend-related, explicitly instruct the subagent to load relevant frontend skills before acting.

## Review Ordering

- Within the main task loop, spec reviewer and code reviewer may run in parallel for the same granularity once the implementer has finished the current scope.
- No checkpoint commit before both are converged.
- During final and regression stages, the paired spec and code reviewers also run in parallel.

## Exception State

Allowed hard stop:

- Worktree scaffold or baseline verification reveals a red baseline and the CTO gave no special override.
- P9 adjudication concludes that the immutable spec itself must change before work can safely continue.
- P9 adjudication cannot establish a safe executable path or binding execution overlay from current authoritative evidence.

When this happens, raise an exception report with:

- repo or worktree absolute path
- immutable spec absolute path
- immutable plan absolute path
- drift adjudication ledger absolute path
- current scope
- exact blocked plan step or steps
- adjudication summary
- ledger entry id or ids
- failing baseline commands
- failing output summary
- read-only evidence showing why the step is unexecutable or spec-drifting
- why the workflow cannot safely continue
- explicit statement of which terminal state applies:
  - `SPEC_RECONVERGENCE_REQUIRED`: the immutable spec must be reconverged first; any later plan or overlay must be derived again from the updated spec before execution resumes
  - `ADJUDICATION_INSUFFICIENT_EVIDENCE`: authoritative evidence is not yet enough to establish a safe executable path or binding execution overlay

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'll implement the small fix myself to save time" | No. P9 does not write code. Your artifact is prompts and orchestration. |
| "I'll report progress after the first task" | No. Partial progress reporting is explicitly forbidden. Finish the workflow or raise an exception. |
| "The spec and plan can drift if implementers discover something" | No. They are immutable references. Route issues through the review loop, not silent reinterpretation. |
| "Reviewers don't need the full prompt every time" | They can compact away memory. Full context must be resent every round. |
| "Reviewers can fix obvious issues directly" | No. Reviewers are read-only. Implementers implement. |
| "The final pass is enough; I can skip task-level reviews" | No. Task-level dual reviews are mandatory. |
| "One squash review is enough" | No. The workflow requires repeated squash or resquash review until the stop condition. |
| "The frozen plan outranks repo reality once execution starts" | No. The plan is derived coordination, not executability truth. Drift evidence must be adjudicated against spec plus actual repo or worktree evidence. |
| "If code matches the spec but not the broken plan, reviewers should force it back to the plan" | No. Return plan drift evidence. P9 adjudicates; do not make spec-correct code worse to preserve a broken plan step. |
| "Any proven wrong plan step means the workflow must terminate immediately" | No. First run the adjudication loop. Only stop if adjudication cannot establish a safe executable path that still respects the immutable spec. |
| "I can just paste the adjudication into later prompts instead of writing a ledger entry" | No. Prompt carry-forward is not a durable artifact. Persist every adjudication in the drift ledger before later rounds continue. |
| "If the investigator says spec reconvergence is required, that really means I should fix the plan" | No. `SPEC_RECONVERGENCE_REQUIRED` means the immutable spec must change first. The plan stays derived from that later spec decision. |

## Red Flags

- Reading large amounts of code or editing code yourself instead of dispatching.
- Returning progress before the whole workflow finishes.
- Forgetting to scaffold `.worktrees/` and docs copies.
- Letting baseline red tests slide.
- Allowing commits before both reviewers converge.
- Reusing subagents across different tasks.
- Letting reviewers modify files.
- Forgetting the compaction reminder or the original prompt preservation rule.
- Skipping `verification-before-completion` or `finishing-a-development-branch`.
- Forcing spec-correct code back toward a broken plan step instead of returning plan drift evidence for adjudication.
- Resuming a later round without the drift adjudication ledger or treating it as optional context.
- Jumping straight to a terminal adjudication state without running the adjudication loop.
- Keeping an unsatisfiable review loop running after adjudication already established that a frozen plan step is wrong.

## Completion

The workflow is complete only when:

1. All task or task-group loops are checkpoint-committed after dual review convergence.
2. The final whole-branch review loop converges.
3. The squash or resquash regression loop reaches `HAPPY_END` or `CONCERNING_END`.
4. `verification-before-completion` is satisfied.
5. `finishing-a-development-branch` is executed.
