# Skills in puku-cli

Skills are reusable prompt templates you invoke with a `/` slash command. A skill injects a structured prompt into the conversation — with optional tool restrictions, argument substitution, model overrides, and execution context — so you can encode repeatable workflows once and reuse them everywhere.

Skills differ from agents in one key way: a skill is a **prompt template**, while an agent is a **persistent AI persona with its own system prompt and memory**. Use skills for well-defined, bounded tasks (summarize this, review that, generate X). Use agents for ongoing collaborative roles (a code reviewer you work with across sessions).

---

## How skills work

When you type `/skill-name` or when the model invokes a skill automatically:

1. puku-cli loads the skill's Markdown body and substitutes any arguments
2. The expanded content is injected into the conversation as a user message
3. The model responds — using only the tools the skill allows, with the model and effort level the skill specifies
4. For **forked** skills, this runs in an isolated sub-agent with its own token budget

---

## Creating a skill

### Option 1 — Write a file manually

Create a `.md` file inside a `SKILL.md` directory structure:

```
.puku-cli/skills/<skill-name>/SKILL.md   ← project skill
~/.puku-cli/skills/<skill-name>/SKILL.md ← personal skill
```

The directory name becomes the slash command name. A skill named `summarize` lives at `.puku-cli/skills/summarize/SKILL.md` and is invoked with `/summarize`.

### Option 2 — Interactive wizard

Run `/skills` in any session, then select **Create new skill**. The wizard walks you through each field.

---

## Skill scopes

| Scope | Path | Committed to git | Available in |
|---|---|---|---|
| Project | `.puku-cli/skills/` | Yes (opt-in) | This project |
| Personal | `~/.puku-cli/skills/` | No | Every project |

When a skill with the same name exists in both locations, the project scope takes precedence within that project.

---

## Skill file format

Every skill is a Markdown file with a YAML frontmatter block followed by the prompt body.

```markdown
---
name: <required>
description: "<required>"
---

Your prompt template goes here.
```

### Minimal example

```markdown
---
name: summarize
description: "Summarize the current file or the content the user just pasted. Output a 3–5 bullet summary."
---

Summarize the content in context. Use 3–5 concise bullets. No headers, no preamble.
```

### Full example

```markdown
---
name: security-review
description: "Perform a security review of the code the user has just written or shared."
allowed-tools: "Read, Grep, Bash"
argument-hint: "[file-path]"
arguments: file
effort: high
model: puku-ai-2.7
context: fork
---

You are a senior application security engineer. Review ${file} for vulnerabilities.

Check for:
1. Injection risks (SQL, shell, template)
2. Authentication and authorization flaws
3. Insecure defaults and hardcoded secrets
4. Unvalidated input at system boundaries
5. Unsafe deserialization or file operations

Output a numbered list of findings. Each finding must include:
- Severity: critical / major / minor
- File and line number
- One-sentence explanation
- A corrected code snippet

End with either APPROVED or CHANGES REQUIRED.
```

---

## Frontmatter field reference

### `name` (required)

The slash command name. Must be 3–50 characters using only letters, digits, and hyphens. Must start and end with a letter or digit.

```yaml
name: summarize          # invoked as /summarize
name: pr-description     # invoked as /pr-description
```

Namespaced skills (stored in nested directories) use a colon separator:

```
.puku-cli/skills/review/security/SKILL.md → /review:security
```

### `description` (required)

Tells puku-cli what the skill does. Shown in the `/skills` menu and used by the model to decide when to auto-invoke the skill.

Write it as a clear statement of what the skill produces or does:

```yaml
description: "Write a conventional commit message based on the staged diff."
description: "Generate JSDoc for the function the user has selected or pasted."
```

### `when-to-use` (optional)

A more detailed trigger condition, separate from the short `description`. The model uses this to decide when to invoke the skill automatically. Be specific about trigger scenarios and example user phrases.

```yaml
when-to-use: >
  Use this skill when the user asks to write, generate, or update a commit message,
  or when they say 'commit this', 'what should the commit say', or 'git commit'.
  Only trigger when there are staged changes visible in context.
```

### `allowed-tools` (optional)

Comma-separated list of tools the skill can use. Omit to allow all tools.

```yaml
allowed-tools: "Read, Grep, Bash"
allowed-tools: "Read, Write, Edit"
allowed-tools: "WebSearch, WebFetch, Read"
```

Restricting tools prevents the skill from taking unexpected actions. A documentation skill that only reads files should not have `Bash` or `Write`.

Available tool names match those listed in the [agents documentation](agents.md#tools-optional).

### `argument-hint` (optional)

A short usage hint shown next to the skill name in autocomplete and the `/skills` menu.

```yaml
argument-hint: "[file-path]"
argument-hint: "<branch-name>"
argument-hint: "[--verbose]"
```

### `arguments` (optional)

Names of positional arguments the skill accepts. Placeholders in the skill body (`${argName}`) are replaced at invocation time.

```yaml
arguments: file
# or multiple:
arguments: [file, format]
```

In the skill body:

```markdown
Review the file at ${file} for correctness and style.
Output the result in ${format} format.
```

Invoke with:

```
/my-skill src/auth.ts markdown
```

### `model` (optional)

Override the model for this specific skill. If omitted, the skill uses the current session model.

```yaml
model: puku-ai-2.7
```

### `effort` (optional)

Controls how much reasoning the model applies.

```yaml
effort: low      # fast, minimal overhead
effort: medium   # balanced (default)
effort: high     # thorough — good for review, analysis, debugging
```

### `context` (optional)

Execution mode. Defaults to `inline`.

```yaml
context: inline   # runs in the current conversation (default)
context: fork     # runs in an isolated sub-agent with its own token budget
```

Use `inline` for short, focused tasks. Use `fork` for complex, long-running skills that should not consume the parent conversation's context window.

### `agent` (optional)

When `context: fork`, specifies which agent type to use for the sub-agent. Defaults to the general-purpose agent.

```yaml
agent: code-reviewer
```

### `disable-model-invocation` (optional)

When `true`, the model cannot invoke this skill automatically via the Skill tool. The skill can still be invoked manually with `/skill-name`.

```yaml
disable-model-invocation: true
```

### `user-invocable` (optional)

When `false`, the skill is hidden from the `/skills` menu and cannot be typed directly as a slash command. It can still be invoked by the model.

```yaml
user-invocable: false
```

### `paths` (optional)

Glob patterns that limit when the skill is suggested. The skill only becomes active after the model has touched files matching one of the patterns.

```yaml
paths: ["src/**/*.ts", "src/**/*.tsx"]
```

### `version` (optional)

Semantic version of the skill. Useful for tracking skill changes in version-controlled projects.

```yaml
version: "1.2.0"
```

---

## Invoking skills

### As a user

Type `/` followed by the skill name in the input prompt:

```
/summarize
/security-review src/auth/login.ts
/pr-description
```

Arguments follow the skill name, separated by spaces.

### Browse available skills

Type `/skills` to open the interactive skills browser. It lists all available skills grouped by source (project, personal, plugin, MCP) with their descriptions and argument hints. Select any skill to invoke it.

### The model invokes skills automatically

When the model determines that a skill is the right tool for a task, it invokes it via the **Skill tool** — without waiting for you to type the slash command. This is driven by the `description` and `when-to-use` fields.

To prevent a skill from being auto-invoked, set `disable-model-invocation: true`.

---

## Namespaced skills

Skills stored in nested subdirectories become namespaced with a colon:

```
.puku-cli/skills/review/security/SKILL.md   → /review:security
.puku-cli/skills/review/performance/SKILL.md → /review:performance
~/.puku-cli/skills/docs/jsdoc/SKILL.md       → /docs:jsdoc
```

Namespacing helps organize large skill libraries and prevents name collisions across teams or plugins.

---

## Argument substitution

Use `${argName}` anywhere in the skill body. The `arguments` frontmatter field defines the names in order.

```markdown
---
name: migrate
description: "Generate a database migration for a schema change."
argument-hint: "<table> <change-description>"
arguments: [table, change]
---

Generate a SQL migration for the `${table}` table with the following change:

${change}

Use the migration style already present in the `migrations/` directory.
Add rollback SQL in a `-- rollback` comment block at the end.
```

Invoke as:

```
/migrate users "add email_verified boolean not null default false"
```

Unrecognized arguments beyond the defined list are appended as-is to the end of the prompt body.

---

## Inline vs. forked execution

### Inline (default)

The skill's expanded content is injected directly into the current conversation. The model responds in context, sharing the same token budget and history.

Best for: quick lookups, short summaries, single-file operations.

### Forked

The skill runs in an isolated sub-agent. The parent conversation is not modified while the skill runs; only the final result is returned.

```yaml
context: fork
```

Best for: long analysis tasks, multi-file reviews, anything that would pollute the parent conversation's context.

Forked skills support the `agent` field to specify which agent handles execution, and they inherit the skill's `effort` and `allowed-tools` settings.

---

## MCP skills

MCP servers can expose **prompts**, which puku-cli surfaces as skills with the naming pattern:

```
<server-name>:<prompt-name>
```

For example, a GitHub MCP server that exposes a `create-pr` prompt appears as `/github:create-pr`.

MCP skills appear in the `/skills` menu under "MCP skills". They work identically to file-based skills from the user's perspective but are defined and maintained by the MCP server — you cannot edit their frontmatter or body locally.

**Security note:** puku-cli never executes shell commands (`!bash` inline execution) from MCP skill bodies. Only local, file-based skills support that capability.

---

## Built-in skills

puku-cli ships with several built-in skills:

| Skill | Description |
|---|---|
| `/loop` | Run a prompt or another skill on a recurring schedule |
| `/remember` | Save something to persistent memory |
| `/schedule` | Create a one-time or recurring scheduled remote agent |
| `/verify` | Run the app and confirm a change works as expected |
| `/simplify` | Analyze code for over-engineering and suggest simplifications |

Type `/skills` to see all currently available built-ins alongside your custom skills.

---

## Sharing skills with your team

Commit `.puku-cli/skills/` to your repository. Every team member who opens the project gets the same skills automatically.

For skills that reference environment-specific paths or secrets, use argument substitution (`${argName}`) or `${ENV_VAR}` expansion so each developer provides their own values at invocation time.

**Recommended structure for a team skill library:**

```
.puku-cli/
  skills/
    review/
      security/SKILL.md      → /review:security
      performance/SKILL.md   → /review:performance
    docs/
      jsdoc/SKILL.md         → /docs:jsdoc
      changelog/SKILL.md     → /docs:changelog
    git/
      commit/SKILL.md        → /git:commit
      pr-description/SKILL.md → /git:pr-description
```

---

## Common skill patterns

### Commit message writer

```markdown
---
name: commit
description: "Write a conventional commit message based on the currently staged diff."
allowed-tools: "Bash"
effort: low
---

Run `git diff --staged`. Write a conventional commit message for these changes.

Format: type(scope): subject
Types: feat, fix, refactor, docs, test, chore, perf
Rules: subject under 72 chars, imperative mood, no period at end.

Output only the commit message — no explanation, no preamble.
```

### Pull request description

```markdown
---
name: pr-description
description: "Generate a pull request title and description from the current branch's commits."
allowed-tools: "Bash, Read"
effort: medium
---

Run `git log main..HEAD --oneline` and `git diff main..HEAD -- . ':(exclude)*.lock'`.

Write a pull request description with:
- A short title (under 70 characters)
- A bullet-point summary of changes (3–5 bullets)
- A test plan checklist
- Any breaking changes, clearly marked

Use the format already in `.github/pull_request_template.md` if one exists.
```

### JSDoc generator

```markdown
---
name: jsdoc
description: "Generate JSDoc comments for the function or class the user has just written or pasted."
argument-hint: "[file-path]"
arguments: file
allowed-tools: "Read, Edit"
effort: low
---

Read ${file}. For every exported function, class, and method that is missing JSDoc, add it.

JSDoc must include:
- One-sentence summary
- @param for each parameter with type and description
- @returns with type and description
- @throws if the function can throw

Do not add JSDoc to private methods or internal helpers.
```

### Security review (forked)

```markdown
---
name: security-review
description: "Run a security review on the files the user specifies or on any recently modified files."
allowed-tools: "Read, Grep, Bash"
argument-hint: "[file-path]"
arguments: target
effort: high
context: fork
---

Security review target: ${target}

Check for:
- Injection (SQL, shell, template, XSS)
- Auth bypass and missing access control
- Secrets hardcoded in source
- Unvalidated input from external sources
- Unsafe file operations and path traversal

Output: numbered findings with severity (critical/major/minor), file:line, explanation, and fix.
End with APPROVED or CHANGES REQUIRED.
```

### Changelog entry

```markdown
---
name: changelog
description: "Write a changelog entry for the last set of commits since the previous release tag."
allowed-tools: "Bash"
effort: medium
---

Run `git log $(git describe --tags --abbrev=0)..HEAD --oneline`.

Write a changelog entry in Keep a Changelog format:

## [Unreleased]

### Added
- ...

### Changed
- ...

### Fixed
- ...

Group commits by type. Skip chore/ci commits. Use plain language, not git commit messages verbatim.
```

---

## Troubleshooting

**Skill is not appearing in `/skills`**
Check that the file is at `.puku-cli/skills/<name>/SKILL.md` (not `.puku-cli/commands/<name>.md`, which is deprecated). Verify the `name` field in frontmatter matches the expected command. Restart puku-cli after adding new skill files — skills are loaded at startup.

**`/skill-name` gives "command not found"**
The `name:` field in the SKILL.md frontmatter must match what you are typing, not the directory name. Check for typos in the frontmatter.

**Arguments are not substituted**
The `arguments:` field must list the argument name, and the skill body must use `${argName}` (exact match, case-sensitive). Check that you are passing arguments after the skill name when invoking.

**Model is not auto-invoking the skill**
Make sure `disable-model-invocation` is not `true`. Improve the `when-to-use` field with specific trigger phrases and conditions. If the description is too vague, the model may not recognize the match.

**Forked skill returns no output**
Verify `context: fork` is set. If an `agent:` is specified, confirm that agent exists in `.puku-cli/agents/` or `~/.puku-cli/agents/`. Check that `allowed-tools` does not exclude tools the skill body needs.

**Skill name conflict between project and personal**
The project scope takes precedence. Rename your personal skill or use a namespace (`category:skill`) to avoid collisions.

**MCP skill not appearing**
The MCP server must be connected. Check the server status with `puku-cli mcp doctor`. MCP skills only appear after a successful server connection.

---

## Reference

### Frontmatter fields

| Field | Required | Type | Description |
|---|---|---|---|
| `name` | Yes | string | Slash command name (3–50 chars, alphanumeric + hyphens) |
| `description` | Yes | string | Short description shown in the menu |
| `when-to-use` | No | string | Detailed trigger conditions for model auto-invocation |
| `allowed-tools` | No | string / array | Tools the skill may use |
| `argument-hint` | No | string | Usage hint shown in autocomplete |
| `arguments` | No | string / array | Positional argument names for `${substitution}` |
| `model` | No | string | Model override for this skill |
| `effort` | No | string | `low`, `medium`, or `high` |
| `context` | No | string | `inline` (default) or `fork` |
| `agent` | No | string | Agent type for forked execution |
| `disable-model-invocation` | No | boolean | Prevent model from auto-invoking |
| `user-invocable` | No | boolean | Hide from menu and slash-command list |
| `paths` | No | array | Glob patterns that gate skill visibility |
| `version` | No | string | Semantic version |

### File locations

| Path | Scope |
|---|---|
| `.puku-cli/skills/<name>/SKILL.md` | Project (committed) |
| `.puku-cli/skills/<category>/<name>/SKILL.md` | Project, namespaced |
| `~/.puku-cli/skills/<name>/SKILL.md` | Personal (all projects) |
| `~/.puku-cli/skills/<category>/<name>/SKILL.md` | Personal, namespaced |

### Session commands

| Command | Description |
|---|---|
| `/skills` | Browse all available skills |
| `/<skill-name>` | Invoke a skill |
| `/<skill-name> <arg1> <arg2>` | Invoke with arguments |
