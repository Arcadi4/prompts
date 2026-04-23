---
name: writing-adrs
description: Use when a design or planning discussion has produced a meaningful architecture or design decision that should be recorded, especially for small teams, solo developers, or coding-agent workflows.
license: CC-BY-NC-4.0
---

# Writing ADRs

## Overview

Write Architecture Decision Records as lightweight decision snapshots, not mini design docs.

**Core principle:** record one meaningful decision, why it was chosen, what tradeoffs came with it, and what should trigger reconsideration later.

This skill is for small agile teams, solo maintainers, and human-plus-agent workflows. It assumes there may be one decision-maker, a few advisors, and no formal architecture board.

## When to Use

Use this skill when:

- a design or planning discussion has produced a real decision
- the user asks to capture, document, or formalize a design choice
- a decision will likely be questioned later
- a choice affects structure, quality attributes, tooling, interfaces, deployment shape, or a hard-to-reverse workflow
- you need a durable record of why one option beat another

Use it **after** design discussion, not before. If the decision is still unsettled, keep designing.

Do **not** use this skill for:

- implementation notes or rollout checklists
- obvious local edits with no lasting tradeoff
- broad solution designs that contain multiple independent decisions
- meeting transcripts or brainstorming dumps

## Default Agent Behavior After Design Decisions

When an agent and user reach a meaningful decision, the agent should explicitly offer to capture it as an ADR.

Use a short prompt such as:

> "We just made a meaningful design decision. Want me to capture it as an ADR so the rationale and tradeoffs do not get lost?"

Offer an ADR when the decision is:

- important enough to revisit later
- likely to affect future implementation choices
- hard to reverse
- the result of real tradeoff discussion

Offer ADR capture after the decision is made; do not create one automatically unless the user asks or the workflow already requires it.

Do not interrupt every tiny choice with ADR ceremony. Offer it when the future reader would reasonably ask, "Why did we do it this way?"

## Scope Sizing

Right-size the ADR to the scope of the decision.

| Scope | Use when | Expected size |
| ----- | -------- | ------------- |
| Microscopic / local / minor | A focused decision inside one component, module, or workflow | 3 to 6 short bullets or paragraphs, often 0.25 to 0.5 page |
| Mid-level / cross-cutting | A decision affects several modules, one service, or a repeated team workflow | Around 1 page |
| Holistic / global / major | A decision changes system shape, platform direction, interfaces, or an expensive-to-reverse constraint | 1 to 2 pages with sharper tradeoff detail |

**Do not scale by importance theater. Scale by blast radius, reversibility, and number of downstream choices affected.**

Even major ADRs should stay concise. If you need long design detail, link to the design doc and keep the ADR focused on the decision.

## Core Pattern

Each ADR should answer these questions, in this order:

1. What decision are we making?
2. What context or forces made this decision necessary?
3. Which serious alternatives were considered?
4. Why did this option win?
5. What consequences or tradeoffs follow from it?
6. What would cause us to revisit or supersede it?

If the ADR cannot answer these questions, even briefly, the thinking is probably not done yet.

## ADR Anatomy

Keep one ADR per file and one decision per ADR.

Suggested sections:

```markdown
# ADR-XXXX: Short decision title

## Status

Proposed | Accepted | Rejected | Superseded by ADR-XXXX

## Date

YYYY-MM-DD

## Context

The situation, forces, constraints, and problem that made this decision necessary.

## Decision

The chosen option, stated plainly.

## Alternatives Considered

- Option A: why it was plausible, why it lost
- Option B: why it was plausible, why it lost

## Consequences

- benefits gained
- costs accepted
- risks introduced
- constraints created

## Revisit Triggers

- conditions that should cause review or supersession

## Related

- links to design docs, issues, PRs, or superseding ADRs
```

Optional for small-team use:

- **Decision driver** if one force dominates
- **Decision maker / advisors** when useful for future context
- **Confidence level** if uncertainty matters

Do not add sections just to look complete. Include only fields that clarify the decision.

## Writing Rules

- Put the decision near the top
- Write in plain, assertive language
- Capture rationale, not just outcome
- Name real alternatives, not strawmen
- State tradeoffs honestly
- Include concrete revisit triggers, not "review later"
- Keep accepted ADRs append-only; supersede them with a new ADR instead of rewriting history
- Store ADRs near the code or docs they govern so they are easy to find

## Small-Team Adaptation

For solo developers and very small teams:

- one person can be the decision-maker
- advisors can be other humans, prior docs, or coding agents used during design
- consensus is helpful but not required for every ADR
- review can be asynchronous and lightweight
- an ADR is successful if it preserves future understanding, not if it satisfies a governance ritual

Treat coding agents as contributors to analysis, not as the authority behind the decision. The ADR should still make clear what the human decided and why.

## Quick Reference

**Write an ADR if:**

- this will shape later implementation
- reversing it would be annoying, risky, or expensive
- multiple plausible options existed
- the tradeoff discussion matters more than the final label

**Do not write an ADR if:**

- it is just a task breakdown
- it is one tiny code tweak with no lasting design meaning
- the choice is still being explored
- the document would just repeat the design doc without a crisp decision

## Common Mistakes

- Writing a design document instead of a decision record
- Recording several decisions in one ADR
- Listing only the winner and hiding the alternatives
- Saying "we can revisit later" without naming triggers
- Treating a solo or 2-person project like it needs committee ceremony
- Mixing implementation rollout steps into the ADR
- Rewriting old ADRs instead of superseding them
- Making microscopic decisions sound global and dramatic

## Rationalization Prevention

| Excuse | Reality |
| ------ | ------- |
| "This is too small to document" | If the tradeoff will matter later, a short ADR is enough |
| "We already have a design doc" | Design docs explain the solution; ADRs preserve the decision and rationale |
| "I will remember why we chose this" | Future-you usually remembers the winner, not the losing options or constraints |
| "I will just update the old ADR later" | Accepted ADRs should be superseded, not rewritten |
| "There were no real alternatives" | If nothing else was plausible, you probably do not need an ADR |
| "I should add rollout steps so it is actionable" | Action items belong in plans or tasks, not in the ADR itself |

## Red Flags - Stop and Fix

- More than one real decision in the same ADR
- No rejected alternatives
- No tradeoffs or downsides
- No revisit trigger
- Status is unclear or missing
- The document reads like a migration plan or implementation checklist
- The scope is inflated far beyond the actual blast radius

**If any of these are true, tighten the ADR before considering it done.**

## Workflow

1. Confirm the decision is actually settled.
2. Decide whether it deserves an ADR.
3. Right-size the record to local, cross-cutting, or global scope.
4. Write one concise ADR for one decision.
5. Link supporting design material instead of copying it.
6. If the decision changes later, write a new ADR that supersedes the old one.
