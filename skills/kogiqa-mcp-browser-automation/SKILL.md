---
name: kogiqa-mcp-browser-automation
description: MCP server for browser automation using natural language through kogiQA, enabling AI agents to interact with web pages without selectors
triggers:
  - automate a web browser
  - interact with a webpage using kogiqa
  - test a web application with browser automation
  - navigate and click elements on a page
  - fill out web forms automatically
  - scrape content from a website
  - debug web applications with browser control
  - automate web testing tasks
---

# kogiQA MCP Browser Automation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

kogiQA MCP is a Model Context Protocol server that provides browser automation capabilities using the kogiQA browser control algorithm. Unlike traditional automation tools, it enables AI agents to interact with web pages through natural language commands without requiring CSS selectors or XPath expressions. This approach saves tokens and simplifies web automation workflows.

**Key advantages:**
- No selector required — interact with elements using natural language descriptions
- Built on Playwright for robust browser automation
- Works with any MCP-compatible client (Claude Desktop, Cursor, VS Code, etc.)
- Lightweight and easy to install

## Installation

### Quick Install

The fastest way to install is using npx:

```bash
npx kogiqa-mcp@latest
```

### Claude Code

```bash
claude mcp add kogiqa-browser npx kogiqa-mcp@latest
```

### VS Code / VS Code Insiders

Install via CLI:

```bash
code --add-mcp '{"name":"kogiqa-browser","command":"npx","args":["kogiqa-mcp@latest"]}'
```

Or add manually to your MCP settings (`settings.json`):

```json
{
  "mcp.servers": {
    "kogiqa-browser": {
      "command": "npx",
      "args": ["kogiqa-mcp@latest"]
    }
  }
}
```

### Cursor

Add to Cursor Settings → MCP → Add new MCP Server:

```json
{
  "kogiqa-browser": {
    "command": "npx",
    "args": ["kogiqa-mcp@latest"]
  }
}
```

### Standard MCP Configuration

For Claude Desktop, Windsurf, Goose, or other MCP clients, add to your MCP configuration file:

```json
{
  "mcpServers": {
    "kogiqa-browser": {
      "command": "npx",
      "args": ["kogiqa-mcp@latest"]
    }
  }
}
```

Configuration file locations:
- **Claude Desktop (macOS):** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Claude Desktop (Windows):** `%APPDATA%\Claude\claude_desktop_config.json`
- **Cursor:** `.cursor/config.json` in your project or user settings
- **VS Code:** User or workspace settings

## Available Tools

Once installed, the MCP server provides several tools for browser automation:

### `kogiqa_navigate`
Navigate to a URL.

**Parameters:**
- `url` (string, required): The URL to navigate to

**Example usage:**
```javascript
// AI agent will call this tool as:
kogiqa_navigate({ url: "https://example.com" })
```

### `kogiqa_click`
Click on an element using natural language description.

**Parameters:**
- `description` (string, required): Natural language description of the element to click

**Example usage:**
```javascript
// No selectors needed - describe the element naturally
kogiqa_click({ description: "the blue login button" })
kogiqa_click({ description: "submit button at the bottom" })
kogiqa_click({ description: "first link in the navigation menu" })
```

### `kogiqa_type`
Type text into an input field.

**Parameters:**
- `description` (string, required): Natural language description of the input field
- `text` (string, required): Text to type

**Example usage:**
```javascript
kogiqa_type({ 
  description: "email input field", 
  text: "user@example.com" 
})
kogiqa_type({ 
  description: "search box at the top", 
  text: "kogiQA MCP" 
})
```

### `kogiqa_get_content`
Extract content from the current page.

**Returns:** HTML or text content of the page

**Example usage:**
```javascript
kogiqa_get_content()
```

### `kogiqa_screenshot`
Take a screenshot of the current page.

**Returns:** Base64-encoded PNG image

**Example usage:**
```javascript
kogiqa_screenshot()
```

### `kogiqa_close`
Close the browser instance.

**Example usage:**
```javascript
kogiqa_close()
```

## Common Patterns

### Login Flow Automation

```javascript
// Navigate to login page
await kogiqa_navigate({ url: "https://app.example.com/login" });

// Fill in credentials
await kogiqa_type({ 
  description: "username field", 
  text: process.env.USERNAME 
});
await kogiqa_type({ 
  description: "password field", 
  text: process.env.PASSWORD 
});

// Submit form
await kogiqa_click({ description: "login button" });

// Verify success
const content = await kogiqa_get_content();
```

### Form Submission

```javascript
// Navigate to form
await kogiqa_navigate({ url: "https://example.com/contact" });

// Fill multiple fields
await kogiqa_type({ description: "name input", text: "John Doe" });
await kogiqa_type({ description: "email input", text: "john@example.com" });
await kogiqa_type({ description: "message textarea", text: "Hello!" });

// Submit
await kogiqa_click({ description: "send button" });
```

### Web Scraping

```javascript
// Navigate to target page
await kogiqa_navigate({ url: "https://example.com/products" });

// Extract content
const content = await kogiqa_get_content();

// Take screenshot for verification
const screenshot = await kogiqa_screenshot();

// Parse content (this would be done by the AI agent)
// Extract product names, prices, etc.
```

### Multi-Step Navigation

```javascript
// Start at homepage
await kogiqa_navigate({ url: "https://example.com" });

// Navigate through site
await kogiqa_click({ description: "products menu item" });
await kogiqa_click({ description: "first product card" });
await kogiqa_click({ description: "add to cart button" });
await kogiqa_click({ description: "checkout button" });

// Capture final state
const screenshot = await kogiqa_screenshot();
```

### Testing with Verification

```javascript
// Navigate to page under test
await kogiqa_navigate({ url: "https://app.example.com/dashboard" });

// Take before screenshot
const before = await kogiqa_screenshot();

// Perform action
await kogiqa_click({ description: "settings button" });

// Verify change
const content = await kogiqa_get_content();
const after = await kogiqa_screenshot();

// Close when done
await kogiqa_close();
```

## Configuration Options

### Headless Mode

By default, kogiQA MCP runs in headless mode. To run with a visible browser (useful for debugging), set environment variables:

```json
{
  "mcpServers": {
    "kogiqa-browser": {
      "command": "npx",
      "args": ["kogiqa-mcp@latest"],
      "env": {
        "HEADLESS": "false"
      }
    }
  }
}
```

### Browser Type

Specify browser (chromium, firefox, webkit):

```json
{
  "mcpServers": {
    "kogiqa-browser": {
      "command": "npx",
      "args": ["kogiqa-mcp@latest"],
      "env": {
        "BROWSER_TYPE": "firefox"
      }
    }
  }
}
```

## Troubleshooting

### Server Not Starting

**Issue:** MCP server fails to start or connect

**Solutions:**
- Ensure Node.js 20+ is installed: `node --version`
- Clear npm cache: `npm cache clean --force`
- Try installing globally: `npm install -g kogiqa-mcp@latest`
- Check MCP configuration syntax in your client's config file

### Browser Launch Failures

**Issue:** Browser fails to launch or crashes

**Solutions:**
- Install Playwright browsers: `npx playwright install`
- Check for system dependencies: `npx playwright install-deps`
- Try different browser: Set `BROWSER_TYPE=firefox` or `webkit`
- Disable headless mode for debugging: `HEADLESS=false`

### Element Not Found

**Issue:** kogiqa cannot find described element

**Solutions:**
- Be more specific in element descriptions: "blue submit button at bottom right"
- Include context: "search input in the header"
- Wait for page to load: Get content first to verify page state
- Take screenshot to see current page state

### Timeouts

**Issue:** Operations timeout waiting for elements

**Solutions:**
- Ensure page has fully loaded before interacting
- Use more specific element descriptions
- Check if element is hidden or covered by other elements
- Verify the element actually exists using `kogiqa_get_content`

### Permission Issues

**Issue:** Browser automation blocked by website

**Solutions:**
- Some sites block automation; this is expected behavior
- Check site's terms of service for automation policies
- Consider adding delays between actions
- Use stealth mode if available in future versions

## Best Practices

1. **Descriptive element references:** Be specific but natural ("red delete button in the sidebar" not just "button")
2. **Verify page state:** Use `kogiqa_get_content` to confirm page loaded before interacting
3. **Handle secrets properly:** Use environment variables, never hardcode credentials
4. **Close browsers:** Call `kogiqa_close` when done to free resources
5. **Progressive navigation:** Navigate step-by-step rather than assuming page state
6. **Take screenshots:** Use for debugging and verification of automation flows
7. **Natural language:** Write descriptions as you would explain to a human

## Resources

- **npm package:** https://www.npmjs.com/package/kogiqa-mcp
- **GitHub repository:** atagon-GmbH/kogiqa-mcp
- **License:** MIT
