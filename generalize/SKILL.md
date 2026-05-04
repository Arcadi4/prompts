---
name: generalize
description: Use when a user provides examples prefixed by "for example", "such as", "e.g.", "like", or ends a list with "etc." / "and so on". For architecture/design tasks, abstract first to identify the concept, then enumerate. For research/clarification tasks, enumerate first — the concept organizes the output but the list is the deliverable. Most valuable for documentation, planning, and research; less necessary for direct implementation with sufficient domain context.
license: CC-BY-SA-4.0
---

# Generalizing From Examples

## Overview

Users often describe something by listing instances of it. They say "like X" or "for example Y" and trust you to understand what larger thing they're pointing at.

**This skill applies primarily to documentation, planning, and research tasks** — contexts where your output structure is shaped by the concepts you identify. For direct implementation tasks where you have sufficient domain context, the skill adds less value; competent agents already produce good plans without it. The skill's leverage is in structuring output: docs, plans, research reports, architecture decisions.

**Two equal imperatives, two different leads:**

| Context | Lead Imperative | Why |
|---|---|---|
| **Architecture / design decisions** (major choices, system-level scope) | **Abstract first** — name the concept, then use it to drive decisions | You can't design without knowing what system you're designing. "Add monitoring, for example track response times" → the concept is Observability, not latency tracking. The concept determines what to build. |
| **Research / clarification tasks** (find, list, investigate, survey) | **Enumerate first** — the user wants all instances. The concept organizes them but isn't the point | "Research AI agent suites such as X and Y" → the user wants a research report. Enumerate comprehensively, then use the concept (Agent Extension Architecture) to structure the report. Enumeration IS the deliverable. |

**Both imperatives are always active** — the question is which one leads based on what the user is asking you to produce. If the output is a decision, the concept leads. If the output is a list, enumeration leads. Either way: enumerate. The concept is a structure multiplier, not a prerequisite.

## When to Use

**This skill is most valuable for documentation, planning, and research tasks** — contexts where your output structure is shaped by the concepts you identify. For direct implementation tasks with sufficient domain context, agents typically produce adequate plans without the skill.

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

Climb these rungs in order. **Which rung drives the climb depends on context:**

- **Architecture/design task → driven from rung 3 (abstract).** The concept determines what to build; enumeration follows.
- **Research/clarification task → driven from rung 5 (enumerate).** Comprehensive enumeration is the goal; the concept organizes it.
- **Either way: all six rungs are visited.** The order just tells you what to prioritize.

```
1. Detect marker     →  "for example" = this is illustrative
2. Isolate example   →  "check auth middleware" — what concept is this an instance of?
3. ▼ ABSTRACT ▼
   Identify concept  →  SECURITY BOUNDARIES (architecture lead) or
                        a structure label (research lead)
                        Ask: what one thing do all these potential examples participate in?
4. Infer scope       →  Architecture: all security-relevant middleware in every route.
                        Research: all agent extension platforms matching the category.
5. ENUMERATE CASES   →  The list the user wanted — always produced, never skipped.
6. Confirm boundary  →  "Does this cover what you meant?"
```

### The Ladder Is Recursive

The ladder is a loop. After you name the concept, climb it again on your own output. Most failures happen here: you think you've identified the transcendental object, but your "concept" is just the example in disguise.

**The test:** Can you name instances of the concept that the example didn't give you? Someone says "exhaustive enumeration (patterns like 1. 2. 3.)." You think your concept is "exhaustive enumeration." But your implementation only detects numbered lists. You didn't identify the concept — you renamed the example. What makes a list exhaustive? Numbered lists are one mechanism. Cardinal words, closure language, definitive conjunctions — these are ALL instances of the concept "closure signal." That's the transcendental object you missed.

**After every abstraction, ask:**
1. "What IS this concept? Not what is it called — what is it?" If someone else read only your concept name, would they generate the same instances?
2. "What would I produce if the user had given a DIFFERENT instance of the same concept?" If your answer changes radically, you're example-bound, not concept-bound.

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

### Conflicting Signals (Multiple Stakeholders)

When a single query contains **conflicting signals from different stakeholders** (e.g., one person says "for example X, Y, etc." and another says "scope to X only"), the enumeration scope is constrained by the **primary requester's directive**, not by treating all input as equal examples to generalize from.

**Resolution hierarchy:**

| Priority | Signal Type | Action |
|---|---|---|
| 1 | **Primary requester's closure directive** | Hard boundary. "Scope to what X originally asked for" means revert to X's examples only. |
| 2 | **Legally-mandated requirements** (GDPR, SOC2, etc.) | Flag as "required but out of scope" — include in a separate section with explicit rationale. Do NOT silently fold into the primary concept. |
| 3 | **Other stakeholders' openness signals** | Treat as additional context, not additional scope. Acknowledge them, defer them, do not enumerate them into the primary output. |

**The rule:** Enumerate the concept the PRIMARY requester defined. If another stakeholder adds a dimension the primary requester didn't address, that dimension is a **competing scope definition**, not an additional example of the same concept. Document the conflict; don't resolve it by inclusion.

**The test:** If you removed the primary requester's message, would your enumerated cases still make sense? If yes, you've drifted. If your enumeration includes domains the primary requester never mentioned, you've drifted.

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

The same example supports multiple levels of abstraction. The goal is to identify the concept the examples participate in — not to pluralize the example.

| Example (shadow) | Widening (still shadows) | Concept (the form) |
|---|---|---|
| "track response times" | "monitor all endpoints" | **Observability**: golden signals across all services |
| "use something like Redis" | "pick a cache" | **Caching layer strategy**: session store, query cache, rate limiter |
| "validate email format" | "validate other fields" | **Input integrity**: format, type, range, sanitization, business rules |
| "research agent suites such as X" | "find other suites" | **Agent extension architecture**: packaging, plugins, distribution |

**How to find the concept:**
1. What singular thing do ALL potential instances participate in? Not "what are more examples" — "what is the ONE thing they share?"
2. What problem is the user solving, for which this example is one approach?
3. If your "concept" is just the example with a wider scope, you haven't found it.

**Acid test:** Present the concept without the example. Would the user say "yes, that's the thing I meant"? If your concept only makes sense when the example is visible, it's not a concept — it's the example paraphrased.

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
| "I'll work through these one at a time" | Enumeration without the concept first = anchor on examples. Abstract, THEN enumerate. |
| "This is getting complicated — let me scope down" | Concept drift under cognitive load. Re-anchor on the abstraction before touching scope. |
| "Actually, the user probably just meant X" | Mid-task concept abandonment. If you identified a concept earlier, you need a reason to change it — fatigue is not a reason. |
| "Compliance/security says this is mandatory, so I must include it" | The question isn't whether the requirement matters. It's whether it belongs in THIS scope, for THIS primary requester. Flag it separately. Don't silently fold a competing dimension into someone else's concept. |

## Red Flags — Stop and Generalize

- "The user mentioned X so I'll just do X"
- "I'll handle the other cases if they ask"
- "If I add Y it might be wrong"
- "They only said X, so Y is out of scope"
- "I'll make the examples the top priority"
- "I'll start broad and narrow down as I go" (You'll narrow to the examples — concept drift.)
- Losing the concept mid-enumeration and listing concrete implementations instead
- Returning to example-anchored language after starting with category-anchored language
- Adding domains from stakeholders other than the primary requester
- Resolving conflicting stakeholder directives by including everything (over-inclusion is not conflict resolution)

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
| **Over-abstracting when the task is research/clarification** | User says "research agent suites such as X and Y" — you spend 40% of output naming the concept instead of enumerating. The user wants the list. Name the concept briefly, then enumerate. The concept serves the enumeration, not the other way around. |
| **Applying the skill to direct implementation** | The agent already has enough context to plan without abstraction. Loading the skill adds overhead without improving output. Reserve the skill for tasks where output STRUCTURE matters: docs, plans, research. |
| Pattern-matching syntax instead of interpreting the signal | "The user used commas, so it must be a sample." Look for the signal, not the format. |
| **Folding competing scopes into the primary concept** | One stakeholder says "X, Y, etc." and another says "scope to X only." You enumerate X, Y, and Z — silently including what the scope-setter excluded. When multiple people give conflicting directives, enumerate the primary requester's concept. Flag the rest separately. Don't resolve conflict by over-inclusion. |

## Validation

After generalizing, verify:

1. **Did I name the concept, or just rename the example?** Present the concept without the example. Would the user recognize it?
2. **Are my enumerated cases all instances of the same concept?** Don't mix categories.
3. **Would the user say "yes, that's what I meant"?**
4. **Did I ask for confirmation at the right level?** (The concept, not a yes/no per item)
5. **Can a reader identify which items the user mentioned?** If yes, restructure by concept.
6. **Can I generate instances the example didn't give me?** If my concept only produces variations of the original example, I renamed — I didn't find the concept.
7. **Did my concept stay stable through enumeration?** Re-read your output start to finish. Do the enumerated items still belong to the original concept, or did the concept drift toward concrete implementations? Goal-shifting is the silent failure — you don't notice it unless you audit. If the end of your output answers a different question than the start, you shifted. **Concrete test:** Does every domain in your output trace back to the primary requester's original examples? If you added domains from other people who weren't the primary requester, you drifted. **Concrete test 2:** Remove the primary requester from the conversation. Would your output still make sense? If yes, your concept has detached from the original ask.

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
