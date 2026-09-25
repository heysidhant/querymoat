# QueryMoat MCP Server Guide

QueryMoat provides an integrated **Model Context Protocol (MCP)** server so AI coding assistants (such as Claude Desktop, Cursor, Antigravity, or custom agents) can inspect and query your project databases safely without leaving your development environment.

## Architecture

The MCP server runs as a background service inside the VS Code Extension Host:
- Binds to `127.0.0.1` on an available local port (default starting port: `42100`).
- Supports JSON-RPC 2.0 and REST endpoints.
- Enforces read-only database query execution by default.
- Maps requests to specific workspaces via Project Identifiers.

## Configuring AI Assistants

The easiest way is the **plug icon** in the QueryMoat dashboard header → **Copy MCP configuration**, which copies a ready-made snippet for your client.

QueryMoat installs a small stdio MCP server at `~/.querymoat/mcp-server.js` and refreshes it whenever the extension starts, so the path in your client's config never changes between QueryMoat versions. The server forwards requests to the QueryMoat extension running for your project; keep the project open in your editor.

### Claude Code

```bash
claude mcp add --transport stdio querymoat node ~/.querymoat/mcp-server.js
```

### Cursor, VS Code, Windsurf, Claude Desktop

Add this to `.cursor/mcp.json`, `.mcp.json`, Windsurf's `mcp_config.json` or `claude_desktop_config.json`. JSON configs don't expand `~`, so use your full home path (on Windows, `C:\\Users\\<you>\\.querymoat\\mcp-server.js`):

```json
{
  "mcpServers": {
    "querymoat": {
      "command": "node",
      "args": ["/Users/<you>/.querymoat/mcp-server.js"]
    }
  }
}
```

The server finds your project from the directory the client starts it in. To point it at a specific project, add `"--project", "/path/to/project"` to `args` or set `QUERYMOAT_PROJECT`.

### HTTP

Clients that support Streamable HTTP can connect directly while the editor is open:

```json
{ "mcpServers": { "querymoat": { "type": "http", "url": "http://127.0.0.1:42100/mcp" } } }
```

The port starts at `42100`; if it is taken, QueryMoat uses the next free port. The dashboard shows the one in use.

## Exposed Tools

### `get-database-info`
Returns high-level metadata (engine type, name, host, server version).
```json
{
  "projectId": "myproject-1a2b3c4d"
}
```

### `get-tables`
Lists all tables and views available in the schema.
```json
{
  "projectId": "myproject-1a2b3c4d",
  "schema": "public"
}
```

### `get-table-schema`
Returns full column definitions, nullability, primary keys, and foreign keys.
```json
{
  "projectId": "myproject-1a2b3c4d",
  "table": "users"
}
```

### `get-table-rows`
Retrieves paginated table records with optional sorting and limit.
```json
{
  "projectId": "myproject-1a2b3c4d",
  "table": "users",
  "page": 1,
  "pageSize": 25
}
```

### `run-query`
Executes an arbitrary SQL query against the active project database.
```json
{
  "projectId": "myproject-1a2b3c4d",
  "query": "SELECT id, email, created_at FROM users WHERE status = 'active' LIMIT 10"
}
```

Response shape:
```json
{
  "columns": [{ "name": "id", "type": "Int" }],
  "rows": [{ "id": 1 }],
  "rowCount": 1,
  "durationMs": 12,
  "resultSets": [ { "columns": [], "rows": [], "rowCount": 0 } ],
  "sandboxed": true
}
```
- `columns` / `rows` / `rowCount` describe the **first** result set.
- `resultSets` is present only when the batch or procedure produced more than one result set, in order.
- `affectedRows` is present only for statements without a result set.
- `sandboxed` is `true` when the statement ran inside a rolled-back sandbox transaction.
- The submitted SQL is not echoed back.

> **Security Note**: When `querymoat.mcp.readOnly` is enabled (default), only `SELECT`/`WITH`/`EXPLAIN`/`SHOW`/`PRAGMA`/`DESCRIBE` statements run. Keywords inside string literals and comments do not trigger the guard; `SELECT ... INTO` and writes hidden after a read do. `EXEC`/`CALL` are rejected unless the target connection sets `"execPolicy": "sandbox"` in `.querymoatrc` (optionally narrowed by `execAllowlist`), in which case the call runs inside a transaction that is always rolled back. Dynamic SQL (`EXEC (...)`, `sp_executesql`, `xp_*`) is never sandboxed. The rejection message names the reason (`contains DELETE`, `execPolicy is 'deny'`, allowlist miss).
