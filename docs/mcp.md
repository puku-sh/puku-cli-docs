# MCP (Model Context Protocol) in puku-cli

MCP connects puku-cli to external tools, data sources, and services through a standard protocol. Once a server is connected, its tools, resources, and prompts become available in your session just like built-in capabilities — puku-cli can read from databases, call APIs, query internal services, and more without any custom glue code.

---

## How it works

An MCP server is a process (local or remote) that exposes:

- **Tools** — callable functions (e.g. `query_database`, `create_github_issue`)
- **Resources** — readable content (e.g. file trees, documents, live data feeds)
- **Prompts** — pre-built conversation templates (e.g. code review workflows)

puku-cli connects to each configured server at startup, fetches its capabilities, and makes them available in your session. Tools appear as `mcp__<server-name>__<tool-name>` and can be called just like any other tool.

---

## Quick setup

### 1. Add a server

```bash
# Local subprocess (stdio)
puku-cli mcp add my-server -- npx -y @my-org/my-mcp-server

# Remote HTTP server
puku-cli mcp add my-api --transport http https://api.example.com/mcp
```

### 2. Verify the connection

```bash
puku-cli mcp doctor my-server
```

### 3. Use it

Start puku-cli. The server's tools and resources are automatically available in your session.

---

## Adding MCP servers

### stdio (local subprocess)

The default transport. puku-cli spawns the server process and communicates via stdin/stdout.

```bash
puku-cli mcp add <name> <command> [args...]
```

**Examples:**

```bash
# npm package
puku-cli mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/dir

# Python script
puku-cli mcp add my-tool -- python /path/to/server.py

# Compiled binary
puku-cli mcp add db-server -- /usr/local/bin/mcp-postgres --database mydb

# With environment variables
puku-cli mcp add vault-server -- npx -y @my-org/vault-mcp \
  -e VAULT_ADDR=https://vault.example.com \
  -e VAULT_TOKEN=s.xxxxxxx
```

**Options for `mcp add`:**

| Flag | Description |
|---|---|
| `-s, --scope <scope>` | Where to store the config: `local`, `project`, or `user` (default: `local`) |
| `-e, --env <key=value>` | Set environment variables (repeatable) |

### HTTP / SSE (remote server)

For remote servers that communicate over HTTP.

```bash
puku-cli mcp add <name> --transport http <url>
puku-cli mcp add <name> --transport sse  <url>
```

**With static headers:**

```bash
puku-cli mcp add my-api --transport http https://api.example.com/mcp \
  -H "Authorization: Bearer my-token" \
  -H "X-Tenant-ID: acme"
```

**With OAuth:**

```bash
puku-cli mcp add my-api --transport http https://api.example.com/mcp \
  --oauth-client-id my-client-id
```

puku-cli opens your browser for the OAuth flow, then caches the token in the platform keychain (macOS Keychain, Windows Credential Manager, or Linux `pass`). Tokens are refreshed automatically.

**Fixed redirect port for OAuth:**

```bash
puku-cli mcp add my-api --transport http https://api.example.com/mcp \
  --client-id my-id \
  --callback-port 8080
```

The redirect URI will be `http://localhost:8080/callback`.

**Options for remote transport:**

| Flag | Description |
|---|---|
| `--transport <type>` | `http` (streamable HTTP) or `sse` (Server-Sent Events) |
| `-H, --header <key=value>` | Static HTTP headers (repeatable) |
| `--oauth-client-id <id>` | OAuth client ID to initiate authorization_code + PKCE flow |
| `--oauth-callback-port <port>` | Fixed local port for OAuth redirect URI |
| `--xaa` | Use XAA (Cross-App Access) enterprise SSO for this server |

### WebSocket

> ⁠Websocket is not currently available. Available transports: stdio, http, sse


## Managing servers

### List all configured servers

```bash
puku-cli mcp list
```

Shows each server's name, transport, scope, and whether it is enabled.

### Get server details

```bash
puku-cli mcp get <name>
```

Shows detailed information about a specific server.

### Remove a server

```bash
puku-cli mcp remove <name>
```

Removes the server from whichever scope it was added to.

### Enable / disable a server

Inside a puku-cli session, use `/mcp` to open the server panel, then select a server to toggle it on or off.

Or use the subcommands:

```
/mcp enable <name>     # re-enable a disabled server
/mcp disable <name>    # disable without removing
/mcp reconnect <name>  # drop and re-establish the connection
```

To enable or disable all servers at once:

```
/mcp enable
/mcp disable
```

---

## Configuration files

MCP servers can be defined in multiple files. puku-cli merges them at startup using a fixed precedence order.

### Config scopes and precedence

| Scope | File | Who controls it |
|---|---|---|
| `enterprise` | Managed policy files | IT/platform team (highest precedence) |
| `user` | `~/.puku-cli/settings.json` | You, across all projects |
| `project` | `.mcp.json` in project root | Team, checked into the repo |
| `local` | `.puku-cli/settings.json` in project | You, for this project only |

When two scopes define a server with the same name, the higher-precedence scope wins.

### `.mcp.json` format

The recommended way to share MCP servers with your team. Commit this file to the repository.

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
    },
    "internal-api": {
      "type": "http",
      "url": "https://api.internal.example.com/mcp",
      "headers": {
        "X-Api-Key": "${INTERNAL_API_KEY}"
      }
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### `settings.json` format

Servers defined under `mcpServers` in any `settings.json` work the same way.

```json
{
  "mcpServers": {
    "my-server": {
      "type": "stdio",
      "command": "node",
      "args": ["/path/to/server.js"]
    }
  }
}
```

### Server config fields

Every server entry supports these fields:

| Field | Type | Description |
|---|---|---|
| `type` | string | Transport type: `stdio`, `http`, `sse`, `ws` |
| `command` | string | Executable to run (stdio only) |
| `args` | string[] | Arguments for the command (stdio only) |
| `env` | object | Extra environment variables (stdio only) |
| `url` | string | Server URL (http, sse, ws only) |
| `headers` | object | Static HTTP headers (http, sse, ws only) |
| `headersHelper` | string | Path to a module that generates headers dynamically |
| `oauth` | object | OAuth configuration (see [OAuth](#oauth)) |

**Environment variable expansion:** Use `${VAR_NAME}` or `$VAR_NAME` in any string value. If a referenced variable is missing at startup, puku-cli warns you and skips connecting to that server.

---

## Tools, resources, and prompts

### Tools

MCP tools appear in the conversation with the prefix `mcp__<server>__<tool>`.

```
mcp__filesystem__read_file
mcp__github__create_issue
mcp__internal-api__query_orders
```

puku-cli calls these automatically when it decides they are the right tool for a task, just like any other built-in tool. You can also ask for them directly:

```
Use mcp__filesystem__read_file to read src/main.ts
```

### Resources

Resources expose readable content — documents, live data, file trees. You can reference them in your session:

```
Read the resource at file:///workspace/schema.sql from the filesystem server
```

puku-cli uses `ListMcpResourcesTool` and `ReadMcpResourceTool` to browse and fetch resource content.

### Prompts

MCP prompts (called *skills* in puku-cli) are pre-built conversation templates. They appear as:

```
mcp__<server>__<prompt-name>
```

Invoke them from the prompt or via `/mcp` to see what prompts a server exposes.

---

## OAuth authentication

puku-cli implements RFC 8252 (OAuth 2.0 for native apps) with PKCE.

### Standard flow

1. Run `puku-cli mcp add my-api --transport http https://api.example.com/mcp --oauth-client-id <id>`
2. puku-cli opens your browser to the authorization endpoint
3. You log in and approve access
4. puku-cli stores the access token and refresh token in the platform keychain
5. On every subsequent start, puku-cli uses the cached token and refreshes it silently before expiry

No browser popup appears again after the first login, unless the refresh token expires or is revoked.

### Fixed redirect port

Some OAuth servers require a pre-registered redirect URI. Use `--oauth-callback-port` to specify the localhost port:

```bash
puku-cli mcp add my-api --transport http https://api.example.com/mcp \
  --oauth-client-id my-id \
  --oauth-callback-port 8080
```

The redirect URI will be `http://localhost:8080/callback`.

### Re-authenticating

If a server shows **Needs auth** in `/mcp`, select it and choose **Reconnect** to trigger a new OAuth flow.

---

## XAA (Cross-App Access) — enterprise SSO

XAA (also called SEP-990 / Cross-App Access) lets you log in once to your company IdP and silently authorize multiple MCP servers — no repeated browser popups.

### 1. Configure the IdP (one time)

```bash
puku-cli mcp xaa setup \
  --issuer https://login.example.com \
  --client-id my-idp-client-id
```

This stores the IdP details in `settings.xaaIdp`. If your IdP uses a confidential client, add `--client-secret`; puku-cli stores the secret in the platform keychain.

### 2. Add servers with XAA

```bash
puku-cli mcp add my-api --transport http https://api.example.com/mcp --xaa
```

### Runtime flow

1. First request: puku-cli opens your browser for IdP login (once)
2. The OIDC `id_token` is cached in the keychain
3. For each `--xaa` server, puku-cli exchanges the cached `id_token` silently — no browser popup
4. Token refresh happens automatically before expiry

---

## Enterprise policy

IT and platform teams can control which MCP servers are allowed using policy files in the enterprise-managed config.

### Allowlist

Only servers matching the allowlist can be added by users.

```json
{
  "allowedMcpServers": [
    { "name": "internal-api" },
    { "urlPattern": "https://*.example.com/*" },
    { "commandPattern": "npx -y @my-org/*" }
  ]
}
```

### Denylist

Servers matching the denylist are blocked even if they are on the allowlist.

```json
{
  "deniedMcpServers": [
    { "name": "blocked-server" },
    { "urlPattern": "https://untrusted.example.com/*" }
  ]
}
```

**Precedence:** denylist > allowlist > unrestricted.

Pattern matching uses `*` as a wildcard matching any characters.

### Managed servers

Servers defined in `managed-mcp.json` are injected at the `enterprise` scope and cannot be removed or disabled by users. Use this to guarantee that audit or security servers are always connected.

---

## Diagnostics

```bash
puku-cli mcp doctor [name]
```

Runs health checks on one server or all servers. Checks:

- Configuration source and scope
- Duplicate definitions across scopes
- Current connection state (connected, needs-auth, failed, pending, disabled)
- Missing environment variables
- Policy violations
- OAuth setup status

**Example output:**

```
✔ filesystem  connected  (project scope, stdio)
✘ internal-api  failed  (local scope, http)
  Finding: INTERNAL_API_KEY is not set. Add it to your environment.
✔ github  connected  (user scope, stdio)

Summary: 2 healthy, 1 blocking issue
```

**Config-only check** (skip live connection tests):

```bash
puku-cli mcp doctor --config-only
```

---

## The `/mcp` panel

Type `/mcp` in any session to open the interactive server panel.

The panel shows:

- All configured servers with their transport type and scope
- Connection status for each server (connected / needs auth / failed / disabled / pending)
- Tools, resources, and prompts exposed by each connected server
- Per-server actions: enable, disable, reconnect

Use the panel to troubleshoot a server that is not connecting, to toggle servers on or off for the current session, or to browse what capabilities are available.

---

## Importing from Claude Desktop

If you use Claude Desktop and have MCP servers configured there, import them with:

```bash
puku-cli mcp import-desktop
```

This reads Claude Desktop's config file and adds the servers to your user scope.

---

## Common server examples

### File system access

```bash
puku-cli mcp add filesystem npx -y @modelcontextprotocol/server-filesystem /path/to/workspace
```

Exposes read and write tools for the specified directory.

### GitHub

```bash
puku-cli mcp add github npx -y @modelcontextprotocol/server-github \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxx
```

Exposes tools for issues, pull requests, repositories, and search.

### PostgreSQL

```bash
puku-cli mcp add postgres npx -y @modelcontextprotocol/server-postgres \
  postgresql://user:pass@localhost/mydb
```

Exposes query and schema tools for your database.

### Brave Search

```bash
puku-cli mcp add brave-search npx -y @modelcontextprotocol/server-brave-search \
  -e BRAVE_API_KEY=your-key
```

Exposes a web search tool using the Brave Search API.

### Custom Python server

```bash
puku-cli mcp add my-tool python /path/to/server.py --port 8080
```

---

## Sharing servers with your team

Commit `.mcp.json` to your repository. Every team member who runs puku-cli in that directory gets the same servers automatically, with no manual setup.

For secrets, use `${ENV_VAR}` references in `.mcp.json` and document which variables each developer needs in their shell profile or `.env` file (which stays out of git).

```json
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://api.internal.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${INTERNAL_API_TOKEN}"
      }
    }
  }
}
```

---

## Troubleshooting

**Server shows "Failed" in `/mcp`**
Run `puku-cli mcp doctor <name>` for a detailed diagnosis. Common causes: wrong command path, missing environment variable, network unreachable.

**Server shows "Needs auth"**
Select the server in `/mcp` and choose **Reconnect** to start the OAuth flow. If using XAA, verify the IdP is configured with `puku-cli mcp xaa setup`.

**Tools are not appearing in the conversation**
Check that the server is "connected" (not "disabled" or "failed") in `/mcp`. Also verify the server actually exposes tools — some servers only expose resources or prompts.

**Environment variable not found**
puku-cli expands `${VAR}` at startup. Make sure the variable is set in the shell where you launch puku-cli, not just in your `.env` file (which is not automatically sourced).

**Server name conflicts between scopes**
The higher-precedence scope wins. Use `puku-cli mcp doctor` to see which scope's definition is active. To override a project server for local development, add a server with the same name at `local` or `user` scope.

**Policy blocked my server**
Your organization's enterprise policy is preventing the server from being added. Contact your IT or platform team to request an allowlist entry, or use an approved server URL pattern.

**OAuth token expired / re-prompting every session**
The refresh token may have been revoked. Clear the cached credentials from your platform keychain and re-run `puku-cli mcp add` to re-authorize.

---

## Reference

### CLI commands

| Command | Description |
|---|---|
| `puku-cli mcp add <name> <cmd> [args]` | Add a stdio server |
| `puku-cli mcp add <name> --transport http <url>` | Add an HTTP server |
| `puku-cli mcp add <name> --transport sse <url>` | Add an SSE server |
| `puku-cli mcp add <name> --transport ws <url>` | Add a WebSocket server |
| `puku-cli mcp remove <name>` | Remove a server |
| `puku-cli mcp list` | List all configured servers |
| `puku-cli mcp doctor [name]` | Diagnose server health |
| `puku-cli mcp doctor --config-only` | Check config without live tests |
| `puku-cli mcp xaa setup` | Configure enterprise IdP for XAA |
| `puku-cli mcp import-desktop` | Import servers from Claude Desktop |

### Session commands

| Command | Description |
|---|---|
| `/mcp` | Open the interactive server panel |
| `/mcp enable [name]` | Enable one or all disabled servers |
| `/mcp disable [name]` | Disable one or all servers |
| `/mcp reconnect <name>` | Drop and re-establish a connection |

### Transport types

| Type | Use case |
|---|---|
| `stdio` | Local subprocess — any executable that speaks MCP over stdin/stdout |
| `http` | Remote server with streamable HTTP |
| `sse` | Remote server with Server-Sent Events |
| `ws` | Remote server with WebSocket |

### Connection states

| State | Meaning |
|---|---|
| Connected | Server is reachable and capabilities are loaded |
| Needs auth | Server requires OAuth or XAA login |
| Failed | Connection attempt failed (check `mcp doctor`) |
| Pending | Reconnecting after a transient failure |
| Disabled | Manually disabled via `/mcp disable` |
