# Claude Code Extension & Architecture Guide

Claude Code operates on a hierarchy of configurable primitives that manage behavior, context, and external capabilities. This guide defines the core components, how they are scoped, and what they do.

## Scoping Convention

All configurations support a dual-scope architecture:

- **Project Scope (`.claude/`)**: Committed to the repository, shared with the team, and applies only to the current workspace.
- **User Scope (`~/.claude/`)**: Personal configurations that apply globally across all your projects.

## 1. Core Primitives

These are the atomic units you author to teach Claude Code how to interact with your codebase.

- **Skills (`skills/<name>/SKILL.md`)** Reusable, repeatable instruction packages. Each skill contains frontmatter describing when Claude should automatically invoke it, alongside optional supporting scripts and reference files. They can also be manually triggered via `/<skill-name>`.

- *Best for:* Encoding complex, multi-step procedures (e.g., codebase reconnaissance, automated package updates).
- **Agents / Subagents (`agents/<name>.md`)** Specialized personas that operate in their own isolated context windows. Frontmatter defines their name, allowed tools, description, and reasoning effort, while the body serves as the system prompt.

- *Best for:* Grinding through heavy tasks (large-scale searches, deep code reviews) without polluting the main conversation's context window.
- **Commands (`commands/<name>.md`)** Lightweight slash commands serving as prompt templates. When a user types `/<name>`, the system runs the prompt, replacing `$ARGUMENTS` with user input.

- *Best for:* Quick, text-based extensions where a full Skill procedure isn't necessary.
- **Rules (Convention via `@import`)** While not a strict native primitive, Rules are a highly supported convention. They are standalone markdown policy files (e.g., banned dependencies, architectural constraints) pulled into the primary `CLAUDE.md` file using `@import` statements.

- *Best for:* Enforcing persistent guardrails that must be loaded into the context of every session.

## 2. Packaging & Distribution

- **Plugins** The native distribution format for Claude Code. A plugin is a bundled package that can include Skills, Agents, Commands, Hooks, MCP servers, and LSP servers.

- *Mechanism:* Installed from a marketplace via `/plugin`, or loaded directly via `claude --plugin-dir <path>` / `--plugin-url <zip-url>`. This replaces the need for manual symlink bootstrapping when sharing environments.

## 3. Automation & Orchestration

- **Hooks** Lifecycle triggers configured in `settings.json`. Hooks enforce deterministic execution (e.g., "always run the linter after an edit tool call") rather than relying on the LLM to remember to do it.
- **Workflows** Deterministic multi-agent orchestration scripts located in `.claude/workflows/`. Used for managing parallel fan-out work, such as large repository migrations or multi-file reviews.
- **Background & Scheduled Agents** Enables asynchronous execution. Sessions can be started in the background (`--bg`), scheduled for later (`/schedule`), or set to run on an interval (`/loop`).

## 4. Context & Memory

- **`CLAUDE.md` Memory Files** The user-authored system prompt entry points. They exist at the user (`~/.claude/CLAUDE.md`), project (`.claude/CLAUDE.md`), and local levels, supporting deep hierarchies via `@import` paths.
- **Auto Memory** A separate, Claude-maintained persistent directory where the agent automatically writes and retrieves learned facts about the project across different sessions.
- **Settings Hierarchy** The configuration cascade: `~/.claude/settings.json` → `.claude/settings.json` → `.claude/settings.local.json`. It controls permissions, model selection, sandboxing, and UI features.

## 5. Tooling & Environment Integration

- **MCP Servers (Model Context Protocol)** Integrations for external tools and live data sources. Configured via `claude mcp` or `.mcp.json` files at the user, project, or local scope.
- **Permission Rules & Sandboxing** Security layers enforced via `settings.json`. Supports granular allow/deny lists per tool (e.g., blocking `Bash(git *)`), global permission modes, and OS-level file/network sandboxing.
- **LSP Support** Language Server Protocol integrations bundled via plugins, granting the agent IDE-level code intelligence and AST awareness.
- **UI Customization** Visual and ergonomic settings including custom statuslines, terminal themes, spinner tips, and keybindings (managed via `~/.claude/keybindings.json`).
