# Introduction to puku-cli

puku-cli is an AI-powered coding assistant that runs entirely in your terminal. It lets you write, edit, debug, and review code through a conversation with an AI — without leaving your shell.

Unlike web-based AI tools, puku-cli operates where developers already work: the command line. It reads your files, runs shell commands, searches your codebase, and executes code on your behalf, all with your approval.

---

## What puku-cli can do

**Code and file operations**
- Read, write, and edit files across your project
- Search code with pattern matching and full-text grep
- Apply targeted edits across multiple files in one turn

**Shell and execution**
- Run Bash and PowerShell commands
- Execute scripts and capture output
- Interact with your local environment (git, npm, docker, etc.)

**Code understanding**
- Explain unfamiliar code and dependencies
- Trace through call stacks and data flows
- Answer questions about any part of your codebase

**Development workflows**
- Review pull requests and suggest improvements
- Generate commit messages from staged diffs
- Scaffold new features, tests, and documentation

**Extensibility**
- Define custom agents — persistent AI personas with their own system prompts, tools, and memory
- Build reusable skills — prompt templates invoked as slash commands
- Connect external tools and APIs via the Model Context Protocol (MCP)
- Automate lifecycle events with hooks (pre-tool, post-tool, session start/end, etc.)

## Platform support

| Platform | Status |
|---|---|
| macOS | Supported — Intel + Apple Silicon |
| Linux | Supported — all major distributions |
| Windows | Supported — PowerShell + WSL |

**IDE integration**: an official VS Code extension (`puku-cli-vscode`) provides a Control Center panel, project-aware launch behavior, and terminal link support.

**Runtime requirement**: Node.js v18 or newer.

---

## How puku-cli is structured

A session is a conversation loop. You type a message; puku-cli reasons about it, calls tools (read files, run commands, search code), and streams a response back. Every tool call is shown to you before it executes, and you can approve, deny, or modify it.

Key concepts:

- **Slash commands** — `/help`, `/review`, `/compact`, `/agents`, `/skills`, `/mcp`, and 90+ more built-in workflows
- **Tools** — the actions puku-cli can take: file I/O, shell execution, web search, code search, notebook editing, and more
- **Agents** — custom AI personas you define once and reuse, each with its own system prompt, allowed tools, and optional persistent memory
- **Skills** — reusable prompt templates stored as Markdown files, invoked as slash commands
- **MCP servers** — external tool providers (databases, APIs, GitHub, etc.) connected via the Model Context Protocol
- **Hooks** — shell scripts that run automatically at lifecycle points (before a tool runs, after a session ends, etc.)

Context is managed automatically. When the conversation grows long, puku-cli compacts prior turns into a summary so you never hit a hard stop mid-task.

---

## Installation

For detailed setup instructions, see:

- [Quick Start](QUICK_START_GUIDE.md) — get running in under two minutes
- [Installation Guide](INSTALLATION_GUIDE.md) — platform-specific prerequisites, permission fixes, and troubleshooting

---

## Next steps

| Topic | Guide |
|---|---|
| Install puku-cli | [Quick Start](QUICK_START_GUIDE.md) |
| Platform setup and troubleshooting | [Installation Guide](INSTALLATION_GUIDE.md) |
| Connect external tools | [MCP servers](mcp.md) |
| Build custom AI personas | [Agents](agents.md) |
| Create reusable workflows | [Skills](skills.md) |
| Automate with lifecycle scripts | [Hooks](hooks.md) |
