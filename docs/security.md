# QueryMoat Security Model & Guidelines

Security and privacy are primary design requirements for QueryMoat. This document outlines the security architecture and threat defenses.

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

- **No Plaintext Passwords in Git**: Configuration in `.querymoatrc` supports `${env:VAR_NAME}` for environment variable interpolation and `credentialKey` for VS Code `SecretStorage`.
- **Automatic Logger Masking**: The `Logger` channel automatically sanitizes passwords, tokens, and secret strings before logging:
  - URLs: `postgres://user:••••••@localhost:5432/db`
  - Key-values: `password=••••••`
- **Output Channel Sanitization**: Database query parameters and credentials are never dumped to public logs.

## 4. Environment Safety & Read-Only Protection

- A connection is locked to **Read-Only** when any of these hold:
  - it points at a **non-local host**: anything other than `localhost`, `127.x.x.x`, `::1`, `host.docker.internal`, a Unix socket or a SQLite file. This catches production databases whatever they are called (`db-1`, `main`, a client's name). Private network addresses such as `10.x` count as remote.
  - its `environment` is a protected tag (`production`, `prod`, `live` by default; `querymoat.readOnlyEnvironments` can add more but never remove these).
  - its name, database or username contains a protected tag as a whole word (`myapp-prod`, `app_production`; not `delivery`). This also catches a production database reached through a local SSH tunnel.
- To edit a remote development database, right-click it in the QueryMoat tree → **Allow Writes on This Remote Database…** and confirm the dialog. The grant is stored in VS Code SecretStorage for that exact host, port, database and user, never in a project file, so an AI agent cannot grant it. **Lock Remote Database Again** revokes it. A grant never unlocks a connection that is tagged or named as production.
- The tree tooltip, the READ-ONLY badge and rejected-query errors name the rule that locked the connection.
- In read-only mode:
  - Inline editing is disabled in the data grid.
  - Delete row and set NULL actions are blocked.
  - Only `SELECT`-shaped statements run (`SELECT`, `WITH`, `EXPLAIN`, `SHOW`, `PRAGMA`, `DESCRIBE`); `INSERT`, `UPDATE`, `DELETE`, `MERGE`, `SELECT ... INTO` and DDL are rejected. String literals and comments are ignored by the guard.
  - `EXEC` / `CALL` are rejected unless the connection sets `execPolicy: "sandbox"`, which runs the call inside a transaction that is always rolled back (optionally limited by `execAllowlist`). Dynamic SQL and `xp_*` procedures are never sandboxed.

## 5. Local MCP Server Security

- **Strict Localhost Binding**: The MCP server binds exclusively to `127.0.0.1`. Requests originating from any other remote address are rejected with HTTP 403 Forbidden.
- **Browser & DNS-Rebinding Protection**: A web page in your browser also connects from `127.0.0.1`, so loopback alone is not enough. The server rejects any request carrying an `Origin` header (browsers send one; MCP clients do not) and any `Host` other than `127.0.0.1:<port>`, `localhost:<port>` or `[::1]:<port>`. No CORS headers are sent, so a browser can never read a response.
- **Read-Only Enforcement**: When `querymoat.mcp.readOnly` is enabled (default `true`), the MCP query tool rejects any statement that is not a `SELECT`, `EXPLAIN`, `WITH`, or metadata query. Procedure calls are allowed only through a connection's `execPolicy: "sandbox"` (rolled-back transaction, allowlist-scoped).
- **Agents Cannot Unlock Themselves**: Workspace settings can turn `querymoat.mcp.readOnly` on but never off. Turning it off in user settings takes effect only after you confirm in a modal dialog, and the confirmation lives in VS Code SecretStorage (OS keychain) rather than in any settings file.
- **Project Isolation**: MCP tool invocations must provide a valid `projectId`. Cross-project access to unconfigured databases is prevented.
