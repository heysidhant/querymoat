# QueryDock MCP Server Guide

QueryDock provides an integrated **Model Context Protocol (MCP)** server so AI coding assistants (such as Claude Desktop, Cursor, Antigravity, or custom agents) can inspect and query your project databases safely without leaving your development environment.

## Architecture

The MCP server runs as a background service inside the VS Code Extension Host:
- Binds to `127.0.0.1` on an available local port (default starting port: `42100`).
- Supports JSON-RPC 2.0 and REST endpoints.
- Enforces read-only database query execution by default.
- Maps requests to specific workspaces via Project Identifiers.

## Configuring AI Assistants

### Claude Desktop / Cursor Configuration

Add QueryDock to your `claude_desktop_config.json` or Cursor MCP settings:

```json
{
  "mcpServers": {
    "querydock": {
      "command": "node",
      "args": [
        "-e",
        "fetch('http://127.0.0.1:42100/v1/tools').then(r => r.json()).then(console.log)"
      ]
    }
  }
}
```

Or connect directly via HTTP SSE / POST to `http://127.0.0.1:<PORT>/tools`.

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

> **Security Note**: When `querydock.mcp.readOnly` is enabled (default), only `SELECT`/`WITH`/`EXPLAIN`/`SHOW`/`PRAGMA`/`DESCRIBE` statements run. Keywords inside string literals and comments do not trigger the guard; `SELECT ... INTO` and writes hidden after a read do. `EXEC`/`CALL` are rejected unless the target connection sets `"execPolicy": "sandbox"` in `.querydockrc` (optionally narrowed by `execAllowlist`), in which case the call runs inside a transaction that is always rolled back. Dynamic SQL (`EXEC (...)`, `sp_executesql`, `xp_*`) is never sandboxed. The rejection message names the reason (`contains DELETE`, `execPolicy is 'deny'`, allowlist miss).
