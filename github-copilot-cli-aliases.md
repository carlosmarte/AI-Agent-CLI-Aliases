# GitHub Copilot CLI Aliases Guide

This document contains a collection of optimized terminal aliases for the GitHub Copilot CLI (the agentic `copilot` command), designed to streamline your workflow, keep permissions explicit, and unlock advanced CLI features. It mirrors the structure of the Claude Code aliases guide so the two can be used side by side.

## How to use these aliases

Install the CLI first (`npm install -g @github/copilot` or `brew install copilot-cli`), then run `copilot login` once.

These are terminal aliases. On macOS, install them as a standalone file under `/Users/Shared/LDE/cli-aliases/` and load that directory from `~/.zshrc`, so every tool's aliases live in one shared place and stay out of your dotfiles.

**Install location (macOS):** `/Users/Shared/LDE/cli-aliases/github-copilot-cli.sh`

Copy the alias blocks from this guide into that file, then add one loader line to `~/.zshrc` that sources everything in the directory:

```bash
# ~/.zshrc
for f in /Users/Shared/LDE/cli-aliases/*.sh; do [ -r "$f" ] && source "$f"; done
```

After saving, run `source ~/.zshrc` to activate the aliases in the current shell. On Linux or with bash, use the same directory layout and put the loader line in `~/.bashrc` instead.

> **Prompt before writing.** Whether a person or an AI agent is following this guide, do not create or overwrite `/Users/Shared/LDE/cli-aliases/github-copilot-cli.sh`, and do not add or change the loader line in `~/.zshrc`, without first showing the user the exact path and the exact content that will be written and getting their explicit confirmation. If `/Users/Shared/LDE/cli-aliases/` does not exist yet, ask before creating it as well.

All aliases call `copilot` from your `PATH`. If you keep it in a fixed location, replace `copilot` with the full path (e.g., `~/.local/bin/copilot`).

---

## 1. Refined Base Aliases

Copilot CLI asks for approval per tool, per path, and per URL by default. These aliases standardize on `--allow-all-tools` for local development, reserving `--yolo` (tools + paths + URLs) for strict sandbox environments only.

```bash
# Standard interactive and bypass modes
alias copy="copilot --allow-all-tools"
alias copyc="copilot --allow-all-tools --continue"
alias cop="copilot"

# Autonomous execution modes using a custom agent definition
alias copa="copilot --allow-all-tools --agent=autonomous"
alias copac="copilot --allow-all-tools --agent=autonomous --continue"
```

The `--agent` flag loads a custom agent by name from your Copilot agents directory. Use it the same way you would a custom system prompt file in Claude Code.

---

## 2. Scripting & Automation (Non-Interactive)

The `-p` flag runs a single prompt and exits, skipping the interactive UI. Pair it with `-s` (`--silent`) to strip stats and decoration so only the agent's response reaches stdout. Ideal for piping data in and out.

```bash
# Ask a quick question and print to stdout (e.g., cat error.log | copp "Summarize these errors")
alias copp="copilot -s -p"

# Same, but emit JSONL for downstream tooling (jq, CI steps)
alias copj="copilot -s --output-format=json -p"

# Fully unattended run: no approval prompts, never pauses to ask the user a question
alias copx="copilot --allow-all-tools --no-ask-user -s -p"
```

Add `--assisted-approval` to a non-interactive alias if you want a safety review of each action instead of blanket approval.

---

## 3. Model, Parallelism, & Cost Control

These aliases pin the session to a specific model or change how much work runs in parallel. Reasoning effort is set through the `effortLevel` key in `settings.json` rather than a flag, and there is no per-session budget flag; use `copilot help billing` to review usage.

```bash
# Force the session to use a specific model
alias cop-haiku="copilot --model=claude-haiku-4.5"
alias cop-sonnet="copilot --model=claude-sonnet-4.5"

# Fleet mode: fan work out across parallel subagents for large, divisible tasks
alias cop-fleet="copilot --allow-all-tools --fleet"
```

The `COPILOT_MODEL` environment variable does the same job as `--model` if you prefer to set it once per shell.

---

## 4. Isolation & Safety

Use these when working on risky refactors or when the agent should only touch part of the filesystem.

```bash
# Allow access to one extra directory outside the working tree (e.g., a sibling package)
alias copd="copilot --add-dir"

# Deny destructive shell commands while leaving everything else interactive
alias cop-safe="copilot --deny-tool='shell(rm)' --deny-tool='shell(git push)'"

# Trust this folder's hooks, MCP servers, and plugins for a single non-interactive run
alias cop-trust="copilot --trust-folder -p"

# Isolate the agent's changes in a new git worktree (Copilot has no built-in worktree flag)
copw() {
  local branch="${1:-copilot/$(date +%Y%m%d-%H%M%S)}"
  local dir="../$(basename "$PWD")-${branch//\//-}"
  git worktree add -b "$branch" "$dir" && (cd "$dir" && copilot --allow-all-tools)
}
```

`--deny-tool` always wins over `--allow-tool`, so `cop-safe` can be combined with `--allow-all-tools` to get a fast session that still refuses the listed commands. Note that `--trust-folder` cannot be combined with `--resume` or `--continue`.

---

## 5. Review & Maintenance

Copilot CLI includes built-in commands for maintaining itself and for sharing session transcripts.

```bash
# Ask for a review of the current branch against main, printed to stdout
alias cop-review="copilot -s -p 'Review the diff between main and HEAD for bugs, risks, and missing tests. Be specific with file and line references.'"

# Update the CLI to the latest release
alias cop-update="copilot update"

# Show the effective configuration and check login state
alias cop-config="copilot config --list"
alias cop-login="copilot login"

# Open the interactive session picker to resume a lost session
alias copr="copilot --resume"

# Run a session and export its transcript to a markdown file when it ends
alias cop-share="copilot --share=./copilot-session.md"
```

For a full list of options and help topics, run `copilot help` or `copilot help <topic>` (topics include `permissions`, `config`, `environment`, `sandbox`, and `billing`).
