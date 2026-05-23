---
name: git-cc
description: git-cc CLI (SKalt/git-cc) reference for writing and validating Conventional Commits as a git extension - covers the `git cc` invocation, message validation, dry-run, config file shape (commit_convention.{yaml,yml,toml}), pass-through flags to `git commit`, and which forms launch a blocking TUI vs which are safe in scripts. Includes a compact Conventional Commits format reference.
license: CC-BY-NC-4.0
---

# git-cc

Comprehensive reference for `git-cc` (SKalt/git-cc) - a git extension that helps write and validate [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

**Version:** 0.2.6 (current as of May 2026)
**Source:** https://github.com/SKalt/git-cc
**Spec:** https://www.conventionalcommits.org/en/v1.0.0/

## Interactive vs Non-Interactive Commands

`git-cc` is primarily an interactive TUI for composing commits, but it has well-defined non-interactive paths. The TUI is **the intended experience for humans** at a real terminal - it only becomes a problem when an automated agent runs it without a TTY.

Forms that launch the TUI (block on `$EDITOR` / `/dev/tty`):

- `git cc` (no args)
- `git cc <partial-prefix>` like `git cc feat` or `git cc 'feat(scope)'` - drops you into the TUI starting partway through
- `git cc <msg>` (positional, without `--no-edit`) - validates the header, then opens `$EDITOR` for the body
- `git cc --redo` - reuses the last message but still opens an editor
- Any invocation that fails validation (unknown type, undefined scope when scopes are configured, header over `header_max_length` with `enforce_header_max_length: true`) - falls back into the TUI to fix the problem

Forms that are non-interactive (safe to script):

- `git cc -m '<complete-conventional-commit>'` - **defaults to `--no-edit`**, the canonical scripted form
- `git cc '<msg>' --no-edit` - same, via the positional form
- `git cc '<msg>' --dry-run` - prints the resulting message and exits without touching git (still requires a valid header)
- `git cc --show-config`, `git cc --version`, `git cc --help`, `git cc --init`, `git cc --generate-shell-completion ...`

If you are running unattended (inside another agent, in CI, or in a scripted shell), prefer the non-interactive forms above and make sure your message is **fully valid** - an invalid type or scope drops back into the TUI even with `--dry-run`.

If you are an interactive user (or a tool/skill that knows how to drive a TUI - e.g. a terminal-multiplexer wrapper, an expect-style harness, a TUI-aware editor), running the interactive forms directly is fine and supported.

When in doubt, run `git cc --help` first - it exits cleanly and shows every flag.

## Prerequisites

### Installation

```bash
# macOS / Linux (Homebrew tap)
brew tap skalt/git-cc
brew install git-cc

# Linux / macOS (installer script - prints help with --help; verify shasum first)
repo=skalt/git-cc; branch=master
curl -sL https://raw.githubusercontent.com/$repo/$branch/scripts/install.sh -o /tmp/install.sh
shasum -a 256 /tmp/install.sh
chmod +x /tmp/install.sh && /tmp/install.sh

# Manual: download a release for your platform from
#   https://github.com/SKalt/git-cc/releases/latest
# Supported formats: tar.gz, apk, brew, deb, exe, rpm

# From source (requires go >= 1.19, $GOPATH/bin on $PATH)
git clone https://github.com/SKalt/git-cc.git
cd git-cc && make install

# Verify installation
git-cc --version          # standalone
git cc --version          # as git extension (any binary on $PATH named git-* works as a subcommand)
```

`git-cc` is **source-provided** software (not OSI open-source). Noncommercial / personal / 30-day commercial trial use is allowed under PolyForm Noncommercial 1.0.0 and PolyForm Free Trial 1.0.0. Continued commercial use requires a paid license: https://github.com/sponsors/skalt/sponsorships?tier_id=335824.

### Generate a Config

A config is optional - without one, `git-cc` validates against the default Angular types. To pin types or add scopes:

```bash
# Initialize a config in the nearest sensible location (.config/commit_convention.yaml)
git cc --init

# Choose the on-disk format (default yaml)
git cc --init --config-format yaml
git cc --init --config-format yml
git cc --init --config-format toml
```

### Show Resolved Config

```bash
# Print the search path and which config file (if any) is active
git cc --show-config
```

## CLI Structure

`git-cc` is a single binary - no nested subcommands. The full surface is one command plus flags.

```
git-cc                              # Equivalent to `git cc` (any flag combination below)
├── (no args)                       # Interactive TUI - compose a commit from scratch
├── <prefix>                        # e.g. `feat`, `feat(api)` - TUI seeded mid-form
├── <message>                       # e.g. `feat: add login` - validate; opens $EDITOR for body unless --no-edit
├── -m <message> [-m <body> ...]    # Validate full message(s); --no-edit is implied
├── --redo                          # Reuse last commit message (interactive)
├── --dry-run                       # Print resulting message; do not commit
├── --init                          # Generate a config file
├── --show-config                   # Print config search path + active config
├── --version                       # Print git-cc version
├── --help / -h                     # Print usage
├── --generate-man-page             # Write a man page into your manpath
└── --generate-shell-completion     # Print bash/zsh/fish/powershell completion to stdout

Pass-through flags (forwarded to `git commit`):
  -a/--all   --allow-empty   --author <str>   --date <str>
  --no-edit  --no-gpg-sign   --no-post-rewrite   --no-signoff
  -n/--no-verify   --verify   -s/--signoff
```

## Configuration

### Config File Resolution Order

`git-cc` searches for a config file named `commit_convention.{yaml,yml,toml}` in this order (first hit wins). Within a directory, `yaml` is preferred over `yml`, and `yml` over `toml`.

1. `${PWD}/`
2. `${REPO_ROOT}/` (skipped if not in a git repo)
3. `${REPO_ROOT}/.config/` (skipped if not in a git repo)
4. `${XDG_CONFIG_HOME}/` (defaults to `~/.config/`)

If no config is found, `git-cc` falls back to the default Angular commit types and accepts any scope.

### Minimal commit_convention.yaml

```yaml
# .config/commit_convention.yaml
# Omit commit_types to use the default Angular set (feat, fix, docs, style,
# perf, test, build, chore, ci, refactor, revert).
commit_types:
  - feat: adds a new feature
  - fix: fixes a bug
  - docs: changes only the documentation
  - chore: maintenance, tooling

# Defining scopes lets `git-cc` validate them and show descriptions in the TUI.
# Omit `scopes` entirely if you don't want scope validation.
scopes:
  - parser: parses conventional commits
  - cli: command-line UI
  - dist: release and distribution

# Header is `type(scope)!: description` - max length applies to that whole line.
header_max_length: 72            # default 72
enforce_header_max_length: false # if true, an over-length header is rejected
```

### TOML Equivalent

```toml
# .config/commit_convention.toml
header_max_length = 72
enforce_header_max_length = false

[[commit_types]]
feat = "adds a new feature"
[[commit_types]]
fix = "fixes a bug"

[[scopes]]
parser = "parses conventional commits"
[[scopes]]
cli = "command-line UI"
```

### Environment

`git-cc` does not define its own env vars. It honors the standard git / shell ones:

```bash
# Editor used for the commit body when not --no-edit
export GIT_EDITOR=vim          # git-specific; takes precedence
export EDITOR=vim              # generic fallback

# XDG search path for the global config
export XDG_CONFIG_HOME=$HOME/.config

# Disable git pagers when scripting
export GIT_PAGER=cat
export PAGER=cat
```

## Writing Commits (Non-Interactive)

The canonical scripted form is `git cc -m '<message>'`. With `-m`, `--no-edit` is implied, so this never blocks on an editor.

### Validate and commit a single header

```bash
# Header only (no body)
git cc -m 'feat: add login page'
git cc -m 'fix(parser): handle empty input'
git cc -m 'docs: update README install steps'
```

### Mark a breaking change

```bash
# `!` before the colon
git cc -m 'refactor(api)!: drop deprecated /v1 endpoints'

# Or use a BREAKING CHANGE: footer (multiple -m flags become paragraphs)
git cc \
  -m 'refactor(api): rewrite request layer' \
  -m 'BREAKING CHANGE: removed /v1; clients must migrate to /v2.'
```

### Multi-paragraph body

```bash
# Each subsequent -m becomes a paragraph in the commit body
git cc \
  -m 'feat(auth): add JWT refresh' \
  -m 'Issues a new token when within 5 minutes of expiry.' \
  -m 'Refs: #123'
```

### Positional form (must add --no-edit to stay non-interactive)

```bash
# Without --no-edit, this opens $EDITOR for the body after validating the header
git cc 'feat: add login page' --no-edit

# Quote messages that contain shell metacharacters (parens, !, spaces, $)
git cc 'fix(parser)!: handle $-prefixed identifiers' --no-edit
```

### Dry run

```bash
# Print the resulting message and the would-be `git commit` invocation; do not commit
git cc -m 'feat: example' --dry-run
git cc 'feat(scope): example' --no-edit --dry-run

# Note: --dry-run still requires a valid header. An unknown type or undefined scope
# drops back into the TUI even with --dry-run.
```

## Writing Commits (Interactive)

These forms launch the TUI. Use them only at a real terminal (or behind a tool/skill that drives a TUI).

```bash
# Full interactive composer
git cc

# Seed the TUI partway through
git cc feat                    # type already chosen, skip to scope/description
git cc 'feat(parser)'          # type + scope chosen, skip to description

# Reuse the last commit's message as a starting point
git cc --redo
```

## Pass-Through Flags (forwarded to `git commit`)

`git-cc` validates the message, then delegates to `git commit` with these flags forwarded as-is. Semantics are identical to `git-commit(1)`.

```bash
# Stage all tracked, modified files (-a)
git cc -m 'fix(api): null-check response' -a

# DCO sign-off
git cc -m 'feat(cli): add --json flag' -s
git cc -m 'feat(cli): add --json flag' --signoff

# Skip pre-commit / commit-msg hooks
git cc -m 'chore: rebase fixup' -n
git cc -m 'chore: rebase fixup' --no-verify

# Force hooks to run (default `--verify true`; spell out for clarity)
git cc -m 'feat: ship it' --verify

# Skip GPG signing for this commit only
git cc -m 'docs: typo' --no-gpg-sign

# Suppress the post-rewrite hook (e.g. inside a rebase script)
git cc -m 'fix: rebase resolution' --no-post-rewrite

# Override author / date metadata (pure passthrough to git commit)
git cc -m 'fix(import): patch CVE-2024-1234' \
  --author 'Jane Doe <jane@example.com>' \
  --date '2026-05-13T10:00:00-04:00'

# Allow an empty commit (no staged changes)
git cc -m 'chore: trigger CI' --allow-empty

# Suppress an auto-added Signed-off-by trailer (when default sign-off is configured)
git cc -m 'feat: skip dco' --no-signoff
```

## Generate Man Page / Shell Completion

```bash
# Drop a man page into your $MANPATH (writes the file - non-interactive)
git cc --generate-man-page

# Print a shell completion script for sourcing
git cc --generate-shell-completion bash > ~/.git-cc-completion.bash
git cc --generate-shell-completion zsh  > ~/.git-cc-completion.zsh
git cc --generate-shell-completion fish > ~/.config/fish/completions/git-cc.fish
git cc --generate-shell-completion powershell > git-cc-completion.ps1
```

## Conventional Commits Reference

These are the rules `git-cc` validates. For broader commit discipline (atomicity, splitting, branching strategy), see the `atomic-git-commits` skill.

### Header format

```
type(scope)!: description

body (optional, blank-line-separated paragraphs)

footer (optional - `Token: value` pairs, e.g. `BREAKING CHANGE:`, `Refs:`, `Closes:`)
```

### Type (required)

A single noun describing the kind of change. Defaults from the Angular convention (override with `commit_types` in config):

| Type       | Use when                                              |
| ---------- | ----------------------------------------------------- |
| `feat`     | Adds a new feature (correlates with `MINOR` semver)   |
| `fix`      | Fixes a bug (correlates with `PATCH` semver)          |
| `docs`     | Documentation only                                    |
| `style`    | Formatting; no logic change                           |
| `perf`     | Performance improvement                               |
| `test`     | Adds or corrects tests                                |
| `build`    | Build system or external dependencies                 |
| `chore`    | Maintenance, tooling; nothing for users to see        |
| `ci`       | Continuous-integration configuration                  |
| `refactor` | Restructuring without behavior change                 |
| `revert`   | Reverts a prior commit                                |

### Scope (optional)

A noun in parentheses describing the affected area: `feat(parser):`, `fix(cli):`. If `scopes` is set in your config, only listed scopes are accepted. If absent, any (or no) scope is accepted.

### Breaking change

Two ways - either is sufficient, both are allowed:

- Append `!` before the colon: `feat(api)!: remove /v1 endpoints`
- Add a `BREAKING CHANGE: <description>` footer (recognized by Conventional Commits tooling and changelog generators)

A breaking change correlates with a `MAJOR` semver bump.

### Description

A short, imperative-mood summary of the change. Keep the full header (`type(scope)!: description`) at or under `header_max_length` (default 72) so it stays readable in `git log --oneline`. Set `enforce_header_max_length: true` in config to make over-length headers a hard error.

### Body

Free-form explanation. Separated from the header by exactly one blank line. Use `-m` repeatedly to write the body inline; otherwise let `git-cc` delegate to `$EDITOR`.

### Footer

Trailers in `Token: value` form (one per line, no blank line within the footer block). The standard recognized tokens are `BREAKING CHANGE:` and `BREAKING-CHANGE:` (treated identically). Custom trailers like `Refs:`, `Closes:`, `Signed-off-by:`, `Reviewed-by:` are passed through to git untouched.

## Global Flags

| Flag                            | Description                                                          |
| ------------------------------- | -------------------------------------------------------------------- |
| `--help` / `-h`                 | Print usage                                                          |
| `--version`                     | Print git-cc version                                                 |
| `--show-config`                 | Print config file search path and the active config                  |
| `--init`                        | Generate a starter `commit_convention.{yaml,yml,toml}`               |
| `--config-format <fmt>`         | Format for `--init` (`yaml`, `yml`, or `toml`; default `yaml`)       |
| `-m` / `--message <str>`        | Pass a complete message; repeat for paragraphs. Implies `--no-edit`. |
| `--redo`                        | Reuse the last commit's message (interactive)                        |
| `--dry-run`                     | Print the resulting message; do not commit                           |
| `--no-edit`                     | Do not launch `$EDITOR` after validating the header                  |
| `-a` / `--all`                  | Pass-through to `git commit -a`                                      |
| `-s` / `--signoff`              | Pass-through to `git commit --signoff`                               |
| `--no-signoff`                  | Suppress an auto-added `Signed-off-by` trailer                       |
| `-n` / `--no-verify`            | Bypass git hooks                                                     |
| `--verify`                      | Run git hooks (default `true`)                                       |
| `--no-gpg-sign`                 | Pass-through to `git commit --no-gpg-sign`                           |
| `--no-post-rewrite`             | Bypass the post-rewrite hook                                         |
| `--allow-empty`                 | Allow a commit with no staged changes                                |
| `--author <str>`                | Pass-through to `git commit --author`                                |
| `--date <str>`                  | Pass-through to `git commit --date`                                  |
| `--generate-man-page`           | Generate a man page into `$MANPATH`                                  |
| `--generate-shell-completion`   | Print a bash/zsh/fish/powershell completion script                   |

## Output Formatting and Recipes

`git-cc` produces no structured output - it either commits, prints the rendered message (`--dry-run`), prints config metadata (`--show-config`), or fails. Compose it with standard text tools.

### Capture a rendered message without committing

```bash
# Get just the would-be commit message (drop the trailing "would run: ..." line)
git cc -m 'feat(api): add /v2/users' --dry-run \
  | sed -n '1,/^would run:/{ /^would run:/d; p; }'
```

### Validate without committing

```bash
# Exit 0 means the header parses and (if scopes/types are configured) is valid.
# Exit non-zero or a TTY error means the header is invalid.
if git cc -m 'feat: ship it' --dry-run >/dev/null 2>&1; then
  echo valid
else
  echo invalid
fi
```

### Stage everything and commit in one shot

```bash
git cc -m 'fix(parser): handle empty input' -a
```

### Generate a config and add it to source control

```bash
git cc --init
git add .config/commit_convention.yaml
git cc -m 'chore(repo): add commit-convention config' --no-verify
```

### Use git-cc as a commit-msg hook

`git-cc` validates messages but is not a commit-msg hook itself. To gate `git commit` on Conventional Commits without running the TUI, parse the message with `git-cc -m "$(cat "$1")" --dry-run` from `.git/hooks/commit-msg`:

```bash
#!/usr/bin/env bash
# .git/hooks/commit-msg
msg=$(cat "$1")
if ! git cc -m "$msg" --dry-run >/dev/null 2>&1; then
  echo "commit-msg: not a valid Conventional Commit" >&2
  echo "  $msg" >&2
  exit 1
fi
```

For richer linting, pair with `commitlint` (separate tool, not part of `git-cc`).
