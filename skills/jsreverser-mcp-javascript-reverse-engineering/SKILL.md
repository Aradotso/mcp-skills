---
name: jsreverser-mcp-javascript-reverse-engineering
description: MCP server for JavaScript reverse engineering in real browser environments with hooks, breakpoints, network tracing, deobfuscation, and environment reconstruction.
triggers:
  - reverse engineer this JavaScript
  - analyze encrypted request parameters
  - hook JavaScript functions in browser
  - deobfuscate minified code
  - extract API signature logic
  - trace network request initiator
  - debug obfuscated frontend code
  - recreate JavaScript runtime environment
---

# JSReverser-MCP JavaScript Reverse Engineering

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

JSReverser-MCP is a specialized MCP server for JavaScript reverse engineering that operates in real browser environments. It helps locate frontend core logic by integrating script retrieval, breakpoint debugging, function hooking, network request tracing, call chain analysis, deobfuscation, and risk assessment into unified MCP tools. Perfect for API analysis, security research, frontend debugging, and understanding encrypted/signed request parameters.

## Core Methodology

The project follows these principles:

1. **Observe-first**: Confirm requests, scripts, functions in browser before intervention
2. **Hook-preferred**: Use minimal hooks for runtime sampling before breakpoints
3. **Breakpoint-last**: Only pause execution when hooks are insufficient
4. **Rebuild-oriented**: Export evidence and reconstruct in Node.js environment
5. **Evidence-first**: Record all findings as task artifacts, not just in conversation
6. **Pure-extraction-after-pass**: Extract pure algorithm only after environment passes

## Installation

### 1. Install and Build

```bash
git clone https://github.com/NoOne-hub/JSReverser-MCP.git
cd JSReverser-MCP
npm install
npm run build
```

The build output is at `build/src/index.js`.

### 2. Quick Start

```bash
npm run start
```

### 3. Configure MCP Client

#### Claude Code

```bash
claude mcp add js-reverse node /ABSOLUTE/PATH/JSReverser-MCP/build/src/index.js
```

#### Cursor Settings

```json
{
  "mcpServers": {
    "js-reverse": {
      "command": "node",
      "args": ["/ABSOLUTE/PATH/JSReverser-MCP/build/src/index.js"]
    }
  }
}
```

#### Codex (config.toml)

```toml
[mcp_servers.js-reverse]
command = "node"
args = ["/ABSOLUTE/PATH/JSReverser-MCP/build/src/index.js"]
```

### 4. Connect to Existing Browser (Optional)

Launch Chrome with remote debugging:

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222

# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
```

Then configure with `--browserUrl`:

```toml
[mcp_servers.js-reverse]
command = "node"
args = [
  "/ABSOLUTE/PATH/JSReverser-MCP/build/src/index.js",
  "--browserUrl=http://localhost:9222"
]
```

## External AI Configuration

JSReverser-MCP can integrate external LLMs for enhanced analysis:

```toml
[mcp_servers.js-reverse]
command = "node"
args = ["/ABSOLUTE/PATH/JSReverser-MCP/build/src/index.js"]

[mcp_servers.js-reverse.env]
DEFAULT_LLM_PROVIDER = "anthropic"
ANTHROPIC_API_KEY = "your_anthropic_key_here"
ANTHROPIC_MODEL = "claude-3-5-sonnet-20241022"

# Alternative: OpenAI
# DEFAULT_LLM_PROVIDER = "openai"
# OPENAI_API_KEY = "your_openai_key_here"
# OPENAI_MODEL = "gpt-4o"

# Alternative: Gemini
# DEFAULT_LLM_PROVIDER = "gemini"
# GEMINI_API_KEY = "your_gemini_key_here"
# GEMINI_MODEL = "gemini-2.0-flash-exp"
```

**AI-dependent features:**
- `understand_code` (required)
- `detect_crypto` (optional with `useAI=true`)
- `deobfuscate_code` (enhanced quality)
- `analyze_target` (optional with `useAI=true`)

## Key Tool Categories

### 1. Page Observation & Script Location

Identify which scripts exist and where target code lives:

```typescript
// List all loaded scripts
list_scripts({ includeSourceMaps: false })

// Get specific script source
get_script_source({ 
  scriptId: "123",
  url: "https://example.com/app.js" 
})

// Find text in single script
find_in_script({
  scriptId: "123",
  searchText: "encrypt",
  maxResults: 10
})

// Search across all scripts
search_in_scripts({
  searchText: "signature",
  maxResults: 20
})
```

### 2. Runtime Hooks & Sampling

Observe runtime behavior with minimal intrusion:

```typescript
// Hook a global function
hook_function({
  functionPath: "window.crypto.subtle.encrypt",
  options: {
    captureArgs: true,
    captureReturn: true,
    captureStack: true
  }
})

// Create reusable hook definition
create_hook({
  hookId: "crypto-monitor",
  code: `
    const original = crypto.subtle.encrypt;
    crypto.subtle.encrypt = function(...args) {
      console.log('[HOOK] encrypt called:', args);
      return original.apply(this, args);
    };
  `
})

// Inject hook into page
inject_hook({ hookId: "crypto-monitor" })

// Retrieve hook data
get_hook_data({ hookId: "crypto-monitor" })

// Trace function calls
trace_function({
  functionName: "generateSignature",
  captureStack: true
})
```

### 3. Breakpoints & Debug Control

Pause execution when hooks aren't enough:

```typescript
// Set breakpoint by line number
set_breakpoint({
  scriptUrl: "https://example.com/sign.js",
  lineNumber: 42
})

// Set breakpoint by code text
set_breakpoint_on_text({
  scriptUrl: "https://example.com/sign.js",
  searchText: "return signature",
  lineOffset: 0
})

// Control execution
resume({ mode: "continue" })
pause()
step_over()
step_into()
step_out()
```

### 4. Network Analysis & Request Tracing

Locate target requests and their initiators:

```typescript
// List all network requests
list_network_requests({
  filterUrl: "api.example.com",
  filterMethod: "POST"
})

// Get request details
get_network_request({
  requestId: "12345.67"
})

// Find who triggered a request
get_request_initiator({
  requestId: "12345.67"
})

// Break when XHR/Fetch fires
break_on_xhr({
  urlPattern: "*sign*"
})
```

### 5. Code Analysis & Deobfuscation

Understand and clean obfuscated code:

```typescript
// Collect page code
collect_code({
  priority: ["inline", "webpack", "vendor"],
  maxSize: 500000
})

// Understand code structure (requires AI)
understand_code({
  source: "function _0x1234(){...}",
  context: "encryption signature generation"
})

// Deobfuscate code
deobfuscate_code({
  source: "var _0x1a2b=['push','length'];...",
  options: {
    renameVariables: true,
    removeDeadCode: true,
    simplifyExpressions: true
  }
})

// Detect crypto usage
detect_crypto({
  source: "...",
  useAI: true
})

// Comprehensive risk analysis
risk_panel({
  useAI: true,
  includeNetworkRisk: true
})
```

### 6. WebSocket Monitoring

Track WebSocket connections and message patterns:

```typescript
// List active WebSocket connections
list_websocket_connections()

// Analyze message patterns
analyze_websocket_messages({
  wsId: "ws-123",
  groupBy: "opcode"
})

// Get messages
get_websocket_messages({
  wsId: "ws-123",
  group: "binary-frames",
  limit: 50
})
```

### 7. Local Rebuild & Environment Reconstruction

Export evidence and recreate runtime in Node.js:

```typescript
// Export rebuild bundle
export_rebuild_bundle({
  taskId: "jd-h5st-20240517",
  includeScripts: true,
  includeNetwork: true,
  includeStorage: true
})

// Compare environment requirements
diff_env_requirements({
  currentEnv: { navigator: true, document: false },
  requiredEnv: { navigator: true, document: true, XMLHttpRequest: true }
})

// Record evidence
record_reverse_evidence({
  taskId: "jd-h5st-20240517",
  evidence: {
    type: "hook-capture",
    functionName: "h5st",
    args: [...],
    returnValue: "..."
  }
})
```

### 8. Session & Login State Management

Save and restore browser sessions:

```typescript
// Save current session
save_session_state({
  snapshotId: "logged-in-state"
})

// Restore session
restore_session_state({
  snapshotId: "logged-in-state"
})

// Export to file
dump_session_state({
  snapshotId: "logged-in-state",
  outputPath: "artifacts/tasks/my-task/session.json"
})

// Load from file
load_session_state({
  source: "artifacts/tasks/my-task/session.json"
})
```

### 9. Page Automation

Minimal automation to trigger target behavior:

```typescript
// Navigate
navigate_page({ url: "https://example.com/login" })

// Query DOM
query_dom({ selector: "button.submit" })

// Click element
click_element({ selector: "button.submit" })

// Type text
type_text({ 
  selector: "input[name='username']",
  text: "testuser"
})

// Take screenshot
take_screenshot({
  outputPath: "artifacts/tasks/my-task/screenshot.png"
})
```

## Standard Workflow Example

### Scenario: Reverse Engineer API Signature Parameter

```typescript
// Step 1: Check browser connection
check_browser_health()

// Step 2: Navigate to target page
navigate_page({ url: "https://example.com/api-page" })

// Step 3: List network requests
const requests = list_network_requests({
  filterUrl: "*api*",
  filterMethod: "POST"
})

// Step 4: Find target request
const targetRequest = get_network_request({ 
  requestId: requests[0].id 
})
// Notice: request has "sign" parameter

// Step 5: Search for "sign" in scripts
const searchResults = search_in_scripts({
  searchText: "sign:",
  maxResults: 10
})

// Step 6: Hook the sign function
hook_function({
  functionPath: "window.generateSign",
  options: {
    captureArgs: true,
    captureReturn: true,
    captureStack: true
  }
})

// Step 7: Trigger request again (click button, etc.)
click_element({ selector: "button.load-data" })

// Step 8: Get hook data
const hookData = get_hook_data({ 
  hookId: "auto-hook-generateSign" 
})
// Review arguments and return value

// Step 9: Get script source for deeper analysis
const scriptSource = get_script_source({
  url: hookData.stack[0].url
})

// Step 10: Deobfuscate if needed
const cleaned = deobfuscate_code({
  source: scriptSource,
  options: { renameVariables: true, removeDeadCode: true }
})

// Step 11: Export rebuild bundle
export_rebuild_bundle({
  taskId: "example-sign-20240517",
  includeScripts: true,
  includeNetwork: true,
  includeStorage: true
})

// Step 12: Record evidence
record_reverse_evidence({
  taskId: "example-sign-20240517",
  evidence: {
    type: "function-hook",
    functionName: "generateSign",
    sampleInput: hookData.calls[0].args,
    sampleOutput: hookData.calls[0].return
  }
})
```

## Task Artifact Structure

Tasks are stored in `artifacts/tasks/<task-id>/`:

```
artifacts/tasks/my-reverse-task/
├── task.json                    # Task metadata
├── runtime-evidence.jsonl       # Hook/trace data
├── network.jsonl                # Network requests
├── scripts.jsonl                # Script sources
├── env/
│   ├── env.js                   # Base environment shims
│   ├── polyfills.js             # Proxy diagnostics, watch
│   ├── entry.js                 # Entry point for local rebuild
│   └── capture.json             # Runtime captures
├── run/                         # Test runs
└── report.md                    # Analysis report
```

**Git Safety**: Only `artifacts/tasks/_TEMPLATE/` is committed. Real task directories stay local.

## Pre-Indexed Parameter Cases

Existing reverse-engineered parameter cases (abstracted, no sensitive data):

- **JD h5st**: `scripts/cases/jd-h5st-pure-node.mjs`
- **Kuaishou falcon**: `scripts/cases/ks-hxfalcon-pure-node.mjs`
- **Douyin a-bogus**: `scripts/cases/douyin-a-bogus-pure-node.mjs`

See `scripts/cases/README.md` for methodology and templates.

## Common Patterns

### Pattern 1: Find Encryption Function

```typescript
// Search for crypto keywords
search_in_scripts({ searchText: "CryptoJS.AES" })

// Hook the encryption call
hook_function({
  functionPath: "CryptoJS.AES.encrypt",
  options: { captureArgs: true, captureReturn: true }
})

// Trigger encryption, review hook data
get_hook_data({ hookId: "auto-hook-CryptoJS.AES.encrypt" })
```

### Pattern 2: Trace Request Parameter Generation

```typescript
// Set breakpoint when request fires
break_on_xhr({ urlPattern: "*api.example.com*" })

// When paused, check call stack
// Identify the function that generated parameters

// Resume and hook that function
resume()
hook_function({ functionPath: "window.buildParams" })

// Trigger again, get hook data
get_hook_data({ hookId: "auto-hook-buildParams" })
```

### Pattern 3: Deobfuscate and Understand

```typescript
// Get obfuscated script
const script = get_script_source({ scriptId: "123" })

// Deobfuscate
const cleaned = deobfuscate_code({
  source: script.source,
  options: {
    renameVariables: true,
    removeDeadCode: true,
    simplifyExpressions: true
  }
})

// Understand with AI (requires provider configured)
understand_code({
  source: cleaned.code,
  context: "API signature generation"
})
```

### Pattern 4: Export and Recreate Locally

```typescript
// After gathering evidence
export_rebuild_bundle({
  taskId: "my-api-20240517",
  includeScripts: true,
  includeNetwork: true,
  includeStorage: true
})

// Check artifacts/tasks/my-api-20240517/env/entry.js
// Run locally:
// node artifacts/tasks/my-api-20240517/env/entry.js

// If errors occur, diff environment
diff_env_requirements({
  currentEnv: {}, // Populated from Node.js runtime check
  requiredEnv: {} // Populated from error messages
})

// Patch env/env.js and env/polyfills.js iteratively
```

## Troubleshooting

### Browser Connection Issues

**Problem**: `check_browser_health` fails

**Solution**:
1. Ensure Chrome is launched with `--remote-debugging-port=9222`
2. Check `--browserUrl` matches (default `http://localhost:9222`)
3. Verify no firewall blocking localhost:9222

### Hook Not Capturing Data

**Problem**: `get_hook_data` returns empty

**Solution**:
1. Confirm hook was injected: check `inject_hook` response
2. Ensure target function was actually called (trigger page action)
3. Check console for hook errors: `list_console_messages()`
4. Verify function path is correct (check `window.functionName` exists)

### Deobfuscation Produces Invalid Code

**Problem**: Deobfuscated code doesn't run

**Solution**:
1. Start with minimal options: `{ renameVariables: false }`
2. Incrementally enable transformations
3. Use `understand_code` to identify why code fails
4. Some VM-based obfuscation requires manual extraction

### Environment Reconstruction Fails

**Problem**: Local rebuild throws errors

**Solution**:
1. Review `runtime-evidence.jsonl` for missing globals
2. Use `diff_env_requirements` to identify gaps
3. Add shims to `env/env.js` one at a time
4. Use `watch` and diagnostics in `env/polyfills.js`
5. Aim for "first divergence" — find exact point where behavior differs

### AI Provider Not Working

**Problem**: `understand_code` fails with "provider not configured"

**Solution**:
1. Set `DEFAULT_LLM_PROVIDER` in MCP server env config
2. Set corresponding API key (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, etc.)
3. Verify key is valid and has API access
4. Check `ANTHROPIC_BASE_URL` if using proxy

### Session State Not Restoring

**Problem**: `restore_session_state` doesn't preserve login

**Solution**:
1. Ensure cookies domain matches current page
2. Save session while still on target domain
3. Check if site uses `httpOnly` cookies (not accessible to JS)
4. Some sites require additional localStorage/sessionStorage

## Reference Documentation

- Full tool reference: `docs/reference/tool-reference.md`
- Workflow guide: `docs/reference/reverse-workflow.md`
- Case safety policy: `docs/reference/case-safety-policy.md`
- Environment patching: `docs/reference/env-patching.md`
- Reverse artifacts: `docs/reference/reverse-artifacts.md`
- Browser connection: `docs/guides/browser-connection.md`
- Client configuration: `docs/guides/client-configuration.md`

## Security & Ethics

- **Never commit sensitive task directories** (real credentials, keys, tokens)
- **Respect target site's terms of service**
- **Use only for authorized testing, research, or debugging your own applications**
- **Abstracted cases in `scripts/cases/` must be sanitized and educational**

This tool is for legitimate reverse engineering, security research, and debugging — not for bypassing security controls on third-party services without permission.
