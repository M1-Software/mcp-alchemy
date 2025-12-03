# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Alchemy is a Model Context Protocol (MCP) server that connects Claude Desktop to databases via SQLAlchemy. It exposes four MCP tools: `all_table_names`, `filter_table_names`, `schema_definitions`, and `execute_query`.

## Development Commands

```bash
# Install dependencies
uv sync
uv pip install <database-driver>  # e.g., psycopg2-binary, pymysql, pymssql

# Run tests (requires SQLite Chinook database)
DB_URL="sqlite:///tests/Chinook_Sqlite.sqlite" uv run python tests/test.py

# Run the MCP server directly for development
DB_URL="sqlite:///tests/Chinook_Sqlite.sqlite" uv run python -m mcp_alchemy.server

# Build for PyPI
uv run python -m build
```

## Architecture

The entire server implementation is in `mcp_alchemy/server.py`:

- **FastMCP server**: Uses `mcp.server.fastmcp.FastMCP` to expose tools
- **Connection pooling**: Single global `ENGINE` with retry logic for transient failures
- **MCP tools**: Four tools decorated with `@mcp.tool()`:
  - `all_table_names()` - Lists all tables
  - `filter_table_names(q)` - Filters tables by substring
  - `schema_definitions(table_names)` - Returns schema with columns, types, PKs, FKs
  - `execute_query(query, params)` - Executes SQL with parameterized queries

## Key Environment Variables

- `DB_URL` (required): SQLAlchemy connection string
- `CLAUDE_LOCAL_FILES_PATH`: Enables full result set export for large queries
- `CLAUDE_FILE_URL_BASE`: Custom URL base for file access (default: jsdelivr CDN)
- `EXECUTE_QUERY_MAX_CHARS`: Output truncation limit (default: 4000)
- `DB_ENGINE_OPTIONS`: JSON string for SQLAlchemy engine kwargs

## Testing

Tests are assertion-based comparisons against expected string output. The test file (`tests/test.py`) imports from the server module and calls functions directly, comparing results to hardcoded expected values.

Docker-based multi-database testing available in `tests/docker-compose.yml` for MySQL and PostgreSQL with the Chinook sample database.

## Version Management

Version is defined in two places that must stay in sync:
- `pyproject.toml` (`version = "..."`)
- `mcp_alchemy/server.py` (`VERSION = "..."`)
