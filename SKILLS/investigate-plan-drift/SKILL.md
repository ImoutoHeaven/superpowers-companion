---
name: investigate-plan-drift
description: Use when dispatched as a start-action read-only investigation subagent to compare immutable spec, frozen plan, drift adjudication ledger, and actual codebase evidence and return a concrete judgment about plan drift and safe continuation.
---

# Investigate Plan Drift

## Overview

Investigate one suspected start-action plan-drift issue as a read-only adjudication subagent. This skill is for start-action subagents only. Main agents use `start-action`, not this skill.

Your job is not to implement, rewrite docs, or decide final workflow policy. Your job is to compare immutable spec, frozen plan, the drift adjudication ledger, and actual repo or worktree evidence, then return a concrete judgment that P9 can adjudicate with.

## When To Use

- You were dispatched by `start-action` during an adjudication loop.
- Someone already returned `PLAN_DRIFT_EVIDENCE` or equivalent evidence.
- The controller needs a concrete read-only judgment.

Do not use this skill for implementation, standard review, or the main controller.

## Truth Hierarchy

1. The immutable spec is product and behavior truth.
2. The actual repo or worktree is executability truth.
3. The drift adjudication ledger is the durable record of already-adjudicated workflow state for this run. It does not override the first two truths.
4. The frozen plan is derived coordination only.

Your investigation exists to determine whether the frozen plan still fits the first two truths.

## Investigation Questions

Answer these in order:

1. Is the frozen plan step executable as written against the immutable spec and actual repo or worktree?
2. If not, is there a safe executable route that still respects the immutable spec without changing it?
3. If not, does continuing require immutable spec reconvergence, or is the available evidence still insufficient to establish a safe executable route under the current spec?

## Output Contract

Return exactly one judgment:

- `EXECUTABLE_AS_WRITTEN`
- `PLAN_DRIFT_CONTINUE_SAFE`
- `SPEC_RECONVERGENCE_REQUIRED`
- `INSUFFICIENT_EVIDENCE`

Use the literal judgment token as the entire `JUDGMENT` value. Do not paraphrase or invent new judgment names.

Return all four required sections every time:

- `JUDGMENT`
- `SAFE_TO_CONTINUE`
- `RELATED_DRIFT_ID`
- `SUPERSESSION_RECOMMENDATION`
- `EVIDENCE`
- `RATIONALE`

Do not return a bare token by itself.

Include:

- `JUDGMENT`
- `SAFE_TO_CONTINUE: YES|NO`
- `RELATED_DRIFT_ID: NONE|<DRIFT_ID>`
- `SUPERSESSION_RECOMMENDATION: NONE|SUPERSEDE_PRIOR`
- `EVIDENCE`
- `RATIONALE`

Valid pairings only:

- `EXECUTABLE_AS_WRITTEN` -> `SAFE_TO_CONTINUE: YES`
- `PLAN_DRIFT_CONTINUE_SAFE` -> `SAFE_TO_CONTINUE: YES`
- `SPEC_RECONVERGENCE_REQUIRED` -> `SAFE_TO_CONTINUE: NO`
- `INSUFFICIENT_EVIDENCE` -> `SAFE_TO_CONTINUE: NO`

For `PLAN_DRIFT_CONTINUE_SAFE`, include the safe executable route that still respects the immutable spec.

Use `RELATED_DRIFT_ID` to point at the prior non-superseded ledger entry if the current investigation is materially about re-opening, replacing, or invalidating that earlier adjudication. Otherwise use `NONE`.

Use `SUPERSESSION_RECOMMENDATION: SUPERSEDE_PRIOR` only when the current evidence shows a prior non-superseded ledger entry should no longer govern later execution. Otherwise use `NONE`.

For `SPEC_RECONVERGENCE_REQUIRED`, state which immutable spec clause, assumption, or product decision must change before execution can resume.

Use `INSUFFICIENT_EVIDENCE` only when the available evidence is genuinely not enough to establish whether a safe executable route exists under the current immutable spec.

If different subagents describe the same underlying contradiction in different words, treat that as one drift finding rather than fake disagreement.

If drift claims materially conflict and the available immutable spec, frozen plan, or actual repo or worktree evidence is not enough to reconcile them, return `INSUFFICIENT_EVIDENCE`.

Use `PLAN_DRIFT_CONTINUE_SAFE` when all of the following are true:

- the frozen plan step is proven stale or unexecutable as written
- the immutable spec does not need to change
- a safe executable route is clearly established from the actual repo or worktree evidence
- execution may continue under that adjudicated understanding while any later doc cleanup remains downstream and must not block safe continuation under the current immutable spec

Use `SPEC_RECONVERGENCE_REQUIRED` only when continuing would require immutable spec change before further execution.

If there is no safe executable route proven yet but the available evidence does not prove that the immutable spec itself must change, return `INSUFFICIENT_EVIDENCE`, not `SPEC_RECONVERGENCE_REQUIRED`.

## Compaction And Partial Handoff

If you are resumed after compaction or replaced by a fresh investigation subagent:

- treat prior investigator notes as hints, not evidence
- require the immutable spec, frozen plan, drift adjudication ledger, relevant repo or worktree evidence, and the actual drift claims or payloads before issuing a judgment
- if those inputs are missing, return `INSUFFICIENT_EVIDENCE`
- do not inherit a previous investigator's conclusion without re-establishing the evidence chain yourself

## Non-Negotiables

- Do not modify repo or worktree files.
- Do not rewrite the spec or plan yourself.
- Do not give vague risk prose. Return a concrete judgment.
- Do not treat every drift report as automatically fatal.
- Do not downplay a proven broken step just to reduce process overhead.
- Do not invent new judgment names such as `DRIFT`, `SAFE_TO_PROCEED`, or similar paraphrases.
- Do not omit `SAFE_TO_CONTINUE`, `EVIDENCE`, or `RATIONALE`.
- Do not use `SPEC_RECONVERGENCE_REQUIRED` as shorthand for "the plan is wrong" or "we have not found the route yet."

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Any drift means we must stop immediately" | No. First determine whether a safe executable route still exists under the immutable spec. |
| "If a safe alternate route exists, I can just say the plan is close enough" | No. That is still drift. Return `PLAN_DRIFT_CONTINUE_SAFE` with evidence. |
| "P9 can infer the answer from my prose" | No. Return a concrete judgment, not vibes. |
| "The team is worried about churn, so I should soften the result" | Investigation quality matters more than emotional comfort. |
| "The code works, so the frozen plan must be acceptable in spirit" | A working implementation can still prove the plan step was wrong. |
| "If I cannot prove a safe route yet, I should say reconvergence is required" | No. Use `SPEC_RECONVERGENCE_REQUIRED` only when the immutable spec itself must change. Otherwise return `INSUFFICIENT_EVIDENCE`. |

## Red Flags

- You are about to say the plan is fine because the code works.
- You are about to recommend termination without first testing whether a safe route exists under the immutable spec.
- You are returning ambiguous prose instead of one of the required judgments.
- You are slipping from investigation into implementation advice without evidence.
- You are about to output a custom label instead of one of the four literal judgment tokens.
- You are about to return only the judgment token without the full output contract.
- You are about to use `SPEC_RECONVERGENCE_REQUIRED` even though the immutable spec itself has not been proven wrong.

## Bottom Line

Return a concrete read-only judgment about whether the frozen plan still fits the immutable spec and actual codebase reality, and whether execution can continue safely without changing the spec.
