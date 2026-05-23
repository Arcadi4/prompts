---
name: independent-design
description: "Use when designing/planning for implementation, and to reflect on your design's domain behavior and real consumers."
license: CC-BY-NC-4.0
---

# Independent Design

## Overview

Independent design means software artifacts serve the system that will run them, not the conversation that produced them. The test is: **would an independent human engineer, planning and programming for themselves, design this the same way?**

Follow this sequence: list the requested elements, identify the core engineering need, then implement for the real domain.

## When to Use

- A design introduces names, structures, or metadata that echo the conversation instead of the domain.
- An element has no current consumer, behavioral effect, user surface, or persistence purpose.
- Verification proves shape, wording, or apparent completeness rather than behavior through the system surface.
- The result seems optimized for satisfying a plan or reviewer checklist more than for serving the running system.

Do not use this to reject normal domain modeling. If a field drives behavior, persistence, validation, routing, permissions, or user output, keep it.

## Workflow

Keep the steps independent and in order. Start with what was requested, then test it against runtime behavior.

```
1. Enumerate requested elements        → what was asked for
2. Classify each element               → runtime behavior, user surface, persistence, or dependent artifact
3. Apply the independent-engineer test → would this exist without the request wording?
4. Delete or rename dependent elements → prefer domain names and single sources of truth
5. Replace static tests                → test behavior through the surface, not copied structure
6. Re-check the design                 → every artifact has a current consumer or domain purpose
```

## Dependent-Design Patterns

Common manifestations, not an exhaustive list:

| Pattern | Smell | Better move |
|---|---|---|
| Conversation-shaped naming | `managerDashboardAccess`, `safeCheckoutStepName`, or `SarahsDashboard` encode request context | Name the domain behavior: `canViewAggregateMetrics`, `checkoutStepId`, `teamMetricsView` |
| UI-flow API design | `POST /checkout/review-and-confirm` mirrors one screen flow | Model the domain action: `POST /orders` or `POST /payments/confirm` |
| Implementation-shaped observability | Metrics like `redis_cache_hits` or `postgres_query_time` leak current storage choices | Track stable behavior: `cache_hits`, `storage_query_duration`, unless the technology is the contract |
| Deployment-shaped config | `awsLambdaTimeout` or `k8sReplicaCount` appears before the product depends on that platform | Use current concepts: `requestTimeout`, `instanceCount`; specialize only when variation exists |
| Design-only metadata | `suggestedFeatures`, `advisoryFlags`, or explanatory `notes` never affect behavior | Put prose in docs, or wire metadata to an actual decision |
| Relationship without use | `relatedItems`, `references`, or `allowedTransitions` no code traverses | Remove, or implement the routing/validation that needs it |
| Structure-only tests | Tests assert exact headings, counts, object keys, or non-empty arrays | Test behavior through the surface |
| Checklist helpers | One-line wrappers or API methods with no semantic compression | Use direct access, or create one meaningful operation |

## Runtime Gates

Before adding any element, answer:

| Gate | Required evidence |
|---|---|
| Consumer | What code reads this outside its definition/test? |
| Decision | Does it change a branch, route, permission, selection, validation, output, or persisted data? |
| Variation | Can it vary meaningfully today? |
| Domain name | Would this name make sense without knowing the request context? |
| Single source | Are repeated representations derived from one authoritative artifact? |
| Test value | Would the test fail if behavior broke, not just if copy changed? |

If the answer is “no” or “maybe later,” do not encode it as production structure. Keep the design independent: current consumers, current domain language, current behavior.

## Test Rules

Bad tests prove generated structure exists:

```ts
expect(config.validationRules).toBeDefined()
expect(document.sections.length).toBeGreaterThan(0)
expect(Object.keys(response)).toContain("metadata")
```

Good tests drive the real surface:

```ts
const result = validateCheckout({ paymentMethod: "expired-card" })
expect(result.canSubmit).toBe(false)
expect(result.errors).toContain("payment-method-expired")
```

Static text checks are acceptable only when the exact text is the product surface or compatibility contract.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The request mentioned this field/name." | User wording is input, not the domain model. |
| "This keeps us future-proof." | Future-proofing without a current consumer is speculative generality. |
| "The structure enforces consistency." | Consistent unused structure is still unused structure. |
| "The tests cover every requested item." | Shape coverage is not behavioral confidence. |
| "The deadline requires proving completeness." | Completeness means the system works through its surface, not that every requested heading or field exists. |
| "It documents intent." | Documentation belongs in docs/comments unless the program consumes it. |
| "It is harmless." | Extra schema fields, fixtures, tests, and names create drift and maintenance cost. |
| "It is a public or internal contract." | Keep it only if a current consumer exists; otherwise it is speculative contract design. |
| "It will support future behavior." | Write the behavior first, then the schema/test that protects it. Future scaffolding is still future-proofing. |
| "It helps observability or debugging." | Observability data must be emitted, queried, or displayed. Undisplayed metadata is not observability. |

## Red Flags

- Adding an element because the request used a phrase.
- Naming artifacts after a requestor, team, UI step, deployment platform, or current vendor instead of domain behavior.
- Creating metadata and immediately describing it as “advisory,” “informational,” or “non-enforcing.”
- Writing tests that would pass with hardcoded fixtures.
- Manually repeating the same concept across multiple representations instead of deriving one from another.
- Naming artifacts after conversation or process vocabulary rather than domain concepts.
- Adding one-line helpers because a checklist said to provide them.
- Calling unused fields contracts, scaffolding, debugging data, or observability without showing the consumer.

When a red flag appears, stop and re-enumerate: what real runtime behavior is this artifact supposed to serve?
