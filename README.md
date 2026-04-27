<p align="center">
  <h1 align="center">idapt MCP</h1>
  <p align="center">Give any LLM powerful tools</p>
</p>

<p align="center">
  <a href="https://idapt.ai/mcp"><img src="https://img.shields.io/badge/MCP-Compatible-blue?style=flat-square" alt="MCP Compatible"></a>
  <a href="https://idapt.ai/pricing"><img src="https://img.shields.io/badge/Tier-Pro%20%2F%20Max-purple?style=flat-square" alt="Pro / Max"></a>
</p>

---

Connect **Claude Code**, **Cursor**, **VS Code**, **Windsurf**, **ChatGPT**, **OpenCode**, and any MCP-compatible tool to [idapt](https://idapt.ai) — your AI workspace with 200+ models, agents, files, tasks, and more.

**No install required.** Just point your tool at the endpoint and authenticate with an API key.

## Quick Start

### 1. Create an API key

Go to [idapt Settings](https://idapt.ai/#settings) and create a new API key with the permissions you need.

### 2. Configure your tool

Choose your tool below and add the configuration.

### 3. Load the idapt skill

Type `/idapt` in your first message to load tool instructions, or add this to your project's system instructions (e.g., `CLAUDE.md`):

```
Always load the /idapt MCP prompt at the start of each conversation and follow its instructions as system-level rules.
```

---

## Setup by Client

### Claude Code

```bash
claude mcp add --transport http idapt https://idapt.ai/api/mcp \
  --header "Authorization: Bearer $IDAPT_API_KEY"
```

### Claude Desktop

Add to your Claude Desktop config file (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "idapt": {
      "url": "https://idapt.ai/api/mcp",
      "transport": "http",
      "headers": {
        "Authorization": "Bearer YOUR_IDAPT_API_KEY"
      }
    }
  }
}
```

### Cursor

Add to `.cursor/mcp.json` in your project root:

```json
{
  "mcpServers": {
    "idapt": {
      "url": "https://idapt.ai/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:IDAPT_API_KEY}"
      }
    }
  }
}
```

> **Note:** Cursor has a ~40 tool limit across all MCP servers.

### VS Code (Copilot)

Add to `.vscode/mcp.json` in your project root:

```json
{
  "servers": {
    "idapt": {
      "url": "https://idapt.ai/api/mcp",
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
      "serverUrl": "https://idapt.ai/api/mcp",
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
      "url": "https://idapt.ai/api/mcp",
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
      "url": "https://idapt.ai/api/mcp",
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
| **Endpoint** | `https://idapt.ai/api/mcp` |
| **Transport** | Streamable HTTP (JSON-RPC 2.0) |
| **Auth Header** | `Authorization: Bearer YOUR_IDAPT_API_KEY` |

---

## Available Tools

| Tool | Description | Actions |
|------|-------------|---------|
| **file** | Manage project files | read, write, create, edit, delete, rename, move, create-folder, grep, glob, semantic-search, text-search |
| **agent** | Manage agent personas | list, read, create, edit |
| **task** | Project task management | create, list, read, update, delete, comment, label-manage, agent-* |
| **web** | Web search and URL fetching | search, fetch |
| **utility** | Models, secrets, and utilities | search-models, secret-*, wait |
| **subagent** | Browse chat history | list-chats, list-messages, read-message |
| **code-execution** | Sandboxed Python/Node | (direct) |
| **bash** | Sandboxed shell command execution | (direct) |
| **media** | Image generation and transcription | generate-image, transcribe |
| **project** | Project lookups | list, read |

---

## Permission Scopes

Each API key has granular permissions:

| Scope | Actions | Description |
|-------|---------|-------------|
| `chat` | read | Browse chat history and messages |
| `files` | read, write | Read and manage project files |
| `agents` | read, write | List, create, and edit agents |
| `tasks` | read, write | Manage project tasks and boards |
| `projects` | read | Access project metadata |
| `user` | read | Read user profile |

---

## Endpoint

```
POST https://idapt.ai/api/mcp
```

JSON-RPC 2.0 over Streamable HTTP. Supports `initialize`, `tools/list`, `tools/call`, `prompts/list`, `prompts/get`, and `ping`.

---

## Links

- [MCP Landing Page](https://idapt.ai/mcp) — Interactive setup guide
- [MCP Overview](https://idapt.ai/help/mcp-overview) — Full documentation
- [MCP Setup](https://idapt.ai/help/mcp-setup) — Detailed setup instructions
- [MCP Tools Reference](https://idapt.ai/help/mcp-tools) — All tools and actions
- [Developers](https://idapt.ai/developers) — API, CLI, and more
- [idapt CLI](https://github.com/idapt/idapt-cli) — Command-line tool
- [idapt API](https://github.com/idapt/idapt-api) — REST API
- [idapt](https://idapt.ai) — The AI workspace
- [Pricing](https://idapt.ai/pricing) — Plans and API key access
