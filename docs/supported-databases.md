# Supported Databases & Dialects

QueryMoat implements a unified driver abstraction layer (`DatabaseDriver`) that provides a uniform developer experience across diverse database management systems.

## 1. SQLite

- **Driver Implementation**: WebAssembly-compiled SQLite (`sql.js`).
- **Compatibility**: 100% pure cross-platform (macOS Apple Silicon & Intel, Linux x64/arm64, Windows). Zero native compilation or Electron ABI mismatch risks.
- **Features**:
  - File-based databases with automatic sync on mutations.
  - In-memory database testing (`:memory:`).
  - Schema inspection via `PRAGMA table_info` and `sqlite_master`.
  - Foreign key relationship inspection via `PRAGMA foreign_key_list`.
  - Index inspection via `PRAGMA index_list`.
  - Query execution plan analysis via `EXPLAIN QUERY PLAN`.

## 2. PostgreSQL

- **Driver Implementation**: `pg` (node-postgres).
- **Features**:
  - Multi-schema support (`public` and user-defined schemas).
  - High-precision data types (UUID, JSONB, Arrays, Timestamps, Enums).
  - Auto-increment sequences detection.
  - Foreign key constraints via `information_schema.table_constraints` and `constraint_column_usage`.
  - Visual execution plan via `EXPLAIN (FORMAT JSON)`.
  - SSL/TLS encryption support with custom CA certificates.

## 3. MySQL & MariaDB

- **Driver Implementation**: `mysql2/promise`.
- **Features**:
  - Databases and tables inspection via `information_schema`.
  - Primary, foreign, and unique indexes via `SHOW INDEX`.
  - Unsigned numbers, ENUM, SET, and JSON datatypes.
  - Auto-increment metadata via `EXTRA` column attributes.
  - Query execution plans via `EXPLAIN FORMAT=JSON`.
  - SSL connection support.

## 4. Microsoft SQL Server (MSSQL / Azure / AWS RDS)

- **Driver Implementation**: `tedious` (pure JavaScript TDS protocol implementation).
- **Features**:
  - Full support for SQL Server 2017, 2019, 2022, Azure SQL, and AWS RDS SQL Server.
  - SSL/TLS encryption with `trustServerCertificate: true` out-of-the-box for cloud endpoints.
  - Multi-schema support (`dbo`, user-defined schemas) via `INFORMATION_SCHEMA`.
  - Primary keys, foreign key relations, and indexes via `sys.indexes` and `INFORMATION_SCHEMA`.
  - Paginated queries using T-SQL `OFFSET ... ROWS FETCH NEXT ... ROWS ONLY`.
  - Configured via `.querymoatrc` (no third-party config files are read).
  - Multiple result sets per batch/procedure, `BIGINT` returned as numbers when safe.
  - Sandboxed `EXEC` on read-only connections (`execPolicy: "sandbox"`, always rolled back).

## Roadmap Databases

Architectural hooks are in place for:
- **MongoDB** via official `mongodb` client (document navigation, inferred schema)
- **DuckDB** via `@duckdb/duckdb-wasm`
- **ClickHouse** via `@clickhouse/client`
- **Redis / Valkey** via `ioredis` (key inspection, TTL, value viewer)
