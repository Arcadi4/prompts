---
name: identifying-slop-tests
description: Use when reviewing, writing, or refactoring tests that assert constants, headings, descriptor counts, schema shape, generated templates, examples, advisory metadata, or plan/rubric compliance instead of runtime behavior.
---

# Identifying Slop Tests

## Overview

Slop testing is testing that proves code satisfied a prompt, plan, rubric, or static design shape instead of proving behavior through a real consumer surface.

Core gate: **Would an independent engineer building this for themselves write this test because it protects real behavior?** If not, treat it as validation theater until proven otherwise.

## When to Use

Use this skill when a test asserts:

- Constants, enum members, descriptor counts, registry keys, headings, labels, descriptions, or prose.
- Prompt/golden-plan/template sections like `## QA Scenarios`, `#### Evidence`, or self-check headings.
- Example objects that parse back into themselves.
- Advisory metadata such as capabilities, recommendations, notes, reasons, or non-enforcing configuration.
- Schema fields that exist only because a plan said the design should contain them.

Do not use this skill to delete every static assertion. Exact text or exact shape is valid when it is a real product surface, public compatibility contract, protocol, migration boundary, or file format consumed by another tool.

## Core Pattern

Validation theater asks, "Did the implementation remember the planned structure?"

Behavior testing asks, "Does the system still work through the surface that depends on this?"

```ts
// Slop: proves the prompt template survived.
expect(prompt).toContain("## QA Scenarios")

// Better: proves the consumer can use the generated artifact.
const agentFile = await syncGeneratedAgent(profile)
expect(await canOpenCodeLoadAgent(agentFile)).toBe(true)
```

## Identification Gates

Ask these before keeping or writing the test:

| Gate | Slop signal | Keep only if |
| --- | --- | --- |
| Behavior | The test can pass while the feature is unusable. | It exercises a real outcome or failure path. |
| Surface | The assertion never leaves internal data shape. | The shape is consumed by a user, tool, API, file format, or runtime. |
| Change detection | Failing test would only mean wording/count changed. | Failing test signals a broken contract or behavior regression. |
| Independence | The test exists because a prompt demanded rigor. | An engineer would write it without seeing the prompt. |
| Replacement | No one can name the behavior it protects. | It can be replaced by a behavior-through-surface check. |

If code could be useless while the test still passes, the test is slop.

## Common Slop Test Types

| Type | Symptom | Replace with |
| --- | --- | --- |
| Fact-checking constants | `expect(count).toBe(18)` | Exercise the consumer that depends on all generated items. |
| Template compliance | `expect(markdown).toContain("## Evidence")` | Parse/load/use the generated document through its consumer. |
| Schema ceremony | Dummy object proves a field is required. | Boundary test for invalid real input or remove the unneeded field. |
| Example tautology | `parse(example) === example` | Behavior that registered/loaded example enables. |
| Advisory metadata | `recommendedCapabilities.reasoningLevel === "deep"` | Runtime routing/enforcement, or delete the metadata. |
| Registry bookkeeping | Object keys equal a hardcoded list. | Tool lookup/dispatch behavior for registered tools. |

## Default Policy

- Do not add tests to make a refactor look rigorous.
- Delete slop tests when their asserted structure is removed.
- Rewrite slop tests only when a real behavior would otherwise lose coverage.
- Prefer existing behavioral tests over new unit tests for deletion/consolidation refactors.
- Add a new test only for an actual runtime boundary, compatibility contract, safety invariant, or user-visible behavior.

## Better Test Targets

Good tests usually protect one of these:

- Generated files can be loaded by the tool that consumes them.
- Invalid input is rejected at a real boundary with an actionable error.
- State transitions preserve invariants and do not corrupt persisted data.
- Destructive actions are prevented or require the intended confirmation.
- Refactors preserve behavior through CLI, API, UI, file sync, or library import surfaces.
- Failure modes leave recoverable state.

## Rationalizations

| Excuse | Reality |
| --- | --- |
| "This proves the plan was followed." | Plans are not runtime behavior. |
| "It documents the expected structure." | Documentation belongs in docs unless a consumer depends on exact structure. |
| "It is cheap coverage." | Coverage that cannot fail for real bugs is noise. |
| "The field may matter later." | Future consumers do not justify current tests. |
| "The schema is important." | Test the boundary behavior, not the schema's self-description. |
| "Without this, refactor could change wording." | Wording matters only if it is a contract or user surface. |

## Red Flags

Stop and re-evaluate when you see:

- `toContain` against prompt headings, plan sections, or generated prose.
- `toEqual(exampleDescriptor)` after parsing or registering the same example.
- Hardcoded descriptor counts, role counts, tool counts, or section counts.
- Assertions over `recommended*`, `metadata`, `notes`, `capabilities`, `reason`, or `mode` fields with no enforcement path.
- Tests named after design artifacts: "golden plan contract", "self-check criteria", "scaffold integrity".
- A test whose failure message would only mean "the code no longer matches the prompt wording."

All of these mean: identify the real consumer surface, replace with a behavior test if needed, otherwise delete the test with the dead structure.
