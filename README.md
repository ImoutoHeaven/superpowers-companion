# superpower-companion

`superpower-companion` is a small skill library that sits on top of `pua` and `superpowers` and adds a tighter orchestration layer between them.

It is not trying to replace either upstream project. It is not a general-purpose skill platform. It is a focused companion library for teams that already like the `pua` and `superpowers` ecosystems, but want a clearer workflow from recap to spec and plan, then from frozen docs to controlled execution.

## What This Repository Is

This repository is best understood as a supplement and orchestrator.

- It does not reimplement `pua`
- It does not reimplement `superpowers`
- It does not aim to become a large catalog of unrelated skills
- It adds a small number of high-leverage workflow entry points

In practice, the split looks like this:

- `pua` provides pressure, role discipline, management posture, and execution intensity
- `superpowers` provides design, planning, debugging, review, and verification workflows
- `superpower-companion` connects those capabilities into more explicit document-driven operating paths

## Why This Exists

The upstream libraries are already strong. The gap this repository tries to close is not capability, but orchestration.

Common situations that motivated this library:

- You already have an approved recap, but you still need a clean way to turn it into a concrete spec and implementation plan
- You already have a spec and a plan, but you want a stricter P9-style controller workflow for execution
- You want to reduce drift during implementation, especially when agents tend to reopen design questions, reinterpret the plan, or report partial progress too early

This library addresses those handoff points.

## Upstream Dependencies

This repository assumes you are working in the orbit of three upstream skill sets.

### `pua`

Primary prerequisite skills come from:

- `pua`
- `p8`
- `p9`
- `loop`

Upstream: <https://github.com/tanweai/pua/>

These provide the role structure, pressure model, execution persistence, and orchestration posture that this repository builds on.

### `superpowers`

This library also depends on workflow skills from `superpowers`, especially:

- `brainstorming`
- `writing-plans`
- `systematic-debugging`

Upstream: <https://github.com/obra/superpowers>

These provide the underlying design, planning, debugging, and verification discipline.

### Recommended Frontend Companion: `impeccable`

For frontend-heavy work, this repository pairs well with:

- `impeccable`

Upstream: <https://github.com/pbakaus/impeccable>

`impeccable` is not a hard dependency here, but it is a strong addition whenever UI quality, layout, interaction, or visual polish matter.

## Included Skills

This repository is intentionally small. At the moment it contains two skills.

### `start-summary`

Use `start-summary` when the conversation already ended in an approved recap, summary, or design conclusion, and you need to turn that into durable execution documents.

It is designed for situations where:

- the design is already settled enough to stop brainstorming
- another engineer or agent is blocked on missing written docs
- ambiguity would be costly during implementation

Its job is to:

- turn the approved recap into a concrete spec
- derive an implementation plan from the frozen spec
- remove optional or ambiguous wording that would leave too much room for implementer interpretation
- converge both documents through review before handoff

In short, `start-summary` turns “we agreed on it” into “it is written down clearly enough to build.”

### `start-action`

Use `start-action` when the spec and implementation plan are already frozen and you want execution to proceed under a strict P9 controller model.

It is designed for situations where:

- you do not want the implementation phase to reopen design
- you want a lead/controller role to coordinate subagents rather than coding directly
- you want explicit review loops, checkpoint commits, squash review, and structured branch closing

Its job is to:

- enforce execution against immutable spec and plan documents
- separate implementer, spec reviewer, and code reviewer responsibilities
- keep the controller in an orchestration role instead of letting it become an ad hoc implementer
- require review convergence before advancing task scopes
- route the end of the workflow through verification and branch-finishing steps

In short, `start-action` turns “the docs are ready” into “execution follows the docs under discipline.”

## Recommended Workflow

This repository is most useful as part of a staged workflow like this:

1. If the design is still open, start with `superpowers` `brainstorming`
2. Once you have a final approved recap, run `start-summary`
3. Produce and freeze the spec
4. Produce and converge the implementation plan
5. Once both documents are stable, run `start-action`
6. Execute the work through a P9/controller-style orchestration flow

The value here is not that this repository adds lots of new skills. The value is that it makes the phase boundaries between upstream skills much more explicit.

## Non-Goals

To keep expectations realistic, this repository is not trying to do the following:

- replace `pua`, `p9`, `loop`, or other upstream execution skills
- replace the broader `superpowers` workflow system
- become a universal skill bundle for every kind of task
- force every project into one oversized process

The goal is narrower: add a few high-value orchestration skills that improve handoffs and reduce ambiguity.

## Repository Layout

The structure is intentionally minimal:

```text
SKILLS/
  start-action/
    SKILL.md
  start-summary/
    SKILL.md
```

That small footprint is deliberate. This repository should stay focused on orchestration skills rather than expanding into a broad skill collection.

## Installation

This repository uses the standard `SKILL.md` layout, so each skill directory can be copied into any agent tool that supports skill folders.

For OpenCode, for example, you can install these skills at either the project level or the global level.

Project-level install:

```bash
mkdir -p .opencode/skills
cp -r SKILLS/start-action .opencode/skills/
cp -r SKILLS/start-summary .opencode/skills/
```

Global install:

```bash
mkdir -p ~/.config/opencode/skills
cp -r SKILLS/start-action ~/.config/opencode/skills/
cp -r SKILLS/start-summary ~/.config/opencode/skills/
```

If you use another tool with a similar skill directory convention, the same approach applies: copy each skill folder, including its `SKILL.md`, into that tool's skill directory.

Before installing this repository, make sure the upstream dependencies you expect to use are already available, especially:

- the role and execution skills from `pua`
- the design, planning, debugging, and verification skills from `superpowers`

## Usage Notes

A few practical guidelines:

1. Do not use `start-action` while the design is still moving
2. Do not skip the spec-writing phase when the recap is still fuzzy; use `start-summary` first
3. Treat this repository as a companion layer, not a standalone workflow system
4. For frontend work with meaningful UI expectations, add `impeccable`

## Design Principles

This repository follows a small set of straightforward principles:

1. Stay small and focused
2. Build on upstream strengths instead of recreating them
3. Freeze spec and plan before pushing execution forward
4. Reduce ambiguity during implementation as much as possible
5. Keep orchestration, implementation, and review responsibilities distinct

If you already use `pua` and `superpowers`, and you want a clearer operating path between “approved summary” and “disciplined execution,” that is exactly what `superpower-companion` is for.
