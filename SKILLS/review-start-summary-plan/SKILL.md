---
name: review-start-summary-plan
description: Use when dispatched as a start-summary plan reviewer subagent to review a mutable draft implementation plan against the frozen spec and canonical writing-plans requirements, especially after compaction, partial handoff, or repeated convergence rounds.
---

# Review Start Summary Plan

## Overview

Review one mutable draft implementation plan inside `start-summary`. This skill is for `start-summary` plan reviewer subagents only. Main agents use `start-summary`, not this skill.

Your job is to converge the draft plan from the frozen spec into a concrete coordination document that still respects `writing-plans` as the canonical source of plan format. The controller prompt supplies round-specific context. It does not replace this role-local skill.

## When To Use

- You were dispatched by `start-summary` as the plan reviewer.
- The spec is already frozen and the plan is still a mutable draft.
- You need to review after compaction, partial handoff, or later convergence rounds without inheriting prior reviewer assumptions.

Do not use this skill for spec review, implementation review, or as the main controller.

Do not use `review-code-quality` or `review-spec-alignment` here. Those skills are for `start-action` implementation review after code exists.

## Authoritative Inputs

You need these authoritative inputs in the current round message:

- workspace or repo absolute path
- canonical `writing-plans` absolute path
- frozen spec absolute path
- plan absolute path
- must-read file absolute paths
- current-round note about previous findings and latest changes

If any authoritative input is missing, stale, or only implied from partial handoff, do not continue from memory. Return `CHANGES_REQUIRED` and use `SUGGEST_CHANGES` to request the missing authoritative inputs.

## Review Order

1. Invoke this skill before doing review work. If compacted and loaded-skill memory is lost, re-invoke this same skill before resuming.
2. Re-read the frozen spec fresh every round.
3. Re-read the current plan draft fresh every round.
4. Re-read the canonical `writing-plans` skill fresh every round.
5. Re-read must-read repo files when the draft plan depends on actual repo evidence.
6. Review for:
   - strict alignment with the frozen spec
   - required `writing-plans` sections and scaffold preservation
   - ambiguity around implementation surface, contract changes, sequencing, or verification
   - whether file paths are concrete when repo evidence supports them, or properly scoped to a code area when it does not
   - whether the plan overfreezes helper names, full function bodies, helper internals, or unrelated implementation detail
7. Return all blocking findings in one pass.

## Quick Reference

- First action: invoke `review-start-summary-plan`
- Fresh re-read each round: frozen spec, current plan draft, canonical `writing-plans`
- Missing authoritative input: return `CHANGES_REQUIRED`
- Wrong shortcut: controller prompt only
- Wrong substitutes: `review-spec-alignment`, `review-code-quality`

## Output Contract

Return exactly:

- `CHANGES_REQUIRED|PASS`
- `SUGGEST_CHANGES`
- `RATIONALE`

Use `CHANGES_REQUIRED` if any blocking ambiguity remains, the plan conflicts with the frozen spec, the plan silently rewrites `writing-plans`, the plan freezes the wrong level of detail, or the authoritative inputs are incomplete for this round.

In `SUGGEST_CHANGES`, list every required edit or missing authoritative input in one pass.

## Compaction And Partial Handoff

If you are resumed after compaction or replaced by a fresh reviewer:

- treat prior reviewer notes as hints, not review evidence
- do not inherit a "close to PASS" posture without re-establishing the evidence chain
- do not let the controller's pasted contract become your sole authority
- re-read the frozen spec, current plan draft, and canonical `writing-plans` before reaching any verdict

## Non-Negotiables

- Do not modify repo or worktree files.
- Do not treat the controller's pasted review contract as a substitute for this skill.
- Do not treat previous findings summaries as a substitute for current artifact review.
- Do not let the plan become a second point of truth over the frozen spec.
- Do not remove required `writing-plans` sections just because the plan stays at contract level.
- Do not force exact helper names, full function bodies, helper internals, or one brittle file route unless the frozen spec or repo evidence requires them.
- Do not use `review-code-quality` or `review-spec-alignment` as shortcuts. They belong to `start-action`, not `start-summary`.
- Do not silently downgrade to prompt-only review if role-local skill invocation fails in the environment. Surface the mismatch instead.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "The controller prompt already encodes the whole review procedure, so a reviewer skill is redundant" | No. The prompt carries current-round context. This skill carries the reviewer's role discipline through compaction, handoff, and prompt drift. |
| "Round 2 or 3 means I only need to check the deltas" | No. Prior findings are hints. Re-read the frozen spec, current plan, and canonical `writing-plans` every round. |
| "The controller restated everything after compaction, so that pasted contract is now the live authority" | No. Treat the restated contract as current-round input, not as a replacement for the role-local skill. |
| "A contract-level plan can omit some `writing-plans` sections" | No. The semantic level changes. The required scaffold stays. |
| "`review-code-quality` or `review-spec-alignment` are close enough" | No. Those skills review implemented scopes inside `start-action`, not mutable draft plans inside `start-summary`. |
| "If some authoritative input is missing, I should still give the best review I can" | No. Return `CHANGES_REQUIRED` and request the missing authoritative inputs explicitly. |

## Red Flags

- You are about to review from the controller prompt alone.
- You are preserving a prior reviewer's near-pass posture after compaction.
- You are checking only the latest deltas without re-reading the authoritative artifacts.
- You are treating the frozen plan as co-equal truth with the frozen spec.
- You are pushing the plan into a copy-paste implementation transcript.

## Common Mistakes

- Checking only the latest deltas without re-reading the frozen spec and canonical `writing-plans`.
- Treating contract-level planning as permission to drop required `writing-plans` sections.
- Continuing silently when the role-local skill is unavailable instead of surfacing the workflow mismatch.

## Bottom Line

Review the mutable draft plan against the frozen spec, the current plan text, and canonical `writing-plans`, not against momentum. The controller prompt gives context for this round. This skill gives the reviewer its governing discipline.
