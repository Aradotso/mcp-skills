---
name: js-reverse-mcp-debugging
description: JavaScript reverse engineering and browser debugging MCP server with anti-detection and agent-first tooling
triggers:
  - debug javascript on this website
  - set a breakpoint on the encryption function
  - analyze websocket messages
  - find where this request is coming from
  - search for crypto functions in the page scripts
  - inspect the call stack at this breakpoint
  - list all loaded scripts on this page
  - capture network request initiators
---

# js-reverse-mcp-debugging

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**js-reverse-mcp** is an MCP server that gives AI agents full JavaScript debugging capabilities: breakpoints, call stacks, scope inspection, network analysis, and WebSocket message capture. Built on Patchright (CDP protocol anti-detection) with optional CloakBrowser (49 C++ fingerprint patches) for strong anti-bot sites.

**Key features:**
- **Headful debugging** — visible browser, breakpoints, step-through, call stacks
- **Persistent sessions** — cookies/localStorage survive restarts
- **Dual anti-detection** — Patchright (protocol layer) + optional CloakBrowser (binary patches)
- **21 MCP tools** — script analysis, breakpoint control, network inspection, WebSocket analysis
- **Zero JS injection** — no `Object.defineProperty` hacks that leak automation signals

## Installation

### NPX (Recommended)

Add to your MCP client configuration:

```json
{
  "mcpServers": {
    "js-reverse": {
      "command": "npx",
      "args": ["js-reverse-mcp"]
    }
  }
}
```

**Claude Code:**
```bash
claude mcp add js-reverse npx js-reverse-mcp
```

**Codex:**
```bash
codex mcp add js-reverse -- npx js-reverse-mcp
```

### Local Install

```bash
git clone https://github.com/zhizhuodemao/js-reverse-mcp.git
cd js-reverse-mcp
npm install
npm run build
```

Then configure with local path:

```json
{
  "mcpServers": {
    "js-reverse": {
      "command": "node",
      "args": ["/path/to/js-reverse-mcp/build/src/index.js"]
    }
  }
}
```

## Configuration Options

**CLI flags (all optional):**

- `--cloak` — Use CloakBrowser binary with 49 C++ fingerprint patches (auto-downloads ~200MB on first run)
- `--isolated` — Use temporary profile (no persistent cookies/localStorage)
- `--browserUrl, -u` — Connect to existing Chrome instance (CDP endpoint, e.g. `http://127.0.0.1:9222`)
- `--logFile` — Write debug logs to file (use with `DEBUG=*` env var)

### Common Configurations

**Default (System Chrome + Persistent Login):**
```json
{
  "mcpServers": {
    "js-reverse": {
      "command": "npx",
      "args": ["js-reverse-mcp"]
    }
  }
}
```

**Anti-Bot Sites (Cloudflare, DataDome, FingerprintJS):**

Pre-download CloakBrowser binary first (one-time, ~30-60s):
```bash
npx cloakbrowser install
```

Then configure:
```json
{
  "mcpServers": {
    "js-reverse-cloak": {
      "command": "npx",
      "args": ["js-reverse-mcp", "--cloak"]
    }
  }
}
```

**Dual Setup (Switch Based on Target):**
```json
{
  "mcpServers": {
    "js-reverse": {
      "command": "npx",
      "args": ["js-reverse-mcp"]
    },
    "js-reverse-cloak": {
      "command": "npx",
      "args": ["js-reverse-mcp", "--cloak"]
    }
  }
}
```

**Connect to Running Chrome:**

1. Launch Chrome with debugging:
   ```bash
   # macOS
   /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --user-data-dir=/tmp/chrome-debug
   
   # Windows
   "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="%TEMP%\chrome-debug"
   ```

2. Configure MCP:
   ```json
   {
     "mcpServers": {
       "js-reverse": {
         "command": "npx",
         "args": ["js-reverse-mcp", "--browserUrl", "http://127.0.0.1:9222"]
       }
     }
   }
   ```

## MCP Tools (21)

### Page & Navigation

| Tool | Purpose |
|------|---------|
| `select_page` | List open pages or switch debugging context by index |
| `new_page` | Create new page and navigate to URL |
| `navigate_page` | Navigate, back, forward, or refresh |
| `select_frame` | List/select iframe execution context |
| `take_screenshot` | Capture page screenshot |

### Script Analysis

| Tool | Purpose |
|------|---------|
| `list_scripts` | List all loaded JavaScript files |
| `get_script_source` | Fetch script source (supports line ranges, character offsets) |
| `save_script_source` | Save full script to local file (large/minified/WASM) |
| `search_in_sources` | Search all scripts for string/regex |

### Breakpoints & Execution

| Tool | Purpose |
|------|---------|
| `set_breakpoint_on_text` | Set breakpoint by searching code text (works in minified code) |
| `break_on_xhr` | Set XHR/Fetch breakpoint by URL pattern |
| `remove_breakpoint` | Remove by ID, URL, or all; auto-resumes execution |
| `list_breakpoints` | List all active breakpoints |
| `get_paused_info` | Get pause state, call stack, scope variables |
| `pause_or_resume` | Toggle pause/resume |
| `step` | Step over/into/out, returns location + source context |

### Network & WebSocket

| Tool | Purpose |
|------|---------|
| `list_network_requests` | List requests or get single request details by reqid |
| `get_request_initiator` | Get JavaScript call stack for network request |
| `get_websocket_messages` | List connections, analyze message patterns, get message details |

### Inspection

| Tool | Purpose |
|------|---------|
| `evaluate_script` | Execute JavaScript (supports breakpoint context, main world, save results/binary to file) |
| `list_console_messages` | List console messages or get single message by msgid |

## Common Workflows

### 1. Basic Reverse Engineering

```typescript
// User: "Debug the encryption on example.com"

// Step 1: Open target page
await use_mcp_tool("js-reverse", "new_page", {
  url: "https://example.com"
});

// Step 2: Find encryption functions
const searchResults = await use_mcp_tool("js-reverse", "search_in_sources", {
  query: "encrypt|crypto|AES|cipher",
  isRegex: true
});

// Step 3: Set breakpoint on encryption function
await use_mcp_tool("js-reverse", "set_breakpoint_on_text", {
  searchText: "function encrypt(data)",
  scriptUrl: searchResults[0].url
});

// Step 4: Trigger action (user does this in browser or via evaluate_script)
// ... breakpoint hits ...

// Step 5: Inspect paused state
const pausedInfo = await use_mcp_tool("js-reverse", "get_paused_info", {});
// Returns: call stack, scope variables, current location

// Step 6: Evaluate in breakpoint context
const params = await use_mcp_tool("js-reverse", "evaluate_script", {
  expression: "data",
  returnByValue: true
});

// Step 7: Step through execution
await use_mcp_tool("js-reverse", "step", {
  action: "into"  // or "over", "out"
});
```

### 2. Network Request Analysis

```typescript
// User: "Find where this API request is initiated"

// Step 1: List network requests
const requests = await use_mcp_tool("js-reverse", "list_network_requests", {});

// Step 2: Find target request
const apiRequest = requests.find(r => r.url.includes("/api/user"));

// Step 3: Get JavaScript call stack
const initiator = await use_mcp_tool("js-reverse", "get_request_initiator", {
  reqid: apiRequest.reqid
});
// Returns: full JS stack trace from initiation point

// Step 4: Set breakpoint at initiation
await use_mcp_tool("js-reverse", "set_breakpoint_on_text", {
  searchText: initiator.callFrames[0].functionName,
  scriptUrl: initiator.callFrames[0].url
});
```

### 3. XHR/Fetch Interception

```typescript
// User: "Break on all requests to /api/encrypt"

// Set XHR breakpoint with URL pattern
await use_mcp_tool("js-reverse", "break_on_xhr", {
  urlPattern: "*/api/encrypt*"
});

// Execution will pause before matching XHR/fetch
// Then inspect request body, headers, call stack
const pausedInfo = await use_mcp_tool("js-reverse", "get_paused_info", {});

// Evaluate request payload
const payload = await use_mcp_tool("js-reverse", "evaluate_script", {
  expression: "arguments[0]",  // xhr.send() argument
  returnByValue: true
});
```

### 4. WebSocket Protocol Analysis

```typescript
// User: "Analyze the WebSocket messages for this trading site"

// Step 1: List WebSocket connections
const wsData = await use_mcp_tool("js-reverse", "get_websocket_messages", {
  action: "list"
});

// Step 2: Analyze message patterns
const analysis = await use_mcp_tool("js-reverse", "get_websocket_messages", {
  action: "analyze",
  wsid: wsData.connections[0].wsid
});
// Returns: message type distribution, size stats, timing patterns

// Step 3: Inspect specific message
const message = await use_mcp_tool("js-reverse", "get_websocket_messages", {
  action: "get",
  wsid: wsData.connections[0].wsid,
  msgid: "msg_123"
});
// Returns: full payload, timestamp, direction (sent/received)
```

### 5. Minified Code Debugging

```typescript
// User: "Set breakpoint on the obfuscated validation function"

// Step 1: Search for function signature in minified code
const results = await use_mcp_tool("js-reverse", "search_in_sources", {
  query: "validate.*password",
  isRegex: true
});

// Step 2: Get context around match
const source = await use_mcp_tool("js-reverse", "get_script_source", {
  scriptId: results[0].scriptId,
  startOffset: results[0].match.offset - 200,
  endOffset: results[0].match.offset + 200
});

// Step 3: Set breakpoint by text match (works in minified code)
await use_mcp_tool("js-reverse", "set_breakpoint_on_text", {
  searchText: results[0].match.line.trim(),
  scriptUrl: results[0].url,
  condition: "password.length > 0"  // optional conditional breakpoint
});
```

### 6. Scope Variable Inspection

```typescript
// After hitting breakpoint, inspect all accessible variables

const pausedInfo = await use_mcp_tool("js-reverse", "get_paused_info", {});

// pausedInfo.scopeChain contains:
// - local variables
// - closure variables
// - global scope

// Evaluate complex expressions in current scope
const result = await use_mcp_tool("js-reverse", "evaluate_script", {
  expression: "Object.keys(this).filter(k => k.startsWith('_'))",
  callFrameId: pausedInfo.callFrames[0].callFrameId,
  returnByValue: true
});
```

### 7. Save Large Script Sources

```typescript
// User: "Save this 5MB minified bundle for analysis"

const scripts = await use_mcp_tool("js-reverse", "list_scripts", {});
const targetScript = scripts.find(s => s.url.includes("bundle.min.js"));

// Save to local file (more reliable than get_script_source for large files)
await use_mcp_tool("js-reverse", "save_script_source", {
  scriptId: targetScript.scriptId,
  outputPath: "/tmp/bundle.min.js"
});

// Now analyze with external tools or search within it
const matches = await use_mcp_tool("js-reverse", "search_in_sources", {
  query: "apiKey.*=.*['\"]([^'\"]+)['\"]",
  isRegex: true,
  scriptId: targetScript.scriptId
});
```

### 8. Multi-Page Debugging

```typescript
// User: "Debug login flow across multiple redirects"

// Step 1: List all open pages
const pages = await use_mcp_tool("js-reverse", "select_page", {});

// Step 2: Switch to login page
await use_mcp_tool("js-reverse", "select_page", {
  index: 0
});

// Step 3: Set breakpoint on form submit
await use_mcp_tool("js-reverse", "set_breakpoint_on_text", {
  searchText: "submitLogin"
});

// Step 4: After redirect, switch to new page
const updatedPages = await use_mcp_tool("js-reverse", "select_page", {});
await use_mcp_tool("js-reverse", "select_page", {
  index: updatedPages.length - 1
});

// Step 5: Continue debugging in new context
const scripts = await use_mcp_tool("js-reverse", "list_scripts", {});
```

## Troubleshooting

### Bot Detection / Access Denied

**Symptoms:** Site returns 403, Cloudflare challenge loops, Zhihu 40362 error

**Solution 1:** Try isolated profile first (rules out state pollution)
```json
"args": ["js-reverse-mcp", "--isolated"]
```

**Solution 2:** Enable CloakBrowser (49 fingerprint patches)

Pre-download binary:
```bash
npx cloakbrowser install
```

Configure:
```json
"args": ["js-reverse-mcp", "--cloak"]
```

**Solution 3:** Clear persistent profile (loses login state)
```bash
rm -rf ~/.cache/chrome-devtools-mcp/chrome-profile
# or for cloak mode:
rm -rf ~/.cache/chrome-devtools-mcp/cloak-profile
```

### Breakpoint Not Hitting

1. **Check if script is loaded:**
   ```typescript
   const scripts = await use_mcp_tool("js-reverse", "list_scripts", {});
   // Verify target script is in list
   ```

2. **Use text-based breakpoint** (works in minified code):
   ```typescript
   await use_mcp_tool("js-reverse", "set_breakpoint_on_text", {
     searchText: "unique_code_snippet",
     scriptUrl: "target.js"
   });
   ```

3. **List active breakpoints:**
   ```typescript
   const breakpoints = await use_mcp_tool("js-reverse", "list_breakpoints", {});
   ```

4. **Check if execution is paused elsewhere:**
   ```typescript
   const pausedInfo = await use_mcp_tool("js-reverse", "get_paused_info", {});
   ```

### Cannot Find Script

**If dynamic/lazy-loaded scripts don't appear:**

1. Navigate to trigger script load:
   ```typescript
   await use_mcp_tool("js-reverse", "navigate_page", {
     action: "goto",
     url: "https://example.com/trigger-page"
   });
   ```

2. Wait for script load, then list:
   ```typescript
   // Give page time to load
   await new Promise(resolve => setTimeout(resolve, 2000));
   const scripts = await use_mcp_tool("js-reverse", "list_scripts", {});
   ```

3. Search across all sources:
   ```typescript
   const results = await use_mcp_tool("js-reverse", "search_in_sources", {
     query: "function_name"
   });
   ```

### WebSocket Messages Not Captured

**If `get_websocket_messages` returns empty:**

1. Ensure page is loaded and WebSocket is connected:
   ```typescript
   const wsData = await use_mcp_tool("js-reverse", "get_websocket_messages", {
     action: "list"
   });
   // Check if connections array is populated
   ```

2. Trigger WebSocket traffic in browser (message capture is passive)

3. List connections includes lifecycle events; messages are captured automatically once connection is established

### Evaluate Script Fails

**Common issues:**

1. **Execution context invalid** — Make sure you're evaluating in the right frame:
   ```typescript
   // List frames first
   const frames = await use_mcp_tool("js-reverse", "select_frame", {});
   
   // Select target frame
   await use_mcp_tool("js-reverse", "select_frame", {
     index: 1  // switch to iframe
   });
   
   // Now evaluate
   await use_mcp_tool("js-reverse", "evaluate_script", {
     expression: "window.secretVar"
   });
   ```

2. **Paused in wrong call frame** — Specify `callFrameId`:
   ```typescript
   const pausedInfo = await use_mcp_tool("js-reverse", "get_paused_info", {});
   
   await use_mcp_tool("js-reverse", "evaluate_script", {
     expression: "localVar",
     callFrameId: pausedInfo.callFrames[0].callFrameId
   });
   ```

3. **Reference error** — Variable not in scope; check `scopeChain`:
   ```typescript
   const pausedInfo = await use_mcp_tool("js-reverse", "get_paused_info", {});
   // Inspect pausedInfo.scopeChain to see available variables
   ```

## Anti-Detection Layer Details

**Protocol Layer (Always Active):**
- Patchright removes `Runtime.enable`, `Console.enable` CDP calls
- Script evaluation in isolated world
- Automation launch flags stripped

**Binary Layer (With `--cloak`):**
- 49 C++ patches: `navigator.webdriver`, canvas, WebGL, audio, GPU, fonts, screen, WebRTC, TLS
- Custom Chromium build (no Google services, no Web Store)
- Persistent fingerprint identity per profile

**Navigation Layer (Always Active):**
- Silent CDP navigation (no early `Network.enable`/`Debugger.enable`)
- Google referer on `new_page`
- Real viewport (no fake 1280×720)

**When to use `--cloak`:** Only when protocol-layer evasion fails. See project docs for details.

## Best Practices

1. **Start simple:** Use default mode; only add `--cloak` if blocked
2. **Persistent sessions:** Default profile saves login state (use `--isolated` for clean state)
3. **Text-based breakpoints:** More reliable than line numbers in minified/dynamic code
4. **Save large sources:** Use `save_script_source` instead of `get_script_source` for big files
5. **Scope inspection:** Always check `get_paused_info().scopeChain` before evaluating variables
6. **Network initiators:** Use `get_request_initiator` to find where requests originate
7. **WebSocket analysis:** Use `analyze` action first to understand message patterns before diving into individual messages

## Security Warning

This tool exposes browser content to MCP clients with full inspection/modification capabilities. Do not use on pages with sensitive information (passwords, payment details, private data).
