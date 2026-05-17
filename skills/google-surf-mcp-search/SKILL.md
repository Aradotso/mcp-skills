---
name: google-surf-mcp-search
description: Google search MCP server with academic PDF extraction, no API key required, CAPTCHA recovery, and parallel search capabilities
triggers:
  - search google without an api key
  - set up google search mcp server
  - extract content from academic papers
  - search and extract web content in parallel
  - configure google-surf-mcp for claude
  - handle captcha in mcp search
  - fetch and extract article content
  - search multiple queries in parallel
---

# google-surf-mcp-search

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What It Does

google-surf-mcp is an MCP server that provides Google search functionality without requiring API keys. It combines three capabilities in one:
- Google search with ad/spam filtering
- URL content extraction (HTML + PDF)
- Academic paper extraction (arXiv, Nature, PubMed, etc.)

Key features:
- Works with actual Google search (not an API wrapper)
- Automatic CAPTCHA recovery with persistent browser profiles
- Parallel search and extraction
- Token-efficient abstract mode for triage
- Built-in rate limiting and caching
- Geometric verification to drop sponsored ads and knowledge panels

## Installation

### Quick Install (npx)

Add to your MCP client config (e.g., `~/.claude.json` for Claude Code):

```json
{
  "mcpServers": {
    "google-surf": {
      "command": "npx",
      "args": ["-y", "google-surf-mcp"]
    }
  }
}
```

### Local Clone Installation

```bash
git clone https://github.com/HarimxChoi/google-surf-mcp
cd google-surf-mcp
npm install
npm run build
```

Config for local installation:

```json
{
  "mcpServers": {
    "google-surf": {
      "command": "node",
      "args": ["/absolute/path/to/google-surf-mcp/build/index.js"]
    }
  }
}
```

### Manual Bootstrap (if auto-bootstrap fails)

```bash
npm run bootstrap
```

With custom paths:

```bash
CHROME_PATH=/usr/bin/google-chrome SURF_TZ=America/New_York npm run bootstrap
```

## Available Tools

### 1. `search` - Single Google Search

Performs a single Google search, returns filtered results (ads removed).

**Parameters:**
- `query` (string, required): Search query
- `limit` (number, optional): Max results, default 10

**Returns:**
- `results[]`: Array of `{ title, url, snippet }`
- `dropped`: Count of filtered results (ads, knowledge panels)
- `dropped_reasons[]`: Why items were dropped
- `cache_hit`: Boolean indicating cache use

**Example Usage:**

```typescript
// Via MCP tool call
{
  "query": "typescript async patterns",
  "limit": 5
}
```

**Response:**

```json
{
  "results": [
    {
      "title": "Async/Await in TypeScript",
      "url": "https://example.com/typescript-async",
      "snippet": "Learn how to use async/await patterns..."
    }
  ],
  "dropped": 2,
  "dropped_reasons": ["sponsored", "knowledge_panel"],
  "cache_hit": false
}
```

### 2. `search_parallel` - Parallel Multi-Query Search

Execute multiple searches in parallel using a worker pool (max 10 queries).

**Parameters:**
- `queries` (string[], required): Array of search queries
- `limit` (number, optional): Max results per query, default 10

**Returns:**
- Array of search results (same format as `search`)

**Example Usage:**

```typescript
{
  "queries": [
    "mcp server best practices",
    "playwright stealth techniques",
    "typescript pdf extraction",
    "google search scraping 2026"
  ],
  "limit": 3
}
```

### 3. `extract` - Fetch and Extract Content

Extract text content from a URL (HTML or PDF).

**Parameters:**
- `url` (string, required): URL to extract
- `max_chars` (number, optional): Character limit, default 100k
- `mode` (string, optional): `"full"` | `"abstract"` | `"metadata"`

**Modes:**
- `full`: Complete article text (HTML via Readability, PDF via unpdf)
- `abstract`: ~1500 chars for triage (PDF page 1 or HTML meta description)
- `metadata`: PDF page count only

**Returns:**
- `content`: Extracted text (markdown for HTML)
- `title`: Document title
- `excerpt`: Short summary
- `length`: Character count
- `is_pdf`: Boolean
- `page_count`: Number (PDFs only)
- `extraction_quality`: `"high"` | `"medium"` | `"low"`

**Example Usage:**

```typescript
// Extract full academic paper
{
  "url": "https://arxiv.org/pdf/2301.12345.pdf",
  "mode": "full"
}

// Quick abstract for triage
{
  "url": "https://nature.com/articles/s41586-023-12345-6",
  "mode": "abstract",
  "max_chars": 2000
}
```

**Response:**

```json
{
  "content": "# Paper Title\n\nAbstract: This paper presents...",
  "title": "Novel Approach to AI Safety",
  "excerpt": "This paper presents a novel approach...",
  "length": 45678,
  "is_pdf": true,
  "page_count": 12,
  "extraction_quality": "high"
}
```

### 4. `search_extract` - Combined Search + Extract

Search and extract content in one call. Efficiently parallelizes extraction.

**Parameters:**
- `query` (string, required): Search query
- `limit` (number, optional): Max results to extract, default 5
- `max_chars` (number, optional): Per-result char limit
- `mode` (string, optional): `"abstract"` (default) | `"full"`

**Best Practices:**
- Use `mode="abstract"` (default) for cheap triage with ~1500-char summaries
- Use `mode="full"` only when you need complete article text (slower, more tokens)

**Returns:**
- `results[]`: Search results enriched with `extracted_content`

**Example Usage:**

```typescript
// Triage mode (default, token-efficient)
{
  "query": "claude mcp server tutorials",
  "limit": 5,
  "mode": "abstract"
}

// Full extraction (when you need complete content)
{
  "query": "machine learning interpretability survey",
  "limit": 3,
  "mode": "full",
  "max_chars": 50000
}
```

**Response:**

```json
{
  "results": [
    {
      "title": "Building MCP Servers",
      "url": "https://example.com/mcp-tutorial",
      "snippet": "Complete guide to MCP servers...",
      "extracted_content": {
        "content": "# Building MCP Servers\n\nMCP (Model Context Protocol)...",
        "title": "Building MCP Servers",
        "length": 1523,
        "is_pdf": false,
        "extraction_quality": "high"
      }
    }
  ]
}
```

### 5. `health` - Server Status

Check server health and configuration.

**Returns:**
- `status`: `"healthy"` | `"degraded"`
- `cascade_mode`: Current stealth mode
- `rate_limiter`: Request counts and limits
- `cache_stats`: Cache size and hit rates
- `config`: Active configuration values

**Example Usage:**

```typescript
// No parameters
{}
```

## Configuration

All configuration via environment variables:

### Essential Variables

```bash
# Chrome binary path (auto-detected if not set)
CHROME_PATH=/usr/bin/google-chrome

# Profile storage (default: ~/.google-surf-mcp)
SURF_PROFILE_ROOT=/custom/path/profiles

# Browser locale and timezone
SURF_LOCALE=en-US
SURF_TZ=America/New_York
```

### Headless & CAPTCHA Recovery

```bash
# Run Chrome visibly (for demos/debugging)
SURF_HEADLESS=false

# Remote debugging mode (headless servers)
SURF_REMOTE_DEBUG=true

# Cloud/serverless mode (fail-fast on CAPTCHA)
SURF_CLOUD_MODE=true
```

### Performance Tuning

```bash
# Idle close timeout (ms), 0 disables
SURF_IDLE_CLOSE_MS=30000

# Rate limit (requests per minute)
SURF_RATE_LIMIT_PER_MIN=10

# Search cache TTL (ms), 0 disables
SURF_CACHE_TTL_SEARCH_MS=86400000

# Cache LRU size
SURF_CACHE_MAX_ENTRIES=1000
```

### Security

```bash
# Allow private IPs in extract (default: false)
SURF_ALLOW_PRIVATE=true

# Ignore TLS errors (auto-on in cloud mode)
SURF_INSECURE_TLS=false

# Disable sandbox (auto-on in cloud mode)
SURF_NO_SANDBOX=false
```

### Advanced

```bash
# Disable cascade fallback (pin single mode)
SURF_CASCADE_DISABLED=true
SURF_USE_STEALTH=true

# Humanlike browsing (off | background | inline)
SURF_HUMANLIKE_MODE=background
```

## Common Patterns

### Pattern 1: Research Assistant

Search academic papers and extract abstracts for quick review:

```typescript
// Step 1: Search and triage with abstracts
const triage = await use_mcp_tool("google-surf", "search_extract", {
  query: "transformer architecture improvements 2026",
  limit: 10,
  mode: "abstract"
});

// Step 2: Extract full text for promising papers
const topPapers = triage.results.slice(0, 3);
const fullTexts = await Promise.all(
  topPapers.map(paper => 
    use_mcp_tool("google-surf", "extract", {
      url: paper.url,
      mode: "full",
      max_chars: 100000
    })
  )
);
```

### Pattern 2: Parallel Research

Search multiple related topics simultaneously:

```typescript
const relatedTopics = await use_mcp_tool("google-surf", "search_parallel", {
  queries: [
    "MCP server authentication patterns",
    "MCP server error handling",
    "MCP server rate limiting",
    "MCP server caching strategies"
  ],
  limit: 5
});

// Process results by topic
relatedTopics.forEach((topicResults, index) => {
  console.log(`Topic ${index + 1}:`, topicResults.results.length, "results");
});
```

### Pattern 3: Content Aggregation

Build a comprehensive knowledge base:

```typescript
// 1. Find relevant sources
const sources = await use_mcp_tool("google-surf", "search", {
  query: "typescript best practices 2026",
  limit: 20
});

// 2. Extract abstracts to filter quality
const abstracts = await Promise.all(
  sources.results.map(result =>
    use_mcp_tool("google-surf", "extract", {
      url: result.url,
      mode: "abstract"
    })
  )
);

// 3. Full extraction for high-quality sources
const highQuality = abstracts
  .filter(a => a.extraction_quality === "high")
  .slice(0, 5);

const fullContent = await Promise.all(
  highQuality.map(a =>
    use_mcp_tool("google-surf", "extract", {
      url: a.url,
      mode: "full"
    })
  )
);
```

### Pattern 4: Health Check Before Heavy Operations

```typescript
// Check server health before batch operations
const health = await use_mcp_tool("google-surf", "health", {});

if (health.status !== "healthy") {
  console.warn("Server degraded, reducing concurrency");
}

const rateLimit = health.rate_limiter.requests_per_minute;
if (rateLimit > 8) {
  // Wait before starting batch
  await sleep(60000);
}
```

## CAPTCHA Recovery Modes

The server handles CAPTCHAs automatically based on environment:

### Mode 1: Local Desktop (default)

```bash
# No config needed - default behavior
```

When CAPTCHA appears:
1. OS notification fires
2. Headed Chrome window opens
3. Human solves CAPTCHA
4. Call automatically retries
5. Profile reputation preserved

### Mode 2: Visible Chrome (demos/debugging)

```bash
SURF_HEADLESS=false
```

- Chrome runs visibly at all times
- CAPTCHA recovery skips notification (user is watching)
- Good for demos and debugging

### Mode 3: Remote Debugging (headless servers)

```bash
SURF_HEADLESS=true
SURF_REMOTE_DEBUG=true
```

When CAPTCHA appears:
1. DevTools port printed to logs
2. Error thrown with instructions
3. SSH port-forward from local machine
4. Open `chrome://inspect` locally
5. Solve CAPTCHA remotely
6. Retry the call

Example SSH forward:
```bash
ssh -L 9222:localhost:9222 your-server
```

### Mode 4: Cloud/Serverless (fail-fast)

```bash
SURF_CLOUD_MODE=true
```

- No CAPTCHA recovery
- Throws `CAPTCHA_REQUIRED` error immediately
- Worker pool disabled
- Sandbox disabled, TLS bypass enabled

## Troubleshooting

### Chrome Not Found

**Error:** `Chrome binary not found`

**Solution:**

```bash
# Find your Chrome installation
which google-chrome
which chromium

# Set explicitly
CHROME_PATH=/usr/bin/google-chrome npm run bootstrap
```

### CAPTCHA Loops

**Symptoms:** Repeated CAPTCHA requests

**Solutions:**

1. Run bootstrap to warm the profile:
```bash
npm run bootstrap
```

2. Reduce request rate:
```bash
SURF_RATE_LIMIT_PER_MIN=5 npx google-surf-mcp
```

3. Check cascade mode:
```typescript
const health = await use_mcp_tool("google-surf", "health", {});
console.log(health.cascade_mode); // Should cycle: none → stealth → humanlike
```

### Empty or No Results

**Check health first:**

```typescript
const health = await use_mcp_tool("google-surf", "health", {});
// Check rate_limiter.requests_per_minute
// Check cache_stats for anomalies
```

**Clear cache if stale:**

```bash
SURF_CACHE_TTL_SEARCH_MS=0 npx google-surf-mcp
```

**Check dropped reasons:**

```typescript
const results = await use_mcp_tool("google-surf", "search", {
  query: "test query"
});
console.log(results.dropped_reasons);
// If all results dropped as "sponsored", selector may be stale
```

### Extraction Failures

**PDF extraction fails:**

```typescript
// Try metadata mode first
const meta = await use_mcp_tool("google-surf", "extract", {
  url: "https://example.com/paper.pdf",
  mode: "metadata"
});
console.log(meta.page_count); // If 0, PDF is inaccessible
```

**SSRF blocked:**

```bash
# Allow private IPs (only if you control the URLs)
SURF_ALLOW_PRIVATE=true npx google-surf-mcp
```

**Low extraction quality:**

```typescript
const result = await use_mcp_tool("google-surf", "extract", {
  url: "https://example.com/article"
});

if (result.extraction_quality === "low") {
  // HTML was poorly structured or blocked
  // Try fetching directly via other means
}
```

### Performance Issues

**Slow first call:**

Normal. First call bootstraps the profile (~4s sequential, ~9s parallel). Subsequent calls are faster (~1.5s).

**Idle timeout too aggressive:**

```bash
# Keep contexts warm longer
SURF_IDLE_CLOSE_MS=120000 npx google-surf-mcp
```

**Too many parallel queries:**

Limit to 10 per `search_parallel` call. For more, batch them:

```typescript
const queries = [...100queries];
const batches = chunk(queries, 10);

for (const batch of batches) {
  const results = await use_mcp_tool("google-surf", "search_parallel", {
    queries: batch
  });
  // Process batch
  await sleep(5000); // Respect rate limits
}
```

## Academic Sources Supported

Inline PDF extraction for:
- arXiv
- bioRxiv, medRxiv
- Nature, Science, Cell
- OpenReview
- NeurIPS, ICML, ICLR proceedings
- JMLR, PMLR
- Springer
- PubMed (via PMC)
- ACL Anthology

All extracted to markdown-formatted text.

## Cache Management

```bash
# Disable search caching
SURF_CACHE_TTL_SEARCH_MS=0

# Increase cache size
SURF_CACHE_MAX_ENTRIES=5000

# Custom cache location
SURF_CACHE_ROOT=/tmp/google-surf-cache
```

Cache namespaces:
- `search`: Google search results (24h TTL default)
- `extract`: URL content extractions (no TTL, LRU only)

## Rate Limiting

Built-in rate limiter prevents Google blocks:

```bash
# Default: 10 requests/minute
SURF_RATE_LIMIT_PER_MIN=10

# Conservative for shared IPs
SURF_RATE_LIMIT_PER_MIN=5

# Aggressive (may trigger CAPTCHAs)
SURF_RATE_LIMIT_PER_MIN=20
```

Check current usage:

```typescript
const health = await use_mcp_tool("google-surf", "health", {});
console.log(health.rate_limiter);
// { requests_per_minute: 7, limit: 10, window_start: "2026-05-17T..." }
```

## Best Practices

1. **Use abstract mode for triage**: Default `search_extract` to `mode="abstract"` to save tokens and time. Only request `mode="full"` when needed.

2. **Batch related queries**: Use `search_parallel` instead of sequential `search` calls.

3. **Check health before batch ops**: Prevents hitting rate limits mid-batch.

4. **Respect cache TTLs**: Default 24h for search is sensible. Don't disable unless debugging.

5. **Handle extraction failures gracefully**: Always check `extraction_quality` and handle `{ error }` responses.

6. **Profile warmth**: First call of the day may be slower. Acceptable for human-in-the-loop workflows.

7. **CAPTCHA strategy**: For long-running agents, use `SURF_CLOUD_MODE=false` and solve CAPTCHAs as they appear to preserve profile reputation.
