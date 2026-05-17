---
name: kagi-session2api-mcp-server
description: Free Kagi Search MCP server using session tokens for web search and URL summarization in AI coding agents
triggers:
  - search the web using Kagi
  - summarize this URL with Kagi
  - set up Kagi MCP server
  - configure Kagi session tokens
  - search for information online
  - get a summary of this article
  - install Kagi search integration
  - use Kagi to find resources
---

# Kagi Session2API MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This MCP server provides free access to Kagi search and summarizer features using session tokens instead of paid API keys. It works with Claude Desktop, Cursor, Windsurf, Cline, and any MCP-compatible client.

## What It Does

- **kagi_search_fetch**: Performs web searches using Kagi's search engine
- **kagi_summarizer**: Summarizes any URL using Kagi's summarization engines
- **Multi-token pooling**: Supports multiple session tokens with round-robin rotation for higher throughput
- **Rate limiting**: Built-in 5 req/s per token with automatic expired token detection
- **Search operators**: Full support for Kagi search operators (site:, lang:, filetype:, etc.)

## Installation

### Quick Install (Recommended)

```bash
pip install kagi-session2api-mcp
```

Or use `uvx` for isolated execution:

```bash
uvx kagi-session2api-mcp
```

### Getting Your Session Token

1. Log in to [kagi.com](https://kagi.com)
2. Navigate to **Settings → Account → Session Link**
3. Copy the token from the session URL: `https://kagi.com/search?token=YOUR_TOKEN_HERE&q=test`
4. Store this token securely (treat it like a password)

## Configuration

### Single Token Setup (Claude Desktop)

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "kagi-session": {
      "command": "uvx",
      "args": ["kagi-session2api-mcp"],
      "env": {
        "KAGI_SESSION_TOKEN": "YOUR_SESSION_TOKEN_HERE"
      }
    }
  }
}
```

### Multiple Tokens (Environment Variable)

For higher throughput with token rotation:

```json
{
  "mcpServers": {
    "kagi-session": {
      "command": "uvx",
      "args": ["kagi-session2api-mcp"],
      "env": {
        "KAGI_SESSION_TOKENS": "TOKEN_1,TOKEN_2,TOKEN_3"
      }
    }
  }
}
```

### Multiple Tokens (Config File - Recommended)

Create `~/.config/kagi-session2api-mcp/config.toml`:

```toml
[kagi]
session_tokens = [
    "YOUR_TOKEN_1_HERE",
    "YOUR_TOKEN_2_HERE",
    "YOUR_TOKEN_3_HERE"
]

# Summarizer engine: cecil (default), agnes, daphne, muriel
summarizer_engine = "cecil"

[client]
timeout = 30
max_retries = 2
```

Reference in MCP config:

```json
{
  "mcpServers": {
    "kagi-session": {
      "command": "uvx",
      "args": ["kagi-session2api-mcp"],
      "env": {
        "KAGI_SESSION_CONFIG": "/home/user/.config/kagi-session2api-mcp/config.toml"
      }
    }
  }
}
```

### Cursor/Windsurf Configuration

Add to `.cursor/mcp.json` or Windsurf settings:

```json
{
  "mcpServers": {
    "kagi-session": {
      "command": "uvx",
      "args": ["kagi-session2api-mcp"],
      "env": {
        "KAGI_SESSION_TOKEN": "YOUR_SESSION_TOKEN_HERE"
      }
    }
  }
}
```

## Using the Tools

### Web Search (kagi_search_fetch)

Basic search:

```
Search for "Python asyncio best practices"
```

The tool returns structured results with:
- Title
- URL
- Snippet
- Published date (if available)

### Search with Operators

Kagi supports powerful search operators:

```
Search for: site:github.com Python MCP server
```

```
Find: "async programming" filetype:pdf lang:en
```

```
Look up: React hooks -site:reddit.com after:2024-01-01
```

**Available operators:**
- `site:domain.com` - Restrict to specific domain
- `-site:domain.com` - Exclude domain
- `filetype:pdf` - Filter by file type (pdf, doc, ppt, etc.)
- `intitle:keyword` - Search in page titles
- `lang:en` - Language filter (en, zh, es, etc.)
- `before:2024-01-01` - Results before date
- `after:2024-01-01` - Results after date
- `"exact phrase"` - Exact phrase match

### URL Summarization (kagi_summarizer)

Basic summarization:

```
Summarize https://example.com/long-article
```

With options:

```python
# In tool call parameters:
{
  "url": "https://example.com/article",
  "summary_type": "takeaway",  # "summary" (prose) or "takeaway" (bullets)
  "engine": "cecil",            # cecil, agnes, daphne, muriel
  "target_language": "EN"       # Language code
}
```

**Summarizer engines:**
- `cecil` (default): Balanced, general-purpose
- `agnes`: Fast, concise
- `daphne`: Detailed, thorough
- `muriel`: Technical, precise

## Common Patterns

### Research Assistant Pattern

```
Search for recent papers on: site:arxiv.org "neural networks" after:2024-01-01
```

Then summarize the most relevant:

```
Summarize https://arxiv.org/abs/2401.12345 with engine daphne
```

### Code Documentation Search

```
Search for: site:docs.python.org asyncio after:2023-01-01
```

### News Aggregation

```
Find latest news: "AI regulation" lang:en after:2024-05-01 -site:twitter.com
```

### Technical Resource Discovery

```
Search: "Docker best practices" filetype:pdf OR site:github.com
```

## Transport Modes

### Stdio (Default for Desktop Clients)

```bash
kagi-session2api-mcp
```

### HTTP Server (Remote Access)

```bash
kagi-session2api-mcp --http --host 0.0.0.0 --port 8000
```

Then connect via HTTP:

```json
{
  "mcpServers": {
    "kagi-session": {
      "url": "http://localhost:8000"
    }
  }
}
```

## Troubleshooting

### "Token expired" or 401 errors

- Session tokens expire after a period of inactivity
- Get a new token from Kagi settings
- Update your configuration with the new token
- Expired tokens are automatically disabled in multi-token pools

### "Rate limit exceeded"

- Each token is limited to 5 requests/second
- Add more tokens to increase throughput
- With N tokens, effective rate = 5×N req/s

### No search results returned

- Check if your query is properly formatted
- Try without operators first
- Verify the token is valid (test on kagi.com)
- Check network connectivity

### Summarizer fails

- Ensure the URL is publicly accessible
- Some sites block automated access
- Try a different summarizer engine
- The summarizer uses an internal Kagi endpoint that may change

### MCP client can't find the server

- Restart your MCP client (Claude Desktop, Cursor, etc.)
- Check the config file path is correct
- Verify `uvx` or `python` is in PATH
- Check logs in MCP client for error messages

### Python version issues

- Requires Python 3.10+
- Check version: `python --version`
- Use `uv` for automatic version management: `uvx kagi-session2api-mcp`

## Security Considerations

⚠️ **Important Security Notes:**

1. **Never commit tokens to version control** - Use environment variables or config files outside your repo
2. **Treat session tokens like passwords** - They provide full account access
3. **Use separate tokens per machine** - If compromised, only one session is affected
4. **Rotate tokens periodically** - Generate new ones from Kagi settings
5. **Monitor account activity** - Check Kagi account for unexpected usage

## Rate Limiting Behavior

| Tokens | Per-Token Rate | Total Throughput |
|--------|----------------|------------------|
| 1      | 5 req/s        | 5 req/s          |
| 2      | 5 req/s        | 10 req/s         |
| 5      | 5 req/s        | 25 req/s         |

The server uses a token bucket algorithm with automatic round-robin rotation. When one token is rate-limited, the next token in the pool is used immediately.

## Programmatic Usage (Python)

While primarily an MCP server, you can also use it as a Python library:

```python
import asyncio
from kagi_session2api_mcp import KagiClient

async def main():
    client = KagiClient(session_token="YOUR_TOKEN")
    
    # Search
    results = await client.search("Python asyncio")
    for result in results:
        print(f"{result['title']}: {result['url']}")
    
    # Summarize
    summary = await client.summarize(
        url="https://example.com/article",
        summary_type="takeaway",
        engine="cecil"
    )
    print(summary)

asyncio.run(main())
```

## Comparison with Official kagimcp

| Feature | kagi-session2api-mcp | Official kagimcp |
|---------|---------------------|------------------|
| Cost | Free (session token) | $25/1000 queries |
| Authentication | Session token | API key |
| Rate limiting | Client-side (5 req/s) | Server-side |
| Multi-token | ✅ Supported | N/A |
| API balance | Not available | Available |
| Stability | Depends on HTML structure | Official API |

## Legal and Risk Warnings

⚠️ **Use at your own risk:**

- Using session tokens may violate [Kagi's Terms of Service](https://kagi.com/terms)
- Potential consequences: account suspension or permanent ban
- Kagi may change HTML structure, breaking functionality
- Authors assume no liability for account actions or data loss
- Not affiliated with or endorsed by Kagi

This tool is for educational and personal use. Consider purchasing official API access for production use.
