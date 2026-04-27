---
name: idapt-mcp
description: Connect to idapt via MCP to access files, agents, knowledge bases, tasks, and 200+ AI models from any MCP-compatible tool.
icon: plug
version: "1.0"
license: MIT
---

# idapt MCP

Connect to idapt via MCP (Model Context Protocol). Access files, agents, knowledge bases, tasks, and 200+ AI models from Claude Code, Cursor, VS Code, and other MCP-compatible tools.

## Setup

### Claude Code

```bash
claude mcp add idapt -- curl -N -s -H "x-api-key: uk_your_key_here" -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"claude","version":"1.0"}}}' https://idapt.ai/api/mcp
```

### Claude Desktop / Cursor / VS Code

Add to your MCP config JSON:

```json
{
  "mcpServers": {
    "idapt": {
      "command": "curl",
      "args": ["-N", "-s", "-H", "x-api-key: uk_your_key_here", "-H", "Content-Type: application/json", "--data", "{\"jsonrpc\":\"2.0\",\"method\":\"initialize\",\"params\":{\"protocolVersion\":\"2024-11-05\",\"capabilities\":{},\"clientInfo\":{\"name\":\"client\",\"version\":\"1.0\"}}}", "https://idapt.ai/api/mcp"]
    }
  }
}
```

Replace `uk_your_key_here` with your API key from Settings > API Keys.

## Tools

All tools use an `action` parameter for routing. Use the `project` parameter to scope operations (defaults to personal project).

| Tool | Actions | Description |
|------|---------|-------------|
| file | read, write, create, edit, delete, rename, move, create-folder, grep, glob, search | File operations |
| agent | list, read, create, edit | Agent management |
| task | create, list, read, update, delete, comment, label | Task management |
| kb | search, create, list, read, edit, delete, note | Knowledge base |
| project | list, read | Project info |
| web | search, fetch | Web search and fetch |
| media | generate-image, transcribe | Image gen and audio |
| code-execution | (direct) | Run Python/Node.js |
| bash | (direct) | Run shell commands |
| subagent | list-chats, list-messages, read-message | Subagent |
| chat | list, read, messages | Chat history |
| utility | search-models, wait | Utilities |
| trigger | list, get, create, update, delete, fire, rotate-secret, runs | Cron + webhook automations |
| notification | send, read | Notify project members + read your inbox |

## Key Patterns

- **Project scoping**: Set `project` to your project slug (e.g., `personal`, `my-project`) or qualified path (`/owner/project`)
- **Resource paths**: Use short names: `report.md` for files, `Coder` for agents, `42` for tasks
- **Discover projects**: `project(action: "list")` to see accessible projects

## Permissions

API keys need these scopes for MCP tools:

| Scope | Tools |
|-------|-------|
| files | file, kb, search |
| agents | agent |
| chat | chat, subagent |
| completions | code-execution, media |
| user | utility, project, notification |
| triggers | trigger |

## Links

- MCP landing page: https://idapt.ai/mcp
- MCP overview: https://idapt.ai/help/mcp-overview
- MCP setup guide: https://idapt.ai/help/mcp-setup
- MCP tools reference: https://idapt.ai/help/mcp-tools
- Create API key: https://idapt.ai/#settings
- Developers hub: https://idapt.ai/developers
