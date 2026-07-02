---
name: browser-mcp-control
description: Control Chrome browser tabs via MCP for AI-assisted web automation using the Browser MCP server and extension
triggers:
  - navigate to a website using browser mcp
  - take a screenshot of the current page
  - click an element in the browser
  - get page content snapshot
  - interact with browser extension
  - automate browser actions through mcp
  - fill out web form using browser
  - extract data from web page
---

# Browser MCP Control

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What Browser MCP Does

Browser MCP is a Model Context Protocol server that bridges AI assistants to your local Chrome browser through a Chrome extension. Unlike headless automation tools, it operates on your **existing browser profile**, preserving login sessions and cookies, reducing CAPTCHA triggers and bot detection.

The architecture uses:
- **MCP Server** (stdio) that AI clients spawn as subprocess
- **WebSocket bridge** (port 9009) for extension communication
- **Chrome Extension** that executes commands in active tabs
- **Optional Redis** for session persistence

## Installation

### 1. Install the MCP Server

```bash
git clone https://github.com/BrowserMCP/mcp.git
cd mcp
npm install
npm run build
```

### 2. Configure Your MCP Client

Add to your MCP settings (Cursor, Claude Desktop, VS Code, etc.):

**Cursor/Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "browsermcp": {
      "command": "node",
      "args": ["/absolute/path/to/mcp/dist/index.js"]
    }
  }
}
```

**VS Code** (`.vscode/settings.json`):

```json
{
  "mcp.servers": {
    "browsermcp": {
      "command": "node",
      "args": ["/absolute/path/to/mcp/dist/index.js"]
    }
  }
}
```

### 3. Install Chrome Extension

Install the Browser MCP extension from [browsermcp.io](https://browsermcp.io) or the Chrome Web Store.

### 4. Connect Extension

1. Restart your AI client to spawn the MCP server
2. Open Chrome and click the Browser MCP extension icon
3. Click **Connect** on the active tab
4. Extension connects via WebSocket to localhost:9009

## Configuration

Create `.env` in the project root:

```bash
# WebSocket port for Chrome extension
WS_PORT=9009

# Logging
LOG_LEVEL=info

# Redis (optional session persistence)
REDIS_ENABLED=false
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0
REDIS_KEY_PREFIX=browsermcp:
REDIS_SESSION_TTL_SECONDS=86400
```

## Core Tools

### Navigation Tools

#### browser_navigate

Navigate to a URL:

```typescript
// Tool call from AI
{
  "tool": "browser_navigate",
  "arguments": {
    "url": "https://github.com/trending",
    "waitUntil": "networkidle"
  }
}
```

**Parameters:**
- `url` (required): Target URL
- `waitUntil` (optional): `load`, `domcontentloaded`, `networkidle`

#### browser_go_back / browser_go_forward

```typescript
{
  "tool": "browser_go_back"
}

{
  "tool": "browser_go_forward"
}
```

#### browser_wait

Wait for page state or timeout:

```typescript
{
  "tool": "browser_wait",
  "arguments": {
    "seconds": 3
  }
}
```

### Interaction Tools

#### browser_click

Click an element by selector or coordinates:

```typescript
// Click by CSS selector
{
  "tool": "browser_click",
  "arguments": {
    "selector": "button.submit-btn"
  }
}

// Click by coordinates
{
  "tool": "browser_click",
  "arguments": {
    "x": 250,
    "y": 400
  }
}
```

**Parameters:**
- `selector` (optional): CSS selector
- `x`, `y` (optional): Pixel coordinates
- `button` (optional): `left`, `right`, `middle`
- `clickCount` (optional): Single/double/triple click

#### browser_type

Type text into an input:

```typescript
{
  "tool": "browser_type",
  "arguments": {
    "selector": "input[name='email']",
    "text": "user@example.com",
    "pressEnter": false
  }
}
```

**Parameters:**
- `selector` (required): CSS selector for input
- `text` (required): Text to type
- `pressEnter` (optional): Submit after typing

#### browser_select_option

Select dropdown option:

```typescript
{
  "tool": "browser_select_option",
  "arguments": {
    "selector": "select#country",
    "value": "USA"
  }
}
```

#### browser_hover

Hover over element:

```typescript
{
  "tool": "browser_hover",
  "arguments": {
    "selector": ".tooltip-trigger"
  }
}
```

#### browser_press_key

Press keyboard keys:

```typescript
{
  "tool": "browser_press_key",
  "arguments": {
    "key": "Enter"
  }
}
```

**Common keys:** `Enter`, `Escape`, `Tab`, `Backspace`, `ArrowDown`, `ArrowUp`

### Inspection Tools

#### browser_snapshot

Get ARIA accessibility tree as YAML:

```typescript
{
  "tool": "browser_snapshot"
}
```

Returns structured page content for AI analysis:

```yaml
- role: navigation
  name: Main navigation
  - role: link
    name: Home
  - role: link
    name: About
- role: main
  - role: heading
    name: Welcome
    level: 1
  - role: article
    - role: heading
      name: Getting Started
      level: 2
```

#### browser_screenshot

Capture page screenshot:

```typescript
{
  "tool": "browser_screenshot",
  "arguments": {
    "quality": 90,
    "fullPage": false
  }
}
```

**Parameters:**
- `quality` (optional): JPEG quality 0-100
- `fullPage` (optional): Capture entire scrollable page

Returns base64-encoded image.

#### browser_get_console_logs

Retrieve browser console messages:

```typescript
{
  "tool": "browser_get_console_logs"
}
```

Returns array of log entries with type, message, timestamp.

## Common Patterns

### Login Flow

```typescript
// 1. Navigate to login page
await useTool("browser_navigate", {
  url: "https://app.example.com/login"
});

// 2. Fill credentials
await useTool("browser_type", {
  selector: "input[name='username']",
  text: process.env.USERNAME
});

await useTool("browser_type", {
  selector: "input[name='password']",
  text: process.env.PASSWORD
});

// 3. Submit
await useTool("browser_click", {
  selector: "button[type='submit']"
});

// 4. Wait for redirect
await useTool("browser_wait", {
  seconds: 2
});
```

### Form Automation

```typescript
// Navigate to form
await useTool("browser_navigate", {
  url: "https://example.com/contact"
});

// Fill text fields
await useTool("browser_type", {
  selector: "#name",
  text: "John Doe"
});

await useTool("browser_type", {
  selector: "#email",
  text: "john@example.com"
});

// Select dropdown
await useTool("browser_select_option", {
  selector: "#topic",
  value: "sales"
});

// Fill textarea
await useTool("browser_type", {
  selector: "textarea#message",
  text: "I'd like to discuss..."
});

// Submit
await useTool("browser_click", {
  selector: "button.submit"
});
```

### Data Extraction

```typescript
// Navigate to target page
await useTool("browser_navigate", {
  url: "https://news.ycombinator.com"
});

// Get structured content
const snapshot = await useTool("browser_snapshot");

// Parse YAML snapshot to extract:
// - Headlines (role: link)
// - Scores (role: text)
// - Comment counts

// Or take screenshot for visual verification
const screenshot = await useTool("browser_screenshot", {
  fullPage: true
});
```

### Multi-Step Workflow

```typescript
// 1. Search
await useTool("browser_navigate", {
  url: "https://github.com/search"
});

await useTool("browser_type", {
  selector: "input[name='q']",
  text: "mcp server",
  pressEnter: true
});

await useTool("browser_wait", { seconds: 2 });

// 2. Filter results
await useTool("browser_click", {
  selector: "button[data-filter='repositories']"
});

await useTool("browser_wait", { seconds: 1 });

// 3. Extract data
const content = await useTool("browser_snapshot");

// 4. Navigate to first result
await useTool("browser_click", {
  selector: ".repo-list-item:first-child a"
});

// 5. Capture repo details
const repoSnapshot = await useTool("browser_snapshot");
```

## Error Handling

### Connection Errors

```typescript
try {
  await useTool("browser_navigate", {
    url: "https://example.com"
  });
} catch (error) {
  if (error.message.includes("No connection")) {
    console.error("Chrome extension not connected. Open extension popup and click Connect.");
  }
}
```

### Timeout Handling

```typescript
// Increase wait time for slow pages
await useTool("browser_navigate", {
  url: "https://slow-site.com",
  waitUntil: "networkidle"
});

await useTool("browser_wait", {
  seconds: 5
});
```

### Element Not Found

```typescript
// Wait before interaction
await useTool("browser_wait", { seconds: 2 });

await useTool("browser_click", {
  selector: ".dynamic-content button"
});

// Or use snapshot to verify element exists
const snapshot = await useTool("browser_snapshot");
// Parse to check for expected elements
```

## Development & Testing

### Run in Development Mode

```bash
npm run dev
```

### Type Checking

```bash
npm run typecheck
```

### Run Tests

```bash
npm test
npm run test:watch
```

### Use MCP Inspector

Debug tools interactively:

```bash
npm run build
npm run inspector
```

### Enable Debug Logging

```bash
LOG_LEVEL=debug node dist/index.js
```

## Troubleshooting

### Extension Not Connecting

**Symptom:** Tools fail with "No connection to browser extension"

**Solutions:**
1. Click Browser MCP extension icon in Chrome
2. Click **Connect** on the active tab
3. Verify WebSocket port 9009 is not blocked
4. Check extension is enabled in `chrome://extensions`

### Port Already in Use

**Symptom:** `Error: listen EADDRINUSE: address already in use :::9009`

**Solutions:**
```bash
# Find process using port 9009
lsof -i :9009
kill -9 <PID>

# Or use different port
WS_PORT=9010 node dist/index.js
```

Update MCP config with new port.

### Tools Timeout

**Symptom:** Operations timeout after 30 seconds

**Solutions:**
1. Ensure browser tab is active and visible
2. Check network connection for slow pages
3. Use `browser_wait` before interactions
4. Verify extension is connected (green indicator)

### No Tools Available

**Symptom:** AI client doesn't show browser tools

**Solutions:**
1. Verify absolute path in MCP config
2. Restart AI client after config changes
3. Check `npm run build` completed successfully
4. Confirm Node.js 18+ installed

### Redis Connection Failures

**Symptom:** Redis errors in logs (when enabled)

**Solutions:**
```bash
# Verify Redis running
redis-cli ping
# Expected: PONG

# Or disable Redis
REDIS_ENABLED=false
```

Server continues without persistence if Redis unavailable.

## Architecture Notes

- **Stdio Transport:** Server communicates with AI client via stdin/stdout
- **Stderr Logging:** All diagnostics go to stderr to preserve MCP protocol
- **Single Connection:** Only one extension WebSocket connection active at a time
- **Session Preservation:** Uses existing Chrome profile, maintaining cookies and login state
- **Type Safety:** End-to-end TypeScript with Zod schema validation

## Security Considerations

- Server runs locally on localhost only
- No data sent to remote servers
- Extension requires explicit user connection per tab
- Use environment variables for credentials, never hardcode
- Redis passwords should be set if exposing port

## Use Cases

- **Web Scraping:** Extract data while authenticated
- **Form Automation:** Fill multi-page forms
- **Testing:** Verify UI workflows with real browser
- **Research:** Navigate sites requiring JavaScript
- **Data Collection:** Gather information from protected content
- **Monitoring:** Check site status and capture evidence
