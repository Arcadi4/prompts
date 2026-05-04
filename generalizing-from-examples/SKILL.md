---
name: generalizing-from-examples
description: Use when a user provides examples prefixed by "for example", "such as", "e.g.", "like", or ends a list with "etc." / "and so on". Apply the Inference Ladder: detect the example marker, abstract the underlying concept, enumerate all related cases the user implied but didn't name, and structure your output around the inferred categories — not around the specific examples the user happened to mention.
license: CC-BY-SA-4.0
---

# Generalizing From Examples

## Overview

Examples are signposts, not fences. When a user says "for example, X", they are pointing at a concept through X — not limiting the scope to X.

**The critical skill is extracting the abstract concept from the trivial description.** Before you can enumerate cases or structure output, you must identify what category the examples belong to. This abstraction step is where most agents underperform: they see "Redis" and think "other caches" (shallow) instead of "the caching layer strategy" (deep). The deeper your abstraction, the more value your enumeration produces.

**Core principle:** Every example marker ("for example", "e.g.", "like", "such as") is the user saying: "I am giving you one instance. Infer the rest."

**The Inference Ladder is recursive.** You apply it to the user's words, produce output, then apply it again to your own output. Most failures occur on the second pass: you think you've generalized, but your "generalization" is just the example wearing a category label. If someone said "detect closure signals like numbered lists" and you built a rule that only checks for numbers, you didn't generalize — you renamed. The ladder didn't break on the first rung (you identified the example marker). It broke on the fourth (you thought you'd abstracted to "closure signals" but your implementation reveals you only handled "numbered lists"). After every generalization, introspect: **did I actually climb, or did I just rename?**

## When to Use

### Triggers — Load this skill when the user:

- Uses example-introducing language: "for example", "e.g.", "i.e.", "like", "such as", "including", "things like", "something like", "kind of like"
- Uses softening qualifiers: "or something", "and so on", "etc.", "and things like that"
- Names a specific tool/library/framework but the context suggests they mean the category: "use something like Redis" → they want caching, not necessarily Redis
- Asks "do X, for example Y, Z" — the "for example" attaches to Y/Z, not to X
- Gives one concrete case when the domain obviously contains many parallel cases

### Do NOT use when:

- The user says "exactly", "specifically", "only", "precisely" without example markers
- The user enumerates a closed set: "the three things are A, B, and C"
- The user gives imperative commands with no softening language
- The scope is inherently singular (one bug, one file, one function)

## Core Pattern

### The Inference Ladder

When you encounter an example, climb these rungs in order:

```
Example given     →  "for example, check auth middleware"
    ↓
Identify marker   →  "for example" = this is illustrative
    ↓
Abstract concept  →  The user wants me to audit SECURITY BOUNDARIES
                     (NOT just "check middleware" — that's task-level, not concept-level)
    ↓                   Ask: what property does this example share with potential peers?
                         Auth middleware = enforces access control before request processing.
                         Peers share that property: rate limiting (DOS protection), input
                         validation (injection prevention), CORS (origin policy), CSRF
                         (forgery protection), audit logging (accountability). All are
                         SECURITY BOUNDARIES in the request pipeline.
    ↓
Infer scope       →  All security-relevant middleware in every route, not just auth
    ↓
Enumerate cases   →  Rate limiting, input validation, CORS, CSRF, audit logging,
                      header sanitization, request size limits...
    ↓
Confirm boundary  →  "I'll audit all security boundaries in the middleware layer — auth,
                      rate limiting, input validation, CORS, CSRF, audit logging. Does
                      that cover what you meant, or should I also check route-level
                       authorization and data-layer access controls?"
```

### The Ladder Is Recursive

The Inference Ladder is not a one-pass filter (user input → output). It is a loop: after you climb it on the user's words, you must climb it again on your own output.

**Where this breaks:** You hear "exhaustive enumeration (patterns like 1. 2. 3.)." You think you've abstracted to "exhaustive enumeration" and move on. But when you implement, your rules only fire on numbered lists. Your "abstraction" was just the example wearing a category label — you renamed, you didn't climb.

**The failure is not at the input stage** (you correctly identified "1. 2. 3." as an example of exhaustive enumeration). **The failure is at the output stage** — your generalization didn't survive its own test. You produced an output that still had an unexamined example acting as a category.

**The introspection step:** After every generalization, ask:
1. "Did I actually climb, or did I just rename?" — If you can't produce category members the example didn't give you, you renamed.
2. "Would someone reading my output know the example I started from?" If yes, the example is still the gravitational center.
3. "What would I produce if the user had given a DIFFERENT example of the same category?" If the answer is "something very different," your generalization is example-bound, not category-bound.

This is not about being "careful" or "checking your work." It is about recognizing that the ladder's hardest rung is the one inside your own head — the abstraction step where you think you've climbed but you've only relabeled.

### Parse Grammar, Not Just Keywords

Example markers have syntactic scope. The marker modifies what comes after it, not the entire sentence:

**User says:** "Add logging to the API routes, for example the user endpoints"
- ❌ Scope: only user endpoints (treats "for example" as "only")
- ❌ Scope: add logging to everything (ignores the example entirely)
- ✅ Scope: user endpoints are one instance of API routes that need logging; apply to all API routes

**User says:** "We need caching, like with Redis"
- ❌ Scope: must use Redis
- ✅ Scope: need a caching layer; Redis is one candidate, evaluate alternatives

**User says:** "Handle edge cases (e.g., null inputs, empty strings, invalid types)"
- ❌ Scope: only null, empty, and invalid-type
- ✅ Scope: all edge cases in the input domain — null/empty/invalid are representative examples

### Closure Signals vs. Openness Signals

The user either signals a **closed set** ("here is the complete list") or an **open set** ("here are representative samples"). The signal is not a specific syntactic pattern — it's the presence of closure language or openness language. Confusing the two is a critical failure mode.

**Closure signals (exhaustive — do NOT add):**

The abstract category is anything that communicates "this list is complete":

| Mechanism | Examples |
|---|---|
| Numbered lists | "1. auth, 2. rate limit, 3. logging" |
| Cardinal words | "the three things are", "both of these", "all four" |
| Definite quantifiers | "the complete list", "everything we need", "the full set" |
| Closing conjunctions | "X, Y, and Z" (final "and" with NO trailing "etc." or "and so on") |
| Closure words | "specifically", "only", "exactly", "precisely", "just", "that's it" |
| Finality language | "these are the ones", "nothing else", "and that's all" |

**Openness signals (illustrative — generalize freely):**

The abstract category is anything that communicates "this list is not complete":

| Mechanism | Examples |
|---|---|
| Example markers | "for example", "e.g.", "such as", "like", "including" |
| Open-ended qualifiers | "etc.", "and so on", "and things like that", "or something" |
| Softening language | "kind of like", "something like", "or whatever" |
| Non-numbered lists with trailing openness | "X, Y, Z, etc." — the "etc." overrides the list form |
| Hedging | "maybe", "probably", "I'm not sure but" |

**The hard rule:** Check for closure signals, not for a specific format. A numbered list is one example of closure — but so is "the only things are", "that's the full list", and "X, Y, and Z" with a final "and" and no "etc." Similarly, "for example" is one example of openness — but so is "etc.", "or something", and any marker that explicitly signals incompleteness.

If you find yourself thinking "but the user used commas so it must be a sample" — you're pattern-matching syntax instead of interpreting the signal. That is the shallow-abstraction failure mode applied to enumeration boundaries. Reverse the reasoning: what signal is the user sending (closure or openness), not what syntax did they use.

### Abstraction Confidence Gates

The number of examples determines how aggressively you should abstract:

| Examples | Confidence | Action |
|---|---|---|
| 1 | Weak — insufficient signal | Infer the pattern, then **must confirm** with user before enumerating widely. One data point could be random. |
| 2-3 | Moderate — pattern emerging | Abstract and enumerate. Present as educated inference: "It sounds like you're talking about [category]. I'd cover [cases]. Does that match your intent?" |
| 4-5 | Strong — deliberate sampling | Abstract and enumerate confidently. Present the category as the real ask: "I understand you need [deep abstraction]. Here's the full picture: [cases]." |
| 6+ | Very strong — the examples ARE the pattern | Infer the user's actual goal and intent directly. The examples were scaffolding. The abstraction IS the request. Enumerate comprehensively. |

### The Enumeration Step

After abstracting the concept, enumerate related cases. This is where generalization produces value.

**Approach:**

1. Name the abstract category: "This is about _____"
2. Ask: "What else belongs in this category?"
3. List candidates including the user's examples nested within, ordered by relevance
4. Frame new items as educated inference: "That also suggests [cases] — would those be in scope?"
5. Let the user trim, don't self-censor

**Example:**

> User: "Make sure we handle errors, for example network timeouts."

You think:
- Category: "transient failure modes in external dependencies"
- Network timeouts → also: DNS failures, connection refused, SSL handshake errors, rate limiting (429), server errors (5xx), partial responses, hanging connections
- Present the enumerated list; let the user say "just timeouts for now"

**Demonstrating understanding through enumeration:**
- Always nest the user's examples within the larger category — shows you heard them AND abstracted
- Frame additions as educated guesses: "Based on [category], I'd also consider [cases]. Do you mean [deeper concept]?"
- If your abstraction might overreach, use "do you mean by": "Do you mean full observability? If so, I'd cover metrics, logs, traces and alerting — not just response times."
- Never replace or discard the user's examples — build outward from them
- One extra example proves you understood the pattern better than five generic ones

### Abstraction Depth

The same example supports multiple levels of abstraction. Deeper is better — it captures the user's actual intent, not just their words.

| Example | Shallow Abstraction (Weak) | Deep Abstraction (Strong) |
|---|---|---|
| "track response times on user endpoints" | "monitor response times on all endpoints" | Observability: latency, error rates, throughput, saturation across all services, with dashboards and alerts |
| "use something like Redis for sessions" | "pick a cache for sessions" | Caching layer strategy: session store, query cache, rate limiter — each with domain-specific requirements |
| "add validation like email format checking" | "validate other fields" | Input integrity: format validation, type checking, range constraints, sanitization, business rule enforcement |
| "research ai agent suites such as X and Y" | "find other agent suites" | Agent extension architecture: component packaging, plugin systems, configuration layering, distribution models, community ecosystem patterns |

**How to deepen your abstraction:**
1. Ask "what property does this specific example share with the things the user is NOT naming?"
2. Ask "what would the user say if I asked them to describe this in one word?"
3. Ask "what problem is the user actually trying to solve, for which this example is one solution?"
4. If your abstraction is just the example pluralized or slightly widened, go one level deeper.

**The acid test:** Could you present your abstraction to the user without mentioning their example, and would they say "yes, exactly"? If your abstraction only makes sense when the user already knows which example you started from, it's too shallow.

### When NOT to Enumerate

Stop generalizing when:
- The example IS the whole thing (domain is inherently singular)
- Enumeration would produce noise not signal (category has 2 items)
- The user explicitly scoped with "only", "just", "specifically"
- You're guessing categories the user would not recognize as related

### Structure Output by Category, Not by Example

Generalizing isn't just about what you include — it's about how you organize it. If you enumerate the right items but organize them around the user's examples (P0 = what they said, P1 = what you added), you're still transcribing with padding. The output structure itself must reflect the abstraction.

**Before (example-anchored — still transcription):**
```
Caching Solutions Research:
  Priority 1: Redis (user mentioned)
  Priority 2: Memcached, Hazelcast
  Priority 3: Other stores
```
→ The user's example is still the gravitational center. You haven't generalized; you've just appended.

**After (category-anchored — true generalization):**
```
Caching Layer Strategy:
  Session store: Redis, Memcached, Dragonfly, Valkey
  Query cache: Redis (read-replica), Elasticache
  Object cache: Hazelcast, Redis
  Selection criteria: latency <1ms, TTL support, clustering
```
→ The category is the organizing principle. No example gets privileged position.

**The test:** Could a reader identify which items the user mentioned? If yes, you're organizing by source, not by concept. Restructure.

**If you must show priorities**, base them on domain importance — not on whether the user said them. The user's examples are not inherently P0.

## Quick Reference

| User says | Agent should NOT do | Agent should DO |
|---|---|---|
| "for example, X" | Scope to X only | Treat X as one instance of a pattern |
| "like X" | Assume X is mandatory | Treat X as illustrative; find alternatives |
| "e.g., X, Y" | Treat X,Y as exhaustive | Treat X,Y as a sample; find the rest |
| "such as X" | Match X literally | Abstract the category X belongs to |
| "including X" | Start and stop at X | Use X as a springboard to find more |
| "something like X" | Try to implement X exactly | Understand what X does that the user values |
| "and so on" / "etc." | Ignore this signal | You MUST enumerate more — the user told you the list is incomplete |
| "or something" | Take the suggestion literally | Recognize the user is unsure; propose the category |

## Enumeration Patterns by Domain

### When the example is a code location or module:

Ask: "What other modules share the same role, layer, or responsibility?"

> "Check the middleware, for example the auth middleware."
> → All middleware in the request pipeline: auth, logging, rate limiting, CORS, body parsing, error handling...

### When the example is a bug or error type:

Ask: "What other inputs, states, or conditions would trigger the same class of bug?"

> "Fix null pointer issues, e.g., when user.profile is undefined."
> → All optional-chain violations on user: user.settings, user.preferences, user.subscriptions...

### When the example is a tool or library:

Ask: "What capability does this tool provide that the user actually needs?"

> "Use something like PostgreSQL for storage."
> → The user needs persistent relational storage; PostgreSQL, MySQL, SQLite are candidates

### When the example is a test case or scenario:

Ask: "What boundary conditions or equivalence classes does this case represent?"

> "Test with edge cases, like an empty cart."
> → All cart states: empty, single item, max items, zero price, negative quantity, concurrent modification...

## Rationalization Prevention

Agents resist generalization for specific, predictable reasons. These are the excuses you will generate in your own reasoning, and they are wrong:

| Rationalization | Reality |
|---|---|
| "The user specifically said X, so that's what they want" | They said "for example X" — the marker is an explicit instruction to generalize. Ignoring it is ignoring the user's words. |
| "I don't want to over-engineer / do extra work" | Under-scoping and missing related cases creates rework. The user will ask "what about Y?" and you'll do it in a second pass. Do it once. |
| "Better to be precise than wrong" | Precision on the example while missing the pattern is precisely wrong. The user's intent was the pattern. |
| "If they wanted Y they would have said Y" | They signaled they want the category by using an example marker. That IS them saying Y — through abstraction. |
| "I'll start with just X and they can ask for more" | This is delegation upward. The user gave you an example so you would do the thinking, not so they would have to enumerate everything. |
| "The example is a good starting point; I'll scope to it" | Examples are starting points for thinking, not for scope. The scope is the pattern, not the instance. |
| "I'm being conservative to avoid mistakes" | Conservatism that ignores the user's explicit generalization signal is not conservative — it's inattentive. |
| "Enumerating more cases is speculating" | Inferring a pattern from an example is not speculation — it's comprehension. The user deliberately chose an example to communicate a pattern. |
| "I'll put the user's examples as P0 and related ones as P1" | Priority-ordering by who said it reveals example-anchored thinking. All items are peers in the category. Prioritize by domain importance, not by whether the user named them. |

## Red Flags — Stop and Generalize

You are under-generalizing when you catch yourself thinking:

- "The user mentioned X so I'll just do X"
- "I'll handle the other cases if they ask"
- "The example is probably what they care about most"
- "If I add Y it might be wrong"
- "They only said X, so Y is out of scope"
- "This is what they literally asked for"
- "I'll make the examples the top priority and add more as lower priority"

**All of these mean: re-read the user's message. Find the example marker. Climb the inference ladder. Enumerate.**

## Default Agent Behavior

When you detect an example marker in the user's request:

1. **Flag it internally.** The marker changes the semantics of everything that follows.
2. **Abstract before executing.** Before writing code or taking action, verbalize the inferred pattern.
3. **Enumerate before scoping.** List candidate cases before deciding what's in scope.
4. **Confirm at the right level of abstraction.** Don't ask "should I also do Y?" — ask "I understand this is about [category]. I'll cover [enumerated cases]. Does that match your intent?"

**Asking for confirmation:**

- ❌ "You mentioned auth middleware. Should I also check rate limiting?" (makes user do your thinking)
- ✅ "I understand you want me to audit all security-relevant middleware — auth, rate limiting, CSRF, input validation. I'll check each. Does that cover what you meant?" (you did the thinking, user confirms)

## Common Mistakes

| Mistake | Fix |
|---|---|
| Treating "for example X" as "only X" | The marker explicitly signals non-exhaustiveness. Treat X as a starting point. |
| Generalizing without confirming the abstraction | Enumerate and ask. Don't silently expand scope. |
| Asking the user to enumerate instead of doing it yourself | The example marker means the user expects you to enumerate. |
| Over-generalizing when the domain is singular | "Fix the typo on line 42" — there is no pattern to infer. Just fix line 42. |
| Ignoring "etc." and "and so on" | These are explicit signals the list is incomplete. You MUST find more items. |
| Treating "like X" as "must use X" | "Like" signals similarity, not identity. Find alternatives that are "like" X. |
| Scoping to the literal words without parsing the marker's grammar | The marker modifies what follows. Parse the syntax of the sentence. |
| Enumerating noise — adding cases that don't belong to the inferred category | Stay within the abstracted concept. Don't add items from a different pattern. |
| **Rename-as-generalize: thinking you've abstracted when you've only relabeled the example** | You hear "exhaustive enumeration (patterns like 1. 2. 3.)" and think your abstraction is "exhaustive enumeration." But when you implement, you only detect numbered lists. Your "abstraction" was just the example with a new label — you never actually identified what makes something exhaustive. The test: list every member of your abstraction without looking at the example. If you can't generate items the example didn't give you, you renamed, you didn't generalize. Re-climb the ladder. |
| **Pattern-matching syntax instead of interpreting the signal** | "The user used commas, so it must be a sample" or "the user used numbers, so it must be exhaustive." Syntax is one mechanism among many. Look for the signal, not the format. |

## Validation

After generalizing, verify:

1. **Does my inferred pattern actually capture what the user's example is an instance of?** If the user says "e.g., check auth middleware" and you infer "all middleware" — is auth middleware representative of all middleware in a way the user would agree with?
2. **Are my enumerated cases all instances of the same abstraction?** Don't mix categories.
3. **Would the user say "yes, that's what I meant"** when they see the enumerated list? If not, your abstraction is wrong.
4. **Did I ask for confirmation at the right level?** Present the category, not a yes/no on each item.
5. **Can a reader identify which items the user specifically mentioned?** If your output structure makes the user's examples stand out (P0, "Primary", "Recommended") — restructure so the category is the headline and all items are peers.
6. **Did I actually generalize, or did I rename the example?** The acid test: can you enumerate members of the category that are NOT derivable from the example alone? If your abstraction only produces items that are obvious variations of the example (e.g., "numbered list" → "more numbered lists"), you renamed, you didn't generalize. Re-climb the ladder from the top — what property defines this category, and what other mechanisms share it?

## Example Dialogues

### Before (literal agent) vs After (generalizing agent)

**User:** "Add input validation to the form fields, for example the email field."

❌ **Literal agent:** *Adds validation only to the email field.*

✅ **Generalizing agent:** "I'll add validation to all form fields — email (format), password (strength), username (length/characters), phone (format). For each I'll add both client-side and server-side validation. Sound right?"

---

**User:** "We should handle API errors better, e.g., the 401 on the login page."

❌ **Literal agent:** *Only handles 401 on the login page.*

✅ **Generalizing agent:** "I'll add structured error handling across the app. For the login page specifically: 401 (bad credentials), 429 (rate limited), 500 (server error), and network failures. I'll also add the same pattern to signup, password reset, and settings pages since they share the same API error surface. That what you had in mind?"

---

**User:** "Cache expensive queries, like the dashboard stats query."

❌ **Literal agent:** *Caches only the dashboard stats query.*

✅ **Generalizing agent:** "I'll audit all expensive queries in the app and add caching where appropriate. Dashboard stats is one — also candidate: user activity feed, leaderboard, analytics aggregations, notification counts. I'll use Redis with TTL-based invalidation. I'll start with dashboard stats as the first one. Does that approach and scope look right?"

---

**User:** "Research caching solutions for our app, like Redis for the session store."

❌ **Example-anchored output:**
```
Caching Research:
  P0: Redis (user specified) — session store, query cache
  P1: Memcached — high-throughput simple caching
  P2: Hazelcast — distributed caching for Java services
```
→ Redis is still the headline. The user's examples plus footnotes.

✅ **Category-anchored output:**
```
Caching Layer Strategy:
  Session Store: Redis, Memcached, Dragonfly, Valkey
  Query Cache: Read-through — Redis (read-replica), Elasticache
  Rate Limiter: Counter-based — Redis (INCR+EXPIRE), local LRU
  Each category has selection criteria independent of whether the user named it.
```
→ The user can't tell which one they mentioned. That's the point.

---

**User:** "We should add monitoring, for example track response times on the user endpoints."

❌ **Shallow abstraction:** *Adds response time tracking to all endpoints.*
(The agent only widened the example's scope, didn't climb the ladder.)

✅ **Deep abstraction:** "I understand you want observability into the system. Response times are one metric. I'll instrument the full golden signals: latency (p50/p95/p99), error rate, throughput, and saturation across all services. Plus dashboards for each dimension and alerting thresholds. For user endpoints that means: response time tracking, error rate by endpoint, request volume by time window, and database connection pool saturation. Does that capture what you meant by monitoring, or should I scope to just latency for now?"
