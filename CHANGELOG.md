# Changelog

## [1.8.7]

### Features
- Implement reAuthentication flow for Google OAuth login and integrate it into the Login component

### Bug Fixes
- Update command reference in MCPSettings error message

---

## [1.8.6]

### Features
- Add `/remote-control` command and puku-cli-app React Native mobile app
- Enhance remote-control mobile app with tabs, session history, and relay improvements
- Implement reconnect token handling in RelayClient and RelaySession
- Enhance useQueueProcessor and useReplBridge for improved command handling and mobile client connection management
- Add client connection event handling to PukuRelayBridge
- Add thinking state management to useRelaySession and update ChatScreen for loading indication
- Enhance initPukuRelayBridge with thinking indicator and permission response handling from app

### Bug Fixes
- Correct capitalization of 'Esc' in user prompts for consistency
- Set defaultEnabled to false for smart-context plugin

### Refactor
- Refactor styles and theme usage across components for consistency

---

## [1.8.5]

### Features
- Switch official marketplace fetching from GCS to Cloudflare CDN
- Add support for legacy plugin manifest directories and improve compatibility
- Add chat usage tracking and hook

### Bug Fixes
- Update error handling for marketplace installation to reflect CDN unavailability
- Enhance compatibility with legacy plugin directories and improve marketplace loading
- Update greeting message in ThemePicker component
- Remove effort notification logic from PromptInput component

### Tests
- Add unit tests for plugin and marketplace validation

---

## [1.8.4]

### Features
- Enhance skill loading: add support for skills from `~/.agents/skills/` directory

### Refactor
- Refactor global config file handling: rename to `puku-cli.json` for better namespace isolation and clarity
- Refactor CI workflow: remove commented provider tests and adjust build step

### Dependencies
- Update lodash-es dependency from 4.18.0 to 4.17.23 for compatibility

---

## [1.8.3]

### Bug Fixes
- Fix `PukuStartup.tsx` getRecentSessions() function that incorrectly read from `~/.puku-cli/sessions/index.json` — sessions are actually stored as individual JSONL files in `~/.puku-cli/projects/<project>/`

---

## [1.8.2]

### Bug Fixes
- Update Puku logo representation and adjust related dimensions

---

## [1.8.1]

### Bug Fixes
- Streamline build process and enhance cross-platform compatibility for smart-context

---

## [1.8.0]

### Features
- Add `/remote-web` command with Cloudflare relay bridge
- Add sidebar colors to theme configuration
- Implement new ASCII logo and gradient color functions for startup screen
- Update onboarding message and GitHub app tagging to reflect Puku-CLI branding

### Bug Fixes
- Correct package name in package.json and update README with badges and usage instructions

---

## [1.7.0]

### Bug Fixes
- Fix long input text wrapping instead of truncating in BaseTextInput
- Correct textInputColumns for full-border input box layout

### Tests
- Add Cursor wrapping tests as regression guard for long-line fix

---

## [1.6.0]

### Cleanup
- Complete `.puku-cli/` path cleanup across the codebase

---

## [1.5.0]

### UI Improvements
- Replace `✻` with `◆` in all display strings
- Update question form symbols: `☐/☒` → `○/◆`, `❯` → `▸`, `✔` → `◆`
- Add sectionLabel/planModeTip theme tokens
- Apply muted blue to feed titles
- Sweep remaining Claude strings, question form pointer color


## [1.1.2]

### Features
- Add friendly display labels for routing redirects and puku_cli MCP tool rendering
- Finalize smart-context MCP tool labels + debrand `--version` output

---

## [1.0.11]

### Features
- Implement smart-context (Strategy 4 — hooks + system prompt)
- Ship smart-context as default built-in plugin with MCP server

### Bug Fixes
- Resolve SessionStart:startup hook error in smart-context

---

## [1.0.10]

### Features
- Add smart-context reference docs

---

## [1.0.9]

### Bug Fixes
- Bundle ripgrep in dev build

## [1.0.8]

### Bug Fixes
- Revert select-option color wrap to avoid Box-in-Text crash
- Remove large db file from tracking and fix /init path

---

## [1.0.7]

### Features
- Add PukuStartup Ink component + UI polish
- Rebrand UI — remove Claude footprint, puku-cli identity

---

## [1.0.6]

### Features
- Add tmux-cli-driver product verification skill
- Adopt subagent-per-test strategy in tmux-cli-driver skill
- Add verify-build, verify-ide, verify-provider skills
- Add verify-compaction skill with test loop

---

## [1.0.5]

### Features
- Replace Bash label with `$` for terminal-style command display
- Replace Claude shimmer spinner with pi-mono braille style
- Replace cooking verbs with puku playful verbs
- Prepare puku-cli-vscode for marketplace publishing

### Bug Fixes
- Use `~/.puku` config dir and correct IDE lockfile PID

---

## [0.1.2]

### Features
- Add puku logo, theme, name
- Add puku worker layer
- Google auth integrated with callback
- Puku dev worker connected for oauth

### Bug Fixes
- Update tagline
- Update title
- Update model name