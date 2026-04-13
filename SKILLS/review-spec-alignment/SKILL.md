---
name: review-spec-alignment
description: Use when dispatched as a start-action spec reviewer subagent to review one implemented scope against the immutable spec first, the drift adjudication ledger second, and to detect when the frozen plan has become a stale or conflicting derived artifact.
---

# Review Spec Alignment

## Overview

Review one implemented start-action scope as the spec reviewer subagent. This skill is for start-action subagents only. Main agents use `start-action`, not this skill.

Your primary question is whether the implementation matches the immutable spec. Your secondary question is whether the frozen plan still functions as a valid derived execution guide after accounting for any relevant non-superseded drift adjudication ledger entries. If the implementation is spec-correct but the frozen plan step is stale, unexecutable, or spec-drifting, return `PLAN_DRIFT_EVIDENCE` instead of forcing the implementation back toward a broken step.

## When To Use

- You were dispatched by `start-action` as the spec reviewer for one scope.
- The implementer already completed the scope and handed back claims plus verification.
- You have the immutable spec path, frozen plan path, and drift adjudication ledger path for the scope.
- You must review behavior and scope alignment without modifying files.

Do not use this skill for implementation, code-quality review, investigation, or the main controller.

## Truth Hierarchy

1. The immutable spec is product and behavior truth.
2. The actual repo or worktree is executability truth.
3. The drift adjudication ledger is the durable record of already-adjudicated workflow state for this run. It does not override the first two truths.
4. The frozen plan is derived coordination only.

Review the implementation against the immutable spec first.

Then decide whether the frozen plan remains a valid guide after accounting for any relevant non-superseded drift-ledger entries. If it does not, return `PLAN_DRIFT_EVIDENCE` with evidence.

## Review Order

1. Re-read the immutable spec for the current scope.
2. Re-read the frozen plan for the current scope.
3. Re-read the drift adjudication ledger for the current scope.
4. Re-read the implementation diff and any must-read repo files.
5. Decide which case you are in:
   - Required review artifacts are missing or only partially handed off: `NEEDS_CONTEXT`.
   - The implementation is spec-correct and the plan remains a valid derived guide after accounting for the ledger: `PASS`.
   - The implementation misses spec requirements, adds forbidden behavior, or is internally inconsistent: `CHANGES_REQUIRED`.
   - The implementation is spec-correct but the plan step is stale, unexecutable, or would require spec drift if enforced literally, and no relevant non-superseded ledger entry already adjudicates that mismatch: `PLAN_DRIFT_EVIDENCE`.
   - A relevant non-superseded ledger entry exists but materially new evidence now disproves it: `PLAN_DRIFT_EVIDENCE`.
6. Return all findings in one pass.

If both ordinary review findings and plan drift evidence exist, return `PLAN_DRIFT_EVIDENCE` and include every blocking finding in the same review so P9 can adjudicate with full context.

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
- the spec requirement that still governs
- the repo or worktree evidence proving the plan mismatch
- whether the implementation is spec-correct despite the broken step
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
- do not upgrade a hunch like "plan probably wrong" into `PLAN_DRIFT_EVIDENCE` without your own evidence

## Non-Negotiables

- Do not modify repo or worktree files.
- Do not nitpick style or code quality. That is not this role.
- Do not force a spec-correct implementation back toward a broken plan step.
- Do not treat the frozen plan as co-equal truth with the immutable spec.
- Do not pass a scope quietly when you have proven the plan is stale.
- Do not re-raise an already-adjudicated drift mismatch when a relevant non-superseded ledger entry already governs it and no materially new evidence disproves it.
- Do not edit the drift adjudication ledger. Only P9/controller may mutate that file.
- Do not issue any verdict from a partial handoff that lacks the required review artifacts.

## Second And Later Rounds

If this is the second or later round on the same scope, invoke `pua` and `systematic-debugging` before acting, then continue following this skill.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "The implementation matches the spec, so I can just PASS" | Not if you also proved the frozen plan step is wrong. Return `PLAN_DRIFT_EVIDENCE` so the controller can adjudicate and prevent split-brain. |
| "The plan mismatch is only documentation churn" | Once proven, it is not cosmetic. It is workflow-relevant drift. |
| "I should force the code back to the literal plan to preserve discipline" | No. Do not make spec-correct code worse to preserve a broken step. |
| "Code quality looks fine, so I can ignore the plan mismatch" | This role is not code-quality-only. Your job includes detecting broken plan guidance after spec review. |
| "The lead wants fast convergence, so I should avoid raising drift" | Fast convergence does not justify hidden contradiction between spec, plan, and implementation. |
| "A previous adjudication was mentioned in prompt text, so I can trust that without the ledger" | No. The durable workflow-state artifact is the drift ledger. If it is missing, return `NEEDS_CONTEXT`. |

## Red Flags

- You are about to pass a scope while noting privately that the frozen plan step is stale.
- You are asking the implementer to modify code only to match a broken file target.
- You are drifting into style or readability nitpicks.
- You are reviewing the plan before deciding whether the implementation satisfies the immutable spec.
- You are about to re-raise the same drift mismatch even though a relevant non-superseded ledger entry already adjudicated it and no new evidence disproves it.

## Bottom Line

Spec review is spec-first. If the implementation is spec-correct but the frozen plan no longer matches executable reality, return `PLAN_DRIFT_EVIDENCE` instead of hiding the contradiction.
