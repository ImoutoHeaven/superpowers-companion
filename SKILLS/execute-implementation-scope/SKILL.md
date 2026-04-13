---
name: execute-implementation-scope
description: Use when dispatched as a start-action implementer subagent to execute an assigned scope from an immutable spec, frozen plan, and drift adjudication ledger, especially when the plan may be stale against actual codebase reality.
---

# Execute Implementation Scope

## Overview

Execute one assigned start-action scope as the implementer subagent. This skill is for start-action subagents only. Main agents use `start-action`, not this skill.

Your primary obligation is to the immutable spec. The actual repo or worktree is executability truth. The drift adjudication ledger is the durable execution-stage overlay artifact for already-adjudicated drift. The frozen plan is a derived coordination artifact that remains authoritative only while it still matches the immutable spec, the ledger, and actual codebase reality.

If a frozen plan step is proven stale, unexecutable, or spec-drifting, do not implement it literally and do not silently substitute a new route. Return `PLAN_DRIFT_EVIDENCE` so P9 can adjudicate.

## When To Use

- You were dispatched by `start-action` as the implementer for one scope.
- You have an assigned scope plus immutable spec, frozen plan, and drift adjudication ledger paths.
- You must implement the scope without reopening design.
- You may discover that the frozen plan no longer matches codebase reality.

Do not use this skill for reviewers, drift investigators, or the main controller.

## Truth Hierarchy

1. The immutable spec is product and behavior truth.
2. The actual repo or worktree is executability truth.
3. The drift adjudication ledger is the durable record of already-adjudicated workflow state for this run. It does not override the first two truths.
4. The frozen plan is derived coordination only.

If the plan conflicts with the spec, the plan is wrong.

If a relevant non-superseded drift-ledger entry already adjudicates a stale plan step, implement against that effective execution contract unless materially new evidence disproves it.

If the plan step is unexecutable against the actual repo or worktree, or executable only by drifting from the immutable spec, return `PLAN_DRIFT_EVIDENCE`.

## Workflow

1. Read the immutable spec in full.
2. Read the frozen plan in full.
3. Read the drift adjudication ledger in full, including the active summary and any relevant non-superseded entries.
4. Read the assigned scope text verbatim.
5. Inspect the current repo or worktree evidence needed for this scope.
6. Decide which case you are in:
   - The plan remains executable and spec-aligned, with no relevant ledger override: implement.
   - A relevant non-superseded ledger entry already defines the effective execution contract for this mismatch: implement against that recorded contract.
   - The plan is unclear or you lack required context: return `NEEDS_CONTEXT`.
   - A non-drift blocker prevents safe completion even though the spec, ledger, and plan are not yet proven contradictory: return `BLOCKED`.
   - The plan is proven stale, unexecutable, or spec-drifting, or materially new evidence disproves a relevant non-superseded ledger entry: return `PLAN_DRIFT_EVIDENCE`.
7. If implementing, finish the scope end to end. Do not leave cleanup for later agents.
8. Run the required verification before claiming completion.
9. Return your status with evidence.

## Compaction And Partial Handoff

If you are resumed after compaction or replaced by a fresh implementer:

- treat prior implementer notes as hints, not authoritative truth
- require the immutable spec, frozen plan, drift adjudication ledger, exact assigned scope, and worktree path before acting
- if any of those are missing, return `NEEDS_CONTEXT`
- do not reconstruct the scope from repo wandering, recent diffs, or a prior agent's intuition alone
- do not convert a previous implementer's suspicion into `PLAN_DRIFT_EVIDENCE` unless you can compare the immutable spec, frozen plan, and actual repo or worktree evidence yourself

## Output Contract

Return exactly one status:

- `DONE`
- `DONE_WITH_CONCERNS`
- `NEEDS_CONTEXT`
- `BLOCKED`
- `PLAN_DRIFT_EVIDENCE`

Use the literal status token as the entire `STATUS` value. Do not paraphrase or invent new status names.

Status meanings:

- `DONE`: scope implemented and verified with no material unresolved concerns.
- `DONE_WITH_CONCERNS`: scope implemented and verified, but you want the controller and reviewers to inspect non-blocking concerns before checkpointing.
- `NEEDS_CONTEXT`: you cannot proceed because required context, path, scope detail, or controller instruction is missing.
- `BLOCKED`: you cannot complete for a non-drift reason that the controller must resolve. If the real issue is stale or spec-drifting plan guidance, use `PLAN_DRIFT_EVIDENCE` instead.
- `PLAN_DRIFT_EVIDENCE`: the frozen plan no longer fits the immutable spec and actual repo or worktree reality.

Include:

- `STATUS`
- `COMPLETION_OR_BLOCKER_SUMMARY`
- `VERIFICATION`
- `RATIONALE`

If returning `PLAN_DRIFT_EVIDENCE`, also include:

- the exact stale or conflicting plan step text
- the affected spec requirement
- the repo or worktree evidence proving the mismatch
- the safe executable route if one is clearly visible
- the relevant `DRIFT_ID` if you believe a current non-superseded ledger entry must be superseded
- explicit statement that you did not silently substitute a route and proceed

## Non-Negotiables

- Do not reopen design.
- Do not silently reinterpret the immutable spec.
- Do not mechanically implement a broken plan step that cannot satisfy the spec.
- Do not silently substitute a new route even if it seems obviously correct.
- Do not ignore a relevant non-superseded drift-ledger entry or treat it as optional context.
- Do not edit the drift adjudication ledger. Only P9/controller may mutate that file.
- Do not treat a partial handoff note as enough context to resume implementation.
- Do not create a commit until both reviewers converge and the controller authorizes the checkpoint commit.
- Do not leave speculative cleanup for later agents.

## Second And Later Rounds

If this is the second or later round on the same scope, invoke `pua` and `systematic-debugging` before acting, then continue following this skill.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "The spec is what matters, so I'll just fix the right file and mention it later" | No. The spec still wins, but a silent route substitution creates unadjudicated plan drift. Return `PLAN_DRIFT_EVIDENCE`. |
| "The lead said to use engineering judgment and keep momentum" | Engineering judgment includes escalating stale plan steps with evidence instead of creating undocumented divergence. |
| "The behavior is obvious, so adjudication is overkill" | Obvious to you is not enough. P9 must decide whether the workflow continues under an adjudicated understanding. |
| "I'll look blocked if I raise drift" | Hiding drift is worse than looking blocked. Silent split-brain poisons later review rounds. |
| "I already spent time exploring, so I should just finish it" | Sunk cost does not authorize a silent substitution. Return the evidence. |
| "The safe route is already in my prompt, so I do not need the drift ledger" | No. Prompt carry-forward is not authoritative workflow state. Read and respect the ledger. |

## Red Flags

- You are about to edit a different file than the frozen plan names without returning `PLAN_DRIFT_EVIDENCE`.
- You are telling yourself that a brief handoff note is enough.
- You are tempted to ship a spec-correct fix while hiding that the plan was wrong.
- You are trying to make the code worse to preserve literal plan alignment.
- You are about to claim completion without fresh verification output.
- You are about to ignore or silently overrule a relevant non-superseded drift-ledger entry.

## Bottom Line

Implement against the immutable spec and actual codebase reality. If the frozen plan no longer fits those two truths, stop and return `PLAN_DRIFT_EVIDENCE` instead of improvising.
