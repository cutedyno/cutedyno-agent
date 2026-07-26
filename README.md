# cutedyno-mcp

MCP server for [CuteDyno](https://cutedyno.com). Gives an AI agent tools to connect social accounts, draft and schedule posts, publish them, and read back what happened.

Works with any MCP client: Claude Desktop, Claude Code, Cursor, VS Code, Windsurf, Cline, and anything else that speaks the protocol.

## Setup

Create an API key at [cutedyno.com/dashboard/api](https://cutedyno.com/dashboard/api), then add the server to your client config.

**Claude Desktop** — `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "cutedyno": {
      "command": "npx",
      "args": ["-y", "cutedyno-mcp"],
      "env": { "CUTEDYNO_API_KEY": "cdyn_live_..." }
    }
  }
}
```

**Cursor** — `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "cutedyno": {
      "command": "npx",
      "args": ["-y", "cutedyno-mcp"],
      "env": { "CUTEDYNO_API_KEY": "cdyn_live_..." }
    }
  }
}
```

**Claude Code**:

```bash
claude mcp add cutedyno --env CUTEDYNO_API_KEY=cdyn_live_... -- npx -y cutedyno-mcp
```

## Environment

| Variable | Required | Default |
| --- | --- | --- |
| `CUTEDYNO_API_KEY` | yes | — |
| `CUTEDYNO_API_URL` | no | `https://api.cutedyno.com` |

Set `CUTEDYNO_API_URL` to point at a local backend during development.

## Hosted transport

If your client supports streamable HTTP, skip the npm package entirely:

```json
{
  "mcpServers": {
    "cutedyno": {
      "url": "https://api.cutedyno.com/mcp",
      "headers": { "Authorization": "Bearer cdyn_live_..." }
    }
  }
}
```

## Tools

Run `npx cutedyno-mcp --tools` for the current list. Every tool maps onto one REST endpoint, so anything an agent can do is also available over HTTP.

| Group | Tools |
| --- | --- |
| Profiles | `list_profiles`, `create_profile`, `get_profile`, `update_profile` |
| Accounts | `list_accounts`, `get_connect_url`, `get_connect_session` |
| Posts | `list_posts`, `get_post`, `create_post`, `update_post`, `cancel_post`, `validate_post`, `publish_post`, `retry_post`, `get_post_history` |
| Comments | `list_comments` |
| Media | `upload_media` |
| Webhooks | `list_webhooks`, `create_webhook`, `test_webhook`, `list_webhook_deliveries`, `delete_webhook` |
| Policy | `get_policy`, `update_policy` |
| Observability | `list_events`, `list_logs` |

## Working with several customers

One API key can cover many profiles. Ask the agent to call `list_profiles`, then pass `profileId` to every other tool. Omit `profileId` and the key's own profile is used, which is what a single-account setup wants.

## Guardrails

Each profile carries a policy: whether human approval is required before publishing, which platforms are allowed, and a daily post cap. Tools respect it, and `get_policy` lets an agent check before it tries. Scope keys narrowly in the dashboard when handing them to an agent.

## Using it as a library

```ts
import { createCuteDynoMcpServer } from 'cutedyno-mcp';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = createCuteDynoMcpServer({ apiKey: process.env.CUTEDYNO_API_KEY! });
await server.connect(new StdioServerTransport());
```

## Links

- [MCP docs](https://cutedyno.com/docs/mcp)
- [Tool reference](https://cutedyno.com/docs/mcp/tools)
- [REST API reference](https://cutedyno.com/docs/api)

MIT
