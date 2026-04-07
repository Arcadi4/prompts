---
name: structural-css-debugging
description: Use when debugging a CSS layout bug, box-model mismatch, or DOM structure problem that keeps breaking under CSS tweaks. If you need the short version, read @structural-css-debugging-concise.md instead.
---

# Structural CSS Debugging

## Overview

Think in boxes, not classes. Most layout bugs are structural: the DOM is split wrong, a wrapper is redundant, or transient state is fighting normal flow.

**Core principle:** give the browser the correct DOM first; only then tune CSS.

## When to Use

- A CSS layout bug keeps coming back after tweaks
- The DOM structure looks suspicious or over-wrapped
- You need to decide whether to change markup or styling first
- Visual inspection shows symptoms but not the root cause

Do **not** use this skill to keep retrying visual-only CSS changes.

## Core Pattern

Before changing CSS, answer:

1. What boxes does the browser think exist here?
2. Which box boundary is causing the bug?
3. Is this boundary necessary, or is it redundant?
4. Should this content really be split into multiple elements at all?

The common failure modes are:

- an extra wrapper makes the browser create the wrong boxes
- two pieces that should flow together were split into separate elements
- transient UI state was left in normal document flow
- a utility-class pile is hiding the structural bug instead of fixing it

Prefer the smallest structural fix:

- remove a wrapper
- merge two nodes into one flow
- move overlays deeper
- let the semantic element own the interaction

If you are tempted to add `nowrap`, `inline-flex`, `grid`, or more wrappers, pause and re-check the DOM first.

## Quick Reference

1. Read the actual DOM, not just JSX.
2. Check browser rules for inline flow, wrapping, sizing, and positioning.
3. Change structure first, then simplify CSS.
4. If stuck, ask for raw HTML and computed styles.

Useful browser concepts to search when needed:

- inline formatting context
- whitespace handling
- line breaking
- flex item sizing
- inline-block behavior
- absolute positioning

## Common Mistakes

- Tweaking classes repeatedly to “try one more thing”
- Using stronger CSS to glue broken structure together
- Reaching for visual tools before checking the DOM
- Assuming the first layout theory is correct
- Focusing on the symptom instead of the box boundary that causes it

## Implementation

- Inspect raw HTML, nesting, and text-node boundaries.
- Search official docs or MDN for the underlying layout model.
- If the issue remains unclear, ask for computed `display`, `white-space`, and `position` values.

Look for a fix that changes structure before style:

- remove a wrapper
- merge siblings that should behave as one unit
- move overlays inside the element that owns the text
- move style responsibility to the element with the right semantics

## Common Questions

- Which boundary is wrong?
- Does the whole unit fail to wrap, or do two parts split apart?
- What does the raw HTML look like?
- What are the computed `display`, `white-space`, and `position` values?

## When Stuck

If the issue persists after a few structural attempts, stop and gather evidence:

1. Ask for raw HTML.
2. Ask for computed styles.
3. Ask which boundary is actually wrong.
4. Search for a similar working pattern in the codebase.

Do not keep editing blindly once the DOM picture is unclear.
