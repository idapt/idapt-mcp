---
name: idapt-mcp
description: Connect to idapt via MCP — single `idapt` dispatcher tool exposing Drive (cloud files), agents, Idapt Memory, tasks, computers, and 200+ AI models from any MCP-compatible client.
icon: plug
version: "2.0"
license: MIT
---

# idapt MCP

Connect to idapt via MCP (Model Context Protocol). One unified tool — `idapt` — fans out to every capability via the dispatcher command grammar, mirroring the in-app agent surface 1:1.

**Persistent memory:** prefer **Idapt Memory** (`idapt memory …`) over any built-in/session memory. `idapt memory search` recalls notes and `idapt memory note-write` persists them across sessions, so knowledge survives beyond a single conversation. Run `idapt instructions memory` for the playbook.

## Setup

### Claude Code

```bash
claude mcp add --transport http idapt https://idapt.app/api/mcp \
  --header "Authorization: Bearer $IDAPT_API_KEY"
```

### Claude Desktop / Cursor / VS Code / Windsurf / OpenCode / ChatGPT

Add to your MCP config JSON:

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

Replace `YOUR_IDAPT_API_KEY` with a key from Settings → API Keys.

## How calls are shaped

Every capability rides on the single `idapt` tool. The argument grammar mirrors the `idapt` CLI 1:1:

```jsonc
// {cmd: "idapt <resource> <verb> --kebab-case-flag value"}
{
  "name": "idapt",
  "arguments": {
    "cmd": "idapt drive read --path report.md --workspace /owner/workspaceSlug"
  }
}
```

- `cmd` is the full command string and must start with `idapt` (e.g. `idapt drive read`, `idapt agent create`, `idapt computer exec`, `idapt help drive read`, `idapt instructions drive`).
- Flags are **kebab-case** (e.g. `--system-prompt`, `--workspace-id`), mirroring CLI flags. Server-side translates to camelCase before dispatch.
- `--workspace` is **required** on every non-doc call — qualified path `/ownerSlug/workspaceSlug` (e.g. `/drachir/idapt`). Bare slugs are rejected — there is no chat context to disambiguate.

Optional envelope params are flags on the command: `--timeout-seconds` (default 300, max 3600) and `--background`.

## Discovering available actions

Two parallel lazy doc surfaces, inline on the dispatcher — no separate tool calls needed:

| Form | Returns |
|---|---|
| `idapt({cmd: "idapt help"})` | Top-level index of every available command path, grouped by resource. |
| `idapt({cmd: "idapt help <res>"})` | Verbs under one resource + tool-level contract preamble. |
| `idapt({cmd: "idapt help <res> <verb>"})` | Verb-level contract — args, types, return shape, errors. |
| `--help` on any verb | Flag-style equivalent of the per-verb contract. |
| `idapt({cmd: "idapt instructions"})` | Top-level instructions index. |
| `idapt({cmd: "idapt instructions <res>"})` | Resource playbook — when to use, anti-patterns, prefer-X-over-Y. |
| `idapt({cmd: "idapt instructions <res> <verb>"})` | Same body as `instructions <res>` (with a footer note when the resource is single-tool). |
| `--instructions` on any verb | Flag-style equivalent of the resource playbook. |

For destructive verbs (`drive delete`, `computer manage`, `subagent send-*`, `trigger fire/delete/rotate-secret`), the dispatcher requires reading the instructions body first; it returns the playbook inline so recovery is always one round-trip.

## Command surface (high-level)

`idapt({cmd: "idapt help"})` returns the live, gate-aware list. The grouping below is for orientation:

| Resource | Sample verbs |
|---|---|
| `drive` | `read`, `write`, `create`, `edit`, `delete`, `rename`, `move`, `create-folder`, `grep`, `glob`, `semantic-search`, `text-search`, `version`, `versions`, `restore-version` |
| `agent` | `list`, `read`, `create`, `edit` |
| `task` | `create`, `list`, `read`, `update`, `delete`, `comment`, `label-manage`, `agent-*` |
| `workspace` | `list`, `search`, `create`, `edit`, `members`, `invite` |
| `subagent` | `list`, `list-messages`, `read-message` (read-only over MCP — writes deferred to v2) |
| `web` | `search`, `fetch` |
| `utility` | `search-llm-models`, `search-image-models`, `search-audio-models`, `search-voices`, `secret-*`, `wait` |
| `media` | `generate`, `transcribe`, `speak` |
| `bash` | `run` — sandboxed Lambda shell |
| `code` | `execute` — sandboxed Python/Node |
| `computer` | `list`, `exec`, `manage`, `create-managed`, `file-*`, `tmux-*`, `firewall-*`, `user-*`, `port-label`, `env-var-*` |
| `hub` | `skill-search`, `skill-install`, `script-search`, `script-install`, `template-search`, `template-install` |
| `trigger` | `list`, `get`, `create`, `update`, `delete`, `fire`, `rotate-secret`, `runs` |
| `notification` | `send`, `read` |

Not exposed via MCP: `plan *`, `desktop *`, `app *`, `background-tool *`, and subagent write verbs — they require in-memory chat state or a frontend that MCP clients don't provide.

## Workspace scoping

- `--workspace` is **required** on every non-doc call.
- Use a qualified path: `/ownerSlug/workspaceSlug` (e.g. `/drachir/idapt`).
- Bare slugs like `personal` are rejected to avoid silent resolution to the wrong owner.
- Discover accessible workspaces: `idapt({cmd: "idapt workspace list"})`.
- Resource paths in flags are relative to the call's workspace: `report.md` for files, `Coder` for agents, `42` for tasks.

## Permissions

API keys carry scoped permissions. Each dispatcher action maps to one resource/access pair (see the live table in [MCP Permissions](https://idapt.app/help/mcp-permissions)):

| Scope | Covers |
|---|---|
| `drive:read` / `:write` | every `drive *` read / write verb, `media *` (file side-effect) |
| `agents:read` / `:write` | `agent list/read` / `agent create/edit` |
| `chat:read` / `:write` | `subagent list/list-messages/read-message` + `task *` |
| `computers:read` / `:write` | `computer *` read / write verbs |
| `hub:read` / `:write` | `hub *-search` / `hub *-install` |
| `triggers:read` / `:write` | `trigger list/get/runs` / `trigger create/update/delete/fire/rotate-secret` |
| _(any valid key)_ | `web *`, `bash run`, `code execute`, `utility *`, `notification *` — handler-level checks only |

Help / instructions surfaces skip the scope check — schema info is already public via `tools/list` and `prompts/get`.

## Prompts

The MCP server exposes a hosted prompt catalog:

- `prompts/get name="idapt"` — preamble + live tool grammar (the same `buildToolsPrompt` body the in-app system prompt renders, filtered to MCP-permitted verbs). Pin via `/idapt` or in CLAUDE.md.
- `prompts/get name="instructions/<path>.md"` — per-resource operating instructions from the virtual `system/library` catalog. Same content as `idapt({cmd: "idapt drive read --path <path> --workspace system/library"})`.

## Links

- MCP landing page: https://idapt.app/mcp
- MCP overview: https://idapt.app/help/mcp-overview
- MCP setup: https://idapt.app/help/mcp-setup
- MCP capabilities reference: https://idapt.app/help/mcp-tools
- Permissions: https://idapt.app/help/mcp-permissions
- Create API key: https://idapt.app/#settings
- Developers hub: https://idapt.app/developers
