---
name: developing-agents
description: Use when implementing or debugging agents, tool-calling assistants, or multi-step AI runtimes and you need correct message conversion, tool-call handling, replayed history, loop continuation, or provider/API compatibility across SDKs, proxies, or direct upstream model APIs.
license: CC-BY-NC-4.0
---

# Developing Agents

## Overview

Use this skill when the hard part is no longer “call a model once”, but “make the runtime behave correctly across multiple steps”.

Focus on implementation, not product design. The main failure modes are almost always:

- wrong message schema at one boundary
- tool call parses correctly but tool result is replayed incorrectly
- first turn works but second turn fails
- streaming and non-streaming behave differently
- provider/proxy compatibility is only partial

## When to Use

- You are implementing tool calling, agent loops, chat replay, memory replay, or resumed sessions
- A model can call tools, but the assistant does not continue correctly after tool execution
- OpenAI-compatible or Anthropic-compatible payloads work for one turn but break on later turns
- An SDK, adapter, or proxy accepts the request but rejects replayed history
- Tool ids, tool names, tool inputs, or tool results are getting mixed up
- Streaming output looks correct, but the final persisted history is wrong

Do **not** use this skill for simple one-shot text generation with no tools and no replayed history.

## API Format References

Before writing adapters, read the actual provider message format instead of guessing.

### OpenAI-style APIs

- Typical shape: chat-completions/messages or responses-style structured input
- Common concepts: `messages`, assistant `tool_calls`, tool messages with `tool_call_id`, streaming deltas, finish reasons
- Docs:
    - https://platform.openai.com/docs/api-reference/chat
    - https://platform.openai.com/docs/guides/function-calling

### Anthropic-style APIs

- Typical shape: `messages` array with structured content blocks
- Common concepts: `tool_use`, `tool_result`, block-based content instead of plain assistant/tool strings
- Docs:
    - https://docs.anthropic.com/en/api/messages
    - https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/overview

### Practical rule

“OpenAI-compatible” and “Anthropic-compatible” usually mean **partially compatible until proven otherwise**. Verify the exact fields you depend on:

- assistant tool-call shape
- tool result shape
- streaming shape
- finish reasons
- replay/resume behavior

## Core Invariants

Keep these separate at all times:

- **tool id**: synthetic runtime/provider identifier
- **tool name**: real executable capability
- **tool input**: structured invocation payload
- **tool output**: result returned after execution

If any of these collapse into one field, replay turns will eventually break.

Also preserve this invariant:

> A tool result is not just something to display. It is input to the next model step.

## Implementation Rules

### 1. Normalize messages at the boundary

Do not pass user-facing message shapes directly into the SDK unless the SDK explicitly accepts them.

You usually need explicit conversion for:

- assistant tool calls
- tool result messages
- content arrays vs plain strings
- provider-specific content blocks

Good:

- client/API messages → runtime/SDK messages
- runtime/SDK messages → upstream/provider messages

Bad:

- letting half-normalized objects leak through multiple layers
- assuming OpenAI-style history is already valid SDK history
- assuming SDK-native messages are already valid upstream payloads

### 2. Treat first turn and replay turn as different test targets

A tool integration is not correct when the model requests a tool once.

It is correct when this full path works:

1. model requests tool
2. runtime executes tool
3. runtime records tool result
4. tool call + tool result are replayed into the next step
5. assistant continues correctly

### 3. Keep a canonical tool registry

Your runtime should have one authoritative source for:

- tool name
- input schema
- execution function
- permission/approval policy if any

Everything else should map back to that registry.

### 4. Preserve message meaning across every conversion

If your system has more than one boundary, verify all of these conversions explicitly:

- client request → runtime message
- runtime message → SDK message
- SDK message → upstream payload
- upstream response → SDK/runtime output
- runtime output → client-visible response

Fixing one direction does not fix the others.

### 5. Streaming must preserve the same semantics as non-streaming

Streaming and non-streaming should agree on:

- tool name
- tool id
- tool input
- finish reason
- replayable history after completion

If the streamed path emits logically different events than the non-streamed path, you have two incompatible runtimes.

## Tool-Call Loop Pattern

The minimal correct loop is:

1. Normalize incoming history into runtime/SDK messages
2. Call the model
3. If there are no tool calls, return assistant output
4. If there are tool calls:
    - validate tool names against the canonical registry
    - execute the tools
    - append tool results in the runtime’s canonical message format
    - call the model again with replayed history
5. Repeat until a real stop condition is reached

Good stop conditions:

- assistant returns normal content with no tool calls
- configured max-step limit reached
- approval required but not granted
- unrecoverable invalid tool call

Bad stop conditions:

- “a tool ran, so we’re done”
- “the provider returned something parseable”
- “the first chunk looked fine”

## Replay and History Rules

Replay bugs are the most common real-world agent bugs.

Always distinguish:

- **client-visible history**
- **runtime history**
- **provider/upstream history**

These are often related, but they are not always identical.

For replayed turns, verify all of these:

- prior assistant tool call still exists
- tool result still points to the correct tool call id
- real tool name is preserved
- tool result is encoded in the schema expected by the next boundary
- the next model step is actually triggered

## Message Conversion Advice

### OpenAI-style history

Typical conversions you may need:

- assistant `tool_calls` → SDK tool-call parts
- tool messages with `tool_call_id` → SDK tool-result parts
- content arrays → text parts or strings depending on the boundary

### Anthropic-style history

Typical conversions you may need:

- `tool_use` blocks → runtime tool-call records
- `tool_result` blocks → runtime tool-result records
- block arrays → client-facing messages or provider-specific replay payloads

### Practical rule

Never convert by field name alone. Convert by **meaning**:

- what is assistant text?
- what is a tool request?
- what is a tool result?
- what must be replayed next turn?

## Debugging Workflow

When tool calling or continuation breaks, inspect these in order:

1. raw client request/history
2. normalized runtime/SDK messages
3. actual upstream payload
4. actual upstream response
5. parsed runtime response
6. replayed follow-up payload
7. final user-visible output

At each step, ask:

- Is the tool name still the real executable name?
- Is the tool id still matched correctly?
- Is the tool input still valid for this boundary?
- Is the tool output still valid for this boundary?
- Should the loop continue from here?

Do not jump directly from final symptom to upstream blame.

## Verification Matrix

Before claiming the integration works, verify all of these:

| Case                      | Must pass                                   |
| ------------------------- | ------------------------------------------- |
| Plain text, no tools      | Assistant returns normal output             |
| First-turn tool call      | Correct tool name, id, and input            |
| Tool result replay        | Assistant continues correctly on next turn  |
| Resumed/persisted history | Replayed tool call + result still work      |
| Streaming tool path       | Same semantics as non-streaming             |
| Tool error path           | Correct error propagation and loop behavior |
| Multi-tool environment    | No synthetic id/name confusion              |

If you only verified first-turn tool calling, you did not verify the agent.

## Quick Reference

If the symptom is:

- **tool called, no continuation** → inspect replayed tool result message shape
- **unavailable synthetic tool name** → inspect tool id vs tool name mapping
- **first turn works, second turn fails** → inspect replay normalization
- **stream works, saved history breaks** → compare streamed events vs persisted messages
- **provider says invalid prompt** → inspect SDK-facing message schema, not just client payload

Common pitfalls from real debugging:

- **SDK accepts first turn but rejects resumed history** → you probably normalized user text but not assistant `tool_calls` and `tool` result messages into SDK-native message parts
- **Tool execution succeeds but final answer is empty** → the loop often stopped because the tool result was displayed but not replayed as input to the next model step
- **Synthetic ids like `toolu_*` leak into the tool name field** → keep `toolCallId` and `toolName` separate; recover the real tool name from the registry or schema match, not from display output
- **Replay turn breaks only in multi-tool environments** → single-tool heuristics hide bugs; verify name recovery when several tools are advertised
- **Targeted tests hang or mislead** → use direct runtime probes when the test runner itself is ambiguous, but still keep a regression test for the confirmed bug

## Common Mistakes

- Treating agent bugs as “model bugs” before checking message conversion
- Testing only first-turn tool requests
- Reconstructing tool names from ids when a canonical registry already exists
- Replaying tool results as display strings when the next boundary expects structured parts
- Assuming OpenAI-compatible or Anthropic-compatible means fully compatible
- Writing “agent development” guidance that focuses on architecture slogans instead of concrete runtime behavior

## Minimal Acceptance Criteria

Do not consider the implementation correct unless all of these are true:

- first-turn tool calls work
- second-turn after tool result works
- at least one replay/resume case works
- streaming and non-streaming agree on semantics
- tool ids and tool names remain distinct throughout the system
- there is a single canonical tool registry
