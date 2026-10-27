# VS Code + Copilot Chat CLI Aliases Guide

This document contains a collection of terminal aliases for driving GitHub Copilot Chat inside VS Code from the command line, using the `code chat` subcommand. It mirrors the structure of the Claude Code aliases guide so the three guides can be used side by side.

Unlike Claude Code or the standalone Copilot CLI, `code chat` always opens the Chat view in a VS Code window. It does not print a response to stdout. Treat these aliases as fast launchers that pre-load a prompt, a mode, and context into the editor.

## How to use these aliases

Make sure the `code` command is on your `PATH` (VS Code: Command Palette → "Shell Command: Install 'code' command in PATH") and that the GitHub Copilot Chat extension is installed and signed in.

These are terminal aliases. On macOS, install them as a standalone file under `/Users/Shared/LDE/cli-aliases/` and load that directory from `~/.zshrc`, so every tool's aliases live in one shared place and stay out of your dotfiles.

**Install location (macOS):** `/Users/Shared/LDE/cli-aliases/vscode-copilot.sh`

Copy the alias blocks from this guide into that file, then add one loader line to `~/.zshrc` that sources everything in the directory:

```bash
# ~/.zshrc
for f in /Users/Shared/LDE/cli-aliases/*.sh; do [ -r "$f" ] && source "$f"; done
```

After saving, run `source ~/.zshrc` to activate the aliases in the current shell. On Linux or with bash, use the same directory layout and put the loader line in `~/.bashrc` instead.

> **Prompt before writing.** Whether a person or an AI agent is following this guide, do not create or overwrite `/Users/Shared/LDE/cli-aliases/vscode-copilot.sh`, and do not add or change the loader line in `~/.zshrc`, without first showing the user the exact path and the exact content that will be written and getting their explicit confirmation. If `/Users/Shared/LDE/cli-aliases/` does not exist yet, ask before creating it as well.

Flags verified against `code chat --help` on VS Code 1.135:

| Flag | Purpose |
|---|---|
| `-m, --mode <mode>` | `ask`, `edit`, `agent`, or a custom mode identifier. Defaults to `agent`. |
| `-a, --add-file <path>` | Attach a file as context. Repeatable. |
| `-r, --reuse-window` | Send the chat to the last active window. |
| `-n, --new-window` | Open an empty window for the chat. |
| `--maximize` | Maximize the Chat view. |
| `--profile <name>` | Open with a specific VS Code profile. |
| trailing `-` | Read stdin as additional context. |

---

## 1. Refined Base Aliases

Agent mode is the default, so the bare command already behaves like Claude Code's interactive session. These aliases pin the window behaviour so a chat never lands in an unexpected editor.

```bash
# Agent mode in the current window, maximized (the everyday default)
alias vsc="code chat -r --maximize"

# Ask-only and edit-only modes for when the agent should not run tools
alias vsc-ask="code chat -r -m ask"
alias vsc-edit="code chat -r -m edit"

# Autonomous execution using a custom chat mode (see note below)
alias vsca="code chat -r --maximize -m autonomous"
alias vscan="code chat -n --maximize -m autonomous"
```

The `autonomous` mode is the VS Code equivalent of Claude Code's custom system-prompt file. Create it as `.github/chatmodes/autonomous.chatmode.md` in the repo (or in your user profile) with the instructions you want applied, then pass its identifier to `-m`.

---

## 2. Scripting & Automation (Pipe Context In)

`code chat` reads stdin when the command ends with `-`. Because the `-` must come last, these are shell functions rather than plain aliases. The prompt still opens in the editor; the piped text is attached as context.

```bash
# Pipe anything into a chat (e.g., cat error.log | vscp "Summarize these errors")
vscp() { code chat -r "$@" -; }

# Pipe the current git diff into an edit-mode session with a prompt
vscd() { git diff | code chat -r -m edit "$@" -; }

# Attach one or more files as context, then prompt
# e.g., vscf src/app.ts src/util.ts "Explain how these interact"
vscf() {
  local args=()
  while [ $# -gt 1 ]; do args+=(-a "$1"); shift; done
  code chat -r "${args[@]}" "$1"
}
```

For truly headless, stdout-returning automation, use the standalone Copilot CLI (`copilot -s -p`) described in the Copilot CLI aliases guide instead.

---

## 3. Model, Profile, & Cost Control

The VS Code CLI has no model flag. The model is chosen from the picker in the Chat view and remembered per mode. What the CLI does offer is `--profile`, which lets you keep separate settings bundles, for example one profile with tool auto-approval enabled and one without.

```bash
# Launch chat under a profile that has tool auto-approval enabled (your "yolo" profile)
alias vsc-auto="code chat -r --maximize --profile copilot-auto"

# Launch chat under a locked-down profile with auto-approval off and fewer extensions
alias vsc-strict="code chat -r --profile copilot-strict"
```

Create the profiles once via Command Palette → "Profiles: New Profile", then set `chat.tools.autoApprove` (and any model defaults) inside each.

---

## 4. Isolation & Safety

Use these when a refactor is risky or when the agent should only see part of the workspace.

```bash
# Open the chat in a fresh, empty window so it does not touch your current workspace
alias vscn="code chat -n --maximize"

# Scope the agent to specific files only (repeat -a as needed)
alias vsc-scope="code chat -r -a"

# Isolate the agent's changes in a new git worktree and open VS Code chat there
vscw() {
  local branch="${1:-copilot/$(date +%Y%m%d-%H%M%S)}"
  local dir="../$(basename "$PWD")-${branch//\//-}"
  git worktree add -b "$branch" "$dir" && code "$dir" && (cd "$dir" && code chat -r --maximize)
}
```

Ask mode (`vsc-ask`) is the safest option when you only want an answer: it never edits files or runs tools.

---

## 5. Review & Maintenance

Pair the chat launcher with the standard `code` extension commands to keep Copilot healthy.

```bash
# Review the current branch against main inside VS Code
vsc-review() {
  git diff main...HEAD | code chat -r --maximize -m ask \
    "Review this diff for bugs, risks, and missing tests. Reference files and lines." -
}

# Install or update the Copilot extensions
alias vsc-copilot-install="code --install-extension GitHub.copilot --install-extension GitHub.copilot-chat"
alias vsc-ext-update="code --update-extensions"

# Confirm which Copilot extensions are installed and their versions
alias vsc-copilot-status="code --list-extensions --show-versions | grep -i copilot"

# Reopen the last-used window (useful after a crash)
alias vscr="code -r ."
```

For the full option list, run `code chat --help` and `code --help`.
