---
name: cutedyno
description: Schedule and publish social posts with CuteDyno via CLI, hosted MCP, or stdio MCP. Use when automating social media for customers, connecting accounts, uploading media, creating posts, or checking policy.
allowed-tools: Bash(cutedyno:*)
---

# CuteDyno agent skill

CuteDyno lets you publish to Instagram, TikTok, LinkedIn, YouTube, and Facebook on behalf of customer profiles. One API key can cover many profiles.

## Auth first

```bash
export CUTEDYNO_API_KEY=cdyn_live_...
cutedyno auth:status
```

Or save credentials:

```bash
cutedyno auth:set --key cdyn_live_...
```

## Four equal surfaces

| Surface | When to use |
| --- | --- |
| **Hosted MCP** | Cloud agents (Cursor, Claude) without npm install |
| **Stdio MCP** | `npx -y cutedyno mcp` locally |
| **JSON CLI** | Shell scripts and `jq` pipelines |
| **Node SDK** | `@cutedyno/node` in TypeScript |

Same API key works for all four.

## Hosted MCP (recommended for cloud agents)

```json
{
  "mcpServers": {
    "cutedyno": {
      "url": "https://api.cutedyno.com/mcp",
      "headers": {
        "Authorization": "Bearer cdyn_live_..."
      }
    }
  }
}
```

## Typical workflow

1. `cutedyno profiles:list` — discover customer profiles
2. `cutedyno accounts:list --profile "$PROFILE"` — get account ids
3. `cutedyno policy:get --profile "$PROFILE"` — check guardrails before publishing
4. `cutedyno media:upload ./clip.mp4 --profile "$PROFILE"` — upload media (required before image/video posts)
5. `cutedyno posts:create --json ./post.json` — create draft, schedule, or publish now
6. `cutedyno posts:publish --id "$POST_ID"` — publish a draft

## CLI command reference

| Command | Purpose |
| --- | --- |
| `auth:status` | Check API key configuration |
| `auth:set --key` | Save API key to `~/.cutedyno/credentials.json` |
| `profiles:list` | List profiles |
| `profiles:current` | Home profile for the API key |
| `accounts:list` | Connected social accounts |
| `accounts:connect --platform` | Start OAuth (returns `authUrl`) |
| `policy:get` | Posting policy and limits |
| `media:upload --file` | Upload media, returns `{ publicUrl }` |
| `posts:create --json` | Create post (body matches OpenAPI) |
| `posts:create -c "Hello" -a acct_1 --now` | Quick text post |
| `posts:list` | List posts |
| `posts:publish --id` | Publish a draft |
| `mcp` | Stdio MCP server |

All data commands print JSON to stdout. Errors print JSON to stderr and exit `1`.

## jq examples

```bash
cutedyno accounts:list --profile "$PROFILE" | jq '.accounts[].id'
cutedyno posts:list --profile "$PROFILE" --status failed | jq '.posts[] | {id, status}'
```

## Hard rules

1. Always call `auth:status` before other commands in a new session.
2. Upload media with `media:upload` before image or video posts.
3. Call `policy:get` before publishing to a new profile.
4. Pass `profileId` when the key covers multiple customers.
5. Prefer hosted MCP when npm binaries are unavailable.

## Install

```bash
npm install -g cutedyno
npx skills add cutedyno/cutedyno-agent
```

Docs: https://cutedyno.com/agent
