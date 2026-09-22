# QueryDock Security Model & Guidelines

Security and privacy are primary design requirements for QueryDock. This document outlines the security architecture and threat defenses.

## 1. Webview Security & Content Security Policy (CSP)

- **Strict CSP**: Every webview panel enforces a strict Content Security Policy with unique nonces:
  ```http
  default-src 'none';
  style-src ${webview.cspSource} 'unsafe-inline';
  script-src 'nonce-${nonce}' ${webview.cspSource};
  img-src ${webview.cspSource} data:;
  font-src ${webview.cspSource};
  connect-src 'none';
  ```
- **Untrusted Database Content**: All database cells are treated as untrusted strings. Even if a database column contains `<script>alert('XSS')</script>`, it is rendered as text and cannot execute JavaScript within the webview.
- **No `eval()`**: Code execution via `eval()` or `Function()` is strictly prohibited.

## 2. SQL Injection Prevention

- **Parameterization**: Values in filters, insertions, updates, and deletes are passed via parameterized placeholders (`$1, $2` in Postgres, `?` in SQLite and MySQL).
- **Identifier Quoting**: Identifiers (table names, schema names, column names) are safely quoted using the appropriate dialect syntax:
  - SQLite & Postgres: `"identifier"` with escaped double quotes
  - MySQL & MariaDB: `` `identifier` `` with escaped backticks
  - SQL Server: `[identifier]` with escaped closing brackets
- **Validation**: Table identifiers are checked against metadata loaded from schema dictionaries.

## 3. Credential Protection & SecretStorage

- **No Plaintext Passwords in Git**: Configuration in `.querydockrc` supports `${env:VAR_NAME}` for environment variable interpolation and `credentialKey` for VS Code `SecretStorage`.
- **Automatic Logger Masking**: The `Logger` channel automatically sanitizes passwords, tokens, and secret strings before logging:
  - URLs: `postgres://user:••••••@localhost:5432/db`
  - Key-values: `password=••••••`
- **Output Channel Sanitization**: Database query parameters and credentials are never dumped to public logs.

## 4. Environment Safety & Read-Only Protection

- Connections labeled with `production`, `prod`, or `live` are locked to **Read-Only** by default (`querydock.readOnlyEnvironments`).
- In read-only mode:
  - Inline editing is disabled in the data grid.
  - Delete row and set NULL actions are blocked.
  - Only `SELECT`-shaped statements run (`SELECT`, `WITH`, `EXPLAIN`, `SHOW`, `PRAGMA`, `DESCRIBE`); `INSERT`, `UPDATE`, `DELETE`, `MERGE`, `SELECT ... INTO` and DDL are rejected. String literals and comments are ignored by the guard.
  - `EXEC` / `CALL` are rejected unless the connection sets `execPolicy: "sandbox"`, which runs the call inside a transaction that is always rolled back (optionally limited by `execAllowlist`). Dynamic SQL and `xp_*` procedures are never sandboxed.

## 5. Local MCP Server Security

- **Strict Localhost Binding**: The MCP server binds exclusively to `127.0.0.1`. Requests originating from any other remote address are rejected with HTTP 403 Forbidden.
- **Read-Only Enforcement**: When `querydock.mcp.readOnly` is enabled (default `true`), the MCP query tool rejects any statement that is not a `SELECT`, `EXPLAIN`, `WITH`, or metadata query. Procedure calls are allowed only through a connection's `execPolicy: "sandbox"` (rolled-back transaction, allowlist-scoped).
- **Project Isolation**: MCP tool invocations must provide a valid `projectId`. Cross-project access to unconfigured databases is prevented.
