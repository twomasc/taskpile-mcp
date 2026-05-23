# MCP Inspector

The [MCP Inspector](https://github.com/modelcontextprotocol/inspector) is Anthropic's official debugging client for any MCP server. Useful for verifying your own connection, exploring the tool list, and dry-running tool calls before wiring them into an agent.

## Run it

```bash
npx @modelcontextprotocol/inspector
```

This opens the Inspector UI in your browser.

## Connect to Taskpile

1. **Transport:** Streamable HTTP
2. **Server URL:** `https://taskpile.app/api/mcp`
3. Click **Connect**.

The Inspector handles the OAuth dance automatically — a browser tab will open for you to sign into Taskpile, then close. You'll land back in the Inspector with the connection live.

## What to try

- **List Tools** — should show 57 tools grouped by category (tasks, projects, tags, teams, integrations).
- **Call `whoami`** — sanity check the auth resolved to the right user.
- **Call `list_tasks` with `{ "status": "today" }`** — quickly see what's on your plate.
- **Call `create_task` with `{ "title": "Test from Inspector #inbox-test @mcp" }`** — verifies the `#project` and `@tag` shorthand parsing works.

If anything fails with a clear error message, that's the contract you should expect from your real client too — Inspector and production clients hit the same endpoint with the same auth.
