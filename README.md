<p align="center"><img src="querydock-logo.png" width="96" alt="QueryDock logo"></p>

<h1 align="center">QueryDock</h1>

<p align="center"><b>Give your AI agent safe access to your database.</b><br>
A read-only-by-default MCP server and a fast database client, inside VS Code, Cursor and Windsurf.</p>

This repository is QueryDock's public home for **bug reports, feature requests and documentation**. The extension's source code is not public.

## Install

- **VS Code:** [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=querydock.querydock), or search "QueryDock" in the Extensions view.
- **Cursor, Windsurf, VSCodium:** [Open VSX](https://open-vsx.org/extension/querydock/querydock), or search "QueryDock" in the Extensions view.

## What it does

- 🛡️ **Safe MCP server for AI agents.** Claude Code, Cursor, Windsurf and Copilot can read your schema and data through a local MCP server that is read-only by default and never writes to production.
- ⚡ **Zero config.** Finds databases in `.env`, `docker-compose.yml`, Laravel, Rails, Django, Supabase and Prisma projects.
- 🗂️ **Database client in your editor.** Browse, edit and export data, run SQL with `EXPLAIN` and history, and inspect schemas, keys and indexes.
- 🏠 **100% local.** No cloud proxy and no telemetry.

Supports **SQLite, PostgreSQL, MySQL, MariaDB and Microsoft SQL Server**.

## CI check

`querydock-check` fails the build when your QueryDock config could let an AI agent or a teammate write to a remote or production database, or leaks a password. Warn-only is opt-in.

```yaml
- uses: actions/checkout@v4
- uses: heysidhant/querydock@v0.2.0
```

Any other CI: `npx querydock-check [dir] [--warn-only] [--format text|github]` (exit `1` on errors).

| Rule | Level | Catches |
| :--- | :--- | :--- |
| `plaintext-password` | error | A password written into the file instead of `${env:VAR}` or `credentialKey` |
| `remote-missing-environment` | error | A connection to a non-local host (or a `${env:...}` host) with no `environment` tag |
| `remote-writable` | error | `"isReadOnly": false` on a remote host |
| `production-writable` | error | `"isReadOnly": false` on a connection tagged or named as production |
| `invalid-json` | error | A config file QueryDock would silently ignore |
| `unknown-environment` | warning | A tag like `prd` that gets no protection |
| `sandbox-without-allowlist` | warning | `execPolicy: "sandbox"` with no `execAllowlist` |
| `workspace-disables-readonly` | warning | `querydock.mcp.readOnly: false` in `.vscode/settings.json` |

## Documentation

- [Configuration (`.querydockrc`)](docs/configuration.md)
- [MCP server for AI agents](docs/mcp.md)
- [Security model](docs/security.md)
- [Supported databases](docs/supported-databases.md)

## Get help

- 🐛 **Found a bug?** [Report it](https://github.com/heysidhant/querydock/issues/new?template=bug_report.yml)
- 💡 **Want a feature?** [Request it](https://github.com/heysidhant/querydock/issues/new?template=feature_request.yml)
- ⭐ **Want Pro early?** [Join the waitlist](https://github.com/heysidhant/querydock/issues/1)

## License

QueryDock is free to use, including at work, under the [QueryDock End User License Agreement](LICENSE). The `querydock-check` CLI is MIT-licensed.
