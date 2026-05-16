---
name: anysearch-mcp-server
description: Unified real-time search MCP server with general web search, 23 vertical domains, batch queries, and URL content extraction
triggers:
  - search the web for information
  - find recent news about a topic
  - look up stock information or financial data
  - search academic papers or research
  - extract content from this URL
  - search multiple queries in parallel
  - search for CVE security vulnerabilities
  - find code examples or repositories
---

# AnySearch MCP Server Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

AnySearch MCP Server is a unified real-time search engine for AI agents, providing general web search, 23 vertical domain searches (finance, academic, security, legal, code, etc.), parallel batch search, and full-page URL content extraction via the MCP protocol.

## What It Does

- **General Web Search**: Natural language queries across the web
- **Vertical Domain Search**: Structured queries in 23 specialized domains
- **Batch Search**: Execute up to 5 independent queries in parallel
- **URL Extraction**: Fetch and convert full page content to Markdown
- **Anonymous Access**: Works without API key (lower rate limits)

## Installation

### Streamable HTTP (Recommended - No Proxy)

For agents supporting MCP spec 2025-03-26+ (OpenCode, Claude Desktop 2025.6+):

**OpenCode** (`~/.opencode/config.json` or `opencode.json`):

```json
{
  "mcp": {
    "anysearch": {
      "type": "streamable-http",
      "url": "https://api.anysearch.com/mcp",
      "headers": {
        "Authorization": "Bearer ${ANYSEARCH_API_KEY}"
      }
    }
  }
}
```

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "anysearch": {
      "type": "streamable-http",
      "url": "https://api.anysearch.com/mcp",
      "headers": {
        "Authorization": "Bearer ${ANYSEARCH_API_KEY}"
      }
    }
  }
}
```

### stdio Transport (Via mcp-remote Proxy)

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "anysearch": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.anysearch.com/mcp",
        "--header",
        "Authorization: Bearer ${ANYSEARCH_API_KEY}"
      ]
    }
  }
}
```

**VS Code Copilot** (`.vscode/mcp.json`):

```json
{
  "servers": {
    "anysearch": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.anysearch.com/mcp",
        "--header",
        "Authorization: Bearer ${ANYSEARCH_API_KEY}"
      ]
    }
  }
}
```

**Cline** (VS Code settings):

```json
{
  "mcpServers": {
    "anysearch": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.anysearch.com/mcp",
        "--header",
        "Authorization: Bearer ${ANYSEARCH_API_KEY}"
      ]
    }
  }
}
```

### SSE Transport (Via supergateway Proxy)

Start the proxy server:

```bash
npx -y supergateway \
  --streamableHttp https://api.anysearch.com/mcp \
  --outputTransport sse \
  --port 8000 \
  --oauth2Bearer ${ANYSEARCH_API_KEY}
```

**Cursor** (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "anysearch": {
      "type": "sse",
      "url": "http://localhost:8000/sse"
    }
  }
}
```

**Windsurf** (`~/.codeium/windsurf/mcp_config.json`):

```json
{
  "mcpServers": {
    "anysearch": {
      "serverUrl": "http://localhost:8000/sse"
    }
  }
}
```

### API Key Configuration

**Optional but recommended**. Get a free key at: https://anysearch.com/console/api-keys

Priority order:
1. `Authorization` header / `--api_key` flag
2. `ANYSEARCH_API_KEY` environment variable
3. `.env` file with `ANYSEARCH_API_KEY=<key>`
4. Anonymous access (lower rate limits)

Set environment variable:

```bash
export ANYSEARCH_API_KEY="your-api-key-here"
```

Or create `.env` file:

```
ANYSEARCH_API_KEY=your-api-key-here
```

## Available Tools

### `search` - General or Vertical Domain Search

Execute a search query. For vertical searches, you MUST call `list_domains` first.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query (format depends on domain) |
| `domain` | string | No | Vertical domain (e.g., `finance`, `academic`) |
| `sub_domain` | string | No | Sub-domain routing key (required for vertical) |
| `sub_domain_params` | object | No | Extra params per sub_domain schema |
| `content_types` | array | No | Filter: `web`, `news`, `code`, `doc`, etc. |
| `zone` | string | No | `cn` or `intl` (required when sub_domain requires) |
| `max_results` | integer | No | 1-100, default 10 |
| `freshness` | string | No | `day`, `week`, `month`, `year` |

**General Web Search Example:**

```json
{
  "query": "quantum computing breakthroughs 2025",
  "max_results": 5,
  "freshness": "month"
}
```

**Vertical Search Example (Stock):**

First, call `list_domains`:

```json
{
  "domain": "finance"
}
```

Then use the discovered `sub_domain` and `query_format`:

```json
{
  "query": "AAPL",
  "domain": "finance",
  "sub_domain": "finance.us_stock",
  "max_results": 5
}
```

**News Search Example:**

```json
{
  "query": "artificial intelligence",
  "content_types": ["news"],
  "freshness": "week",
  "max_results": 10
}
```

**Academic Search Example:**

```json
{
  "query": "10.1038/s41586-019-1666-5",
  "domain": "academic",
  "sub_domain": "academic.doi"
}
```

### `list_domains` - Query Vertical Domain Directory

**MUST be called before vertical search** to discover available sub_domains and their query formats.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | One of | Single domain to query |
| `domains` | array | One of | Batch up to 5 domains |

**Single Domain Example:**

```json
{
  "domain": "finance"
}
```

**Multiple Domains Example:**

```json
{
  "domains": ["finance", "academic", "security"]
}
```

**Returns Markdown table with:**
- `sub_domain`: Routing key for search
- `description`: What the sub_domain searches
- `query_format`: Exact format required (e.g., raw ticker, DOI)
- `params_schema`: JSON schema for optional parameters
- `zone`: If `CN`, must set `zone: "cn"` in search

### `batch_search` - Parallel Search Execution

Execute 2-5 independent search queries in parallel. Single failure doesn't block others.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `queries` | array | Yes | 1-5 query objects, each with same fields as `search` |

**Example:**

```json
{
  "queries": [
    {
      "query": "AAPL",
      "domain": "finance",
      "sub_domain": "finance.us_stock"
    },
    {
      "query": "python async http client",
      "domain": "code",
      "sub_domain": "code.general"
    },
    {
      "query": "CVE-2024-1234",
      "domain": "security",
      "sub_domain": "security.cve"
    }
  ]
}
```

### `extract` - URL Content Extraction

Fetch full page content from a URL and return as Markdown. HTML pages only, truncated at 50,000 characters.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | Target URL (`http://` or `https://`) |

**Example:**

```json
{
  "url": "https://en.wikipedia.org/wiki/Quantum_computing"
}
```

## Supported Domains

**23 vertical domains available:**

- `code` - Code repositories, packages, documentation
- `tech` - Technology news, products, reviews
- `fashion` - Fashion trends, brands, products
- `travel` - Flights, hotels, destinations
- `home` - Home improvement, furniture, decor
- `ecommerce` - Products, prices, reviews
- `gaming` - Games, platforms, news
- `film` - Movies, reviews, showtimes
- `music` - Songs, artists, albums
- `finance` - Stocks, crypto, markets, companies
- `academic` - Papers, patents, research
- `legal` - Laws, regulations, cases
- `business` - Companies, industries, markets
- `ip` - Patents, trademarks, copyrights
- `security` - CVEs, vulnerabilities, threats
- `education` - Courses, schools, materials
- `health` - Medical, wellness, drugs
- `religion` - Religious texts, organizations
- `geo` - Geographic data, maps, places
- `environment` - Climate, conservation, ecology
- `energy` - Energy sources, efficiency, policy
- `ugc` - User-generated content, forums

## Common Usage Patterns

### Pattern 1: General Web Search

When user asks an open-ended question:

```json
{
  "query": "best practices for microservices architecture",
  "max_results": 10
}
```

### Pattern 2: Recent News

When user wants current information:

```json
{
  "query": "SpaceX launches",
  "content_types": ["news"],
  "freshness": "week",
  "max_results": 5
}
```

### Pattern 3: Vertical Search Workflow

When user mentions structured identifiers (stock ticker, CVE, DOI, etc.):

**Step 1:** Identify the domain and call `list_domains`

```json
{
  "domain": "security"
}
```

**Step 2:** Parse the response to find the correct `sub_domain` and `query_format`

**Step 3:** Execute search with exact format

```json
{
  "query": "CVE-2024-1234",
  "domain": "security",
  "sub_domain": "security.cve"
}
```

### Pattern 4: Multi-Intent Queries

When user has multiple independent questions:

```json
{
  "queries": [
    {
      "query": "TSLA",
      "domain": "finance",
      "sub_domain": "finance.us_stock"
    },
    {
      "query": "electric vehicle market trends",
      "freshness": "month"
    }
  ]
}
```

### Pattern 5: Deep Content Extraction

When search results provide URLs but user needs full content:

**Step 1:** Search

```json
{
  "query": "kubernetes best practices",
  "max_results": 3
}
```

**Step 2:** Extract top result

```json
{
  "url": "https://kubernetes.io/docs/concepts/configuration/overview/"
}
```

### Pattern 6: Code Search

When user needs code examples:

```json
{
  "query": "react hooks useEffect",
  "domain": "code",
  "sub_domain": "code.general",
  "max_results": 10
}
```

### Pattern 7: Academic Research

When user references DOI or research topic:

```json
{
  "query": "10.1038/nature12373",
  "domain": "academic",
  "sub_domain": "academic.doi"
}
```

## Decision Flow

Use this logic to choose the right tool:

```
User query
  |
  +-- Has structured identifiers? (ticker, CVE, DOI, IATA, patent, etc.)
  |     YES -> 1) list_domains(domain) -> discover sub_domain & query_format
  |             2) search with domain + sub_domain + exact format
  |
  +-- Multiple independent questions?
  |     YES -> batch_search with array of queries
  |
  +-- Need full article/page content beyond snippets?
  |     YES -> search -> extract(url) from results
  |
  +-- Filter by content type or freshness?
  |     YES -> search with content_types and/or freshness params
  |
  +-- Otherwise -> search (general, natural language)
```

## Vertical Search Requirements

Before executing vertical search, you MUST:

1. **Call `list_domains`** for the target domain
2. **Follow `query_format`** exactly (e.g., raw ticker "AAPL", not "Apple stock")
3. **Use correct `sub_domain`** that matches user intent
4. **Set `zone: "cn"`** if the sub_domain requires it (check `zone` field)
5. **Include `sub_domain_params`** if the schema requires additional fields

## Troubleshooting

### Issue: "Invalid query format for vertical search"

**Cause:** Query doesn't match the `query_format` from `list_domains`

**Solution:** 
1. Call `list_domains` for the domain
2. Check the `query_format` field
3. Transform user input to match exact format (e.g., extract ticker symbol from natural language)

### Issue: "Missing required zone parameter"

**Cause:** Sub_domain requires `zone: "cn"` but it wasn't provided

**Solution:**
1. Check `list_domains` response for `zone` field
2. If `zone: "CN"`, add `"zone": "cn"` to search call

### Issue: "Anonymous rate limit exceeded"

**Cause:** No API key configured, hitting lower anonymous limits

**Solution:**
1. Get free API key from https://anysearch.com/console/api-keys
2. Set `ANYSEARCH_API_KEY` environment variable
3. Restart agent to pick up new key

### Issue: "SSE/stdio proxy not working"

**Cause:** Proxy server not running or incorrect configuration

**Solution for SSE:**
1. Ensure `supergateway` proxy is running: `npx -y supergateway --streamableHttp https://api.anysearch.com/mcp --outputTransport sse --port 8000 --oauth2Bearer ${ANYSEARCH_API_KEY}`
2. Check proxy is accessible: `curl http://localhost:8000/sse`
3. Verify agent config points to `http://localhost:8000/sse`

**Solution for stdio:**
1. Use `mcp-remote` proxy (simpler, auto-detects transport)
2. Verify `npx` can download packages
3. Check agent restart picked up new config

### Issue: "No results for vertical search"

**Cause:** Wrong `sub_domain` selected or malformed query

**Solution:**
1. Review `list_domains` output again
2. Match user intent to `description` field of sub_domains
3. Verify query matches `query_format` exactly
4. Try general search first to validate the entity exists

### Issue: "Extract returns truncated content"

**Cause:** Page content exceeds 50,000 character limit

**Solution:**
1. This is expected behavior for very long pages
2. Use search results snippets for overview
3. Extract multiple specific URLs if needed
4. For extremely long docs, consider asking user which sections to prioritize

## Configuration Quick Reference

| Agent | Transport | Proxy? | Config File |
|-------|-----------|--------|-------------|
| OpenCode | Streamable HTTP | No | `opencode.json` |
| Claude Desktop 2025.6+ | Streamable HTTP | No | `claude_desktop_config.json` |
| Claude Desktop (legacy) | stdio | Yes (mcp-remote) | `claude_desktop_config.json` |
| VS Code Copilot | stdio | Yes (mcp-remote) | `.vscode/mcp.json` |
| Cline | stdio | Yes (mcp-remote) | VS Code settings |
| Cursor | SSE | Yes (supergateway) | `.cursor/mcp.json` |
| Windsurf | SSE | Yes (supergateway) | `mcp_config.json` |

## Best Practices

1. **Always call `list_domains` before vertical search** - Don't assume you know the sub_domains
2. **Use batch_search for multiple intents** - More efficient than sequential calls
3. **Prefer general search for exploratory queries** - Vertical search requires exact formats
4. **Extract URLs sparingly** - Search snippets are often sufficient
5. **Set appropriate `max_results`** - Default 10 is good, reduce for speed, increase for thoroughness
6. **Use `freshness` for time-sensitive queries** - News, events, recent developments
7. **Never include API keys in chat** - Always use environment variables
8. **Check zone requirements** - CN sub_domains require `zone: "cn"` parameter

## Security Notes

- All queries and URLs are sent to `https://api.anysearch.com`
- Do not use for sensitive data (passwords, personal info, trade secrets)
- API keys grant higher rate limits - protect them as secrets
- Anonymous access works but has lower limits
