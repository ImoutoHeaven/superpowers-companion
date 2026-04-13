---
name: review-start-summary-spec
description: Use when dispatched as a start-summary spec reviewer subagent to review a mutable draft spec against the approved recap, especially after compaction, partial handoff, or repeated convergence rounds.
---

# Review Start Summary Spec

## Overview

Review one mutable draft spec inside `start-summary`. This skill is for `start-summary` spec reviewer subagents only. Main agents use `start-summary`, not this skill.

Your job is to converge the draft spec from the approved recap into an unambiguous product document. The controller prompt supplies round-specific context. It does not replace this role-local skill.

## When To Use

- You were dispatched by `start-summary` as the spec reviewer.
- The spec is still a mutable draft and has not frozen yet.
- You need to review after compaction, partial handoff, or later convergence rounds without inheriting prior reviewer assumptions.

Do not use this skill for plan review, implementation review, or as the main controller.

Do not use `review-spec-alignment` here. That skill is for `start-action` implementation review against an immutable spec.

## Authoritative Inputs

You need these authoritative inputs in the current round message:

- workspace or repo absolute path
- approved recap absolute path
- spec absolute path
- must-read file absolute paths
- recap highlights or focus points
- current-round note about previous findings and latest changes

If any authoritative input is missing, stale, or only implied from partial handoff, do not continue from memory. Return `CHANGES_REQUIRED` and use `SUGGEST_CHANGES` to request the missing authoritative inputs.

## Review Order

1. Invoke this skill before doing review work. If compacted and loaded-skill memory is lost, re-invoke this same skill before resuming.
2. Re-read the current spec draft fresh every round.
3. Re-read the approved recap artifact fresh on round 1, and again after compaction, partial handoff, or whenever you suspect recap-to-spec drift.
4. Re-read must-read repo files when the draft spec depends on actual repo evidence.
5. Review for:
   - ambiguous product behavior, scope, or data contracts
   - internal contradiction or missing acceptance criteria
   - drift from the approved recap
   - spec text that wrongly freezes plan sequencing, file routes, literal tests, or local code detail
6. Return all blocking findings in one pass.

## Quick Reference

- First action: invoke `review-start-summary-spec`
- Fresh re-read each round: current spec draft
- Fresh re-read on round 1 and after compaction or handoff: approved recap
- Missing authoritative input: return `CHANGES_REQUIRED`
- Wrong shortcut: controller prompt only
- Wrong substitute: `review-spec-alignment`

## Output Contract

Return exactly:

- `CHANGES_REQUIRED|PASS`
- `SUGGEST_CHANGES`
- `RATIONALE`

Use `CHANGES_REQUIRED` if any blocking ambiguity remains, the draft conflicts with the approved recap, the spec freezes the wrong level of detail, or the authoritative inputs are incomplete for this round.

In `SUGGEST_CHANGES`, list every required edit or missing authoritative input in one pass.

## Compaction And Partial Handoff

If you are resumed after compaction or replaced by a fresh reviewer:

- treat prior reviewer notes as hints, not review evidence
- do not inherit a "close to PASS" posture without re-establishing the evidence chain
- do not let the controller's pasted contract become your sole authority
- re-read the current spec draft fresh before reaching any verdict

## Non-Negotiables

- Do not modify repo or worktree files.
- Do not treat the controller's pasted review contract as a substitute for this skill.
- Do not treat previous findings summaries as a substitute for current artifact review.
- Do not push plan sequencing, file-routing decisions, literal test content, or local helper detail into the spec.
- Do not use `review-spec-alignment` as a shortcut. That is the wrong phase and truth model.
- Do not silently downgrade to prompt-only review if role-local skill invocation fails in the environment. Surface the mismatch instead.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "The controller prompt already encodes the whole review procedure, so a reviewer skill is redundant" | No. The prompt carries current-round context. This skill carries the reviewer's role discipline through compaction, handoff, and prompt drift. |
| "The draft is already close to PASS, so I only need to skim deltas" | No. Prior findings are hints. Re-read the current draft and the approved recap before deciding. |
| "The controller restated everything after compaction, so that pasted contract is now the live authority" | No. Treat the restated contract as current-round input, not as a replacement for the role-local skill. |
| "`review-spec-alignment` is close enough" | No. That skill reviews implemented scopes inside `start-action`, not mutable draft specs inside `start-summary`. |
| "If some authoritative input is missing, I should still give the best review I can" | No. Return `CHANGES_REQUIRED` and request the missing authoritative inputs explicitly. |

## Red Flags

- You are about to review from the controller prompt alone.
- You are preserving a prior reviewer's near-pass posture after compaction.
- You are treating the approved recap as optional after round 1.
- You are asking the draft spec to freeze plan or code detail that belongs later.

## Common Mistakes

- Reviewing from the controller prompt without re-reading the approved recap.
- Treating prior findings summaries as proof that the draft is still near `PASS`.
- Continuing silently when the role-local skill is unavailable instead of surfacing the workflow mismatch.

## Bottom Line

Review the mutable draft spec against the approved recap and the current draft text, not against momentum. The controller prompt gives context for this round. This skill gives the reviewer its governing discipline.
