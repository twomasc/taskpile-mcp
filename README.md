# Taskpile MCP

Public documentation for **Taskpile**'s Model Context Protocol server. Taskpile is an AI-first task manager designed to be used through your chat client — capture tasks in Claude, plan in ChatGPT, finish in Taskpile.

- App: <https://taskpile.app>
- MCP endpoint: `https://taskpile.app/api/mcp`
- Transport: Streamable HTTP (JSON-RPC 2.0)
- Protocol versions supported: `2025-06-18`, `2025-03-26`, `2024-11-05`

This repository contains **only public-facing documentation** for the MCP integration. The Taskpile app itself is a hosted commercial product — its source isn't here, but everything you need to connect a client and use the MCP tools is.

## Connecting

Any MCP-compatible client can connect by pointing at the URL above. Authentication uses OAuth 2.1 with Dynamic Client Registration + PKCE — the client handles this automatically; users sign in to Taskpile in a browser tab when prompted.

Verified clients:

| Client | Status |
| --- | --- |
| Claude.ai (Connectors) | ✅ |
| Claude Desktop (via `mcp-remote`) | ✅ |
| Claude Code | ✅ |
| ChatGPT (Connectors) | ✅ |
| Le Chat (Mistral) | ✅ |
| Cursor | ✅ |
| Zed | ✅ |
| MCP Inspector | ✅ |
| Lovable.app | ✅ |

## Quick start by client

See [`examples/`](./examples) for ready-to-paste config snippets:

- [Claude Desktop](./examples/claude-desktop.json)
- [Cursor](./examples/cursor.json)
- [Cline (VS Code)](./examples/cline.json)
- [MCP Inspector](./examples/mcp-inspector.md)

For hosted clients like Claude.ai and ChatGPT, add Taskpile via their built-in connector UI and paste the URL above; no local config needed.

## Tools

Taskpile exposes **57 tools** spanning task CRUD, projects, tags, search, bulk operations, teams, and account integrations like morning digest. The full machine-readable schema (names, descriptions, JSON Schema for inputs) is in [`tools.json`](./tools.json) — auto-generated from the live server, so it's always in sync with what `tools/list` returns.

A quick taste:

- `create_task`, `list_tasks`, `update_task`, `complete_task`, `delete_task`
- `assign_horizon` (today_morning, tomorrow, next_week, etc.), `delegate_task`, `set_recurrence`
- `list_projects`, `create_project`, `archive_project`
- `search_tasks` (free-text + status/horizon filters)
- `bulk_complete`, `bulk_delete`, `bulk_assign_horizon` (up to 200 ids per call)
- `changes_since` (incremental sync), `get_today`, `get_review_queue`
- `get_morning_digest`, `configure_morning_digest`
- `whoami`, `get_inbox_email`
- Teams: `create_team`, `invite_to_team`, `accept_invitation`, `assign_task`, `convert_project_to_shared`, etc.
- `search` and `fetch` (ChatGPT Connectors built-in tools, mapped to Taskpile)

## Conventions

A few project-wide conventions worth knowing if you're writing a prompt:

- **Tags use `@tag_name`. Projects use `#project_name`.** Do NOT use `#` for tags — `#foo` is silently treated as a project name.
- `create_task` parses `#project` and `@tag` shorthand from the title. Only the FIRST `#` token becomes the project; the rest are stripped.
- To set tags at creation, pass `tags: string[]` and/or put `@tag` in the title. Both forms work and are de-duplicated.
- `update_task` does **not** parse `#`/`@` shorthand from the title. Use `tags` / `addTags` / `removeTags` and `projectId` to change those.
- Prefer dedicated verbs over `update_task` when they exist: `complete_task`, `assign_horizon`, `delegate_task`.
- Bulk endpoints accept up to 200 ids per call.

## Auth flow

If you're building a client and want to know what to expect:

1. Client sends an unauthenticated request to `https://taskpile.app/api/mcp` and receives `401` with `WWW-Authenticate: Bearer realm="taskpile", resource_metadata="https://taskpile.app/.well-known/oauth-protected-resource"`.
2. Client fetches `https://taskpile.app/.well-known/oauth-protected-resource` and `…/oauth-authorization-server` to discover endpoints.
3. Client POSTs to `https://taskpile.app/oauth/register` for Dynamic Client Registration (RFC 7591).
4. Client kicks off the standard authorization-code + PKCE flow, opens a browser at `/oauth/authorize`, exchanges the code at `/oauth/token`, and receives an `access_token`.
5. Subsequent requests carry `Authorization: Bearer <token>`. Tokens are valid for 30 days; no refresh-token flow — clients re-run the authorize/token dance on expiry.

`HEAD /api/mcp` returns `200` without auth and is intended for liveness probes only.

## Privacy + data handling

- Taskpile is GDPR-compliant. Tasks, projects, and tags created via MCP are user-owned and visible only to the authenticated user (and their team members for shared projects).
- The MCP integration sends task content to Taskpile's servers (EU-hosted). It does NOT route through Anthropic, OpenAI, or any other LLM provider — those are *clients* of the MCP server, not the server itself.
- Personal access tokens can be created in Taskpile → Settings → Integrations and revoked at any time.

## Issues and feedback

This repo is for documentation. Bug reports about the MCP server, feature requests for new tools, or questions about the API — please open an issue here and we'll triage.

For Taskpile app issues (unrelated to MCP), please contact us through the app at <https://taskpile.app>.

## License

Documentation in this repository is MIT-licensed (see `LICENSE`). The Taskpile app itself is a hosted commercial product and is not licensed for redistribution.
