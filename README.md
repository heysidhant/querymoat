<p align="center"><img src="querydock-logo.png" width="96" alt="QueryDock logo"></p>

<h1 align="center">QueryDock</h1>

<p align="center"><b>Give your AI agent safe access to your database.</b><br>
A read-only-by-default MCP server and a fast database client, inside VS Code, Cursor and Windsurf.</p>

<p align="center">
<a href="https://marketplace.visualstudio.com/items?itemName=querydock.querydock"><img src="https://img.shields.io/visual-studio-marketplace/v/querydock.querydock" alt="Marketplace version"></a>
<a href="https://marketplace.visualstudio.com/items?itemName=querydock.querydock"><img src="https://img.shields.io/visual-studio-marketplace/i/querydock.querydock" alt="Installs"></a>
<a href="https://marketplace.visualstudio.com/items?itemName=querydock.querydock&ssr=false#review-details"><img src="https://img.shields.io/visual-studio-marketplace/r/querydock.querydock" alt="Rating"></a>
</p>

This repository is QueryDock's public home for **bug reports, feature requests and documentation**. The extension's source code is not public.

## Install

- **VS Code:** [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=querydock.querydock), or search "QueryDock" in the Extensions view.
- **Cursor, Windsurf, VSCodium:** coming soon to Open VSX. Until then, download the `.vsix` from the Marketplace page and use **Extensions: Install from VSIX…**.

## What it does

- 🛡️ **Safe MCP server for AI agents.** Claude Code, Cursor, Windsurf and Copilot can read your schema and data through a local MCP server that is read-only by default and never writes to production.
- ⚡ **Zero config.** Finds databases in `.env`, `docker-compose.yml`, Laravel, Rails, Django, Supabase and Prisma projects.
- 🗂️ **Database client in your editor.** Browse, edit and export data, run SQL with `EXPLAIN` and history, and inspect schemas, keys and indexes.
- 🏠 **100% local.** No cloud proxy and no telemetry.

Supports **SQLite, PostgreSQL, MySQL, MariaDB and Microsoft SQL Server**.

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

QueryDock is free to use, including at work, under the [QueryDock End User License Agreement](LICENSE).
