---
name: playwright-mcp-server
description: Browser automation MCP server using Playwright's accessibility tree for LLM-friendly web interaction
triggers:
  - automate a web browser
  - interact with a web page
  - use playwright mcp
  - navigate to a website
  - click on an element
  - fill out a form
  - take a screenshot with playwright
  - get page accessibility tree
---

# Playwright MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Playwright MCP server provides browser automation capabilities through the Model Context Protocol. It enables LLMs to interact with web pages using Playwright's accessibility tree instead of screenshots, making it fast, lightweight, and deterministic.

## What It Does

- **Structured interaction**: Uses accessibility snapshots instead of vision models
- **Browser automation**: Navigate, click, fill forms, extract data from web pages
- **Multi-browser support**: Chromium, Firefox, and WebKit
- **LLM-optimized**: Returns structured data that's easy for language models to parse

## Installation

Add to your MCP client configuration:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

### Configuration Options

Configure via `args` in your MCP config:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--allowed-hosts", "example.com,*.trusted-domain.com",
        "--browser", "chromium",
        "--headless", "false"
      ]
    }
  }
}
```

Common options:
- `--allowed-hosts <hosts...>`: Comma-separated hosts (or `*` to disable check). Env: `PLAYWRIGHT_MCP_ALLOWED_HOSTS`
- `--browser <browser>`: Choose `chromium`, `firefox`, or `webkit`
- `--headless <true|false>`: Run browser in headless mode
- `--timeout <ms>`: Default timeout for operations

## Available Tools

The MCP server exposes these tools to LLM agents:

### 1. `playwright_navigate`

Navigate to a URL and get accessibility snapshot.

**Parameters:**
- `url` (string, required): URL to navigate to

**Example usage:**
```typescript
// Agent will call this tool
{
  "url": "https://example.com"
}
```

**Returns:** Accessibility tree snapshot of the page

### 2. `playwright_click`

Click an element identified by its accessibility role and name.

**Parameters:**
- `selector` (string, required): Element selector (role, text, or CSS)
- `button` (string, optional): `left`, `right`, or `middle` (default: `left`)

**Example usage:**
```typescript
{
  "selector": "button[name='Submit']",
  "button": "left"
}
```

### 3. `playwright_fill`

Fill an input field with text.

**Parameters:**
- `selector` (string, required): Input field selector
- `value` (string, required): Text to fill

**Example usage:**
```typescript
{
  "selector": "input[name='email']",
  "value": "user@example.com"
}
```

### 4. `playwright_screenshot`

Take a screenshot of the page or element.

**Parameters:**
- `selector` (string, optional): Element to screenshot (defaults to full page)
- `path` (string, optional): File path to save screenshot

**Example usage:**
```typescript
{
  "selector": "div.main-content",
  "path": "./screenshots/content.png"
}
```

### 5. `playwright_evaluate`

Execute JavaScript in the page context.

**Parameters:**
- `expression` (string, required): JavaScript to execute

**Example usage:**
```typescript
{
  "expression": "document.title"
}
```

### 6. `playwright_snapshot`

Get current accessibility snapshot without navigation.

**Example usage:**
```typescript
{}
```

## Common Patterns

### Form Automation

```typescript
// Navigate to login page
await playwright_navigate({ url: "https://app.example.com/login" });

// Fill credentials
await playwright_fill({
  selector: "input[name='username']",
  value: "user@example.com"
});

await playwright_fill({
  selector: "input[type='password']",
  value: process.env.USER_PASSWORD  // Use env vars for secrets
});

// Submit form
await playwright_click({
  selector: "button[type='submit']"
});

// Verify login
const snapshot = await playwright_snapshot({});
// Parse snapshot to confirm successful login
```

### Data Extraction

```typescript
// Navigate to data page
await playwright_navigate({ url: "https://example.com/products" });

// Extract product information
const products = await playwright_evaluate({
  expression: `
    Array.from(document.querySelectorAll('.product')).map(p => ({
      name: p.querySelector('.name')?.textContent,
      price: p.querySelector('.price')?.textContent
    }))
  `
});
```

### Multi-Step Workflow

```typescript
// Search flow
await playwright_navigate({ url: "https://example.com" });

await playwright_fill({
  selector: "input[placeholder='Search']",
  value: "playwright automation"
});

await playwright_click({
  selector: "button[aria-label='Search']"
});

// Wait for results by getting snapshot
const results = await playwright_snapshot({});

// Click first result
await playwright_click({
  selector: "a.result-item:first-child"
});

// Capture final page
await playwright_screenshot({
  path: "./evidence/result-page.png"
});
```

### Testing Accessibility

```typescript
// Navigate to page under test
await playwright_navigate({ url: "https://myapp.com/dashboard" });

// Get accessibility tree
const snapshot = await playwright_snapshot({});

// The snapshot will show:
// - Missing ARIA labels
// - Elements without proper roles
// - Navigation structure
// Parse snapshot to validate accessibility
```

## Accessibility Selectors

Playwright MCP uses accessible selectors. Prefer:

```typescript
// Good: Role-based selectors
"button[name='Submit']"
"link[name='Documentation']"
"textbox[name='Email']"

// Good: ARIA attributes
"[aria-label='Close dialog']"
"[role='navigation']"

// Okay: Text content
"text=Click here"

// Last resort: CSS selectors
"div.modal > button.close"
```

## Working with Snapshots

Accessibility snapshots are structured representations of the page:

```typescript
// Example snapshot structure
{
  "role": "WebArea",
  "name": "Example Page",
  "children": [
    {
      "role": "button",
      "name": "Submit",
      "focusable": true
    },
    {
      "role": "textbox",
      "name": "Email",
      "value": "user@example.com"
    }
  ]
}
```

Use snapshots to:
- Understand page structure
- Identify interactive elements
- Validate content presence
- Find navigation paths

## Troubleshooting

### Host Not Allowed

**Error:** `Host not allowed: example.com`

**Solution:** Add to allowed hosts:
```json
{
  "args": [
    "@playwright/mcp@latest",
    "--allowed-hosts", "example.com,*.example.com"
  ]
}
```

Or disable checks (development only):
```json
{
  "args": ["@playwright/mcp@latest", "--allowed-hosts", "*"]
}
```

### Element Not Found

**Error:** `Element not found: button[name='Submit']`

**Solutions:**
1. Get current snapshot to see available elements:
   ```typescript
   const snapshot = await playwright_snapshot({});
   ```

2. Use more flexible selectors:
   ```typescript
   // Instead of exact match
   "button[name*='submit']"  // Contains 'submit'
   "text=/submit/i"           // Case-insensitive regex
   ```

3. Wait for element by evaluating:
   ```typescript
   await playwright_evaluate({
     expression: `
       new Promise(resolve => {
         const check = () => {
           if (document.querySelector('button[name="Submit"]')) {
             resolve(true);
           } else {
             setTimeout(check, 100);
           }
         };
         check();
       })
     `
   });
   ```

### Timeout Issues

**Error:** `Timeout waiting for element`

**Solution:** Increase timeout:
```json
{
  "args": ["@playwright/mcp@latest", "--timeout", "60000"]
}
```

Or wait explicitly:
```typescript
await playwright_evaluate({
  expression: "new Promise(r => setTimeout(r, 2000))"
});
```

### Browser Not Launching

**Error:** `Failed to launch browser`

**Solutions:**
1. Check browser installation:
   ```bash
   npx playwright install chromium
   ```

2. Try different browser:
   ```json
   {
     "args": ["@playwright/mcp@latest", "--browser", "firefox"]
   }
   ```

3. Run in headed mode for debugging:
   ```json
   {
     "args": ["@playwright/mcp@latest", "--headless", "false"]
   }
   ```

## Best Practices

1. **Use environment variables for secrets:**
   ```typescript
   // Never hardcode credentials
   await playwright_fill({
     selector: "input[type='password']",
     value: process.env.PASSWORD
   });
   ```

2. **Take snapshots for debugging:**
   ```typescript
   // Before and after critical actions
   const beforeSnapshot = await playwright_snapshot({});
   await playwright_click({ selector: "button[name='Delete']" });
   const afterSnapshot = await playwright_snapshot({});
   ```

3. **Prefer accessibility selectors:**
   ```typescript
   // Robust to UI changes
   "button[name='Save']"
   // vs fragile CSS
   "div.container > div:nth-child(2) > button.primary"
   ```

4. **Handle navigation timing:**
   ```typescript
   await playwright_navigate({ url: "https://example.com" });
   // Snapshot after navigate includes wait for load
   const snapshot = await playwright_snapshot({});
   ```

5. **Combine tools for complex workflows:**
   ```typescript
   // Navigate → Inspect → Act → Verify
   await playwright_navigate({ url: "..." });
   const structure = await playwright_snapshot({});
   await playwright_fill({ ... });
   await playwright_click({ ... });
   const result = await playwright_snapshot({});
   ```

## vs Playwright CLI

Use **Playwright MCP** when:
- Building chat-based automation agents
- Need persistent browser state across tool calls
- Iterative reasoning over page structure
- Long-running autonomous workflows

Use **[Playwright CLI + SKILLS](https://github.com/microsoft/playwright-cli)** when:
- Working with coding agents that favor CLI tools
- Token efficiency is critical
- Managing large codebases alongside automation
- Need concise, purpose-built commands
