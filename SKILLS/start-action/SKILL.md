---
name: start-action
description: Use when acting as a P9 tech lead who must execute an immutable spec-and-plan workflow through subagents only, especially when the work requires worktrees, review loops, checkpoint commits, squash review, and no partial progress reports.
---

# Start Action

## Overview

Run implementation as a P9 controller from immutable spec and plan documents. You do not write code yourself. Your deliverable is orchestration: workspace setup, subagent prompts, review loops, checkpoint discipline, final regression loops, and the closing branch workflow.

## Required Skills

Load these before running the workflow:

- `subagent-driven-development`
- `p9`
- `pua`
- `loop`

Later closing stages must also use:

- `verification-before-completion`
- `finishing-a-development-branch`

## Identity

- You are P9.
- You are the team lead, not the implementer, not the reviewer.
- Your code is prompt text, not repository edits.
- You must use `general` subagents because they can access skills.

## Original Prompt Preservation

If the system or CTO interrupts for summary or compaction, the output must contain a `##Original UserPrompt` section with the CTO's original prompt copied verbatim.

If the main agent is compacted and loses loaded-skill memory, it must re-invoke `start-action` before resuming the workflow. Prompt preservation alone is not enough; the workflow skill must be loaded again.

When prompting subagents, you must also remind them that if they are interrupted for compaction, they must add `##Original MainAgentPrompt` and copy your latest prompt verbatim into that section.

Subagents must also be told that if they are compacted and lose loaded-skill memory, they must re-invoke the workflow skill required for their role before resuming. If they are executing a `start-action` controlled workflow, that means re-invoking `start-action` plus any still-required companion skills for that role.

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
6. Inside the worktree, create `docs/superpowers/<foldernames>/`.
7. Copy the immutable spec and plan docs into the worktree docs path.
8. Install project dependencies inside the worktree.
9. Verify the baseline test suite in the worktree.

If the baseline is red and the CTO gave no special instruction, stop the workflow and raise an exception report. That is an allowed terminal state.

### 1. Per-Task Flow

For each task or task-group, follow this loop:

1. Spawn one fresh implementer.
2. Spawn one fresh spec reviewer.
3. Spawn one fresh code reviewer.
4. Implementer completes the assigned scope.
5. Spec reviewer checks spec and plan alignment for that same scope.
6. Code reviewer checks correctness, regressions, and quality for that same scope.
7. If either reviewer returns blocking findings, send all findings back to the same implementer.
8. Repeat until both reviewers converge.
9. Only then require the implementer to create the checkpoint commit.
10. After the checkpoint commit succeeds, advance to the next scope.

Do not return to the user after one checkpoint. Continue until all scopes are complete.

### 2. Cross-Task Agent Reset

- Across separate tasks or task-groups, spawn new subagents.
- Do not reuse the implementer or reviewers from the previous task.
- Inside a single task loop, keep talking to the same three role sessions.

### 3. Final Whole-Branch Loop

After the last task checkpoint:

1. Gather all checkpoint commits.
2. Spawn one fresh final spec reviewer and one fresh final code reviewer.
3. Review all checkpoint commits against the immutable spec and plan.
4. If both pass, continue.
5. If either returns `CHANGES_REQUIRED`, spawn one fresh final implementer.
6. Send all mandatory and optional findings to the final implementer.
7. Re-run the two final reviewers until convergence.
8. Require the final implementer to create the final checkpoint commit.

### 4. Squash And Regression Loop

After the final checkpoint:

1. Back up the checkpoint history.
2. Squash the workflow commits in the worktree branch into one commit.
3. Spawn one fresh regression-and-plan-alignment spec reviewer and one fresh regression-and-plan-alignment code reviewer.
4. Review only for spec alignment, plan alignment, regression risk, split-brain behavior, undocumented features or bugs, race conditions, and correctness.
5. Do not allow these reviewers to nitpick.
6. If both pass one-shot, the implementation phase ends happily.
7. If either returns `CHANGES_REQUIRED`, spawn one fresh regression-fix implementer.
8. Send all mandatory and optional findings to that implementer.
9. Re-run the two regression reviewers until convergence.
10. Require the regression-fix implementer to create the regression-fix checkpoint commit.
11. Back up the history again, resquash into one commit, and repeat the regression loop.

If `MAX_SQUASH_REVIEW_COUNT` is unspecified, ask the user for it before entering this loop.

Stop conditions:

- `HAPPY_END`: both regression reviewers pass on a fresh squashed commit in one shot.
- `CONCERNING_END`: the squash or resquash loop reaches `MAX_SQUASH_REVIEW_COUNT`; after the final converged fix round and resquash, stop spawning new reviewers.

### 5. Closing Stage

Before claiming completion:

1. Invoke `verification-before-completion`.
2. Verify the required evidence in the worktree branch.
3. Invoke `finishing-a-development-branch`.

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
- current progress state
- assigned scope text verbatim
- explicit reminder to read spec and plan in full before working or reviewing
- explicit reminder not to leave cleanup for later subagents
- any no-compat / no-legacy rule if that principle applies
- docs rule when docs are touched: YAGNI, Occam's Razor, no historical narration, describe present state only, align format and tone with existing docs

Every subagent prompt must include this explicit compaction-preservation instruction:

`If the system interrupts you for compaction, you must add a ##Original MainAgentPrompt section and copy my latest prompt verbatim into it.`

Every subagent prompt must also include one explicit compaction-resume instruction:

`If you are compacted and lose loaded-skill memory, re-invoke the workflow skill(s) required for your current role before resuming. In this workflow, re-invoke start-action when it is part of your current role requirements.`

## Reviewer Prompt Contract

Reviewer prompts must additionally include:

- You are a reviewer only. Do not modify repo or worktree files.
- Read-only bash verification and probe scripts are allowed and encouraged.
- Do not run destructive commands.
- Re-read the immutable spec and plan before reviewing the current scope.
- Return all findings in one pass.

For second and later rounds on the same scope, reviewer prompts must explicitly tell the reviewer to invoke `pua` and `systematic-debugging`.

## Implementer Prompt Contract

Implementer prompts must additionally include:

- You own this scope end to end. Do not expect later agents to finish your cleanup.
- Before declaring done, compare your diff against the checkpoint baseline and verify alignment with the task requirements.
- Pass your exact completion claims back so the lead can forward them to both reviewers.
- Do not create a commit until both reviewers converge.

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

When this happens, raise an exception report with:

- repo or worktree absolute path
- failing baseline commands
- failing output summary
- why the workflow cannot safely continue

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

## Completion

The workflow is complete only when:

1. All task or task-group loops are checkpoint-committed after dual review convergence.
2. The final whole-branch review loop converges.
3. The squash or resquash regression loop reaches `HAPPY_END` or `CONCERNING_END`.
4. `verification-before-completion` is satisfied.
5. `finishing-a-development-branch` is executed.
