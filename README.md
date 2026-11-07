# wechat-mcp-server

MCP server and Claude Code channel for [wechat-cli](https://github.com/OliviaY1/wechat-cli). It gives an AI session tools for WeChat, and pushes new messages into a running Claude Code session. All WeChat work happens in `wechat-cli`.

```text
wechat-cli watch ──> SQLite ──> events --follow ──> this server ──> Claude Code session
                                                         │
                       wechat-cli send, read, history <──┘
```

`wechat-cli watch` runs on its own and must already be running. This server is started by Claude Code and stops with the session. A saved cursor means messages that arrive while it's stopped are pushed on the next start. The first ever start begins from the newest event.

## Setup

Needs Node 23.6 or later, which runs `server.ts` directly.

```sh
npm install
```

Add this to `.mcp.json`, with your own paths:

```json
{
  "mcpServers": {
    "wechat-mac": {
      "command": "node",
      "args": ["/path/to/wechat-mcp-server/server.ts"],
      "env": { "WECHAT_CLI": "/path/to/wechat-cli" }
    }
  }
}
```

Start Claude Code with the channel enabled. It shows a confirmation screen, and only works in an interactive session:

```sh
claude --dangerously-load-development-channels server:wechat-mac
```

Other MCP clients can use the tools, and `wechat_events` in place of channel push.

## Tools

| Tool | What it does |
|---|---|
| `wechat_status` | Reports whether WeChat is ready |
| `wechat_chats` | Lists chat names |
| `wechat_read` | Reads recent messages live from WeChat |
| `wechat_send` | Sends text to a chat |
| `wechat_history` | Reads saved messages |
| `wechat_search` | Searches saved messages |
| `wechat_backfill` | Starts saving older messages in the background |
| `wechat_events` | Reads queued events after a cursor |

## Push

Every message from someone else is pushed as `<channel source="wechat-mac" chat="..." sender="...">`. Your own messages are never pushed, so Claude doesn't answer itself. Gap events aren't pushed, but `wechat_events` returns them.

To push only some chats, set `WECHAT_PUSH_CHATS` to a JSON array of exact names. This doesn't limit who Claude can message.

```json
"env": { "WECHAT_PUSH_CHATS": "[\"Exact Chat Name\"]" }
```

## Test results (2026-11-07)

Tested with a scripted MCP client against the real CLI and WeChat. Push was tested by adding fake events and sending the real notification.

| Test | Result |
|---|---|
| `initialize` declares `claude/channel` and returns instructions | Pass |
| All 8 tools | Pass |
| Push of a message from someone else (0.117 s) | Pass |
| Own message not pushed | Pass |
| Catch-up after restart | Pass |
| `WECHAT_PUSH_CHATS` filter | Pass |
| Follower stops with the server | Pass |

## Not tested yet

- A real Claude Code session receiving a push.
- A real incoming message going through `watch` to Claude.
