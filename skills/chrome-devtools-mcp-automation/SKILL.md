---
name: chrome-devtools-mcp-automation
description: Expert guidance for Chrome DevTools MCP server - browser automation, debugging, and performance analysis for AI agents
triggers:
  - automate chrome browser with devtools
  - debug web application with chrome devtools
  - analyze page performance with chrome
  - take screenshots and inspect network requests
  - record performance traces in chrome
  - control chrome browser from my agent
  - inspect browser console and errors
  - navigate and interact with web pages using puppeteer
---

# Chrome DevTools MCP Automation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Chrome DevTools MCP (`chrome-devtools-mcp`) is a Model Context Protocol (MCP) server that gives AI coding agents full access to Chrome DevTools. It enables reliable browser automation via Puppeteer, advanced debugging (network inspection, console messages, screenshots), and performance analysis (trace recording, CrUX data).

## Installation

### As MCP Server (Standard Mode)

Add to your MCP client configuration (e.g., Cursor, Claude Code, Cline, VS Code):

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

**With options:**

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest",
        "--headless",
        "--no-usage-statistics",
        "--no-performance-crux"
      ]
    }
  }
}
```

### Slim Mode (Basic Browser Tasks Only)

For simpler automation without performance tools:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--slim", "--headless"]
    }
  }
}
```

### Connect to Existing Browser

To connect to an already-running Chrome instance (useful for tools like Antigravity):

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest",
        "--browser-url=http://127.0.0.1:9222"
      ]
    }
  }
}
```

### CLI Usage (Without MCP)

```bash
# Install globally
npm install -g chrome-devtools-mcp

# Run CLI commands
chrome-devtools-mcp navigate --url https://example.com
chrome-devtools-mcp screenshot --output screenshot.png
chrome-devtools-mcp console-logs
```

## Key Configuration Options

| Flag | Description |
|------|-------------|
| `--headless` | Run Chrome in headless mode (no UI) |
| `--slim` | Enable slim mode (basic tools only) |
| `--no-usage-statistics` | Opt out of Google usage statistics collection |
| `--no-performance-crux` | Disable Chrome UX Report (CrUX) API calls |
| `--browser-url=URL` | Connect to existing Chrome instance at URL |
| `--user-data-dir=PATH` | Use custom Chrome profile directory |

### Environment Variables

```bash
# Disable usage statistics
export CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS=1

# Disable update checks
export CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS=1

# Use in CI environment (auto-disables statistics)
export CI=true
```

## Core Capabilities

### 1. Browser Navigation & Interaction

**Navigate to a URL:**
```typescript
// MCP tool call: navigate
{
  "url": "https://example.com",
  "waitUntil": "networkidle0"
}
```

**Click elements:**
```typescript
// MCP tool call: click
{
  "selector": "button.submit",
  "waitForNavigation": true
}
```

**Fill forms:**
```typescript
// MCP tool call: type
{
  "selector": "input[name='email']",
  "text": "user@example.com",
  "delay": 100
}
```

**Extract page content:**
```typescript
// MCP tool call: evaluate
{
  "script": "document.querySelector('h1').textContent",
  "returnByValue": true
}
```

### 2. Screenshots & Visual Inspection

**Full page screenshot:**
```typescript
// MCP tool call: screenshot
{
  "fullPage": true,
  "path": "./screenshots/page.png"
}
```

**Element screenshot:**
```typescript
// MCP tool call: screenshot
{
  "selector": "div.product-card",
  "path": "./element.png"
}
```

### 3. Network Inspection

**Monitor network requests:**
```typescript
// MCP tool call: network-logs
{
  "filter": {
    "type": "fetch",
    "status": 200
  }
}
```

**Analyze failed requests:**
```typescript
// MCP tool call: network-logs
{
  "filter": {
    "failed": true
  }
}
```

### 4. Console & Debugging

**Get console messages:**
```typescript
// MCP tool call: console-logs
{
  "level": "error"  // or "warning", "info", "log"
}
```

**Execute JavaScript in page context:**
```typescript
// MCP tool call: evaluate
{
  "script": `
    const errors = [];
    window.addEventListener('error', (e) => {
      errors.push({ message: e.message, stack: e.error.stack });
    });
    errors;
  `,
  "returnByValue": true
}
```

### 5. Performance Analysis

**Record performance trace:**
```typescript
// MCP tool call: start-trace
{
  "categories": ["devtools.timeline", "v8.execute", "disabled-by-default-v8.cpu_profiler"]
}

// ... perform actions ...

// MCP tool call: stop-trace
{
  "path": "./traces/performance.json"
}
```

**Get performance metrics:**
```typescript
// MCP tool call: get-metrics
{
  "includeMemory": true
}
```

**Analyze Core Web Vitals:**
```typescript
// MCP tool call: web-vitals
{
  "url": "https://example.com",
  "includeCrux": true  // Include real-user data from CrUX
}
```

## Common Patterns

### Pattern 1: E2E Test Automation

```typescript
// 1. Navigate to login page
// Tool: navigate
{ "url": "https://app.example.com/login" }

// 2. Fill credentials
// Tool: type
{ "selector": "#username", "text": "testuser" }
{ "selector": "#password", "text": "secure_password" }

// 3. Submit form
// Tool: click
{ "selector": "button[type='submit']", "waitForNavigation": true }

// 4. Verify success
// Tool: evaluate
{ "script": "window.location.pathname === '/dashboard'" }

// 5. Capture proof
// Tool: screenshot
{ "path": "./test-evidence/login-success.png" }
```

### Pattern 2: Performance Audit

```typescript
// 1. Start trace recording
// Tool: start-trace
{ "categories": ["devtools.timeline", "loading", "blink.user_timing"] }

// 2. Navigate to page
// Tool: navigate
{ "url": "https://example.com/product/123", "waitUntil": "networkidle0" }

// 3. Stop trace
// Tool: stop-trace
{ "path": "./audits/product-page-trace.json" }

// 4. Get Core Web Vitals
// Tool: web-vitals
{ "url": "https://example.com/product/123", "includeCrux": true }

// 5. Analyze metrics
// Tool: get-metrics
{ "includeMemory": true }
```

### Pattern 3: Debugging Production Issues

```typescript
// 1. Navigate to problematic page
// Tool: navigate
{ "url": "https://example.com/checkout" }

// 2. Monitor network for failed requests
// Tool: network-logs
{ "filter": { "failed": true } }

// 3. Capture console errors
// Tool: console-logs
{ "level": "error" }

// 4. Get JavaScript errors with stack traces
// Tool: evaluate
{
  "script": `
    window.__errors = [];
    window.addEventListener('error', (e) => {
      __errors.push({
        message: e.message,
        filename: e.filename,
        lineno: e.lineno,
        colno: e.colno,
        stack: e.error?.stack
      });
    });
    setTimeout(() => __errors, 5000);
  `,
  "returnByValue": true
}

// 5. Screenshot for visual context
// Tool: screenshot
{ "fullPage": true, "path": "./debug/error-state.png" }
```

### Pattern 4: Data Extraction (Web Scraping)

```typescript
// 1. Navigate to target page
// Tool: navigate
{ "url": "https://example.com/products" }

// 2. Wait for dynamic content
// Tool: wait-for-selector
{ "selector": "div.product-list", "timeout": 5000 }

// 3. Extract structured data
// Tool: evaluate
{
  "script": `
    Array.from(document.querySelectorAll('.product-card')).map(card => ({
      title: card.querySelector('h3').textContent.trim(),
      price: card.querySelector('.price').textContent.trim(),
      url: card.querySelector('a').href,
      inStock: !card.querySelector('.out-of-stock')
    }))
  `,
  "returnByValue": true
}

// 4. Handle pagination
// Tool: click
{ "selector": "button.next-page", "waitForNavigation": true }
```

### Pattern 5: Visual Regression Testing

```typescript
// 1. Set consistent viewport
// Tool: set-viewport
{ "width": 1280, "height": 720, "deviceScaleFactor": 1 }

// 2. Navigate to page
// Tool: navigate
{ "url": "https://example.com/landing" }

// 3. Wait for animations to complete
// Tool: wait-for-timeout
{ "timeout": 2000 }

// 4. Take baseline screenshot
// Tool: screenshot
{
  "fullPage": true,
  "path": "./visual-tests/baseline.png"
}

// 5. Compare against previous baseline (in your test framework)
// Tool: screenshot
{
  "fullPage": true,
  "path": "./visual-tests/current.png"
}
```

## Troubleshooting

### Browser Won't Start

**Issue:** MCP server fails with "Browser not found"

**Solution:**
```bash
# Specify Chrome executable path
export CHROME_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"

# Or use Chrome for Testing
npx @puppeteer/browsers install chrome@stable
```

### Port Already in Use

**Issue:** "Port 9222 already in use"

**Solution:**
```bash
# Find and kill existing Chrome instance
lsof -ti:9222 | xargs kill -9

# Or use custom port
# In MCP config, add: "--remote-debugging-port=9223"
```

### Timeout Errors

**Issue:** "Navigation timeout of 30000 ms exceeded"

**Solution:**
```typescript
// Increase timeout for slow pages
// Tool: navigate
{
  "url": "https://slow-site.com",
  "timeout": 60000,
  "waitUntil": "domcontentloaded"  // Less strict than "networkidle0"
}
```

### Headless Mode Issues

**Issue:** Site behaves differently in headless mode

**Solution:**
```json
// Remove --headless flag temporarily to debug
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

### Memory Leaks in Long-Running Sessions

**Issue:** Chrome consumes excessive memory over time

**Solution:**
```typescript
// Close and reopen browser periodically
// Tool: close-browser
{}

// Browser will auto-restart on next tool call
```

### Source Map Errors

**Issue:** Stack traces not showing original source

**Solution:**
```bash
# Ensure source maps are deployed with your app
# Check Network tab for .map file requests
# Tool: network-logs
{ "filter": { "type": "other" } }
```

### Corporate Proxy/Firewall

**Issue:** Cannot download Chrome or connect to CrUX API

**Solution:**
```bash
# Set proxy environment variables
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080

# Disable CrUX if API calls fail
# Add flag: --no-performance-crux
```

## Requirements

- **Node.js:** v20.19 or newer (latest maintenance LTS)
- **Chrome:** Current stable version or newer
- **npm:** Any recent version
- **OS:** macOS, Linux, or Windows 11+

## Privacy & Data Collection

- **Usage Statistics:** Enabled by default. Opt out with `--no-usage-statistics` flag or `CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS=1` env var.
- **CrUX API:** Performance tools may send trace URLs to Google CrUX API. Disable with `--no-performance-crux`.
- **Browser Data:** All browser content is exposed to MCP clients. Do not use with sensitive/personal data you don't want to share.

## Best Practices

1. **Use Slim Mode for Simple Tasks:** If you only need navigation and screenshots, use `--slim` to reduce startup time.
2. **Headless by Default in CI:** Always use `--headless` in automated environments.
3. **Set Timeouts Appropriately:** Adjust navigation timeouts based on expected page load times.
4. **Clean Up Resources:** For long-running scripts, periodically close the browser to free memory.
5. **Version Pin in Production:** Use `chrome-devtools-mcp@X.Y.Z` instead of `@latest` for reproducible builds.
6. **Monitor Network:** Check `network-logs` before assuming page load issues are browser-related.
7. **Leverage Source Maps:** Deploy source maps in staging/test environments for better debugging.

## Additional Resources

- **Tool Reference:** [Standard](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md) | [Slim](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/slim-tool-reference.md)
- **CLI Documentation:** [CLI Guide](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/cli.md)
- **Troubleshooting:** [Full Guide](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md)
- **Puppeteer API:** [Official Docs](https://pptr.dev/)
- **Chrome DevTools Protocol:** [Protocol Reference](https://chromedevtools.github.io/devtools-protocol/)
