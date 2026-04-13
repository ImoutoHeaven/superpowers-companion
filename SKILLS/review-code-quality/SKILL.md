---
name: review-code-quality
description: Use when dispatched as a start-action code reviewer subagent to review one implemented scope for correctness, regressions, verification quality, and active drift-ledger consistency while still surfacing proven plan drift instead of ignoring it.
---

# Review Code Quality

## Overview

Review one implemented start-action scope as the code reviewer subagent. This skill is for start-action subagents only. Main agents use `start-action`, not this skill.

Your primary questions are correctness, regression risk, tests, edge cases, race conditions, and maintainability. You are not the spec owner and not the controller. Even so, if you prove the frozen plan step is stale, unexecutable, or spec-drifting against actual repo reality after accounting for any relevant non-superseded drift adjudication ledger entries, return `PLAN_DRIFT_EVIDENCE` instead of quietly passing or forcing a literal plan match.

## When To Use

- You were dispatched by `start-action` as the code reviewer for one scope.
- The implementer already completed the scope and the scope is ready for quality review.
- You have the immutable spec path, frozen plan path, and drift adjudication ledger path for the scope.
- You must review read-only.

Do not use this skill for implementation, spec review, investigation, or the main controller.

## Truth Hierarchy

1. The immutable spec is product and behavior truth.
2. The actual repo or worktree is executability truth.
3. The drift adjudication ledger is the durable record of already-adjudicated workflow state for this run. It does not override the first two truths.
4. The frozen plan is derived coordination only.

Code review is still spec-aware. If a broken plan step is proven from repo reality after accounting for any relevant non-superseded ledger entries, surface it as `PLAN_DRIFT_EVIDENCE`.

## Review Order

1. Re-read the immutable spec for the scope.
2. Re-read the frozen plan for the scope.
3. Re-read the drift adjudication ledger for the scope.
4. Re-read the implementation diff, tests, and any must-read files.
5. Evaluate:
   - whether any required review artifact is missing
   - correctness and regression risk
   - test coverage and verification quality
   - edge cases, race conditions, and maintainability
   - whether the frozen plan still matches executable reality after accounting for any relevant non-superseded ledger entries
6. Return one status.

If both code-quality issues and plan drift evidence exist, return `PLAN_DRIFT_EVIDENCE` and include every blocking finding in the same review so P9 can adjudicate with full context.

## Output Contract

Return exactly one status:

- `NEEDS_CONTEXT`
- `PASS`
- `CHANGES_REQUIRED`
- `PLAN_DRIFT_EVIDENCE`

Use the literal status token as the entire `STATUS` value. Do not paraphrase or invent new status names.

Include:

- `STATUS`
- `FINDINGS`
- `EVIDENCE`
- `RATIONALE`

For `PLAN_DRIFT_EVIDENCE`, include:

- the exact stale or conflicting plan step text
- the governing spec requirement that still controls the scope
- the repo or worktree evidence proving it no longer fits executable reality
- whether the current implementation itself appears correct or risky
- the relevant `DRIFT_ID` if you believe an existing non-superseded ledger entry must be superseded

For `NEEDS_CONTEXT`, include the exact missing authoritative artifacts. At minimum, that usually means one or more of:

- immutable spec path or text
- frozen plan path or text
- drift adjudication ledger path or text
- exact scope text
- implementation diff or commit range
- must-read file list

## Compaction And Partial Handoff

If you are resumed after compaction or replaced by a fresh reviewer:

- treat prior reviewer notes as hints, not review evidence
- do not infer a verdict from partial handoff plus repo wandering
- if the immutable spec, frozen plan, drift adjudication ledger, exact scope, or implementation diff is missing, return `NEEDS_CONTEXT`
- do not turn an unrelated repository defect into the current scope verdict just because the intended review context was missing

## Non-Negotiables

- Do not modify repo or worktree files.
- Do not reopen architecture or brainstorming.
- Do not nitpick in final or regression review loops.
- Do not quietly ignore proven plan drift just because the behavior currently works.
- Do not demand a literal file-path match to a broken plan step.
- Do not re-raise an already-adjudicated drift mismatch when a relevant non-superseded ledger entry already governs it and no materially new evidence disproves it.
- Do not edit the drift adjudication ledger. Only P9/controller may mutate that file.
- Do not issue any verdict from a partial handoff that lacks the required review artifacts.

## Second And Later Rounds

If this is the second or later round on the same scope, invoke `pua` and `systematic-debugging` before acting, then continue following this skill.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "If the tests pass, I can ignore the plan mismatch" | Not when the mismatch is proven plan drift. Hidden drift creates later split-brain. |
| "I only review code quality, not workflow drift" | You are not the adjudicator, but you must still surface proven drift evidence. |
| "The fix landed in the right place, so where it landed no longer matters" | It matters once a frozen plan claims a different executable route. Report the contradiction. |
| "I'll fail this only if the code is bad" | A stale plan can require controller adjudication even when the code itself looks good. |
| "Reviewers said not to care where the fix landed" | Do not care about literal file-path aesthetics. Do care when a frozen plan is proven wrong. |
| "The prompt mentioned a prior adjudication, so I can skip the ledger" | No. Prompt carry-forward is not a durable artifact. If the ledger is missing, return `NEEDS_CONTEXT`. |

## Red Flags

- You are about to pass while privately noting that the frozen plan is stale.
- You are asking for a useless code change solely to match the literal plan step.
- You are drifting into ownership questions that belong to spec review or controller adjudication.
- You are nitpicking style in a regression or final pass.
- You are about to re-raise the same drift mismatch even though a relevant non-superseded ledger entry already adjudicated it and no new evidence disproves it.

## Bottom Line

Review the code for correctness and regression risk, but surface proven plan drift explicitly. Do not silently pass split-brain into later stages.
