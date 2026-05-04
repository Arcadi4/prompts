---
name: generalizing-from-examples
description: Use when a user provides examples prefixed by "for example", "such as", "e.g.", "like", or ends a list with "etc." / "and so on". Apply the Inference Ladder: detect the example marker, abstract the underlying concept, enumerate all related cases the user implied but didn't name, and structure your output around the inferred categories — not around the specific examples the user happened to mention.
license: CC-BY-SA-4.0
---

# Generalizing From Examples

## Overview

Examples are signposts, not fences. When a user says "for example, X", they are pointing at a concept through X — not limiting the scope to X. Every example marker ("for example", "e.g.", "like", "such as") is the user saying: "I am giving you one instance. Infer the rest."

**The critical skill is extracting the abstract concept, not widening the example.** Agents typically underperform here: they see "Redis" and think "other caches" (shallow — pluralizing) instead of "the caching layer strategy" (deep — climbing). The deeper your abstraction, the more value your enumeration produces.

## When to Use

### Triggers — Load this skill when the user:

- Uses example-introducing language: "for example", "e.g.", "like", "such as", "including", "things like", "something like"
- Uses softening qualifiers: "or something", "and so on", "etc.", "and things like that"
- Names a specific tool/library but the context suggests they mean the category: "use something like Redis" → they want caching, not necessarily Redis
- Gives one concrete case when the domain obviously contains many parallel cases

### Do NOT use when:

- The user says "exactly", "specifically", "only", "precisely" without example markers
- The user uses closure signals: numbered lists, cardinal words ("the three things"), final "and" without "etc."
- The scope is inherently singular (one bug, one file, one function)

## Core Pattern

### The Inference Ladder

Climb these rungs in order:

```
1. Detect marker     →  "for example" = this is illustrative
2. Isolate example   →  "check auth middleware" — but what category is this?
3. Abstract concept  →  SECURITY BOUNDARIES (not just "all middleware")
                        Ask: what property does this example share with potential peers?
4. Infer scope       →  All security-relevant middleware in every route
5. Enumerate cases   →  Rate limiting, input validation, CORS, CSRF, audit logging...
6. Confirm boundary  →  "I'll audit all security boundaries — auth, rate limiting,
                        input validation, CORS, CSRF, audit logging. Does that cover
                        what you meant?"
```

### The Ladder Is Recursive

The ladder is a loop, not a one-pass filter. After you climb it on user input, climb it again on your own output. Most failures occur on this second pass: you think you've generalized, but your "abstraction" is just the example wearing a category label.

**The test:** Can you enumerate category members the example didn't give you? If someone says "exhaustive enumeration (patterns like 1. 2. 3.)" and your implementation only detects numbered lists, you renamed — you didn't generalize. What makes something exhaustive? Numbered lists are ONE mechanism. Closure words, cardinal phrases, definitive conjunctions are equally valid.

**Introspection after every generalization:**
1. "Did I actually climb, or did I just rename?"
2. "What would I produce if the user had given a DIFFERENT example of the same category?"

### Closure Signals vs. Openness Signals

The user signals either a **closed set** ("this list is complete") or an **open set** ("these are samples"). The signal is the category of language, not the specific syntax.

**Closure signals (exhaustive — do NOT add):**

| Mechanism | Examples |
|---|---|
| Numbered lists | "1. auth, 2. rate limit, 3. logging" |
| Cardinal words | "the three things are", "both of these" |
| Definite quantifiers | "the complete list", "everything we need" |
| Closing conjunctions | "X, Y, and Z" (final "and" with NO "etc.") |
| Closure words | "specifically", "only", "exactly", "just", "that's it" |
| Finality language | "these are the ones", "nothing else" |

**Openness signals (illustrative — generalize freely):**

| Mechanism | Examples |
|---|---|
| Example markers | "for example", "e.g.", "such as", "like", "including" |
| Open-ended qualifiers | "etc.", "and so on", "and things like that" |
| Softening language | "kind of like", "something like", "or whatever" |
| Trailing openness | "X, Y, Z, etc." — the "etc." overrides the list form |
| Hedging | "maybe", "probably", "I'm not sure but" |

Check for the signal, not the format. "But the user used commas" is pattern-matching syntax, not interpreting intent.

### Abstraction Confidence Gates

| Examples | Confidence | Action |
|---|---|---|
| 1 | Weak | Infer the pattern, then **must confirm** with user before enumerating widely. |
| 2-3 | Moderate | Abstract and enumerate. Present as educated inference. |
| 4-5 | Strong | Present the category as the real ask. Enumerate confidently. |
| 6+ | Very strong | The examples ARE the pattern. The abstraction IS the request. Enumerate comprehensively. |

### The Enumeration Step

1. Name the abstract category
2. List candidates including the user's examples nested within
3. Frame additions as educated inference: "Based on [category], I'd also consider [cases]."
4. If your abstraction might overreach: "Do you mean [deeper concept]? If so, I'd cover [cases]."
5. Let the user trim, don't self-censor

Never replace the user's examples — build outward from them. One extra example proves you understood the pattern better than five generic ones.

### Abstraction Depth

The same example supports multiple levels of abstraction. Deeper captures intent, not just words.

| Example | Shallow (Weak) | Deep (Strong) |
|---|---|---|
| "track response times" | "monitor all endpoints" | Observability: golden signals across all services |
| "use something like Redis" | "pick a cache" | Caching layer strategy: session store, query cache, rate limiter |
| "validate email format" | "validate other fields" | Input integrity: format, type, range, sanitization, business rules |
| "research agent suites such as X" | "find other suites" | Agent extension architecture: packaging, plugins, distribution |

**How to deepen:**
1. What property does this example share with things the user ISN'T naming?
2. What problem is the user actually solving, for which this example is one solution?
3. If your abstraction is just the example pluralized, go one level deeper.

**Acid test:** Could you present your abstraction without mentioning their example, and would they say "yes, exactly"?

### When NOT to Enumerate

- The example IS the whole thing (domain is inherently singular)
- The user used closure signals
- You're guessing categories the user wouldn't recognize as related

### Structure Output by Category, Not by Example

The output structure itself must reflect the abstraction. If you enumerate the right items but organize them around the user's examples (P0 = what they said), you're transcribing with padding.

**Before (example-anchored):**
```
Caching Research:
  Priority 1: Redis (user mentioned)
  Priority 2: Memcached, Hazelcast
```
→ The user's example is the gravitational center. You haven't generalized; you've appended.

**After (category-anchored):**
```
Caching Layer Strategy:
  Session store: Redis, Memcached, Dragonfly, Valkey
  Query cache: Redis (read-replica), Elasticache
  Object cache: Hazelcast, Redis
```
→ The category organizes. No example gets privileged position.

**Test:** Could a reader identify which items the user mentioned? If yes, restructure.

## Quick Reference

| User says | Agent should NOT do | Agent should DO |
|---|---|---|
| "for example, X" | Scope to X only | Treat X as one instance of a pattern |
| "like X" | Assume X is mandatory | Treat X as illustrative; find alternatives |
| "e.g., X, Y" | Treat X,Y as exhaustive | Treat X,Y as a sample; find the rest |
| "such as X" | Match X literally | Abstract the category X belongs to |
| "including X" | Start and stop at X | Use X as a springboard |
| "something like X" | Try to implement X exactly | Understand what X does that the user values |
| "etc." / "and so on" | Ignore this signal | You MUST enumerate more — the user told you the list is incomplete |
| "or something" | Take literally | Recognize the user is unsure; propose the category |

## Rationalization Prevention

Agents resist generalization for predictable reasons. These excuses are wrong:

| Rationalization | Reality |
|---|---|
| "The user specifically said X, so that's what they want" | They said "for example X" — the marker IS an instruction to generalize. |
| "I don't want to over-engineer" | Under-scoping creates rework. The user will ask "what about Y?" Do it once. |
| "Better to be precise than wrong" | Precision on the example while missing the pattern is precisely wrong. |
| "If they wanted Y they would have said Y" | They signaled the category by using an example marker. That IS them saying Y. |
| "I'll start with X and they can ask for more" | This is delegation upward. The user expects you to do the thinking. |
| "The example is a good starting point; I'll scope to it" | Examples are starting points for thinking, not for scope. |
| "I'm being conservative to avoid mistakes" | Conservatism that ignores explicit generalization signals is inattentive. |
| "Enumerating more is speculating" | Inferring a pattern from an example is comprehension, not speculation. |
| "I'll put the user's examples as P0" | Priority by who said it = example-anchored thinking. All items are peers. |

## Red Flags — Stop and Generalize

- "The user mentioned X so I'll just do X"
- "I'll handle the other cases if they ask"
- "If I add Y it might be wrong"
- "They only said X, so Y is out of scope"
- "I'll make the examples the top priority"

**All of these mean: re-read the user's message. Find the example marker. Climb the ladder.**

## Common Mistakes

| Mistake | Fix |
|---|---|
| Treating "for example X" as "only X" | The marker signals non-exhaustiveness. X is a starting point. |
| Generalizing without confirming the abstraction | Enumerate and ask. Don't silently expand scope. |
| Asking the user to enumerate | The example marker means the user expects YOU to enumerate. |
| Over-generalizing when the domain is singular | "Fix the typo on line 42" — no pattern to infer. |
| Ignoring "etc." and "and so on" | Explicit signals the list is incomplete. You MUST find more items. |
| Treating "like X" as "must use X" | "Like" signals similarity, not identity. |
| **Rename-as-generalize: relabeling the example instead of climbing** | You hear "exhaustive enumeration (patterns like 1. 2. 3.)" and implement a rule that only fires on numbered lists. You renamed, you didn't generalize. The test: list every member of your abstraction without looking at the example. If you can't produce items the example didn't give you, re-climb the ladder. |
| Pattern-matching syntax instead of interpreting the signal | "The user used commas, so it must be a sample." Look for the signal, not the format. |

## Validation

After generalizing, verify:

1. **Does my inferred pattern capture what the example is an instance of?**
2. **Are my enumerated cases all instances of the same abstraction?**
3. **Would the user say "yes, that's what I meant"?**
4. **Did I ask for confirmation at the right level?** (Category, not yes/no per item)
5. **Can a reader identify which items the user mentioned?** If yes, restructure by category.
6. **Did I actually generalize, or rename?** Can you enumerate members the example didn't give you?

## Example Dialogues

**User:** "Add input validation to the form fields, for example the email field."

❌ **Literal:** *Adds validation only to the email field.*

✅ **Generalizing:** "I'll add validation to all form fields — email (format), password (strength), username (length/characters), phone (format). Both client-side and server-side. Sound right?"

---

**User:** "We should handle API errors better, e.g., the 401 on the login page."

❌ **Literal:** *Only handles 401 on the login page.*

✅ **Generalizing:** "I'll add structured error handling across the app. Login page: 401, 429, 500, network failures. Same pattern on signup, password reset, settings — they share the API error surface. That what you had in mind?"

---

**User:** "We should add monitoring, for example track response times on the user endpoints."

❌ **Shallow:** *Adds response time tracking to all endpoints.* (Widened the example, didn't climb.)

✅ **Deep:** "I understand you want observability. Response times are one golden signal. I'll instrument latency (p50/p95/p99), error rate, throughput, and saturation across all services — plus dashboards and alerting thresholds. For user endpoints: response time tracking, error rate by endpoint, request volume by time window, DB connection pool saturation. Does that capture monitoring, or scope to latency for now?"
