---
name: gemini-search-mcp-web-search
description: MCP server providing free, unlimited web search powered by Google AI Mode (Gemini) without requiring API keys
triggers:
  - search the web using gemini
  - set up free web search for my AI agent
  - integrate google gemini search into claude
  - add web search capability without api key
  - configure gemini-search-mcp server
  - use google ai mode for real-time search
  - enable unlimited web search in cursor
  - connect gemini search to my mcp client
---

# gemini-search-mcp Web Search Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`gemini-search-mcp` is an MCP server that provides real-time web search capabilities to AI agents (Claude Desktop, Cursor, Windsurf, etc.) using Google's AI Mode powered by Gemini. It bypasses traditional search API limitations by using Chrome DevTools Protocol (CDP) to execute searches through a real browser, giving unlimited, free access to Google Search results without requiring API keys.

The server exposes two MCP tools:
- **`web_search(query)`** - Search the web and get synthesized answers grounded in real-time results
- **`ask(prompt)`** - General questions where AI Mode auto-decides whether to search

Additionally, it can run as an OpenAI-compatible API server for non-MCP clients.

## Installation

### Basic Installation

```bash
pip install -e .
```

### With Undetected ChromeDriver (CAPTCHA Resistance)

```bash
pip install -e '.[undetected]'
```

### Requirements

- Python 3.10+
- Chrome, Edge, or Chromium browser installed
- No API keys required

## MCP Server Setup

### Claude Code

```bash
claude mcp add gemini-search -- gemini-search-mcp
```

### Claude Desktop

Edit `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "gemini-search": {
      "command": "gemini-search-mcp",
      "args": [],
      "env": {
        "HEADLESS": "1",
        "BROWSER_CHANNEL": "chrome"
      }
    }
  }
}
```

### Cursor / Windsurf

Add to your MCP settings (`.cursor/mcp.json` or similar):

```json
{
  "mcpServers": {
    "gemini-search": {
      "command": "gemini-search-mcp",
      "args": []
    }
  }
}
```

## Running as OpenAI-Compatible API

Start the server:

```bash
gemini-search --port 8080
```

Use with any OpenAI-compatible client:

```python
import openai

client = openai.OpenAI(
    base_url="http://localhost:8080/v1",
    api_key="not-needed"
)

response = client.chat.completions.create(
    model="gemini-search",
    messages=[
        {"role": "user", "content": "What are the latest AI developments in 2026?"}
    ]
)

print(response.choices[0].message.content)
```

```bash
# cURL example
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-search",
    "messages": [{"role": "user", "content": "Bitcoin price today"}]
  }'
```

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CDP_URL` | None | Chrome DevTools URL (e.g., `http://127.0.0.1:9222`). Connects to existing Chrome |
| `BROWSER_CHANNEL` | `chrome` | Browser: `chrome`, `msedge`, or `chromium` |
| `HEADLESS` | `1` | Set to `0` to show browser window |
| `GEMINI_SEARCH_USER_DATA_DIR` | None | Persistent Chrome profile directory for cookie reuse |
| `GEMINI_SEARCH_CDP_PORT` | `19250` | CDP port for self-launched Chrome |
| `GEMINI_SEARCH_BROWSER_BACKEND` | `subprocess` | Backend: `subprocess` or `undetected` |
| `GEMINI_SEARCH_PROXY_SERVER` | None | Proxy (e.g., `socks5://127.0.0.1:7897`) |
| `GEMINI_SEARCH_CHROMEDRIVER` | None | Chromedriver path for `undetected` backend |

### Example with Custom Configuration

```json
{
  "mcpServers": {
    "gemini-search": {
      "command": "gemini-search-mcp",
      "args": [],
      "env": {
        "HEADLESS": "1",
        "BROWSER_CHANNEL": "msedge",
        "GEMINI_SEARCH_USER_DATA_DIR": "/home/user/.cache/gemini-search-profile",
        "GEMINI_SEARCH_PROXY_SERVER": "socks5://127.0.0.1:7897"
      }
    }
  }
}
```

## Using the MCP Tools

### web_search Tool

Use when you need real-time web information:

```python
# In an MCP client context, the agent would call:
# web_search("latest AI regulation news 2026")

# Returns synthesized answer like:
# "The EU AI Act enforcement began on June 1, 2026, requiring..."
```

**Example queries:**
- `web_search("current weather in Tokyo")`
- `web_search("latest SpaceX launch date")`
- `web_search("Python 3.13 new features")`
- `web_search("best laptop 2026 under $1000")`

### ask Tool

Use for general questions where AI Mode decides if web search is needed:

```python
# ask("what is 1847 * 293")
# Returns: "541171" (computed without web search)

# ask("who won the latest Formula 1 race")
# Triggers web search and returns current results
```

## Common Patterns

### Persistent Profile for CAPTCHA Avoidance

If Google shows CAPTCHA for temporary profiles, create a persistent profile:

```bash
# First run: visible mode, solve CAPTCHA if needed
mkdir -p ~/.local/share/gemini-search-mcp/chrome-profile
gemini-search --no-headless --user-data-dir ~/.local/share/gemini-search-mcp/chrome-profile
```

Then configure for headless reuse:

```json
{
  "mcpServers": {
    "gemini-search": {
      "command": "gemini-search-mcp",
      "args": [],
      "env": {
        "GEMINI_SEARCH_USER_DATA_DIR": "/home/user/.local/share/gemini-search-mcp/chrome-profile",
        "HEADLESS": "1"
      }
    }
  }
}
```

### Using with Proxy

```bash
export GEMINI_SEARCH_PROXY_SERVER="socks5://127.0.0.1:7897"
gemini-search-mcp
```

### Connect to Existing Chrome Instance

Launch Chrome with remote debugging:

```bash
# Linux/Mac
google-chrome --remote-debugging-port=9222 --user-data-dir=/tmp/chrome-profile

# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir=C:\temp\chrome-profile
```

Configure MCP server to use it:

```json
{
  "mcpServers": {
    "gemini-search": {
      "command": "gemini-search-mcp",
      "args": [],
      "env": {
        "CDP_URL": "http://127.0.0.1:9222"
      }
    }
  }
}
```

### Docker Deployment

```bash
# Using docker-compose
docker compose up -d

# Access at http://localhost:8080
```

Create `docker-compose.yml`:

```yaml
version: '3.8'
services:
  gemini-search:
    build: .
    ports:
      - "8080:8080"
    environment:
      - HEADLESS=1
      - BROWSER_CHANNEL=chrome
    command: gemini-search --port 8080 --host 0.0.0.0
```

## Troubleshooting

### CAPTCHA Issues

**Problem:** Google shows `/sorry/` CAPTCHA page

**Solution 1:** Use persistent profile (see above)

**Solution 2:** Try undetected-chromedriver backend:

```bash
pip install -e '.[undetected]'

# Test first
python scripts/uc_google_probe.py --out-json uc-probe.json

# If probe shows ok=true and captcha=false, use it
gemini-search-mcp --browser-backend undetected
```

**Solution 3:** Use a different browser channel:

```bash
export BROWSER_CHANNEL=msedge
gemini-search-mcp
```

### Browser Launch Failures

**Problem:** Chrome fails to start

**Solutions:**
- Verify Chrome is installed: `google-chrome --version` or `chromium --version`
- Try different browser: `export BROWSER_CHANNEL=chromium`
- Use visible mode for debugging: `export HEADLESS=0`
- Check CDP port isn't in use: `lsof -i :19250`

### Connection Issues

**Problem:** MCP client can't connect to server

**Solutions:**
- Check server is running: `ps aux | grep gemini-search`
- Verify stdio communication (MCP uses stdio, not HTTP)
- Check logs in Claude Desktop: `~/Library/Logs/Claude/` (Mac) or `%APPDATA%\Claude\logs` (Windows)
- Restart Claude Desktop after config changes

### Rate Limiting

**Problem:** Getting throttled despite "unlimited" claim

**Solutions:**
- The tool avoids rate limits via real Chrome fingerprint
- If throttled, you may have network/IP issues
- Try with proxy: `export GEMINI_SEARCH_PROXY_SERVER=socks5://127.0.0.1:7897`
- Ensure using real Chrome, not chromium-headless-shell

### Slow Response Times

**Problem:** Searches take longer than expected ~1.5s average

**Solutions:**
- First request is slower (browser startup)
- Use persistent browser via `CDP_URL` for faster subsequent requests
- Check network latency to Google
- Reduce proxy overhead if using one

### DOM Structure Changes

**Problem:** Search results not parsing correctly

**Solutions:**
- Google may have updated their AI Mode DOM structure
- Check for project updates: `git pull origin main`
- Report issues on GitHub with example query
- Use `--no-headless` to visually inspect what Chrome sees

## Advanced Usage

### Custom Search Parameters

The server uses Google's AI Mode endpoint internally. For advanced control:

```python
# When calling from Python code (not common for MCP usage)
from gemini_search_mcp import search

result = search.web_search(
    query="Python best practices 2026",
    # Additional parameters handled internally
)
```

### Multiple Concurrent Searches

Each MCP tool call is independent. The server handles concurrency:

```python
# AI agent can make multiple calls
# web_search("topic A")
# web_search("topic B")
# Results arrive as they complete
```

### Integration with LangChain

```python
from langchain.tools import Tool
import requests

def gemini_search(query: str) -> str:
    response = requests.post(
        "http://localhost:8080/v1/chat/completions",
        json={
            "model": "gemini-search",
            "messages": [{"role": "user", "content": query}]
        }
    )
    return response.json()["choices"][0]["message"]["content"]

search_tool = Tool(
    name="GeminiWebSearch",
    func=gemini_search,
    description="Search the web using Google AI Mode for current information"
)
```

## Key Differences from Alternatives

- **vs Tavily/SerpAPI**: No API key, no cost, no quotas
- **vs Grok MCP**: Uses Google Search instead of X.ai, no API key needed
- **vs Direct HTTP**: Avoids TLS fingerprint detection via real Chrome browser
- **vs Browser automation**: Optimized CDP approach, faster than Selenium
