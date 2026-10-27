# Claude Code CLI Aliases Guide

This document contains a collection of optimized terminal aliases for Claude Code, designed to streamline your workflow, improve security, and unlock advanced CLI features.

## How to use these aliases

These are terminal aliases. On macOS, install them as a standalone file under `/Users/Shared/LDE/cli-aliases/` and load that directory from `~/.zshrc`, so every tool's aliases live in one shared place and stay out of your dotfiles.

**Install location (macOS):** `/Users/Shared/LDE/cli-aliases/claude-code.sh`

Copy the alias blocks from this guide into that file, then add one loader line to `~/.zshrc` that sources everything in the directory:

```bash
# ~/.zshrc
for f in /Users/Shared/LDE/cli-aliases/*.sh; do [ -r "$f" ] && source "$f"; done
```

After saving, run `source ~/.zshrc` to activate the aliases in the current shell. On Linux or with bash, use the same directory layout and put the loader line in `~/.bashrc` instead.

> **Prompt before writing.** Whether a person or an AI agent is following this guide, do not create or overwrite `/Users/Shared/LDE/cli-aliases/claude-code.sh`, and do not add or change the loader line in `~/.zshrc`, without first showing the user the exact path and the exact content that will be written and getting their explicit confirmation. If `/Users/Shared/LDE/cli-aliases/` does not exist yet, ask before creating it as well.

---

## 1. Refined Base Aliases

These aliases improve upon your original setup by using the portable `~/` path instead of a hardcoded home directory. They also standardize on `--permission-mode bypassPermissions` for local development, reserving `--dangerously-skip-permissions` only for strict sandbox environments.

```bash
# Standard interactive and bypass modes
alias cldy="~/.local/bin/claude --permission-mode bypassPermissions"
alias cldyc="~/.local/bin/claude --permission-mode bypassPermissions --continue"
alias cld="~/.local/bin/claude -d"

# Fable model sessions (bypass permissions, optionally continuing the last session)
alias fabley="~/.local/bin/claude --model fable --permission-mode bypassPermissions"
alias fablec="~/.local/bin/claude --model fable --permission-mode bypassPermissions --continue"
```

---

## 2. Scripting & Automation (Non-Interactive)

The `-p` (`--print`) flag is incredibly powerful for chaining commands, as it skips the interactive UI, trust prompts, and returns plain text. Ideal for piping data in and out.

```bash
# Ask a quick question and print to stdout (e.g., cat error.log | cldp "Summarize these errors")
alias cldp="~/.local/bin/claude -p"
```

---

## 3. Effort, Cost, & Model Control

These aliases help you manage API usage, forcing Claude to either think deeper for complex tasks or stay within a strict budget for simple ones.

```bash
# Max effort mode for deep reasoning, complex tasks, and architectural refactoring
alias cld-max="~/.local/bin/claude --effort max"

# Cost-conscious mode (sets a hard API budget limit of $1.00 for the session)
alias cld-cheap="~/.local/bin/claude --max-budget-usd 1.00"

# Force the session to use a specific model (e.g., Sonnet)
alias cld-sonnet="~/.local/bin/claude --model sonnet"
```

---

## 4. Isolation & Safety

Use these when working on risky refactors or troubleshooting Claude Code itself.

```bash
# Start a session in a new git worktree to isolate Claude's code changes from your current branch
alias cldw="~/.local/bin/claude -w"

# Start a session in safe-mode (disables CLAUDE.md, MCP servers, and hooks) for debugging
alias cld-safe="~/.local/bin/claude --safe-mode"
```

---

## 5. Review & Maintenance

Claude Code includes built-in tools for maintaining your repository and checking its own health.

```bash
# Trigger a cloud-hosted, multi-agent code review on your current branch or PR
alias cld-review="~/.local/bin/claude ultrareview"

# Run a health check and fix local configuration issues
alias cld-fix="~/.local/bin/claude doctor"

# Open the interactive session picker to resume a lost session
alias cldr="~/.local/bin/claude -r"
```
