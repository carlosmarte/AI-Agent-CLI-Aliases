# OpenAI Codex CLI Aliases Guide

This document contains a collection of optimized terminal aliases for the OpenAI Codex CLI (the `codex` command), designed to streamline your workflow, keep approvals and sandboxing explicit, and unlock advanced CLI features. It mirrors the structure of the Claude Code aliases guide so the guides can be used side by side.

## How to use these aliases

Install the CLI first (`npm install -g @openai/codex` or `brew install --cask codex`), then run `codex login` once.

These are terminal aliases. On macOS, install them as a standalone file under `/Users/Shared/LDE/cli-aliases/` and load that directory from `~/.zshrc`, so every tool's aliases live in one shared place and stay out of your dotfiles.

**Install location (macOS):** `/Users/Shared/LDE/cli-aliases/openai-codex.sh`

Copy the alias blocks from this guide into that file, then add one loader line to `~/.zshrc` that sources everything in the directory:

```bash
# ~/.zshrc
for f in /Users/Shared/LDE/cli-aliases/*.sh; do [ -r "$f" ] && source "$f"; done
```

After saving, run `source ~/.zshrc` to activate the aliases in the current shell. On Linux or with bash, use the same directory layout and put the loader line in `~/.bashrc` instead.

> **Prompt before writing.** Whether a person or an AI agent is following this guide, do not create or overwrite `/Users/Shared/LDE/cli-aliases/openai-codex.sh`, and do not add or change the loader line in `~/.zshrc`, without first showing the user the exact path and the exact content that will be written and getting their explicit confirmation. If `/Users/Shared/LDE/cli-aliases/` does not exist yet, ask before creating it as well.

All aliases call `codex` from your `PATH`. If you keep it in a fixed location, replace `codex` with the full path. Flags below were checked against the official CLI reference; run `codex --help` and `codex exec --help` to confirm them on your installed version.

---

## 1. Refined Base Aliases

Codex combines two controls: an approval policy (`-a`) and a sandbox level (`-s`). These aliases standardize on `--approve-for-me` for local development, which pairs `on-request` approvals with the `workspace-write` sandbox. The `--yolo` flag (alias of `--dangerously-bypass-approvals-and-sandbox`) is reserved for strict sandbox environments only, as it removes both approvals and the sandbox.

```bash
# Standard interactive and bypass modes
alias cdxy="codex --approve-for-me"
alias cdxyc="codex --approve-for-me resume --last"
alias cdx="codex"

# Fully autonomous inside the workspace: no approval pauses, writes limited to the repo
alias cdxa="codex -a never -s workspace-write"
alias cdxac="codex -a never -s workspace-write resume --last"
```

Keep `--yolo` out of everyday aliases. If you need it, type it deliberately: `codex --yolo`.

---

## 2. Scripting & Automation (Non-Interactive)

`codex exec` runs a single prompt headless and prints the result to stdout. Pass `-` as the prompt to read it from stdin, which makes piping straightforward.

```bash
# Ask a quick question and print to stdout (e.g., cat error.log | cdxp "Summarize these errors")
cdxp() { codex exec -a never -s read-only "$@"; }

# Emit newline-delimited JSON events for downstream tooling (jq, CI steps)
cdxj() { codex exec --json -a never -s read-only "$@"; }

# Headless run that may edit files, saving only the final message to a file
cdxx() { codex exec -a never -s workspace-write -o ./codex-last-message.md "$@"; }
```

These are functions rather than aliases so that `"$@"` passes the prompt through cleanly. The read-only sandbox on `cdxp` and `cdxj` means a stray prompt can't modify your tree.

---

## 3. Effort, Cost, & Model Control

Codex has no budget flag, but reasoning effort and model are both overridable per session. Effort is a config key set with `-c`; the model is set with `-m`. Check `codex --help` for the model IDs your account exposes and substitute below.

```bash
# Max reasoning effort for deep tasks and architectural refactoring
alias cdx-max="codex -c model_reasoning_effort=high"

# Low reasoning effort for quick, cheap questions and small edits
alias cdx-cheap="codex -c model_reasoning_effort=low"

# Force the session to use a specific model
alias cdx-mini="codex -m gpt-5-mini"

# Live web search instead of cached results
alias cdx-search="codex --search"

# Route to a local open-source model via Ollama
alias cdx-local="codex --oss --local-provider ollama"
```

Named profiles let you bundle model, effort, and sandbox settings in `~/.codex/config.toml` and switch with `-p <profile>`, for example `codex -p deep`.

---

## 4. Isolation & Safety

Use these when working on risky refactors or when the agent should only read.

```bash
# Start a session in a new managed git worktree to isolate Codex's changes from your branch
alias cdxw="codex --worktree"

# Read-only mode: the agent can explore and explain but cannot write or run mutating commands
alias cdx-ro="codex -s read-only"

# Grant write access to one extra directory outside the repo (e.g., cdxd ../shared-lib)
alias cdxd="codex --add-dir"

# Run the agent against a different directory without cd-ing there
alias cdxc="codex -C"
```

`--worktree` works for new interactive sessions, `codex exec`, and `codex fork`, but not with `codex resume`. The sandbox levels are `read-only`, `workspace-write`, and `danger-full-access`; the last is best avoided outside a container.

---

## 5. Review & Maintenance

Codex ships a non-interactive review command and the usual health and session tools.

```bash
# Review the current branch against main
alias cdx-review="codex review --base main"

# Review only uncommitted changes, or a single commit
alias cdx-review-wip="codex review --uncommitted"
alias cdx-review-commit="codex review --commit"

# Run a health check on the local installation and configuration
alias cdx-fix="codex doctor"

# Resume the most recent session in this directory, or pick one by ID
alias cdxr="codex resume --last"
alias cdx-resume="codex resume"

# Pull the latest diff from a Codex cloud task into the working tree
alias cdx-apply="codex apply"

# Inspect configured MCP servers and feature flags
alias cdx-mcp="codex mcp list"
alias cdx-features="codex features list"
```

For the full option list, run `codex --help`, or `codex <command> --help` for `exec`, `review`, `resume`, `mcp`, `sandbox`, and `cloud`.
