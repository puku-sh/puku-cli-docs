# Custom Agents in puku-cli

Agents are specialized AI assistants you define once and reuse across every session. Each agent has its own system prompt, tool set, model choice, and optional persistent memory — giving you a permanent team of focused experts rather than a single general-purpose assistant.

---

## What agents can do

- **Automate repetitive workflows** — a test-runner that fires after every code change, a commit-message writer triggered before every commit
- **Enforce quality gates** — a code-reviewer that checks for security issues, a documentation checker that flags missing JSDoc
- **Persist domain knowledge** — an architecture agent that accumulates knowledge of your codebase across sessions via memory
- **Parallelize work** — multiple agents run concurrently via the `Agent` tool, letting one orchestrator delegate to specialists

---

## Two ways to create an agent

### Option 1 — Interactive wizard (recommended)

Run `/agents` in any puku-cli session, then select **Create new agent**. The wizard walks you through:

1. **Location** — project (`.puku-cli/agents/`) or personal (`~/.puku-cli/agents/`)
2. **Method** — generate with AI or configure manually
3. **Agent type / name** — the identifier used to call the agent
4. **Description** — when puku-cli should automatically invoke this agent
5. **System prompt** — the agent's instructions and persona
6. **Tools** — which tools the agent is allowed to use
7. **Model** — puku-ai-2.7 or Inherit from parent
8. **Color** — visual label in the agents list

### Option 2 — Write the file manually

Create a `.md` file in one of these directories:

| Scope | Path | Available in |
|---|---|---|
| Project | `.puku-cli/agents/<name>.md` | This project only |
| Personal | `~/.puku-cli/agents/<name>.md` | Every project |

The filename does not matter — puku-cli uses the `name:` field inside the file.

---

## Agent file structure

Every agent is a Markdown file with a YAML frontmatter block followed by the system prompt.

```markdown
---
name: <required>
description: "<required>"
tools: <optional>
model: <optional>
memory: <optional>
effort: <optional>
color: <optional>
---

Your system prompt goes here.
```

### Minimal example

```markdown
---
name: commit-message-writer
description: "Use this agent after the user asks to stage or commit changes. Write a concise, conventional commit message based on the staged diff."
---

You are a senior engineer who writes commit messages. Follow conventional commits format: type(scope): subject. Keep the subject under 72 characters. Use the imperative mood. Do not include a body unless explicitly asked.
```

### Full example with every field

```markdown
---
name: code-reviewer
description: "Use this agent when the user has finished writing a feature, function, or module and asks for a review. Check for bugs, security issues, style violations, and missing tests."
tools: Read, Grep, Bash
model: puku-ai-2.7
effort: high
memory: project
color: blue
---

You are a principal software engineer performing code review. Your responsibilities:

1. **Correctness** — identify logic errors, off-by-one bugs, null/undefined edge cases
2. **Security** — flag injection risks, insecure defaults, exposed secrets, unvalidated input
3. **Performance** — spot O(n²) loops, unnecessary re-renders, blocking I/O
4. **Maintainability** — flag functions over 50 lines, missing error handling, unclear naming
5. **Test coverage** — note untested branches and suggest specific test cases

Format your output as:
- A summary paragraph (2–3 sentences)
- A numbered list of findings, each with severity (critical / major / minor) and a suggested fix
- A final "LGTM" or "Needs changes" verdict

Ask for the file paths if they are not already in context.
```

---

## Frontmatter field reference

### `name` (required)

The agent's unique identifier. Must be 3–50 characters, using only letters, digits, and hyphens. Must start and end with a letter or digit.

```yaml
name: test-runner        # good
name: api-docs-writer    # good
name: my_agent           # bad — underscores not allowed
name: -agent             # bad — cannot start with hyphen
```

### `description` (required)

Tells puku-cli **when** to invoke this agent. This is the most important field for agent-triggered workflows — puku-cli reads it to decide whether the current context matches. Write it like a precise trigger condition, not a capability summary.

```yaml
# Good — specific triggering condition
description: "Use this agent when the user explicitly asks to run tests, when a function or module has just been written, or when the user says 'check if this works'."

# Less effective — describes capability, not trigger
description: "This agent runs tests."
```

**Tips for a precise description:**
- Start with "Use this agent when…"
- List 2–4 specific triggering scenarios
- Include example user phrases that should trigger it: `"when the user says 'review this' or 'LGTM?'"`
- Mention what context must be present: `"when there are staged git changes"`
- Maximum 5000 characters; recommended 100–500 characters for most agents

### `tools` (optional)

Comma-separated list of tools the agent is allowed to use. Omit the field (or set `*`) to give the agent access to all tools.

```yaml
tools: Read, Grep, Bash            # read-only + shell
tools: Read, Write, Edit, Bash     # full file access + shell
tools: WebSearch, WebFetch, Read   # research agent
tools: *                           # all tools (default when omitted)
```

**Available tool names:**

| Category | Tools |
|---|---|
| File system | `Read`, `Write`, `Edit`, `Glob`, `Grep` |
| Shell | `Bash`, `PowerShell` |
| Web | `WebSearch`, `WebFetch` |
| Agents & skills | `Agent`, `Skill` |
| Tasks | `TaskCreate`, `TaskGet`, `TaskList`, `TaskUpdate`, `TaskStop` |
| Scheduling | `CronCreate`, `CronDelete`, `CronList` |
| Notebooks | `NotebookEdit` |
| Git isolation | `EnterWorktree`, `ExitWorktree` |
| Planning | `EnterPlanMode`, `ExitPlanMode` |
| User interaction | `AskUserQuestion` |

**Principle of least privilege:** Only grant the tools the agent actually needs. A documentation agent that only reads files should not have `Bash` or `Write`.

### `model` (optional)

Overrides the model for this specific agent. If omitted, the agent uses the same model as the parent session.

```yaml
model: puku-ai-2.7    # use the puku-ai-2.7 model explicitly
model: inherit          # same as parent session (default behavior)
```

Omitting `model:` is equivalent to `model: inherit`.

### `memory` (optional)

Gives the agent persistent memory across sessions. When set, the agent reads its `MEMORY.md` at startup and can write to it during the session to accumulate knowledge.

```yaml
memory: project    # stored in .puku-cli/agent-memory/<name>/  (project-scoped)
memory: user       # stored in ~/.puku-cli/agent-memory/<name>/ (personal, cross-project)
memory: local      # stored in .puku-cli/agent-memory-local/<name>/ (project, never committed)
```

| Scope | Committed to git | Shared across machines | Use case |
|---|---|---|---|
| `project` | Depends on `.gitignore` | Yes (if committed) | Team-shared knowledge base |
| `user` | No | No | Personal preferences and notes |
| `local` | No | No (unless using remote mount) | Machine-local scratch notes |

When `memory:` is set, `Read`, `Write`, and `Edit` are automatically added to the agent's tool set if not already present.

**In the system prompt**, tell the agent what to remember:

```markdown
**Update your agent memory** as you discover architectural decisions, recurring patterns,
important file locations, and project-specific conventions.
```

### `effort` (optional)

Controls how much reasoning the agent applies per response.

```yaml
effort: low      # fast, minimal overhead — good for simple tasks
effort: medium   # balanced — good for most agents
effort: high     # thorough — good for review, analysis, and debugging
```

### `color` (optional)

A visual label shown next to the agent's name in the `/agents` list.

```yaml
color: red | blue | green | yellow | purple | orange | pink | cyan
```

---

## Writing effective system prompts

The system prompt is the agent's complete instruction manual. It is written in the Markdown body of the file, below the `---` closing the frontmatter block.

### Structure that works well

```markdown
You are a <role/persona> specializing in <domain>.

## Responsibilities
- Clear responsibility 1
- Clear responsibility 2
- Clear responsibility 3

## Process
Describe the step-by-step approach the agent should follow.

## Output format
Specify exactly what the response should look like — headers, bullet points,
code blocks, line length, etc.

## Constraints
- What the agent must NOT do
- Edge cases to handle explicitly
- When to ask for clarification vs. proceed
```

### Tips for maximum output precision

**Be specific about output format.** Vague prompts produce vague output. Specify whether you want prose, bullet lists, code blocks with language tags, numbered findings, or a pass/fail verdict.

```markdown
# Vague
Review the code and give feedback.

# Precise
Output a numbered list. Each item must contain: severity (critical/major/minor),
the exact file and line number, a one-sentence explanation, and a corrected
code snippet in a fenced block. End with either "APPROVED" or "CHANGES REQUIRED".
```

**Define the persona sharply.** `"You are an expert"` is weaker than `"You are a senior backend engineer at a fintech company who prioritizes correctness and auditability over brevity."`

**State explicit constraints.** Tell the agent what it must not do:

```markdown
- Do not modify test files unless explicitly asked
- Do not suggest architectural changes — only point out bugs
- Never output more than 20 findings; group minor issues
```

**Describe escalation behavior.** Tell the agent when to ask questions vs. proceed:

```markdown
If the file path is not in context, ask for it before proceeding.
If the scope of the task is ambiguous, list your assumptions and ask for confirmation.
```

**Give examples inline.** A single concrete example is worth ten sentences of abstract instruction:

```markdown
Example finding:
[MAJOR] src/auth/login.ts:42 — Password comparison uses `==` instead of a
timing-safe function, enabling timing attacks.
```

---

## Memory — making agents smarter over time

When an agent has `memory: project` (or `user` / `local`), it builds up knowledge across sessions. Here is how to get the most out of it.

### Tell the agent what to write

In the system prompt, specify what domain knowledge is worth persisting:

```markdown
**Update your agent memory** as you discover:
- File locations for key modules (auth, DB schema, API routes)
- Recurring code patterns and conventions in this project
- Known flaky tests and their root causes
- Architectural decisions and the reasoning behind them
```

### What gets written

The agent writes to `MEMORY.md` in its memory directory using the `Write` tool. The file is plain Markdown. On the next session, its contents are injected into the agent's system prompt automatically.

### Scoping memory to your needs

Use `memory: project` when the knowledge is codebase-specific and you want teammates to benefit. Use `memory: user` for personal preferences that apply everywhere. Use `memory: local` for scratch notes you never want to commit.

---

## Common agent patterns

### Code reviewer

```markdown
---
name: code-reviewer
description: "Use this agent when the user finishes writing a feature and asks for review, or says 'review this', 'LGTM?', or 'is this good?'."
tools: Read, Grep, Bash
effort: high
color: blue
---

You are a principal engineer performing peer code review. Check for: correctness, security vulnerabilities (injection, auth bypass, secrets in code), performance issues (N+1 queries, blocking calls), and missing error handling. Output a numbered list of findings with severity (critical/major/minor), file:line, and suggested fix. End with APPROVED or CHANGES REQUIRED.
```

### Test runner

```markdown
---
name: test-runner
description: "Use this agent after a function, class, or module has been written or modified. Run the relevant tests and report results. Trigger when the user says 'run tests', 'check if this works', or 'does it pass?'."
tools: Bash, Read
color: green
---

You are a QA engineer. Run the project's test suite scoped to the files that were just changed. Use the project's test command (check package.json or Makefile). Report: pass/fail counts, failing test names with error messages, and whether coverage changed. If tests pass, output TESTS PASSING. If they fail, summarize the failures concisely and stop — do not attempt fixes unless asked.
```

### Commit message writer

```markdown
---
name: commit-message-writer
description: "Use this agent when the user asks to commit changes, stage files, or write a commit message. Also trigger when the user says 'commit this' or 'git commit'."
tools: Bash, Read
color: yellow
---

You are a developer who writes precise, conventional commit messages. Run `git diff --staged` to see what is staged. Follow conventional commits: type(scope): subject. Types: feat, fix, refactor, docs, test, chore, perf. Keep subject under 72 characters, imperative mood, no period. Add a body only if the change is non-obvious. Output only the commit message — no preamble, no explanation.
```

### Architecture guide (with memory)

```markdown
---
name: architect
description: "Use this agent for questions about the codebase structure, where to put new files, which modules handle what, or when the user asks 'where is X' or 'how does Y work'."
tools: Read, Glob, Grep
memory: project
color: purple
---

You are the lead architect for this project. You know every module, every key abstraction, and every design decision. Answer questions about the codebase structure concisely and point to specific files and line numbers. When you discover something new about the codebase, update your memory so you remember it next session.

**Update your agent memory** as you discover: module responsibilities, key file locations, API boundaries, data flow, and architectural constraints.
```

---

## Agent locations and scope

Agents can live in two places:

```
.puku-cli/agents/          ← project agents (checked in with the repo)
~/.puku-cli/agents/        ← personal agents (available in all projects)
```

**Project agents** are ideal for team workflows — commit them to the repository so everyone on the team gets the same agents automatically.

**Personal agents** are ideal for your personal style preferences, shortcuts, and cross-project tools.

You can have agents with the same name in both locations. The project agent takes precedence within a project.

---

## Calling agents explicitly

You can call any agent by name in a prompt:

```
Use the code-reviewer agent on src/auth/login.ts
```

Or spawn one from another agent's system prompt using the `Agent` tool with the agent's `name` value.

---

## Troubleshooting

**Agent is not triggering automatically**
The description is the trigger signal. Make it more specific and action-oriented. Add concrete user phrases: `"when the user says 'commit' or 'git commit'"`.

**Agent uses the wrong tools**
Check the `tools:` field. If empty, the agent has all tools. Add explicit tool restrictions to prevent unexpected behavior.

**Generation fails with "Invalid agent configuration"**
The AI generation step could not produce a valid JSON configuration. Try shortening or simplifying your description prompt, or switch to Manual configuration to write the system prompt yourself.

**Agent forgets things between sessions**
Add `memory: project` (or `user`) to the frontmatter and add explicit memory instructions to the system prompt telling the agent what to record.

**Name validation error**
Agent names must be 3–50 characters, alphanumeric and hyphens only, starting and ending with a letter or digit. No spaces, underscores, or special characters.
