---
name: datagouv-mcp-server
description: Use the data.gouv.fr MCP server to search, explore, and analyze French Open Data datasets through AI chatbots
triggers:
  - "How do I connect to the data.gouv.fr MCP server?"
  - "Set up the datagouv MCP server in Claude Desktop"
  - "Search French open data using MCP"
  - "Configure datagouv MCP for ChatGPT"
  - "What datasets are available on data.gouv.fr?"
  - "Run the datagouv MCP server locally"
  - "Connect datagouv MCP to Cursor"
  - "Install the French open data MCP server"
---

# datagouv-mcp-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The **data.gouv.fr MCP Server** is a Model Context Protocol (MCP) server that enables AI chatbots (Claude, ChatGPT, Gemini, etc.) to search, explore, and analyze datasets from [data.gouv.fr](https://www.data.gouv.fr), the French national Open Data platform, directly through conversation.

Instead of manually browsing the website, users can ask natural language questions like:
- "Quels jeux de données sont disponibles sur les prix de l'immobilier?"
- "Montre-moi les dernières données de population pour Paris"

**Key Features:**
- Read-only access to French Open Data (no API key required)
- Search datasets by keywords, topics, and filters
- Explore dataset metadata, resources, and organizations
- Works with all major MCP-compatible chatbots
- Public hosted instance at `https://mcp.data.gouv.fr/mcp`
- Self-hostable with Docker or Python

## Installation & Configuration

### Using the Public Hosted Instance (Recommended)

The easiest way to use this MCP server is to connect to the public instance at `https://mcp.data.gouv.fr/mcp`. No installation required.

### Client-Specific Configuration

#### Claude Desktop

Add to `~/.config/Claude/claude_desktop_config.json` (Linux), `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS), or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "datagouv": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.data.gouv.fr/mcp"
      ]
    }
  }
}
```

**Windows-specific fix:** If the server doesn't connect, add this at the root level:

```json
{
  "isUsingBuiltInNodeForMcp": false,
  "mcpServers": {
    "datagouv": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.data.gouv.fr/mcp"]
    }
  }
}
```

#### ChatGPT (Paid Plans Only)

1. Go to `Settings` → `Apps and connectors`
2. Enable **Developer mode** in `Advanced settings`
3. Go to `Connectors` → `Browse connectors` → `Add a new connector`
4. Set URL to `https://mcp.data.gouv.fr/mcp` and save

#### Cursor

In Cursor Settings, search for "MCP" and add:

```json
{
  "mcpServers": {
    "datagouv": {
      "url": "https://mcp.data.gouv.fr/mcp",
      "transport": "http"
    }
  }
}
```

#### VS Code

Run **MCP: Open User Configuration** from Command Palette, then add:

```json
{
  "servers": {
    "datagouv": {
      "url": "https://mcp.data.gouv.fr/mcp",
      "type": "http"
    }
  }
}
```

#### Windsurf

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "datagouv": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.data.gouv.fr/mcp"]
    }
  }
}
```

#### Le Chat (Mistral)

1. Go to `Intelligence` → `Connectors`
2. Click `Add connector` → `Custom MCP Connector`
3. Name it (e.g., "DataGouv")
4. Set URL to `https://mcp.data.gouv.fr/mcp`
5. Leave authentication disabled and click **Create**

#### HuggingChat

1. Click + icon → `MCP Servers` → `Manage MCP Servers`
2. Click `+ Add Server`
3. Enter Server Name (e.g., "Data Gouv")
4. Set Server URL to `https://mcp.data.gouv.fr/mcp`
5. Click `Add Server`, then `Health Check` to verify

## Running Locally

### Prerequisites

- Docker & Docker Compose (recommended)
- OR Python with [uv](https://github.com/astral-sh/uv) installed

### With Docker (Recommended)

```bash
# Clone the repository
git clone git@github.com:datagouv/datagouv-mcp.git
cd datagouv-mcp

# Run with default settings (port 8000, prod environment)
docker compose up -d

# Run with custom settings
MCP_PORT=8007 DATAGOUV_API_ENV=demo LOG_LEVEL=DEBUG docker compose up -d

# Stop
docker compose down
```

### With Python/uv

```bash
# Clone the repository
git clone git@github.com:datagouv/datagouv-mcp.git
cd datagouv-mcp

# Install dependencies
uv sync

# Create environment file
cp .env.example .env

# Edit .env as needed (optional)
# MCP_HOST=127.0.0.1
# MCP_PORT=8007
# DATAGOUV_API_ENV=prod
# LOG_LEVEL=INFO

# Load environment variables
set -a && source .env && set +a

# Start the server
uv run main.py
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_HOST` | `0.0.0.0` | Host to bind to (use `127.0.0.1` for local dev) |
| `MCP_PORT` | `8000` | Port for the MCP HTTP server |
| `MCP_ENV` | `local` | Environment name (for Sentry): `local`, `prod`, `preprod`, `demo` |
| `DATAGOUV_API_ENV` | `prod` | data.gouv.fr environment: `prod` or `demo` |
| `LOG_LEVEL` | `INFO` | Python logging level: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |
| `SENTRY_DSN` | (unset) | Sentry DSN for error monitoring (optional) |
| `SENTRY_SAMPLE_RATE` | `1.0` | Sentry trace sampling rate (0.0-1.0) |

## Using the MCP Server

Once configured, you can interact with the MCP server through your AI chatbot by asking questions in natural language.

### Example Queries

**Search datasets:**
```
"Find datasets about real estate prices in France"
"Quels jeux de données existent sur la pollution de l'air?"
"Show me population data for Paris"
```

**Explore organizations:**
```
"What datasets does INSEE publish?"
"List all datasets from the Ministry of Health"
```

**Dataset details:**
```
"Give me information about dataset ID abc123"
"What resources are available in this dataset?"
```

## Available MCP Tools

The server exposes read-only tools for searching and exploring data.gouv.fr:

### `search_datasets`
Search for datasets by keywords, topics, organizations, etc.

**Parameters:**
- `q` (string, optional): Search query
- `page_size` (int, optional): Results per page (default: 20)
- `page` (int, optional): Page number (default: 1)
- Additional filters: `organization`, `tag`, `badge`, `featured`, `temporal_coverage`, `granularity`, `schema`, `license`

### `get_dataset`
Retrieve detailed information about a specific dataset.

**Parameters:**
- `dataset_id` (string, required): The dataset ID or slug

### `list_resources`
List all resources (files, APIs) within a dataset.

**Parameters:**
- `dataset_id` (string, required): The dataset ID or slug

### `get_resource`
Get detailed information about a specific resource.

**Parameters:**
- `resource_id` (string, required): The resource ID

## Code Examples

### Python Client Example

```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def search_datasets():
    server_params = StdioServerParameters(
        command="npx",
        args=["mcp-remote", "https://mcp.data.gouv.fr/mcp"]
    )
    
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            # Initialize the connection
            await session.initialize()
            
            # Search for datasets
            result = await session.call_tool(
                "search_datasets",
                arguments={"q": "population", "page_size": 5}
            )
            
            print(result)

asyncio.run(search_datasets())
```

### Connecting a Custom Local Server

If you're running the server locally on port 8007:

```json
{
  "mcpServers": {
    "datagouv-local": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://127.0.0.1:8007"
      ]
    }
  }
}
```

### Docker Compose Custom Configuration

```yaml
# docker-compose.yml
services:
  mcp-server:
    build: .
    ports:
      - "8007:8007"
    environment:
      - MCP_HOST=0.0.0.0
      - MCP_PORT=8007
      - MCP_ENV=prod
      - DATAGOUV_API_ENV=prod
      - LOG_LEVEL=INFO
      - SENTRY_DSN=${SENTRY_DSN}
    restart: unless-stopped
```

## Common Patterns

### Searching with Filters

When helping users search datasets, combine text queries with filters:

```python
# Search for recent environmental datasets from a specific org
result = await session.call_tool(
    "search_datasets",
    arguments={
        "q": "environment climate",
        "organization": "ademe",
        "badge": "climate-change",
        "page_size": 10
    }
)
```

### Progressive Exploration

1. Start with broad search
2. Get dataset details with `get_dataset`
3. List resources with `list_resources`
4. Get specific resource details with `get_resource`

### Handling Pagination

```python
# Get first page
page1 = await session.call_tool(
    "search_datasets",
    arguments={"q": "transport", "page": 1, "page_size": 20}
)

# Get next page
page2 = await session.call_tool(
    "search_datasets",
    arguments={"q": "transport", "page": 2, "page_size": 20}
)
```

## Troubleshooting

### Server Not Connecting in Claude Desktop (Windows)

**Symptom:** Server appears in list but never connects, no tools visible

**Solution:** Add `"isUsingBuiltInNodeForMcp": false` to the root of `claude_desktop_config.json`:

```json
{
  "isUsingBuiltInNodeForMcp": false,
  "mcpServers": { ... }
}
```

See [issue #69](https://github.com/datagouv/datagouv-mcp/issues/69)

### Connection Timeout

**Symptom:** Server fails to respond or times out

**Solutions:**
1. Verify the public instance is up: `curl https://mcp.data.gouv.fr/mcp`
2. Check your network/firewall settings
3. Try running a local instance instead

### Local Server Won't Start

**Symptom:** Error when running `uv run main.py`

**Solutions:**
1. Ensure `uv` is installed: `pip install uv`
2. Verify Python version compatibility (check `pyproject.toml`)
3. Check `.env` file exists and is loaded
4. Review logs with `LOG_LEVEL=DEBUG`

### No Results Returned

**Symptom:** Search returns empty results

**Solutions:**
1. Verify `DATAGOUV_API_ENV` is set correctly (`prod` vs `demo`)
2. Try broader search terms
3. Check if specific filters are too restrictive
4. Test the same query on https://www.data.gouv.fr directly

### CORS Issues (Browser-Based Clients)

**Symptom:** CORS errors in browser console

**Solution:** The public instance should handle CORS. If self-hosting, ensure your server configuration allows CORS from your client origin.

## Development & Testing

### Run Tests

```bash
# Install dev dependencies
uv sync --dev

# Run tests
uv run pytest

# Run with coverage
uv run pytest --cov=src
```

### Enable Debug Logging

```bash
LOG_LEVEL=DEBUG uv run main.py
```

### Test Against Demo Environment

```bash
DATAGOUV_API_ENV=demo uv run main.py
```

This connects to https://demo.data.gouv.fr instead of production.

## Resources

- **Project Repository:** https://github.com/datagouv/datagouv-mcp
- **Public Instance:** https://mcp.data.gouv.fr/mcp
- **data.gouv.fr Platform:** https://www.data.gouv.fr
- **MCP Specification:** https://modelcontextprotocol.io
- **Feedback Form:** https://tally.so/r/KYMboX

## License

MIT License - See the repository for full details.
