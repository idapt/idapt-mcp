<p align="center">
  <h1 align="center">idapt MCP</h1>
  <p align="center">Give any LLM powerful tools — one unified dispatcher</p>
</p>

<p align="center">
  <a href="https://idapt.app/mcp"><img src="https://img.shields.io/badge/MCP-Compatible-blue?style=flat-square" alt="MCP Compatible"></a>
  <a href="https://idapt.app/pricing"><img src="https://img.shields.io/badge/Tier-Pro%20%2F%20Max-purple?style=flat-square" alt="Pro / Max"></a>
</p>

---

Connect **Claude Code**, **Cursor**, **VS Code**, **Windsurf**, **ChatGPT**, **OpenCode**, and any MCP-compatible tool to [idapt](https://idapt.app) — your AI workspace with 200+ models, agents, Drive (cloud files), tasks, computers, and more.

**No install required.** Point your tool at the endpoint and authenticate with an API key.

**One tool slot, full surface.** idapt exposes a single MCP tool — `idapt` — that fans out to every capability via a dispatcher command grammar. Frees the rest of your client's tool budget for other MCPs and keeps the tool count predictable.

## Quick Start

### 1. Create an API key

Go to [idapt Settings](https://idapt.app/#settings) and create a new API key with the permissions you need.

### 2. Configure your tool

Choose your tool below and add the configuration.

### 3. Load the idapt skill

Type `/idapt` in your first message to load the live tool grammar + instructions, or pin it in your workspace's system instructions (e.g. `CLAUDE.md`):

```
Always load the /idapt MCP prompt at the start of each conversation and follow its instructions as system-level rules.
```

---

## Setup by Client

### Claude Code

```bash
claude mcp add --transport http idapt https://idapt.app/api/mcp \
  --header "Authorization: Bearer $IDAPT_API_KEY"
```

### Claude Desktop

Add to your Claude Desktop config file (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "idapt": {
      "url": "https://idapt.app/api/mcp",
      "transport": "http",
      "headers": {
        "Authorization": "Bearer YOUR_IDAPT_API_KEY"
      }
    }
  }
}
```

### Cursor

Add to `.cursor/mcp.json` in your workspace root:

```json
{
  "mcpServers": {
    "idapt": {
      "url": "https://idapt.app/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:IDAPT_API_KEY}"
      }
    }
  }
}
```

> idapt takes a single slot in Cursor's MCP tool budget — every capability is reachable via the one `idapt` tool.

### VS Code (Copilot)

Add to `.vscode/mcp.json` in your workspace root:

```json
{
  "servers": {
    "idapt": {
      "url": "https://idapt.app/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:IDAPT_API_KEY}"
      }
    }
  }
}
```

### Windsurf

Add an MCP server in Windsurf settings:

```json
{
  "mcpServers": {
    "idapt": {
      "serverUrl": "https://idapt.app/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:IDAPT_API_KEY}"
      }
    }
  }
}
```

### OpenCode

Add to `opencode.json`:

```json
{
  "mcpServers": {
    "idapt": {
      "url": "https://idapt.app/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:IDAPT_API_KEY}"
      }
    }
  }
}
```

### ChatGPT

Add as an MCP server in ChatGPT settings:

```json
{
  "mcpServers": {
    "idapt": {
      "url": "https://idapt.app/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_IDAPT_API_KEY"
      }
    }
  }
}
```

### Other MCP Client

Use these connection details with any MCP-compatible tool:

| Setting | Value |
|---------|-------|
| **Endpoint** | `https://idapt.app/api/mcp` |
| **Transport** | Streamable HTTP (JSON-RPC 2.0) |
| **Auth Header** | `Authorization: Bearer YOUR_IDAPT_API_KEY` |

---

## How calls are shaped

Every capability rides on the single `idapt` tool:

```jsonc
{
  "name": "idapt",
  "arguments": {
    "cmd": "idapt drive read --path report.md --workspace /owner/workspaceSlug"
  }
}
```

- `cmd` is the full command string and must start with `idapt`, matching the `idapt` CLI 1:1 — `idapt drive read`, `idapt agent create`, `idapt computer exec`, `idapt help`, `idapt instructions drive`, etc.
- Flags are kebab-case (mirroring CLI flags); the server translates to camelCase before dispatch.
- `--workspace` is **required** on every non-doc call — qualified path `/ownerSlug/workspaceSlug`. Bare slugs are rejected.

Discover available actions via the dispatcher's lazy doc surfaces (no separate tool calls):

| Call | Returns |
|---|---|
| `idapt({cmd: "idapt help"})` | Top-level command index, grouped by resource |
| `idapt({cmd: "idapt help <res>"})` | Verbs under one resource |
| `idapt({cmd: "idapt help <res> <verb>"})` | Per-verb contract (args, types, errors) |
| `idapt({cmd: "idapt instructions <res>"})` | Resource playbook (when, why, anti-patterns) |
| `--help` / `--instructions` | Flag-style equivalents on any verb |

For destructive verbs (`drive delete`, `computer manage`, `subagent send-*`, `trigger fire/delete/rotate-secret`), the dispatcher requires reading the instructions body first; the body returns inline so recovery is one round-trip.

---

## Command surface (orientation)

| Resource | Sample verbs |
|---|---|
| **drive** | read, write, create, edit, delete, rename, move, create-folder, grep, glob, semantic-search, text-search, version, versions, restore-version |
| **agent** | list, read, create, edit |
| **task** | create, list, read, update, delete, comment, label-manage, agent-* |
| **workspace** | list, search, create, edit, members, invite |
| **subagent** | list, list-messages, read-message (read-only over MCP) |
| **web** | search, fetch |
| **utility** | search-llm-models, search-image-models, search-audio-models, search-voices, secret-*, wait |
| **media** | generate, transcribe, speak |
| **bash** | run — sandboxed Lambda shell |
| **code** | execute — sandboxed Python/Node |
| **computer** | list, exec, manage, create-managed, file-*, tmux-*, firewall-*, user-*, port-label, env-var-* |
| **hub** | skill-search, skill-install, script-search, script-install, template-search, template-install |
| **trigger** | list, get, create, update, delete, fire, rotate-secret, runs |
| **notification** | send, read |

Not exposed via MCP: `plan *`, `desktop *`, `app *`, `background-tool *`, and subagent write verbs — they require in-memory chat state or a frontend that MCP clients don't provide.

---

## Permission Scopes

Each API key carries scoped permissions. Help / instructions surfaces skip the scope check (schema info is already public via `tools/list` and `prompts/get`).

| Scope | Sample command paths |
|---|---|
| `drive:read` / `:write` | `drive *` read / write verbs, `media *` (file side-effect) |
| `agents:read` / `:write` | `agent list/read` / `agent create/edit` |
| `chat:read` / `:write` | `subagent list/list-messages/read-message`, `task *` |
| `computers:read` / `:write` | `computer *` read / write verbs |
| `hub:read` / `:write` | `hub *-search` / `hub *-install` |
| `triggers:read` / `:write` | `trigger list/get/runs` / `trigger create/update/delete/fire/rotate-secret` |
| `user:read` | required for MCP route access |
| _(any valid key)_ | `web *`, `bash run`, `code execute`, `utility *`, `notification *` — handler-level checks only |

---

## Endpoint

```
POST https://idapt.app/api/mcp
```

JSON-RPC 2.0 over Streamable HTTP. Supports `initialize`, `tools/list`, `tools/call`, `prompts/list`, `prompts/get`, and `ping`.

---

## Links

- [MCP Landing Page](https://idapt.app/mcp) — Interactive setup guide
- [MCP Overview](https://idapt.app/help/mcp-overview) — Full documentation
- [MCP Setup](https://idapt.app/help/mcp-setup) — Detailed setup instructions
- [MCP Capabilities Reference](https://idapt.app/help/mcp-tools) — Live command paths
- [Permissions](https://idapt.app/help/mcp-permissions) — Scope → command mapping
- [Developers](https://idapt.app/developers) — API, CLI, and more
- [idapt CLI](https://github.com/idapt/idapt-cli) — Command-line tool with matching command grammar
- [idapt API](https://github.com/idapt/idapt-api) — REST API
- [idapt](https://idapt.app) — The AI workspace
- [Pricing](https://idapt.app/pricing) — Plans and API key access
