# Gemini CLI Aliases Guide

This document contains a collection of optimized terminal aliases for Google's Gemini CLI (the `gemini` command), designed to streamline your workflow, keep approvals explicit, and unlock advanced CLI features. It mirrors the structure of the Claude Code aliases guide so the guides can be used side by side.

## How to use these aliases

Install the CLI first (`npm install -g @google/gemini-cli`) and run `gemini` once to sign in.

These are terminal aliases. On macOS, install them as a standalone file under `/Users/Shared/LDE/cli-aliases/` and load that directory from `~/.zshrc`, so every tool's aliases live in one shared place and stay out of your dotfiles.

**Install location (macOS):** `/Users/Shared/LDE/cli-aliases/gemini-cli.sh`

Copy the alias blocks from this guide into that file, then add one loader line to `~/.zshrc` that sources everything in the directory:

```bash
# ~/.zshrc
for f in /Users/Shared/LDE/cli-aliases/*.sh; do [ -r "$f" ] && source "$f"; done
```

After saving, run `source ~/.zshrc` to activate the aliases in the current shell. On Linux or with bash, use the same directory layout and put the loader line in `~/.bashrc` instead.

> **Prompt before writing.** Whether a person or an AI agent is following this guide, do not create or overwrite `/Users/Shared/LDE/cli-aliases/gemini-cli.sh`, and do not add or change the loader line in `~/.zshrc`, without first showing the user the exact path and the exact content that will be written and getting their explicit confirmation. If `/Users/Shared/LDE/cli-aliases/` does not exist yet, ask before creating it as well.

All aliases call `gemini` from your `PATH`. If you keep it in a fixed location, replace `gemini` with the full path. Flags below were verified against `gemini --help` on Gemini CLI 0.63.0.

---

## 1. Refined Base Aliases

Gemini CLI prompts for approval before each tool call by default. These aliases standardize on `--approval-mode yolo` for local development and `--approval-mode auto_edit` when you want edits to flow but shell commands to pause. The `--skip-trust` flag removes the workspace trust prompt for the session.

```bash
# Standard interactive and bypass modes
alias gmy="gemini --approval-mode yolo --skip-trust"
alias gmyc="gemini --approval-mode yolo --skip-trust --resume latest"
alias gm="gemini -d"

# Auto-approve file edits only; shell commands still ask
alias gma="gemini --approval-mode auto_edit --skip-trust"
alias gmac="gemini --approval-mode auto_edit --skip-trust --resume latest"

# Seed an interactive session with an opening prompt (e.g., gmi "Explain this repo")
alias gmi="gemini --approval-mode yolo --skip-trust -i"
```

The `-y` / `--yolo` flag is shorthand for `--approval-mode yolo`. The long form is used here so the aliases read clearly next to `auto_edit` and `plan`.

---

## 2. Scripting & Automation (Non-Interactive)

The `-p` (`--prompt`) flag runs headless and prints the response to stdout. Anything on stdin is prepended to the prompt, so piping works without extra flags. Ideal for chaining commands.

```bash
# Ask a quick question and print to stdout (e.g., cat error.log | gmp "Summarize these errors")
alias gmp="gemini -p"

# Same, but emit JSON for downstream tooling (jq, CI steps)
alias gmj="gemini -o json -p"

# Stream JSON events as they arrive (useful for long tasks and progress bars)
alias gmjs="gemini -o stream-json -p"

# Fully unattended run: no approval prompts, no trust prompt
alias gmx="gemini --approval-mode yolo --skip-trust -p"
```

---

## 3. Model, Cost, & Local Control

Gemini CLI has no effort or budget flag. Cost is controlled by choosing the model: Pro for deep reasoning, Flash for cheap and fast. Run `/model` inside a session to see the IDs currently available to your account and substitute them below.

```bash
# Deep reasoning for complex tasks and architectural refactoring
alias gm-pro="gemini --model gemini-2.5-pro"

# Cost-conscious mode for quick questions and simple edits
alias gm-flash="gemini --model gemini-2.5-flash"

# Headless cheap mode for scripts (e.g., git diff | gmp-flash "Write a commit message")
alias gmp-flash="gemini --model gemini-2.5-flash -p"
```

For fully local inference, `gemini gemma` manages routing to local Gemma models; run `gemini gemma --help` for its subcommands.

---

## 4. Isolation & Safety

Use these when working on risky refactors or when the agent should be read-only or sandboxed.

```bash
# Start a session in a new git worktree to isolate Gemini's code changes from your current branch
alias gmw="gemini -w"

# Run inside the sandbox (container or seatbelt, depending on your settings)
alias gms="gemini -s"

# Plan mode: read-only, the agent can explore and propose but cannot edit or run tools
alias gm-plan="gemini --approval-mode plan"

# Add an extra directory to the workspace (e.g., gmd ../shared-lib)
alias gmd="gemini --include-directories"

# Load only specific extensions instead of all of them (e.g., gme my-ext)
alias gme="gemini -e"
```

Tool allow and deny rules now live in the Policy Engine. Pass `--policy <file-or-dir>` to load additional policy files for a session; the older `--allowed-tools` flag is deprecated.

---

## 5. Review & Maintenance

Gemini CLI has no built-in review command, so review is a canned headless prompt. Session and extension management are built in.

```bash
# Review the current branch against main, printed to stdout
gm-review() {
  git diff main...HEAD | gemini -p \
    "Review this diff for bugs, risks, and missing tests. Reference files and lines."
}

# Resume a session: gmr latest, or gmr 3 (see gm-sessions for the index)
alias gmr="gemini --resume"
alias gm-sessions="gemini --list-sessions"

# Inspect installed extensions and configured MCP servers
alias gm-ext="gemini extensions list"
alias gm-mcp="gemini mcp list"

# Resume in bypass mode when you want to pick up where an autonomous run stopped
alias gmry="gemini --approval-mode yolo --skip-trust --resume"
```

For the full option list, run `gemini --help`, or `gemini <command> --help` for the `mcp`, `extensions`, `skills`, `hooks`, and `gemma` subcommands.
