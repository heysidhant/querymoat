# QueryDock Configuration Reference

This guide covers all options for configuring QueryDock via `.querydockrc` and VS Code user/workspace settings.

## Project Configuration: `.querydockrc`

Place `.querydockrc` (or `.querydockrc.json`) in your repository root. This file can be shared with your team.

### Schema Example
```json
[
  {
    "id": "app-sqlite",
    "name": "Local SQLite Database",
    "type": "sqlite",
    "path": "./database/database.sqlite",
    "environment": "development"
  },
  {
    "id": "app-postgres",
    "name": "Local PostgreSQL",
    "type": "postgres",
    "host": "127.0.0.1",
    "port": 5432,
    "database": "myapp",
    "username": "postgres",
    "password": "${env:DB_PASSWORD}",
    "schema": "public",
    "environment": "development",
    "ssl": false
  },
  {
    "id": "app-mysql",
    "name": "MySQL Production Replica",
    "type": "mysql",
    "host": "replica.example.internal",
    "port": 3306,
    "database": "store",
    "username": "readonly_app",
    "credentialKey": "mysql-replica-pass",
    "environment": "production",
    "isReadOnly": true
  }
]
```

### Configuration Fields

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `name` | string | **Yes** | Human-readable connection display name |
| `type` | string | **Yes** | Database engine: `sqlite`, `postgres`, `mysql`, `mariadb` |
| `id` | string | No | Unique ID (auto-generated if omitted) |
| `path` | string | **Yes (SQLite)** | File path (absolute or workspace-relative) |
| `host` | string | **Yes (Network)** | Hostname or IP address |
| `port` | number | **Yes (Network)** | Port number (e.g. 5432 for Postgres, 3306 for MySQL) |
| `database` | string | **Yes (Network)** | Database name |
| `username` | string | **Yes (Network)** | Database username |
| `password` | string | No | Password string or `${env:VAR}` reference |
| `credentialKey` | string | No | Key referencing VS Code SecretStorage |
| `environment` | string | No | `local`, `development`, `staging`, `production`, `test` |
| `isReadOnly` | boolean | No | Force read-only protection (SELECT-shaped statements only) |
| `execPolicy` | `deny` \| `sandbox` | No | While read-only: `deny` (default) rejects `EXEC`/`CALL`; `sandbox` runs them inside a transaction that is always rolled back |
| `execAllowlist` | string[] | No | Procedures the sandbox may run — case-insensitive globs, optional `schema.` prefix (`["EM3_*", "dbo.UTIL_*_GET_*"]`). Empty = any |
| `connectionTimeoutMs` | number | No | Connect timeout (network drivers) |
| `schema` | string | No | Default schema (PostgreSQL) |
| `ssl` | boolean/object | No | SSL configuration options |

---

## VS Code Settings (`querydock.*`)

Configure these in VS Code Settings (`Cmd+,` / `Ctrl+,`):

- **`querydock.debug`** (`boolean`, default: `false`)  
  Enables verbose logging in the QueryDock output channel.

- **`querydock.defaultPageSize`** (`enum: [25, 50, 100, 250, 500]`, default: `100`)  
  Default number of rows retrieved per page in the Data Grid.

- **`querydock.enableAutoDiscovery`** (`boolean`, default: `true`)  
  Automatically scans workspace for `.env`, `docker-compose.yml`, and framework database configurations.

- **`querydock.enableCodeLens`** (`boolean`, default: `true`)  
  Displays "QueryDock: Open Table" CodeLens above recognized model classes and SQL statements.

- **`querydock.enableMcp`** (`boolean`, default: `true`)  
  Starts the local Model Context Protocol server.

- **`querydock.mcp.readOnly`** (`boolean`, default: `true`)  
  Restricts MCP SQL queries to read-only statements (`SELECT`, `EXPLAIN`, `WITH`, `SHOW`, `PRAGMA`, `DESCRIBE`). Procedure calls additionally require the connection's `execPolicy: "sandbox"`.

- **`querydock.readOnlyEnvironments`** (`string[]`, default: `["production", "prod", "live"]`)  
  Environment tags that automatically enforce read-only locking on connections.
