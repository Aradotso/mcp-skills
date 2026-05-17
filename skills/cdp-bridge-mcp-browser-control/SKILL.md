---
name: cdp-bridge-mcp-browser-control
description: Control and automate real browser sessions through CDP, preserving login state and cookies for LLM-driven interactions
triggers:
  - "connect to my browser tabs"
  - "execute JavaScript in the current page"
  - "scan the current webpage content"
  - "take a screenshot of this page"
  - "get cookies from the browser"
  - "navigate to a URL in the browser"
  - "switch between browser tabs"
  - "wait for page element to load"
---

# CDP Bridge MCP Browser Control

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

CDP Bridge MCP is a Model Context Protocol server that bridges LLM clients to **real, active browser sessions** through Chrome DevTools Protocol (CDP) and a companion browser extension. Unlike headless automation tools, it connects to your already-open, already-logged-in browser tabs, preserving authentication state, cookies, and rendered page content.

**Key differentiators:**
- **Reuses real login sessions** — no need to re-authenticate or transfer cookies
- **Works with current browser state** — connects to tabs you already have open
- **LLM-optimized page scanning** — filters HTML to preserve useful content while reducing tokens
- **Automatic CSP handling** — falls back to CDP when content security policies block script injection
- **Lightweight setup** — no separate browser instances or complex configuration

## Installation

### 1. Install the MCP Server

The server is available via PyPI and can be run with `uvx`:

```bash
# Test with stdio mode (default)
uvx cdp-bridge@latest

# Run as HTTP server (for multi-client scenarios)
uvx cdp-bridge@latest --transport streamable-http --port 8000
```

### 2. Load Browser Extension

1. Navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select the `src/cdp_bridge/tmwd_cdp_bridge` folder from the repository

The extension automatically attempts to connect to `127.0.0.1:18765` (WebSocket) and retries every ~5 seconds if the server isn't running yet.

### 3. Configure MCP Client

**For stdio mode (Claude Desktop, most local clients):**

```json
{
  "mcpServers": {
    "cdp-bridge": {
      "command": "uvx",
      "args": ["cdp-bridge@latest"]
    }
  }
}
```

**For streamable-http mode (shared/Docker deployments):**

First start the server:
```bash
uvx cdp-bridge@latest --transport streamable-http --port 8000
```

Then configure the client:
```json
{
  "mcpServers": {
    "cdp-bridge": {
      "type": "streamableHttp",
      "url": "http://127.0.0.1:8000/mcp"
    }
  }
}
```

**Claude Code:**
```bash
# stdio
claude mcp add cdp-bridge uvx cdp-bridge@latest

# streamable-http
claude mcp add cdp-bridge --transport streamable-http http://127.0.0.1:8000/mcp
```

**Codex:**
```bash
# stdio
codex mcp add cdp-bridge uvx cdp-bridge@latest

# streamable-http
codex mcp add cdp-bridge --transport streamable-http --url http://127.0.0.1:8000/mcp
```

## Available Tools

| Tool | Purpose |
|------|---------|
| `browser_get_tabs` | List all connected browser tabs with URLs and titles |
| `browser_scan` | Extract page content as simplified HTML or plain text |
| `browser_execute_js` | Run JavaScript in the active tab |
| `browser_switch_tab` | Change which tab is active for MCP operations |
| `browser_batch` | Execute multiple CDP/extension commands atomically |
| `browser_wait` | Poll a JavaScript condition until true or timeout |
| `browser_navigate` | Navigate the active tab to a URL |
| `browser_screenshot` | Capture page screenshot as base64 PNG |
| `browser_cookies` | Read cookies for the current domain |

## Common Usage Patterns

### Get Available Tabs

```python
# When user asks: "what tabs do I have open?"
result = await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_get_tabs",
    arguments={}
)

# Result structure:
# {
#   "tabs": [
#     {"id": "tab_0", "url": "https://example.com", "title": "Example Domain"},
#     {"id": "tab_1", "url": "https://github.com", "title": "GitHub"}
#   ],
#   "active_tab": "tab_0"
# }
```

### Scan Page Content

```python
# Extract simplified HTML (default)
result = await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_scan",
    arguments={
        "tab_id": "tab_0",
        "format": "html"  # or "text" for plain text
    }
)

# Returns cleaned HTML with scripts/styles removed
# Preserves semantic structure for LLM consumption
```

**How `browser_scan` works:**
- Removes `<script>`, `<style>`, `<svg>`, hidden elements
- Keeps text content, links, headings, form controls
- Converts `<img>` to `[Image: alt_text]`
- Ideal for reducing token usage while preserving page semantics

### Execute JavaScript

```python
# Extract data or interact with the page
result = await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_execute_js",
    arguments={
        "tab_id": "tab_0",
        "code": """
            return {
                title: document.title,
                links: Array.from(document.querySelectorAll('a'))
                    .slice(0, 10)
                    .map(a => ({href: a.href, text: a.innerText}))
            };
        """
    }
)

# Result contains the returned object
```

**Execution modes:**
1. **Primary**: Uses `chrome.scripting.executeScript` in MAIN world
2. **Fallback**: Uses CDP `Runtime.evaluate` if CSP blocks injection

### Wait for Dynamic Content

```python
# Wait for element to appear (useful for SPAs)
result = await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_wait",
    arguments={
        "tab_id": "tab_0",
        "condition": "document.querySelector('.dynamic-content') !== null",
        "timeout": 10,  # seconds
        "interval": 0.5  # poll every 500ms
    }
)

# Returns True if condition met, raises timeout error otherwise
```

### Navigate and Screenshot

```python
# Navigate to a URL
await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_navigate",
    arguments={
        "tab_id": "tab_0",
        "url": "https://example.com"
    }
)

# Wait for page load
await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_wait",
    arguments={
        "tab_id": "tab_0",
        "condition": "document.readyState === 'complete'",
        "timeout": 30
    }
)

# Capture screenshot
screenshot = await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_screenshot",
    arguments={
        "tab_id": "tab_0",
        "format": "png"  # returns base64-encoded image
    }
)
```

### Read Cookies

```python
# Get cookies for authenticated session analysis
result = await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_cookies",
    arguments={
        "tab_id": "tab_0"
    }
)

# Returns array of cookie objects:
# [{"name": "session_id", "value": "...", "domain": "example.com", ...}]
```

### Batch Operations (Advanced)

```python
# Execute multiple CDP commands in sequence
result = await use_mcp_tool(
    server_name="cdp-bridge",
    tool_name="browser_batch",
    arguments={
        "tab_id": "tab_0",
        "commands": [
            {
                "method": "Runtime.evaluate",
                "params": {
                    "expression": "document.title",
                    "returnByValue": True
                }
            },
            {
                "method": "Network.getCookies",
                "params": {}
            }
        ]
    }
)

# Returns array of results matching command order
```

## Real-World Examples

### Extract Article Content from Logged-In Site

```python
# User is already logged in to medium.com
# 1. Find the article tab
tabs = await use_mcp_tool("cdp-bridge", "browser_get_tabs", {})
article_tab = next(t for t in tabs["tabs"] if "medium.com" in t["url"])

# 2. Switch to it
await use_mcp_tool("cdp-bridge", "browser_switch_tab", {"tab_id": article_tab["id"]})

# 3. Extract article text
content = await use_mcp_tool(
    "cdp-bridge",
    "browser_execute_js",
    {
        "tab_id": article_tab["id"],
        "code": """
            const article = document.querySelector('article');
            return article ? article.innerText : 'No article found';
        """
    }
)
```

### Monitor Dashboard Data

```python
# User has analytics dashboard open
# Poll for updated metrics every 30 seconds

while True:
    metrics = await use_mcp_tool(
        "cdp-bridge",
        "browser_execute_js",
        {
            "tab_id": "tab_0",
            "code": """
                return {
                    visitors: document.querySelector('.visitor-count')?.innerText,
                    revenue: document.querySelector('.revenue')?.innerText,
                    timestamp: Date.now()
                };
            """
        }
    )
    
    # Process metrics...
    await asyncio.sleep(30)
```

### Fill Form in Authenticated Session

```python
# Navigate to form page
await use_mcp_tool(
    "cdp-bridge",
    "browser_navigate",
    {"tab_id": "tab_0", "url": "https://example.com/settings"}
)

# Wait for form to load
await use_mcp_tool(
    "cdp-bridge",
    "browser_wait",
    {
        "tab_id": "tab_0",
        "condition": "document.querySelector('form#settings') !== null",
        "timeout": 10
    }
)

# Fill and submit
await use_mcp_tool(
    "cdp-bridge",
    "browser_execute_js",
    {
        "tab_id": "tab_0",
        "code": """
            const form = document.querySelector('form#settings');
            form.querySelector('input[name="email"]').value = 'user@example.com';
            form.querySelector('input[name="notifications"]').checked = true;
            form.submit();
        """
    }
)
```

## Troubleshooting

### Extension Shows "ERR_CONNECTION_REFUSED"

**Normal behavior** on first load. The extension retries connection every ~5 seconds. Once you invoke any MCP tool (e.g., `browser_get_tabs`), the server starts and the extension connects automatically within seconds.

### No Tabs Appear in `browser_get_tabs`

1. Verify the extension is loaded and enabled at `chrome://extensions/`
2. Check the extension's service worker console for connection status
3. Ensure MCP server is running (invoke any tool to start it in stdio mode)
4. Confirm WebSocket server is listening on `127.0.0.1:18765`

### JavaScript Execution Fails with CSP Error

The extension **automatically falls back** to CDP `Runtime.evaluate` when CSP blocks `chrome.scripting`. If both fail:
- Check browser console for specific CSP violations
- Try wrapping code in `(function() { ... })()`
- Avoid accessing restricted APIs (e.g., cross-origin iframes)

### Tab IDs Change Unexpectedly

Tab IDs are session-based and reset when:
- The extension reloads
- The browser restarts
- WebSocket reconnects

Always call `browser_get_tabs` before targeting a specific tab if state may have changed.

### High Token Usage from `browser_scan`

1. Use `"format": "text"` for plain text extraction (fewer tokens)
2. Limit scope by executing JS to extract specific DOM subtrees first:

```python
await use_mcp_tool(
    "cdp-bridge",
    "browser_execute_js",
    {
        "tab_id": "tab_0",
        "code": "document.body.innerHTML = document.querySelector('main').innerHTML"
    }
)

# Then scan the reduced page
await use_mcp_tool("cdp-bridge", "browser_scan", {"tab_id": "tab_0"})
```

## Configuration Options

### Server Launch Arguments

```bash
uvx cdp-bridge@latest [OPTIONS]

Options:
  --transport [stdio|streamable-http]  Transport mode (default: stdio)
  --port INTEGER                       HTTP port for streamable-http mode (default: 8000)
```

### Extension Configuration

Edit `src/cdp_bridge/tmwd_cdp_bridge/background.js` to customize:

```javascript
// WebSocket server address
const WS_URL = 'ws://127.0.0.1:18765';

// Reconnection interval (ms)
const RECONNECT_INTERVAL = 5000;
```

## Architecture Notes

- **Transport modes**: `stdio` for single-client (subprocess), `streamable-http` for multi-client (persistent server)
- **Extension communication**: WebSocket (primary) + HTTP long-polling (fallback)
- **JavaScript execution**: MAIN world via `chrome.scripting`, falls back to CDP for CSP-restricted pages
- **Session management**: Each connected tab gets a unique session ID, managed by `TMWebDriver`

## Requirements

- Python 3.10+
- Chrome or Chromium-based browser
- Browser extension loaded in developer mode
- Network access to `127.0.0.1:18765` (WebSocket) and `127.0.0.1:18766` (HTTP)
