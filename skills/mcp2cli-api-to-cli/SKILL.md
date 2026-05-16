---
name: mcp2cli-api-to-cli
description: Turn any MCP, OpenAPI, or GraphQL server into a CLI at runtime with zero codegen, saving 96-99% of tokens wasted on tool schemas
triggers:
  - create a CLI from an API specification
  - connect to an MCP server over HTTP
  - turn this OpenAPI spec into a command line tool
  - query a GraphQL endpoint from the command line
  - save API connection settings for reuse
  - use mcp2cli to interact with this service
  - generate a skill from an OpenAPI spec
  - list available tools from an MCP server
---

# mcp2cli

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

mcp2cli converts any MCP server, OpenAPI specification, or GraphQL endpoint into a CLI at runtime without code generation. It dramatically reduces token consumption by replacing repeated tool schema transmissions with simple CLI calls — saving 96-99% of tokens on every LLM turn.

## Installation

```bash
# Run directly without installing
uvx mcp2cli --help

# Or install globally
uv tool install mcp2cli

# Install as a skill for AI agents
npx skills add knowsuchagency/mcp2cli --skill mcp2cli
```

## Core Concepts

mcp2cli operates in four modes:
- **MCP HTTP/SSE**: Connect to MCP servers over HTTP with SSE or streamable HTTP transport
- **MCP stdio**: Launch and communicate with local MCP servers via stdio
- **OpenAPI**: Generate CLI from OpenAPI 3.x specs (JSON or YAML)
- **GraphQL**: Introspect and query GraphQL endpoints

All modes support:
- Dynamic command generation (no codegen step)
- Automatic caching with configurable TTL
- OAuth 2.0 flows (authorization code + PKCE, client credentials)
- Secret management (env vars, files)
- Usage tracking and intelligent tool ranking

## MCP HTTP Mode

Connect to MCP servers over HTTP with automatic transport negotiation:

```bash
# List available tools
mcp2cli --mcp https://mcp.example.com/sse --list

# Call a tool
mcp2cli --mcp https://mcp.example.com/sse search --query "rust async"

# With authentication
mcp2cli --mcp https://mcp.example.com/sse \
  --auth-header "x-api-key:env:MCP_API_KEY" \
  query --sql "SELECT * FROM users LIMIT 10"

# Force specific transport (skip auto-negotiation)
mcp2cli --mcp https://mcp.example.com/sse --transport sse --list

# Search tools by name or description
mcp2cli --mcp https://mcp.example.com/sse --search "database"
```

## MCP stdio Mode

Launch local MCP servers and communicate via stdio:

```bash
# List tools from filesystem MCP server
mcp2cli --mcp-stdio "npx @modelcontextprotocol/server-filesystem /tmp" --list

# Read a file
mcp2cli --mcp-stdio "npx @modelcontextprotocol/server-filesystem /tmp" \
  read-file --path /tmp/data.json

# Pass environment variables to the server
mcp2cli --mcp-stdio "node ./custom-server.js" \
  --env DATABASE_URL=env:DATABASE_URL \
  --env DEBUG=1 \
  search --query "test"
```

## OpenAPI Mode

Generate CLI from OpenAPI specifications:

```bash
# Remote spec
mcp2cli --spec https://petstore3.swagger.io/api/v3/openapi.json --list

# Local spec with base URL override
mcp2cli --spec ./openapi.yaml --base-url https://api.example.com --list

# Call an endpoint
mcp2cli --spec ./openapi.json --base-url https://api.example.com \
  list-pets --status available --limit 20

# POST with JSON body from stdin
echo '{"name": "Fido", "species": "dog"}' | \
  mcp2cli --spec ./openapi.json create-pet --stdin

# With authentication
mcp2cli --spec ./openapi.json \
  --auth-header "Authorization:Bearer env:API_TOKEN" \
  create-item --name "New Item"
```

## GraphQL Mode

Query GraphQL endpoints with automatic introspection:

```bash
# List all queries and mutations
mcp2cli --graphql https://api.example.com/graphql --list

# Execute a query (auto-generates selection set)
mcp2cli --graphql https://api.example.com/graphql users --limit 10

# Execute a mutation
mcp2cli --graphql https://api.example.com/graphql \
  create-user --name "Alice" --email "alice@example.com"

# Override selection set fields
mcp2cli --graphql https://api.example.com/graphql \
  users --fields "id name email createdAt"

# With authentication
mcp2cli --graphql https://api.example.com/graphql \
  --auth-header "Authorization:Bearer env:GRAPHQL_TOKEN" \
  users
```

## OAuth Authentication

All modes support OAuth 2.0 flows:

```bash
# Authorization code + PKCE (opens browser)
mcp2cli --mcp https://mcp.example.com/sse --oauth --list
mcp2cli --spec https://api.example.com/openapi.json --oauth list-users
mcp2cli --graphql https://api.example.com/graphql --oauth users

# Client credentials (machine-to-machine)
mcp2cli --spec https://api.example.com/openapi.json \
  --oauth-client-id "env:OAUTH_CLIENT_ID" \
  --oauth-client-secret "env:OAUTH_CLIENT_SECRET" \
  list-resources

# With specific scopes
mcp2cli --graphql https://api.example.com/graphql \
  --oauth --oauth-scope "read:users write:users" \
  users

# Local spec — provide base URL for OAuth discovery
mcp2cli --spec ./openapi.json \
  --base-url https://api.example.com \
  --oauth \
  --list
```

Tokens are cached in `~/.cache/mcp2cli/oauth/` and automatically refreshed.

## Secret Management

Avoid passing secrets as CLI arguments:

```bash
# Read from environment variable
mcp2cli --mcp https://mcp.example.com/sse \
  --auth-header "Authorization:env:MY_API_TOKEN" \
  --list

# Read from file
mcp2cli --spec ./openapi.json \
  --oauth-client-secret "file:/run/secrets/client_secret" \
  --oauth-client-id "file:/run/secrets/client_id" \
  --list

# Works with secret managers
vault kv get -field=token secret/api | \
  MY_TOKEN=$(cat) mcp2cli --mcp https://mcp.example.com/sse \
  --auth-header "Authorization:env:MY_TOKEN" \
  search --query "data"
```

## Bake Mode — Save Connection Settings

Create reusable named configurations:

```bash
# Create baked tool from OpenAPI spec
mcp2cli bake create petstore \
  --spec https://api.example.com/openapi.json \
  --auth-header "Authorization:Bearer env:PETSTORE_TOKEN" \
  --exclude "delete-*,update-*" \
  --methods GET,POST \
  --cache-ttl 7200

# Create baked tool from MCP stdio server
mcp2cli bake create github \
  --mcp-stdio "npx @modelcontextprotocol/server-github" \
  --env GITHUB_TOKEN=env:GITHUB_TOKEN \
  --include "search-*,list-*" \
  --exclude "delete-*"

# Create baked tool from GraphQL endpoint
mcp2cli bake create hasura \
  --graphql https://hasura.example.com/v1/graphql \
  --auth-header "x-hasura-admin-secret:env:HASURA_SECRET"

# Use baked tool with @ prefix
mcp2cli @petstore --list
mcp2cli @petstore list-pets --limit 5
mcp2cli @github search-repos --query "mcp server"
mcp2cli @hasura users --limit 10

# Manage baked tools
mcp2cli bake list
mcp2cli bake show petstore
mcp2cli bake update petstore --cache-ttl 3600
mcp2cli bake remove petstore

# Install as standalone script
mcp2cli bake install petstore
# Creates ~/.local/bin/petstore wrapper

# Install to custom directory
mcp2cli bake install petstore --dir ./scripts/
```

### Filtering Options

```bash
# Include only specific tool patterns
mcp2cli bake create myapi \
  --spec ./openapi.json \
  --include "list-*,get-*,search-*"

# Exclude dangerous operations
mcp2cli bake create myapi \
  --spec ./openapi.json \
  --exclude "delete-*,destroy-*"

# Limit to specific HTTP methods (OpenAPI only)
mcp2cli bake create myapi \
  --spec ./openapi.json \
  --methods GET,POST
```

## Usage-Aware Tool Ranking

Reduce token costs with intelligent tool ranking:

```bash
# Default list (sorted by usage frequency when available)
mcp2cli @myapi --list

# Top 10 most-used tools, compact output (~20 tokens)
mcp2cli @myapi --list --top 10 --compact

# Sort by most recently used
mcp2cli @myapi --list --sort recent

# Alphabetical sort
mcp2cli @myapi --list --sort alpha

# Show full descriptions
mcp2cli @myapi --list --verbose
```

Usage data is tracked locally in `~/.cache/mcp2cli/usage.json`.

## Output Control

```bash
# Pretty-print JSON (auto-enabled for TTY)
mcp2cli --spec ./openapi.json --pretty list-users

# Raw response body (no JSON parsing)
mcp2cli --spec ./openapi.json --raw get-binary-data

# Limit output to first N records
mcp2cli --spec ./openapi.json list-logs --head 20

# TOON output (40-60% fewer tokens for LLMs)
mcp2cli --mcp https://mcp.example.com/sse --toon list-large-dataset

# Pipe-friendly output
mcp2cli --spec ./openapi.json list-users | jq '.[] | .email'
```

## Caching

Control spec and tool list caching:

```bash
# Force refresh (bypass cache)
mcp2cli --spec https://api.example.com/openapi.json --refresh --list

# Custom TTL (7 days)
mcp2cli --spec https://api.example.com/openapi.json --cache-ttl 604800 --list

# Custom cache key
mcp2cli --spec https://api.example.com/openapi.json --cache-key prod-api --list

# Override cache directory
MCP2CLI_CACHE_DIR=/tmp/my-cache mcp2cli --spec ./openapi.json --list
```

Default cache location: `~/.cache/mcp2cli/`
Default TTL: 3600 seconds (1 hour)

## Common Patterns

### Create a Skill from an API

```bash
# Generate a skill configuration for an OpenAPI service
mcp2cli --spec https://api.example.com/openapi.json --list

# Create baked tool for easier access
mcp2cli bake create myservice \
  --spec https://api.example.com/openapi.json \
  --auth-header "Authorization:Bearer env:MYSERVICE_TOKEN"

# Use in skill workflows
mcp2cli @myservice list-resources --format json | jq '.[] | select(.active == true)'
```

### Multi-Environment Setup

```bash
# Development environment
mcp2cli bake create myapi-dev \
  --spec https://dev-api.example.com/openapi.json \
  --auth-header "Authorization:Bearer env:DEV_TOKEN"

# Production environment
mcp2cli bake create myapi-prod \
  --spec https://api.example.com/openapi.json \
  --auth-header "Authorization:Bearer env:PROD_TOKEN"

# Use environment-specific tools
mcp2cli @myapi-dev test-endpoint --data "test"
mcp2cli @myapi-prod get-metrics --period "24h"
```

### Chaining with MCP Servers

```bash
# Use filesystem MCP to read config, then call API
CONFIG=$(mcp2cli --mcp-stdio "npx @modelcontextprotocol/server-filesystem /etc" \
  read-file --path /etc/myapp/config.json)

echo "$CONFIG" | jq -r '.api_endpoint' | \
  xargs -I {} mcp2cli --spec {}/openapi.json --list
```

### Filtering Large Tool Lists

```bash
# Search tools related to users
mcp2cli @myapi --search "user"

# List only creation operations
mcp2cli @myapi --list | grep "^create-"

# Get compact list of top tools for LLM context
mcp2cli @myapi --list --top 20 --compact
```

## Python API Usage

While mcp2cli is primarily a CLI tool, you can use its components programmatically:

```python
from mcp2cli.openapi import load_spec
from mcp2cli.client import make_request
import asyncio

async def call_api():
    # Load OpenAPI spec
    spec = await load_spec("https://api.example.com/openapi.json")
    
    # Make request
    response = await make_request(
        spec=spec,
        operation_id="listPets",
        params={"limit": 10},
        auth_headers={"Authorization": "Bearer token"}
    )
    
    return response

result = asyncio.run(call_api())
```

## Troubleshooting

### OAuth Flow Fails

```bash
# Clear cached tokens
rm -rf ~/.cache/mcp2cli/oauth/

# Retry with verbose output
mcp2cli --spec ./openapi.json --oauth --list --verbose
```

### MCP Server Connection Issues

```bash
# Test transport explicitly
mcp2cli --mcp https://mcp.example.com/sse --transport sse --list
mcp2cli --mcp https://mcp.example.com/sse --transport streamable --list

# Check server logs if using stdio
mcp2cli --mcp-stdio "npx @modelcontextprotocol/server-filesystem /tmp" \
  --env DEBUG=* \
  --list
```

### Cache Issues

```bash
# Force refresh
mcp2cli --spec ./openapi.json --refresh --list

# Clear all cache
rm -rf ~/.cache/mcp2cli/

# Use temporary cache location
MCP2CLI_CACHE_DIR=/tmp/test-cache mcp2cli --spec ./openapi.json --list
```

### Tool Not Found After Baking

```bash
# Check baked tool exists
mcp2cli bake list

# Verify configuration
mcp2cli bake show myapi

# Recreate with explicit filters
mcp2cli bake update myapi --include "*"
```

### Large Response Truncation

```bash
# Use --head to limit records
mcp2cli @myapi list-all --head 100

# Use --raw to get full response
mcp2cli @myapi list-all --raw > output.json

# Use TOON for token efficiency
mcp2cli @myapi list-all --toon
```

## Configuration

Configuration directory: `~/.config/mcp2cli/`
Cache directory: `~/.cache/mcp2cli/`

Override with environment variables:
- `MCP2CLI_CONFIG_DIR`: Configuration directory
- `MCP2CLI_CACHE_DIR`: Cache directory

Baked tools are stored in `~/.config/mcp2cli/baked.json`
Usage tracking in `~/.cache/mcp2cli/usage.json`
OAuth tokens in `~/.cache/mcp2cli/oauth/`
