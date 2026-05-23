---
name: opencode-cli
description: opencode CLI (anomalyco/opencode) comprehensive reference for the TUI, headless server, ACP server, web UI, sessions, agents, MCP servers, providers/auth, models, plugins, GitHub agent integration, stats, export/import, and debugging utilities from the command line. Includes guidance on which commands launch blocking interactive UIs and how to avoid them in scripted contexts.
license: CC-BY-NC-4.0
---

# opencode CLI

Comprehensive reference for the `opencode` CLI (anomalyco/opencode) - the open source AI coding agent.

**Version:** 1.15.10 (current as of May 2026)
**Source:** https://github.com/anomalyco/opencode
**Docs:** https://opencode.ai/docs

## Interactive vs Non-Interactive Commands

Some `opencode` commands launch a full-screen TUI or another long-running interactive process and will **block the terminal** until the user exits. That is the intended behavior for human use - it only becomes a problem when an automated agent runs them without a human at the keyboard.

Commands that block on an interactive UI / long-running process:

- `opencode` (no subcommand) and `opencode <project-path>` - full-screen TUI
- `opencode attach <url>` - remote TUI
- `opencode serve`, `opencode web`, `opencode acp` - long-running servers
- `opencode mcp add`, `opencode mcp auth <name>` - interactive prompts
- `opencode providers login` (alias `opencode auth login`) - interactive picker / browser
- `opencode github install` - interactive setup wizard
- `opencode db` (no query) - opens an interactive `sqlite3` shell
- `opencode debug wait` - blocks indefinitely

If you are running unattended (e.g. inside another agent, in CI, or in a scripted shell), prefer the non-interactive equivalents:

- Use `opencode run "<prompt>"` instead of the bare `opencode` TUI for one-shot prompts
- Pass `--format json` to `opencode run` and `opencode session list` for machine-readable output
- Pass an explicit query to `opencode db "SELECT ..."` instead of opening the bare shell
- Drive `opencode serve` from a process supervisor; don't shell out to it from a single-shot script

If you are an interactive user (or a tool/skill that knows how to drive a TUI - e.g. a terminal-multiplexer wrapper, an ACP-aware editor, an expect-style harness), running the interactive commands directly is fine and supported.

When in doubt about an unfamiliar subcommand, run `opencode <subcommand> --help` first - the help text exits cleanly and tells you whether the command is interactive.

## Prerequisites

### Installation

```bash
# YOLO install (auto-detects platform, respects $OPENCODE_INSTALL_DIR / $XDG_BIN_DIR)
curl -fsSL https://opencode.ai/install | bash

# macOS / Linux (Homebrew tap, recommended - always up to date)
brew install anomalyco/tap/opencode

# macOS / Linux (official Homebrew formula)
brew install opencode

# npm / bun / pnpm / yarn
npm i -g opencode-ai@latest

# Windows
scoop install opencode
choco install opencode

# Arch Linux
sudo pacman -S opencode      # stable
paru -S opencode-bin         # AUR latest

# Cross-platform version manager
mise use -g opencode

# Nix
nix run nixpkgs#opencode
nix run github:anomalyco/opencode   # latest dev branch

# Verify installation
opencode --version
```

### Authentication

```bash
# Interactive provider login (opens browser or prompts for API key)
opencode auth login

# Login to a specific provider, skipping the picker
opencode auth login -p anthropic
opencode auth login -p openai -m "API Key"

# List configured providers and credentials
opencode auth list
opencode auth ls

# Log out
opencode auth logout

# `providers` is the canonical name; `auth` is an alias
opencode providers list
opencode providers login
opencode providers logout
```

Credentials are stored at `~/.local/share/opencode/auth.json`. Provider env vars (e.g. `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GITHUB_TOKEN`, AWS / Azure / Vertex creds) are auto-detected when set.

### Showing Paths and Debug Info

```bash
# Show all opencode XDG paths (data, config, cache, state, log, repos, tmp)
opencode debug paths

# Show version, OS, terminal, and enabled plugins
opencode debug info
```

## CLI Structure

```
opencode                     # Root - start TUI in cwd (interactive)
├── [project]                # Start TUI in given project path (interactive)
├── run                      # One-shot prompt (non-interactive)
├── attach <url>             # Connect TUI to a running server (interactive)
├── serve                    # Headless HTTP server (long-running)
├── web                      # Server + browser UI (long-running)
├── acp                      # Agent Client Protocol (stdio JSON-RPC)
├── completion               # Print shell completion script
├── upgrade [target]         # Upgrade opencode binary
├── uninstall                # Uninstall opencode
├── providers (alias: auth)  # AI providers / credentials
│   ├── list (ls)
│   ├── login [url]
│   └── logout
├── agent                    # Manage agents
│   ├── create
│   └── list
├── mcp                      # MCP servers
│   ├── add
│   ├── list (ls)
│   ├── auth [name]
│   │   └── list (ls)
│   ├── logout [name]
│   └── debug <name>
├── plugin <module>          # Install plugin (alias: plug)
├── models [provider]        # List models
├── session                  # Sessions
│   ├── list
│   └── delete <sessionID>
├── export [sessionID]       # Export session as JSON
├── import <file>            # Import session JSON / share URL
├── stats                    # Token usage and cost
├── github                   # GitHub agent integration
│   ├── install
│   └── run
├── pr <number>              # Checkout GitHub PR + run opencode
├── db                       # SQLite database tools
│   ├── [query]
│   ├── path
│   └── migrate
└── debug                    # Diagnostics
    ├── config
    ├── info
    ├── paths
    ├── startup
    ├── agent <name>
    ├── skill
    ├── scrap
    ├── v2
    ├── snapshot { track, patch, diff }
    ├── lsp { diagnostics, symbols, document-symbols }
    ├── rg { tree, files, search }
    ├── file { read, status, list, search, tree }
    └── wait
```

## Configuration

### Config Files and Resolution Order

Config is **deep-merged** in this order (later wins):

1. Remote config from `<authdomain>/.well-known/opencode` (org defaults, fetched at auth time)
2. Global: `~/.config/opencode/opencode.json`
3. `OPENCODE_CONFIG` env var path
4. Project: `opencode.json` (or `opencode.jsonc`) found by walking up from cwd to the nearest git root
5. `.opencode/` subdirs in project: `agents/`, `commands/`, `modes/`, `plugins/`, `skills/`, `tools/`, `themes/`
6. `OPENCODE_CONFIG_CONTENT` env var (inline JSON, runtime override)
7. Managed: `/Library/Application Support/opencode/` (macOS), `/etc/opencode/` (Linux), `%ProgramData%\opencode` (Windows)
8. macOS managed prefs (`ai.opencode.managed`, MDM `.mobileconfig` - highest, not user-overridable)

JSON Schema URL: `https://opencode.ai/config.json` (draft 2020-12). Config files support **JSONC** (comments + trailing commas).

### Show Resolved Config

```bash
# Print fully resolved configuration (after merge)
opencode debug config
```

### Minimal opencode.json

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "anthropic/claude-haiku-4-5",
  "default_agent": "build",
  "logLevel": "INFO",
  "share": "manual",
  "instructions": ["AGENTS.md", "docs/*.md"],
  "permission": { "edit": "ask", "bash": "ask" }
}
```

### Variable Substitution in Config

```jsonc
{
  "model": "{env:OPENCODE_MODEL}",
  "mcp": {
    "github": {
      "type": "remote",
      "url": "https://api.githubcopilot.com/mcp",
      "headers": { "Authorization": "Bearer {env:GITHUB_TOKEN}" }
    }
  },
  "instructions": ["{file:./AGENTS.md}"]
}
```

`{env:VAR}` reads an env var. `{file:path}` inlines file contents (relative to the config dir, or absolute when starting with `/` or `~`).

### TUI Config

The TUI keeps its own file, schema `https://opencode.ai/tui.json`:

- Default: `~/.config/opencode/tui.json`
- Override: `OPENCODE_TUI_CONFIG` env var

Keys: `theme`, `keybinds`, `scroll_speed`, `scroll_acceleration`, `diff_style`, `mouse`, `leader_timeout`, `attention`. Legacy `theme` / `keybinds` / `tui` keys inside `opencode.json` are auto-migrated.

### Environment Variables

```bash
# Config file overrides
export OPENCODE_CONFIG=/path/to/custom-opencode.json
export OPENCODE_CONFIG_DIR=/path/to/config-dir   # changes resolution of agents/commands/modes/plugins/skills
export OPENCODE_CONFIG_CONTENT='{"model":"anthropic/claude-sonnet-4-5"}'
export OPENCODE_TUI_CONFIG=/path/to/tui.json

# Behavior toggles (set to '1' / 'true' to enable)
export OPENCODE_AUTO_SHARE=1
export OPENCODE_DISABLE_AUTOUPDATE=1
export OPENCODE_DISABLE_AUTOCOMPACT=1
export OPENCODE_DISABLE_PRUNE=1
export OPENCODE_DISABLE_TERMINAL_TITLE=1
export OPENCODE_DISABLE_DEFAULT_PLUGINS=1
export OPENCODE_DISABLE_LSP_DOWNLOAD=1
export OPENCODE_DISABLE_MODELS_FETCH=1
export OPENCODE_DISABLE_MOUSE=1
export OPENCODE_ENABLE_EXPERIMENTAL_MODELS=1

# Claude Code interop
export OPENCODE_DISABLE_CLAUDE_CODE=1            # ignore .claude entirely
export OPENCODE_DISABLE_CLAUDE_CODE_PROMPT=1     # skip ~/.claude/CLAUDE.md
export OPENCODE_DISABLE_CLAUDE_CODE_SKILLS=1     # skip .claude/skills

# Permissions (inline JSON, layered over config)
export OPENCODE_PERMISSION='{"edit":"ask","bash":{"npm install":"allow","*":"ask"}}'

# Server auth (basic auth for `serve` / `web` / `attach`)
export OPENCODE_SERVER_PASSWORD='supersecret'
export OPENCODE_SERVER_USERNAME='opencode'

# Misc
export OPENCODE_GIT_BASH_PATH='C:\Program Files\Git\bin\bash.exe'
export OPENCODE_CLIENT='cli'                     # client identifier
export OPENCODE_MODELS_URL=https://models.dev    # custom models registry
export OPENCODE_DEV_DEBUG=true                   # writes to .opencode/debug.log

# Experimental flags (subject to change)
export OPENCODE_EXPERIMENTAL=1                   # master toggle
export OPENCODE_EXPERIMENTAL_FILEWATCHER=1
export OPENCODE_EXPERIMENTAL_LSP_TOOL=1
export OPENCODE_EXPERIMENTAL_LSP_TY=1            # Python ty
export OPENCODE_EXPERIMENTAL_PLAN_MODE=1
export OPENCODE_EXPERIMENTAL_BASH_DEFAULT_TIMEOUT_MS=120000
export OPENCODE_EXPERIMENTAL_OUTPUT_TOKEN_MAX=8192

# Provider keys (auto-detected, no opencode-specific name needed)
export ANTHROPIC_API_KEY=...
export OPENAI_API_KEY=...
export GEMINI_API_KEY=...
export GROQ_API_KEY=...
export OPENROUTER_API_KEY=...
export XAI_API_KEY=...
export GITHUB_TOKEN=...
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_REGION=...
export AZURE_OPENAI_ENDPOINT=...
export AZURE_OPENAI_API_KEY=...
export VERTEXAI_PROJECT=...
export VERTEXAI_LOCATION=...

# Proxy (always include localhost in NO_PROXY to avoid loops with the local server)
export HTTPS_PROXY=http://proxy.corp:8080
export HTTP_PROXY=http://proxy.corp:8080
export NO_PROXY=localhost,127.0.0.1,.corp
export NODE_EXTRA_CA_CERTS=/etc/ssl/corp-ca.pem
```

There is **no** `OPENCODE_LOG_LEVEL`, `OPENCODE_THEME`, `OPENCODE_DISABLE_SHARE`, or `OPENCODE_MODEL` env var. Use `--log-level` (CLI) or `logLevel` (config) for logging, the `theme` key in `tui.json` for theming, and `"share": "disabled"` in `opencode.json` to disable sharing.

## Running opencode (TUI / one-shot)

The bare `opencode` command launches the TUI and **blocks the terminal**. Pass a project path positional to start in a different directory.

### Start TUI

```bash
# Start TUI in current directory (interactive - use `opencode run` for scripts)
opencode

# Start TUI in a specific project path
opencode /path/to/project

# Start TUI with a specific model and agent
opencode -m anthropic/claude-sonnet-4-5 --agent build

# Resume the most recent session
opencode -c
opencode --continue

# Resume a specific session by id
opencode -s ses_abc123

# Fork an existing session (requires --continue or --session)
opencode -c --fork

# Start TUI with an initial prompt
opencode --prompt "Refactor authentication module"

# Run without external plugins (for debugging plugin issues)
opencode --pure

# Bind the embedded server to a custom port / host
opencode --port 4096 --hostname 127.0.0.1

# Enable mDNS discovery (so other clients can find this instance)
opencode --mdns --mdns-domain opencode.local

# Allow extra origins for the embedded server CORS
opencode --cors http://localhost:3000 --cors http://localhost:5173

# Print server logs to terminal
opencode --print-logs --log-level DEBUG
```

## Run (non-interactive prompts)

Use `opencode run` to send a single prompt and exit. This is the right entry point for scripts and CI.

### Basic Run

```bash
# One-shot prompt
opencode run "Write a Python script that lists S3 buckets"

# Multi-word message (positional message array)
opencode run "Add error handling" "to src/api/server.ts"

# Pick model / agent / reasoning variant
opencode run "Audit this file" -m anthropic/claude-opus-4-5
opencode run "Plan a refactor" --agent plan
opencode run "Solve this puzzle" --variant high      # high / max / minimal

# Show provider thinking blocks in the output
opencode run "Plan it" --thinking

# Auto-share the resulting session
opencode run "Triage build errors" --share

# Set a session title (defaults to truncated prompt)
opencode run "Land issue #123" --title "Issue 123 fix"

# Run a custom slash command instead of free-form prompt
opencode run --command /test "src/api/server.test.ts"
```

### Continue / Resume Sessions

```bash
# Continue the most recent session
opencode run -c "Now write tests for that"

# Continue a specific session
opencode run -s ses_abc123 "Now write tests"

# Fork the session (branch off without modifying the original)
opencode run -c --fork "Try a different approach"
```

### Attach Files to a Run

```bash
# Attach one or more files (-f repeatable)
opencode run "Review this code" -f src/server.ts -f src/server.test.ts

# Attach an image (auto-resized per attachment.image config)
opencode run "Describe this screenshot" -f screenshot.png
```

### Output Format

```bash
# Default: pretty terminal output
opencode run "ping"

# Raw JSON event stream (one event per line - good for scripting)
opencode run "ping" --format json

# Pipe JSON events to jq
opencode run "ping" --format json | jq -c 'select(.type=="message")'

# Capture full session as JSON via export afterwards
opencode run "ping" --format json
opencode export | jq .
```

### Run Against a Remote Server

```bash
# Send the prompt to a running `opencode serve` instance instead of starting a local server
opencode run "Help" --attach http://localhost:4096

# With basic auth (defaults pulled from OPENCODE_SERVER_USERNAME / OPENCODE_SERVER_PASSWORD)
opencode run "Help" --attach http://server:4096 -u opencode -p hunter2

# Pass --dir to set the working directory on the remote server
opencode run "Refactor" --attach http://server:4096 --dir /srv/project
```

### Risk-Sensitive Flags

```bash
# Auto-approve all permissions that haven't been explicitly declined
# (DANGEROUS - lets the agent edit files / run commands without prompting)
opencode run "Apply the patch" --dangerously-skip-permissions

# Direct interactive split-footer mode (combines run + interactive prompt)
opencode run -i "Start a chat about this codebase"

# Demo mode (lets you run interactive demo slash commands)
opencode run --demo /tutorial
```

## Attach (remote TUI)

Connect a local TUI to a running `opencode serve` / `opencode web` instance.

```bash
# Attach to a remote server (interactive - launches TUI)
opencode attach http://localhost:4096

# With basic auth (defaults from OPENCODE_SERVER_USERNAME / OPENCODE_SERVER_PASSWORD)
opencode attach http://server:4096 -u opencode -p hunter2

# Set the working directory used on the server
opencode attach http://server:4096 --dir /srv/project

# Resume a session on the remote server
opencode attach http://server:4096 -c
opencode attach http://server:4096 -s ses_abc123

# Fork the session on the remote server
opencode attach http://server:4096 -c --fork
```

## Serve (headless server)

Run a headless HTTP server. Other clients (TUI, web UI, IDE plugins) can connect.

```bash
# Start headless server on default port (4096) and 127.0.0.1 (long-running)
opencode serve

# Bind to all interfaces on a specific port
opencode serve --hostname 0.0.0.0 --port 4096

# Enable mDNS discovery so peers on the LAN can find it
opencode serve --mdns --mdns-domain dev.opencode.local

# Allow additional CORS origins
opencode serve --cors http://localhost:3000 --cors http://localhost:5173

# Add basic auth (any client must send these creds)
OPENCODE_SERVER_PASSWORD='hunter2' opencode serve --hostname 0.0.0.0 --port 4096
```

The OpenAPI 3.1 schema is exposed at `http://<host>:<port>/doc`. SSE event stream lives at `/event` (first event is `server.connected`).

## Web (server + browser UI)

```bash
# Start server and open the web UI in the default browser (long-running)
opencode web

# On a fixed port / host
opencode web --port 4097 --hostname 127.0.0.1

# Enable mDNS, custom CORS
opencode web --mdns --cors http://localhost:3000
```

## ACP (Agent Client Protocol)

`opencode acp` exposes opencode as an ACP subprocess speaking JSON-RPC over stdin/stdout. Used by Zed, JetBrains, Avante.nvim, CodeCompanion.nvim, etc.

```bash
# Start ACP server (typically launched by an IDE, not directly)
opencode acp

# Bind embedded server side-channel to a port (rarely needed)
opencode acp --port 4096 --hostname 127.0.0.1

# Set the working directory the ACP session runs in
opencode acp --cwd /path/to/project

# mDNS / CORS for the side-channel server
opencode acp --mdns --mdns-domain opencode.local --cors http://localhost:3000
```

ACP supports tools, MCP servers, AGENTS.md, formatters, and the permission system. `/undo` and `/redo` slash commands are not supported in ACP mode.

## Agents

Agents combine a system prompt, model, mode, and permissions. They live as Markdown files with YAML frontmatter under `~/.config/opencode/agents/` (global) or `.opencode/agents/` (project). Filename = agent name.

### Manage Agents

```bash
# List all agents (built-in + custom)
opencode agent list

# Inspect an agent's resolved config (model, permissions, prompt)
opencode debug agent build
opencode debug agent plan
opencode debug agent <my-agent>

# Execute a single tool in an agent's context (debug)
opencode debug agent build --tool read --params '{"file":"package.json"}'
opencode debug agent build --tool grep --params '{"pattern":"TODO","include":"*.ts"}'

# Create a new agent file (the bare command launches an interactive form;
# pass all flags non-interactively)
opencode agent create \
  --path ~/.config/opencode/agents \
  --description "Code reviewer that focuses on security" \
  --mode subagent \
  --permissions read,grep,glob,webfetch \
  -m anthropic/claude-sonnet-4-5
```

`--permissions` (alias `--tools`) is comma-separated; valid tool ids: `bash`, `read`, `edit`, `glob`, `grep`, `webfetch`, `task`, `todowrite`, `websearch`, `lsp`, `skill`. Default is `all`.

### Agent File Format

```markdown
---
description: "Security-focused code reviewer"
mode: subagent              # primary | subagent | all
model: anthropic/claude-sonnet-4-5
temperature: 0.1
top_p: 0.9
steps: 10                   # max iterations
color: "#ff6b6b"            # hex or theme color name
hidden: true                # subagent only - hide from @ autocomplete
disable: false
permission:
  edit: deny
  bash: ask
  webfetch: allow
prompt: |
  You are a senior code reviewer focused on security.
  Audit changes for SQL injection, XSS, and auth bypass.
---

Optional extra prompt body in Markdown after the frontmatter.
```

Built-ins:

- `build` - primary, full toolset, default agent
- `plan` - primary, read-only; asks for edits and bash
- `general` - subagent, full toolset
- `explore` - subagent, read-only
- `scout` - subagent, external research

Unknown frontmatter keys pass through as model options (e.g. `reasoningEffort: high`, `textVerbosity: low`).

## MCP Servers (Model Context Protocol)

opencode reads MCP server config from the `mcp` key in `opencode.json`. Tools from a server are namespaced as `<server-name>_<tool>`.

### Manage MCP Servers

```bash
# List configured servers + connection status (renders a Unicode table)
opencode mcp list
opencode mcp ls

# Add an MCP server (interactive - drive only from a real terminal)
opencode mcp add

# Authenticate with an OAuth-enabled remote MCP server (INTERACTIVE)
opencode mcp auth my-mcp

# List OAuth-capable servers and their auth status
opencode mcp auth list
opencode mcp auth ls

# Remove stored OAuth credentials for a server
opencode mcp logout my-mcp

# Debug an OAuth connection (handshake + token introspection)
opencode mcp debug my-mcp
```

OAuth tokens are cached at `~/.local/share/opencode/mcp-auth.json`. opencode auto-detects OAuth on a `401` response (RFC 7591 Dynamic Client Registration). Set `"oauth": false` on the server to disable.

### MCP Config (in opencode.json)

```jsonc
{
  "mcp": {
    // Local stdio server
    "filesystem": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-filesystem", "/srv"],
      "enabled": true,
      "environment": { "LOG_LEVEL": "info" },
      "timeout": 5000
    },

    // Remote HTTP / SSE server
    "github": {
      "type": "remote",
      "url": "https://api.githubcopilot.com/mcp",
      "enabled": true,
      "headers": { "Authorization": "Bearer {env:GITHUB_TOKEN}" },
      "timeout": 5000
    },

    // Remote with explicit OAuth dynamic registration
    "linear": {
      "type": "remote",
      "url": "https://mcp.linear.app/sse",
      "enabled": true,
      "oauth": {
        "clientId": "{env:LINEAR_CLIENT_ID}",
        "clientSecret": "{env:LINEAR_CLIENT_SECRET}",
        "scope": "read write",
        "callbackPort": 19876,
        "redirectUri": "http://127.0.0.1:19876/mcp/oauth/callback"
      }
    }
  },

  // Restrict which MCP tools agents can call
  "tools": { "github_*": true, "filesystem_delete*": false }
}
```

## Models

```bash
# List all available models (provider/model-id, one per line)
opencode models

# List for a single provider
opencode models anthropic
opencode models openai

# Show extra metadata (cost, context, capabilities)
opencode models --verbose

# Refresh the models cache from models.dev
opencode models --refresh
```

Custom registry: set `OPENCODE_MODELS_URL` to a `models.dev`-compatible URL.

## Sessions

Sessions are persisted in opencode's SQLite database at `~/.local/share/opencode/opencode.db`.

### List / Delete Sessions

```bash
# List recent sessions (table - default)
opencode session list

# Newest 20, JSON output for scripting
opencode session list -n 20 --format json

# Pipe to jq
opencode session list --format json | jq '.[].id'

# Delete a session by id
opencode session delete ses_abc123
```

### Export / Import

```bash
# Export the most recent session to stdout as JSON
opencode export

# Export a specific session
opencode export ses_abc123 > session.json

# Sanitize sensitive transcript and file data before export
opencode export ses_abc123 --sanitize > session-redacted.json

# Import from a JSON file
opencode import ./session.json

# Import from a share URL
opencode import https://opncd.ai/s/abcd1234
```

### Stats

```bash
# All-time stats
opencode stats

# Last 30 days
opencode stats --days 30

# Top 10 tools by usage
opencode stats --tools 10

# Top 5 models by usage
opencode stats --models 5

# Filter to the current project
opencode stats --project ""

# Filter to a specific project path
opencode stats --project /path/to/project
```

## Plugins

opencode plugins are JS/TS modules implementing hooks (`tool.execute.before`, `session.created`, `permission.asked`, etc.) and optionally registering custom tools. Local plugins live under `.opencode/plugins/` (project) or `~/.config/opencode/plugins/` (global). npm plugins are listed in the `plugin` array in `opencode.json` and auto-installed via Bun (cached at `~/.cache/opencode/node_modules/`).

```bash
# Install an npm plugin and add it to project opencode.json
opencode plugin opencode-wakatime
opencode plug opencode-wakatime

# Install scoped npm plugin
opencode plugin "@example/opencode-notifier"

# Install in global config (~/.config/opencode/opencode.json)
opencode plugin opencode-wakatime --global
opencode plugin opencode-wakatime -g

# Replace an already-installed plugin (force update version)
opencode plugin opencode-wakatime --force
opencode plugin opencode-wakatime -f
```

### Plugin Skeleton

```typescript
// .opencode/plugins/my-plugin.ts
import type { Plugin } from "@opencode-ai/plugin"

export const MyPlugin: Plugin = async ({ project, directory, worktree, client, $ }) => ({
  "tool.execute.before": async ({ tool, args }) => {
    if (tool === "bash") console.log("[bash]", args.command)
  },
  "session.created": async ({ session }) => {
    await $`echo ${session.id} >> ~/sessions.log`
  },
})
```

If a plugin tool name matches a built-in, the plugin tool wins. For local plugins with deps, drop a `package.json` next to them - opencode runs `bun install` on startup.

## GitHub Agent

The `opencode github` subcommand wires up the opencode GitHub App (`github.com/apps/opencode-agent`) so PRs and issues can be driven by `/oc` / `/opencode` comments.

```bash
# Interactive: install the GitHub App, generate .github/workflows/opencode.yml,
# and walk through repo secrets configuration (drive only from a real terminal)
opencode github install

# Run the GitHub agent locally (used inside the action runner)
opencode github run --token github_pat_xxx
opencode github run --event ./mock-issue_comment.json --token github_pat_xxx
```

### Workflow Skeleton (manual setup)

```yaml
# .github/workflows/opencode.yml
name: opencode
on:
  issue_comment: { types: [created] }
  pull_request_review_comment: { types: [created] }

jobs:
  opencode:
    if: contains(github.event.comment.body, '/oc') || contains(github.event.comment.body, '/opencode')
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 1
          persist-credentials: false
      - uses: anomalyco/opencode/github@latest
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        with:
          model: anthropic/claude-sonnet-4-5
          # share: true
          # agent: build
          # prompt: "Review for security regressions"
```

Comment triggers in issues / PRs:

- `/oc explain this issue` - reads thread, replies inline
- `/oc fix this` - opens a branch + PR
- `/oc add error handling here` (on a PR review line) - auto-detects file, lines, and diff context

### Pull Request Workflow Helper

```bash
# Fetch a PR by number, check it out, and start opencode in the worktree
opencode pr 123
opencode pr 4567
```

## Database (SQLite)

opencode persists sessions and snapshots in SQLite. The bare `opencode db` opens an interactive `sqlite3` shell - pass a query to keep it non-interactive.

```bash
# Print the database file path
opencode db path

# Run a one-shot SQL query (default format: tsv)
opencode db "SELECT id, title, updated FROM session ORDER BY updated DESC LIMIT 10"

# JSON output
opencode db --format json "SELECT id, title FROM session LIMIT 5"

# Migrate legacy JSON-on-disk data into SQLite (merges with existing)
opencode db migrate

# Interactive sqlite3 shell - drive only from a real terminal
opencode db
```

## Upgrade / Uninstall

```bash
# Upgrade to latest
opencode upgrade

# Upgrade to a specific version (with or without leading 'v')
opencode upgrade 1.15.10
opencode upgrade v1.15.10

# Force the upgrade method
opencode upgrade --method brew
opencode upgrade --method npm
opencode upgrade --method bun
opencode upgrade --method curl
opencode upgrade --method choco
opencode upgrade --method scoop

# Uninstall
opencode uninstall --dry-run        # show what would be removed
opencode uninstall --keep-config    # leave ~/.config/opencode alone
opencode uninstall --keep-data      # leave sessions / snapshots
opencode uninstall -c -d            # keep both
opencode uninstall --force          # skip confirmation
```

## Shell Completion

```bash
# Print zsh completion script (default if no shell flag)
opencode completion

# Install for zsh
opencode completion >> ~/.zshrc

# Install for bash
opencode completion -s bash > ~/.opencode-completion.bash
echo 'source ~/.opencode-completion.bash' >> ~/.bashrc

# Install for fish
opencode completion -s fish > ~/.config/fish/completions/opencode.fish
```

## Debug Subcommands

`opencode debug` exposes diagnostics that mirror the LSP, ripgrep, file-system, and snapshot tools the agent uses internally. Most subcommands are one-shot and safe to script; `opencode debug wait` blocks indefinitely (it exists to keep the runner alive while attaching a debugger).

### Resolved Configuration / Environment

```bash
# Show fully resolved config (after merge)
opencode debug config

# Show version, OS, terminal, plugins
opencode debug info

# Show all opencode XDG paths
opencode debug paths

# Print startup time in milliseconds
opencode debug startup
```

### Agents and Skills

```bash
# Inspect an agent's merged config and prompt
opencode debug agent build
opencode debug agent <my-agent>

# Run a single tool in an agent's context (params accept JSON or JS object literal)
opencode debug agent build --tool read --params '{"file":"package.json"}'

# Dump every available skill (large output - auto-saved to tool-output)
opencode debug skill

# List historical / registered projects
opencode debug scrap

# v2 plugin/provider catalog
opencode debug v2
```

### LSP

```bash
# Diagnostics for a file
opencode debug lsp diagnostics src/server.ts

# Workspace symbol search
opencode debug lsp symbols "createServer"

# Document symbols by URI
opencode debug lsp document-symbols file:///abs/path/src/server.ts
```

### Ripgrep

```bash
# File tree (limited)
opencode debug rg tree --limit 200

# List files matching a query / glob
opencode debug rg files --query server --glob "*.ts" --limit 100

# Search file contents
opencode debug rg search "TODO"
opencode debug rg search "createServer" --glob "src/**/*.ts" --glob "!**/*.test.ts" --limit 50
```

### Filesystem

```bash
# Read a file as JSON
opencode debug file read package.json

# File status info
opencode debug file status

# List a directory
opencode debug file list src/

# Search files by query
opencode debug file search Server

# Directory tree (defaults to cwd)
opencode debug file tree
opencode debug file tree src/
```

### Snapshots

```bash
# Track current snapshot state
opencode debug snapshot track

# Show patch for a snapshot hash
opencode debug snapshot patch <hash>

# Show diff for a snapshot hash
opencode debug snapshot diff <hash>
```

### Wait

```bash
# Block forever - useful only when keeping the runner alive for a debugger
opencode debug wait
```

## Sharing

Sharing publishes a session to a CDN-served URL like `https://opncd.ai/s/<id>`.

```bash
# Auto-share a single run
opencode run "Plan a refactor" --share

# Set sharing globally via config (manual is the default)
# opencode.json: "share": "manual" | "auto" | "disabled"
```

In the TUI, `/share` and `/unshare` slash commands toggle sharing for the current session. There is **no** env var to disable sharing globally - use `"share": "disabled"` in config (or push it via managed config in enterprise).

## Global Flags

| Flag                       | Description                                                 |
| -------------------------- | ----------------------------------------------------------- |
| `--help` / `-h`            | Show help for command (use this FIRST)                      |
| `--version` / `-v`         | Show opencode version                                       |
| `--print-logs`             | Print server logs to terminal                               |
| `--log-level LEVEL`        | `DEBUG` / `INFO` / `WARN` / `ERROR`                         |
| `--pure`                   | Run without external plugins                                |
| `--port N`                 | Server port (default `0` = random; `serve`/`web` use 4096)  |
| `--hostname HOST`          | Server hostname (default `127.0.0.1`)                       |
| `--mdns`                   | Enable mDNS discovery                                       |
| `--mdns-domain DOMAIN`     | mDNS domain (default `opencode.local`)                      |
| `--cors ORIGIN`            | Extra CORS origin (repeatable)                              |
| `-m` / `--model SPEC`      | `provider/model` model spec                                 |
| `-c` / `--continue`        | Resume the most recent session                              |
| `-s` / `--session ID`      | Resume a specific session                                   |
| `--fork`                   | Fork the resumed session (requires `-c` or `-s`)            |
| `--prompt TEXT`            | Initial prompt for the TUI                                  |
| `--agent NAME`             | Agent to use                                                |

`opencode run`-only flags worth highlighting: `--format json`, `-f/--file`, `--share`, `--variant`, `--thinking`, `--attach`, `-p/--password`, `-u/--username`, `--dir`, `--title`, `--replay`, `--replay-limit`, `-i/--interactive`, `--dangerously-skip-permissions`, `--demo`, `--command`.

## Output Formatting

### `--format json` (run / session list / db)

```bash
# Stream raw events from a single run, one JSON object per line
opencode run "ping" --format json

# Filter to message events with jq
opencode run "ping" --format json | jq -c 'select(.type=="message")'

# Tail the latest assistant response
opencode run "ping" --format json | jq -r 'select(.type=="message" and .role=="assistant") | .content'

# Sessions as JSON for scripting
opencode session list --format json | jq -r '.[] | "\(.id)\t\(.title)"'
opencode session list --format json | jq '[.[] | select(.directory|startswith(env.PWD))]'
```

The full per-line event schema for `run --format json` is not documented in the official docs. Inspect a sample run for the exact shape on your installed version, or stream the related SSE feed at `GET /event` on a running `opencode serve`.

### Database queries (TSV / JSON)

```bash
# TSV (default) - friendly to awk / column
opencode db "SELECT id, title FROM session LIMIT 5"

# JSON - friendly to jq
opencode db --format json "SELECT id, title FROM session LIMIT 5" | jq .

# Recent project + session pairs
opencode db --format json \
  "SELECT s.id, s.title, p.worktree FROM session s JOIN project p ON s.project_id = p.id ORDER BY s.updated DESC LIMIT 10"
```

### Common scripting recipes

```bash
# Most recent session id
SES=$(opencode session list -n 1 --format json | jq -r '.[0].id')

# Continue scripted prompts in the same session
opencode run -s "$SES" --format json "Now write tests" | jq -c '.'

# Auth status as JSON-ish (table only by default - parse with awk if needed)
opencode auth list

# Grep recent assistant messages from the DB
opencode db --format json "SELECT content FROM message WHERE role='assistant' ORDER BY created DESC LIMIT 5" \
  | jq -r '.[].content'
```
