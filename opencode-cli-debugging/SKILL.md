---
name: opencode-cli-debugging
description: Use when debugging OpenCode CLI integrations, plugin development, provider wiring, tool execution failures, or session-level behavior and you need reproducible non-interactive runs, boundary tracing, and safe alternatives to opening the interactive TUI.
license: CC-BY-NC-4.0
---

# OpenCode CLI Debugging

## Overview

Use this skill to debug OpenCode behavior **as a real client**, not just with unit fixtures. The core principle is: **separate CLI behavior, proxy/API behavior, and model behavior into different boundaries and prove which one is failing before you change code**.

This skill is especially useful for plugin development, model/provider adapters, proxy debugging, tool-call failures, empty assistant continuations, and agent-loop issues.

## When to Use

- Real OpenCode runs behave differently from your unit tests
- Tool calls appear in the UI but fail, hang, or use the wrong tool name
- Assistant continuation is empty after a tool result
- You need to inspect a specific OpenCode session or replay a real request
- You are tempted to open the TUI, but the environment cannot drive it safely

Do **not** use this skill when a plain unit test failure already isolates the bug with no CLI/runtime uncertainty.

## Core Workflow

1. **Pick the right reproduction surface**
    - Prefer **non-interactive** commands such as `opencode run --format json ...` when you need real-client behavior.
    - Avoid launching the generic interactive TUI if your environment cannot control it safely.
    - If even `opencode` is constrained, use session tools, HTTP probes, logs, tmux-managed background processes, and direct provider calls instead.

2. **Freeze the runtime you are testing**
    - Start or restart the target proxy/service cleanly.
    - Health-check the exact endpoint the CLI will hit.
    - Verify the model name the CLI actually exposes; advertised models may differ from what your server supports internally.

3. **Trace one boundary at a time**
    - CLI event stream / session output
    - OpenAI-compatible request/response surface
    - SDK prompt/message normalization
    - Upstream provider request/response
    - Tool execution / tool result replay

4. **Capture evidence from a real failing case**
    - Save the exact command, session id, model id, and request payload shape.
    - For tool issues, record separately:
        - tool id
        - tool name
        - tool arguments
        - tool result shape
        - follow-up assistant continuation

5. **Only then patch the narrowest boundary**
    - Wrong tool name on first response → response parsing / tool-name recovery
    - Wrong tool name on replay turn → history serialization boundary
    - Tool result accepted but no continuation → message-shape conversion or loop boundary
    - Empty assistant text with no errors → inspect the follow-up request payload before changing generation logic

## Safe Reproduction Patterns

### 1. Real CLI run without opening TUI

```bash
opencode run --model <provider/model> --format json --dangerously-skip-permissions "<prompt>"
```

Use this when you need the real OpenCode request path, tool registry, and event stream.

### 2. Session inspection

- Read metadata first (`session_info`)
- Then read visible messages (`session_read`)
- Compare what the user saw with what your proxy claims happened

This is ideal when a user gives you a session id and says “the tool ran but nothing came back.”

### 3. tmux-managed background service

Use tmux for long-running proxies or local providers so you can restart and inspect them deterministically.

Typical loop:

```bash
tmux kill-session -t my-proxy
tmux new-session -d -s my-proxy "node server.js"
curl http://127.0.0.1:8081/v1/models
```

## Boundary Debugging Checklist

### Tool-name failures

Ask these in order:

1. Is the **tool id** synthetic (`toolu_*`, `call_*`, etc.)? That is usually normal.
2. Is the **tool name** also synthetic? That is usually wrong.
3. Is the failure on:
    - the first model response, or
    - a replayed follow-up turn after tool history is fed back in?

Those are different bugs and often live at different boundaries.

### Empty continuation after tool result

Check:

1. Did the tool actually execute?
2. Did the proxy receive the tool result back?
3. Did the follow-up request include the assistant tool call **and** the tool result in the right schema?
4. Did the model return assistant text, another tool call, or nothing?

If you cannot answer all four with evidence, you are not ready to patch.

## Patterns That Worked Repeatedly

### Distinguish first-turn bugs from replay-turn bugs

The same visible symptom can come from different causes:

- **First-turn bug:** provider returns a synthetic tool id/name combination and your parser recovers the wrong tool name.
- **Replay-turn bug:** the first turn succeeded, but when you serialize assistant/tool history back upstream, you accidentally turn the synthetic id into the tool name.

Treat these as separate regressions.

### Verify the client-facing contract, not just internal types

A proxy can be internally “correct” yet still break the CLI if it changes:

- finish reason spelling (`tool-calls` vs `tool_calls`)
- tool name vs tool id placement
- streaming chunk shape
- second-turn message schema

Always verify the actual CLI-visible contract.

### Add one regression per boundary

Good examples:

- first response with synthetic id but correct recovered tool name
- replay turn with synthetic call id but real tool name preserved upstream
- tool result round trip producing a normal assistant continuation

## Common Mistakes

### Opening the TUI when you cannot drive it

If your environment cannot operate the TUI, this is not debugging — it is deadlocking yourself.

Use non-interactive runs, sessions, HTTP probes, tmux, and local process inspection instead.

### Treating tool ids as tool names

Provider-generated ids are often synthetic. Real tool names must match the registered tools the client actually knows about.

### Trusting fixtures too early

Fixtures are excellent for regression coverage, but they do not prove real-client behavior. Reproduce once with the actual CLI path before concluding the bug is fixed.

### Debugging only one layer

If you only inspect the provider response or only inspect the CLI output, you miss the transformation layer where most integration bugs live.

## Quick Reference

| Problem | First place to inspect |
| ------- | ---------------------- |
| Wrong/unavailable tool name | Provider response parsing and replay serialization |
| Tool ran but assistant stayed empty | Follow-up message schema and agent-loop boundary |
| Unit tests pass but OpenCode fails | Real `opencode run --format json` path |
| Session looks incomplete | `session_info` + `session_read` + proxy logs |
| Proxy changes not taking effect | Clean restart + health check |

## Minimal Success Criteria

Do not call the bug fixed until you have all of these:

1. A real or realistic reproduction case
2. A precise failing boundary
3. A regression test for that boundary
4. A fresh runtime verification against the real surface the user cares about
