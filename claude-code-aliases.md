# Claude Code CLI Aliases Guide

This document contains a collection of optimized terminal aliases for Claude Code, designed to streamline your workflow, improve security, and unlock advanced CLI features.

## How to use these aliases

Add these lines to your shell configuration file (e.g., `~/.zshrc`, `~/.bashrc`, or `~/.bash_aliases`). After saving the file, run `source ~/.zshrc` (or the equivalent for your shell) to activate them.

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
