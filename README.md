# mcp-ai-agent-server

[![CI](https://github.com/thiagoger/mcp-ai-agent-server/actions/workflows/ci.yml/badge.svg)](https://github.com/thiagoger/mcp-ai-agent-server/actions/workflows/ci.yml)
[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org)
[![Protocol: MCP](https://img.shields.io/badge/protocol-MCP-7C3AED.svg)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Give an AI agent a tool and let it do the work:

> **You:** Generate 20 companies of demo data with seed 7 and confirm the foreign keys are clean.
>
> **Claude** *(calls `generate_dataset` over MCP)* → 20 companies, 282 users, 360 invoices, 704 line items. Referential integrity: **PASS**.

That round trip is what this repo is about. It's a small, readable [Model Context Protocol](https://modelcontextprotocol.io) server in Python that turns plain functions into tools any MCP client (Claude Desktop, Claude Code, an IDE agent) can call. I build these against OAuth-guarded SaaS APIs at work; this is the same skeleton with the credentials stripped out, so the wiring is easy to read.

## The whole server is this shape

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("synthetic-data")

@mcp.tool()
def generate_dataset(companies: int = 8, seed: int = 42) -> dict:
    """Generate referentially-intact demo data and return row counts."""
    ...
```

Type hints become the tool schema the model sees. No glue code, no JSON-schema by hand. What the agent gets:

| | name | what it does |
|---|------|--------------|
| tool | `generate_dataset` | build a dataset, return row counts + a sample invoice |
| tool | `validate_dataset` | return just the referential-integrity report |
| resource | `schema://dataset` | the relational schema the server produces |

## Plug it into Claude

```bash
pip install -r requirements.txt
python server.py        # talks MCP over stdio
```

```jsonc
// claude_desktop_config.json
{
  "mcpServers": {
    "synthetic-data": { "command": "python", "args": ["/abs/path/server.py"] }
  }
}
```

Restart the client and the three capabilities show up. Ask for data; the agent calls the tool and hands you back a clean dataset.

## Where the real version differs

A production server swaps the in-memory generator for an upstream API call and adds an OAuth 2.0 client right here. I kept that boundary deliberately obvious and credential-free so this stays a teaching copy, not a liability. The data generator itself is the standalone [synthetic-data-forge](https://github.com/thiagoger/synthetic-data-forge).

## Tested

`pytest` checks the generator's integrity and determinism *and* boots the MCP server to confirm it registers its tools. Green on Python 3.10 to 3.12.

## License

MIT.
