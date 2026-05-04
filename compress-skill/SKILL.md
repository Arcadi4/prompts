---
name: compress-skill
description: Use when skill documents are too long, bloated with repetitive prose, or the user asks to compress/shorten/tighten/trim a SKILL.md. Also use when a skill has overlapping sections, duplicate explanations, or verbose examples that inflate word count without adding behavioral content.
---

# Compress Skill

## Overview

Compression is a structured reduction — not indiscriminate cutting. Every compression must preserve behavioral fidelity: if a fresh agent reads the compressed skill, they must produce the same behavior as with the original.

**Core principle:** Compress prose, never compress rules. Signal tables, validation checks, rationalization tables, and red flags are behavioral content. Exposition, examples, commentary, and duplicate explanations are compression targets.

## Core Pattern

### The Four Compression Categories

Every compression operation belongs to one of four categories. Enumerate opportunities first, then categorize:

| Category | Signal | Operation | Example |
|---|---|---|---|
| **Merge** | Two sections say the same thing in different words | Combine into one section; keep the clearer version | Overview says "only for docs/planning/research" and When To Use repeats it verbatim |
| **Fold** | Adjacent sections share a parent concept; the smaller one is elaboration, not a distinct topic | Inline the smaller section into the larger one | "Ladder Is Recursive" (9 lines of philosophy) → inlined as 3 lines in Inference Ladder |
| **Trim** | Prose is verbose but instruction is simple; filler words surround a command | Cut filler; keep the instruction, remove the commentary | "The next thing you should do after this is to enumerate the cases" → "Enumerate cases" |
| **Prune** | Multiple checks test the same underlying property | Keep the strongest check; drop redundant ones | "Did my concept stay stable?" and "Did the concept drift?" — same test, keep one |

### Compression Workflow

1. **Enumerate opportunities.** Read the skill end-to-end. List every specific compression you see — don't categorize yet. "Merging paragraph X with Y" is an opportunity.
2. **Categorize.** Assign each opportunity to Merge, Fold, Trim, or Prune. If an opportunity doesn't fit cleanly, reconsider — you might be about to cut behavioral content.
3. **Sort by safety.** Trims are safest (cut commentary). Merges are safe (keep behavioral). Folds need care (preserve both messages). Prunes risk data loss. Do trims and merges first; re-read after folds and prunes.
4. **Apply in order.** Execute trims → merges → folds → prunes. This avoids compressing something that later operations would eliminate anyway.
5. **Validate.** Run the acid test on the result.

### The Acid Test

For every compression, ask: **"If I gave this compressed skill to a fresh agent, would they produce the same behavior as with the original?"**

- No → revert the compression
- Yes, but only because surrounding context compensates → the skill is fragile; keep the original
- Yes, the instruction is intact → proceed

### Safety Rules

**Never compress:**

| Content Type | Reason |
|---|---|
| Signal tables (Closure Signals, Openness Signals, Confidence Gates) | These encode behavioral rules in compact form. Removing rows removes rules. |
| Rationalization Prevention tables | Each row counters a specific rationalization. Remove one, the loophole reopens. |
| Validation checklists | Multiple checks test different failure modes. Pruning is possible only when checks genuinely overlap. |
| Red Flags lists | Each flag is a self-check trigger. |
| Conflicting Signals / resolution hierarchies | These are decision trees, not prose. |

**Safe to compress:**

- Duplicate sentences across sections (merge)
- Verbose framing around simple instructions (trim)
- Examples that are longer than the pattern they illustrate (trim)
- "Philosophical" paragraphs that don't encode a behavioral rule (fold or cut)
- Commentary: "This is important because..." — unless the "because" encodes a rule not stated elsewhere

## Quick Reference

| If you see | Do this |
|---|---|
| Same sentence in two sections | **Merge** — keep one copy |
| Small section elaborating a parent concept | **Fold** — inline it |
| "The next thing you should do is..." | **Trim** — just say what to do |
| Two validation checks asking the same thing | **Prune** — keep the stronger one |
| A code example longer than the rule it teaches | **Trim** — single-line example |
| Any behavioral table row | **Never touch** unless genuinely redundant |

## Do NOT Compress

- Skills under 200 words — they're already at target
- Skills where every section teaches something distinct
- When you can't distinguish behavioral content from exposition (read more carefully)

## Common Mistakes

| Mistake | Fix |
|---|---|
| Cutting a "redundant" sentence that actually encodes a rule not stated elsewhere | Before cutting, verify: is this rule encoded anywhere else in the skill? If not, keep it. |
| Merging sections that serve different purposes | "When to Use" and "Triggers" may overlap but answer different questions. Check audience intent before merging. |
| Pruning validation checks because they "feel similar" | Two checks that feel similar may catch different failure modes. Only prune when the underlying test is identical. |
| Compressing signal tables | These are data, not prose. Never compress. |
| Over-applying to a skill that doesn't need it | If the skill is already ≤ 200 words, compression causes harm — you'll cut behavioral content to meet an imaginary target. |
