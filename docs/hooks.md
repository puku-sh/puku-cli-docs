# puku-cli Hooks

Hooks let you run your own shell scripts at specific points in puku-cli's lifecycle — before a tool runs, after it completes, when a session starts, and more. Use them to enforce policies, log activity, send notifications, or automatically inject context.

---

## Configuration

Hooks are defined in a `settings.json` file. puku-cli checks three locations in order; later sources override earlier ones:

| File | Scope |
|---|---|
| `~/.puku-cli/settings.json` | User-level — applies to every project |
| `.puku-cli/settings.json` | Project-level — checked into the repo |
| `.puku-cli/settings.local.json` | Local overrides — not checked into the repo |

### Basic structure

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/your-script.sh"
          }
        ]
      }
    ]
  }
}
```

Each hook event key (`PreToolUse`, `PostToolUse`, etc.) holds an array of **matchers**. A matcher specifies which tool or pattern triggers the hook, and which hook commands to run.

| Field | Required | Description |
|---|---|---|
| `matcher` | No | Regex matched against the tool name (e.g. `"Bash"`, `".*"`, `"FileEdit\|FileWrite"`). Omit to match all tools. |
| `hooks` | Yes | Array of hook command objects |

Each hook command object:

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `"command"` (shell script), `"prompt"`, `"agent"`, or `"http"` |
| `command` | Yes (for `"command"` type) | Path to the script or inline shell command |

---

## Hook Events

| Event | When it fires |
|---|---|
| `SessionStart` | Once when puku-cli starts a new session |
| `SessionEnd` | When a session ends |
| `UserPromptSubmit` | Each time the user submits a message |
| `PreToolUse` | Before any tool executes — can block execution |
| `PostToolUse` | After a tool succeeds |
| `PostToolUseFailure` | After a tool fails |
| `Stop` | When Claude stops responding (end of turn) |
| `StopFailure` | When Claude stops due to an error |
| `SubagentStart` | When a subagent (parallel task) is created |
| `SubagentStop` | When a subagent finishes |
| `PreCompact` | Before context compaction runs |
| `PostCompact` | After context compaction runs |
| `Notification` | When puku-cli emits a notification |
| `PermissionRequest` | When a permission dialog would appear (hook can approve/deny) |
| `PermissionDenied` | When the user denies a permission |
| `Setup` | During initial setup |
| `Elicitation` | When Claude asks the user a structured question |
| `ElicitationResult` | After the user answers a structured question |
| `CwdChanged` | When the working directory changes |
| `FileChanged` | When a watched file changes |
| `WorktreeCreate` | When a git worktree is created |
| `WorktreeRemove` | When a git worktree is removed |
| `ConfigChange` | When settings change |
| `InstructionsLoaded` | When CLAUDE.md instructions are loaded |
| `TaskCreated` | When a task is created |
| `TaskCompleted` | When a task completes |
| `TeammateIdle` | When a teammate agent becomes idle |

---

## Hook Script Basics

When puku-cli fires a hook, it runs your script and passes a JSON object to its **stdin**. The script can:

- **Do nothing** — exit 0, puku-cli continues normally.
- **Block** — exit with code 2 or write JSON with `"continue": false` to stdout.
- **Approve/deny a permission** — write JSON with `"decision": "approve"` or `"decision": "block"`.
- **Inject context** — write JSON with `"systemMessage"` or `"hookSpecificOutput.additionalContext"`.

### What puku-cli sends to stdin (PreToolUse example)

```json
{
  "session_id": "abc123",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf /tmp/work"
  }
}
```

### Exit codes

| Exit code | Meaning |
|---|---|
| `0` | Success — puku-cli continues |
| `2` | Block — puku-cli stops execution and shows stderr as the error message |
| Other non-zero | Non-blocking error — logged but does not stop execution |

---

## JSON Output Format

Your script can write a JSON object to **stdout** to control puku-cli's behavior. If both JSON stdout and a non-zero exit code are present, the JSON is processed first.

### Blocking a tool (PreToolUse)

```json
{
  "continue": false,
  "stopReason": "Cannot delete sensitive files like .env"
}
```

`stopReason` is shown to the user as the error message.

### Approving or denying with a decision field

```json
{
  "decision": "block",
  "reason": "This command is not allowed in production branches"
}
```

```json
{
  "decision": "approve"
}
```

### Adding context to Claude's next response

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "additionalContext": "Note: this project has a linting step that should run after file edits."
  }
}
```

### Adding a system-level warning message

```json
{
  "systemMessage": "Warning: you are about to modify a production database."
}
```

### Suppressing hook output from the transcript

```json
{
  "suppressOutput": true
}
```

### Async hooks (long-running scripts)

If your script needs more than a few seconds, signal that it is running asynchronously:

```json
{
  "async": true,
  "asyncTimeout": 60
}
```

`asyncTimeout` is in seconds (default: 60). The script must still write its final result to stdout before the timeout.

---

## Examples

### Block deletion of sensitive files

Save as `.puku-cli/scripts/block_sensitive_rm.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
CMD=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if echo "$CMD" | grep -qiE "(\.env|secret|credential|\.pem|\.key)"; then
  echo '{"continue": false, "stopReason": "Cannot delete sensitive files"}'
  exit 2
fi

exit 0
```

`settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/your/project/.puku-cli/scripts/block_sensitive_rm.sh"
          }
        ]
      }
    ]
  }
}
```

### Log every tool call

```bash
#!/bin/bash
INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name // "unknown"')
TIMESTAMP=$(date -Iseconds)
echo "$TIMESTAMP  $TOOL  $INPUT" >> ~/.puku-cli/tool-usage.log
exit 0
```

### Inject a reminder before file writes

```bash
#!/bin/bash
INPUT=$(cat)
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)

if [ "$BRANCH" = "main" ] || [ "$BRANCH" = "master" ]; then
  echo "{\"systemMessage\": \"Warning: you are editing files directly on $BRANCH.\"}"
fi

exit 0
```

`settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "FileEdit|FileWrite",
        "hooks": [
          {
            "type": "command",
            "command": "/your/project/.puku-cli/scripts/warn-main-branch.sh"
          }
        ]
      }
    ]
  }
}
```

### Run a linter after every file edit

```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

if [[ "$FILE" == *.ts ]] || [[ "$FILE" == *.tsx ]]; then
  npx eslint --fix "$FILE" 2>&1
fi

exit 0
```

`settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "FileEdit",
        "hooks": [
          {
            "type": "command",
            "command": "/your/project/.puku-cli/scripts/lint-after-edit.sh"
          }
        ]
      }
    ]
  }
}
```

### Run a script at session start

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/your/project/.puku-cli/scripts/session-setup.sh"
          }
        ]
      }
    ]
  }
}
```

No `matcher` is needed for `SessionStart` because there is no tool name to match.

---

## Hook Script Tips

**Read all of stdin before processing.** puku-cli writes the full JSON payload to stdin, so always read it fully:

```bash
INPUT=$(cat)
```

**Use `jq` to parse fields.** The payload is a JSON object:

```bash
TOOL=$(echo "$INPUT" | jq -r '.tool_name // empty')
CMD=$(echo "$INPUT" | jq -r '.tool_input.command // empty')
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
```

**Make scripts executable:**

```bash
chmod +x .puku-cli/scripts/your-script.sh
```

**Use absolute paths** in `command` fields. puku-cli may run from any working directory.

**Keep scripts fast.** Hooks run synchronously (unless async) and add latency to every tool call they match. For anything slow, use async mode.

---

## Debugging Hooks

If your hooks are not firing, check these common problems:

**1. Wrong file location** — puku-cli only reads `settings.json` from the three locations listed above. Confirm the file exists and is in the right place.

**2. JSON parse error** — Any syntax error in `settings.json` silently prevents loading. Validate with:

```bash
jq . .puku-cli/settings.json
```

**3. Invalid hook type** — `type` must be exactly `"command"`, `"prompt"`, `"agent"`, or `"http"`. Any other value causes that hook entry to be dropped.

**4. Script not executable** — On macOS/Linux, run `chmod +x /path/to/your-script.sh`.

**5. Wrong matcher regex** — The `matcher` value is a regex matched against the tool name. `"Bash"` matches exactly; `".*"` matches all tools. Test with:

```bash
echo "Bash" | grep -E "YourMatcherHere"
```

**6. Check debug logs** — Run puku-cli with verbose logging to see hook activity:

```bash
puku-cli --verbose
```

Look for lines like:
```
Settings hooks: captured 2 matcher(s) across 1 event(s) (PreToolUse)
Hook matchers for PreToolUse: 2 total (2 settings, 0 registered, 0 session)
```

---

## Full `settings.json` Reference

A settings file with multiple hook events:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/your/project/.puku-cli/scripts/session-init.sh"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/your/project/.puku-cli/scripts/pre-bash.sh"
          }
        ]
      },
      {
        "matcher": "FileEdit|FileWrite",
        "hooks": [
          {
            "type": "command",
            "command": "/your/project/.puku-cli/scripts/pre-file-edit.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "/your/project/.puku-cli/scripts/audit-log.sh"
          }
        ]
      }
    ]
  }
}
```
