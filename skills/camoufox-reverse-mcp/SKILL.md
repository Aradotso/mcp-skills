```markdown
---
name: camoufox-reverse-mcp
description: Anti-fingerprint browser MCP server for JS reverse engineering with 35 tools including JSVMP analysis, hook tracing, and C++ engine-level property tracking
triggers:
  - analyze JavaScript obfuscation with anti-detection browser
  - reverse engineer JSVMP protected endpoints
  - hook and trace JavaScript functions with Camoufox
  - intercept network requests with fingerprint evasion
  - debug anti-scraping protections
  - trace DOM property access at C++ engine level
  - capture crypto API calls in browser
  - export browser state for reverse engineering
---

# camoufox-reverse-mcp

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

An MCP server that enables AI coding assistants to perform JavaScript reverse engineering using **Camoufox**, an anti-fingerprint browser. Provides 35 tools for API parameter analysis, JS static/dynamic analysis, function hooking, network interception, JSVMP bytecode analysis, and C++ engine-level property tracking.

## What It Does

This MCP server wraps Camoufox (Firefox-based anti-detection browser) with Playwright automation, offering:

- **C++ engine-level fingerprint spoofing** (undetectable by JS)
- **Persistent function hooking** with auto-reinjection after navigation
- **JSVMP analysis** via instrumentation and interpreter proxying
- **Network traffic capture** with request/response modification
- **Property access tracing** at Firefox C++ engine level (requires camoufox-reverse fork)
- Works against RS, AK, JY, CF and other anti-bot systems

## Installation

### Method 1: AI-Assisted (Recommended)

In your AI coding tool (Cursor/Claude Code), paste:

```
Install camoufox-reverse-mcp from https://github.com/WhiteNightShadow/camoufox-reverse-mcp
```

### Method 2: Manual

```bash
git clone https://github.com/WhiteNightShadow/camoufox-reverse-mcp.git
cd camoufox-reverse-mcp
pip install -e .
```

## Configuration

### Cursor (`.cursor/mcp.json`)

```json
{
  "mcpServers": {
    "camoufox-reverse": {
      "command": "python",
      "args": ["-m", "camoufox_reverse_mcp"]
    }
  }
}
```

### Claude Code (with proxy and humanization)

```json
{
  "mcpServers": {
    "camoufox-reverse": {
      "command": "python",
      "args": [
        "-m", "camoufox_reverse_mcp",
        "--proxy", "http://127.0.0.1:7890",
        "--geoip",
        "--humanize"
      ]
    }
  }
}
```

### CLI Options

```bash
python -m camoufox_reverse_mcp \
  --headless \                    # Headless mode
  --proxy http://host:port \      # HTTP/SOCKS5 proxy
  --geoip \                       # Match fingerprint to proxy IP
  --humanize                      # Add human-like mouse/keyboard delays
```

## Core Tool Categories

### Browser Control (9 tools)

```python
# Launch browser with optional C++ property tracing
launch_browser(headless=False, enable_trace=True)

# Navigate with hook pre-injection
navigate(url="https://example.com", wait_until="networkidle", pre_inject_hooks=["xhr", "crypto"])

# Screenshot (full page or element)
take_screenshot(path="/tmp/page.png", full_page=True)

# Wait for elements or URL patterns
wait_for(selector="#result", timeout=5000)
wait_for(url_pattern="**/api/data", timeout=10000)

# Page interaction
click(selector="button.submit")
type_text(selector="input#search", text="query")

# Get page info
get_page_info()  # Returns URL, title, viewport
```

### Script Analysis (3 tools)

```python
# List all loaded scripts
scripts(action="list")
# Returns: [{"url": "https://site.com/app.js", "script_id": "s1", "size": 45231}, ...]

# Get script source code
scripts(action="get", script_url="https://site.com/vmp.js")

# Search across all scripts (auto-detects minified code)
search_code(keyword="sign", script_url=None, max_results=50)
# For minified: returns char-level context
# For formatted: returns line-level context
```

### Hooking & Tracing (5 tools)

```python
# Trace function calls (non-invasive)
hook_function(
    target="window.btoa",
    mode="trace",
    persistent=True  # Auto-reinject after navigation
)

# Intercept and modify function
hook_function(
    target="window.generateSign",
    mode="intercept",
    custom_code="""
    const original = arguments[0];
    console.log('[HOOK] Input:', original);
    const result = ORIGINAL_FUNC(original);
    console.log('[HOOK] Output:', result);
    return result;
    """,
    persistent=True
)

# Inject preset hooks
inject_hook_preset("xhr")           # XMLHttpRequest
inject_hook_preset("fetch")         # Fetch API
inject_hook_preset("crypto")        # crypto.subtle
inject_hook_preset("websocket")     # WebSocket
inject_hook_preset("debugger_bypass")  # Anti-debugger bypass
inject_hook_preset("cookie")        # document.cookie
inject_hook_preset("runtime_probe") # Function.prototype.toString protection

# Remove all hooks
remove_hooks()

# Get console logs (includes hook output)
get_console_logs()
```

### Network Analysis (6 tools)

```python
# Start capturing network traffic
network_capture(action="start")

# List requests with filters
list_network_requests(
    url_filter="*/api/*",
    method="POST",
    resource_type="xhr",
    status_code=200,
    domain="example.com"
)

# Get full request details
get_network_request(request_id="r42", max_body_size=10000)
# Returns: {
#   "url": "...",
#   "method": "POST",
#   "headers": {...},
#   "postData": "...",
#   "response": {
#     "status": 200,
#     "headers": {...},
#     "body": "..."
#   }
# }

# Get call stack of request initiator
get_request_initiator(request_id="r42")
# Returns: [
#   {"url": "app.js", "lineNumber": 234, "functionName": "sendRequest"},
#   ...
# ]

# Intercept requests
intercept_request(action="start", url_pattern="**/api/login")
intercept_request(action="modify", url_pattern="**/api/*", 
                  modify={"headers": {"X-Custom": "value"}})
intercept_request(action="block", url_pattern="**/tracker.js")
intercept_request(action="stop")
```

### JSVMP Reverse Engineering (3 tools)

#### Understanding JSVMP Types

| Type | Examples | ✅ Recommended Path | ❌ Avoid |
|------|----------|---------------------|----------|
| **Signature-based** (environment is signature) | RS 5/6, AK sensor_data | `instrumentation(action="install")` | `pre_inject_hooks`, `hook_jsvmp_interpreter(mode="proxy")` |
| **Behavior-based** (parameter signing) | TK JSVMP, JY gt4 | `hook_jsvmp_interpreter(mode="proxy")` | — |
| **Pure obfuscation** | Generic JS obfuscators | Any combination | — |

```python
# Method 1: Source-level instrumentation (AST rewriting)
# Best for: Signature-based JSVMP (RS, AK)
instrumentation(
    action="install",
    url_pattern="**/vmp_*.js",
    mode="ast"  # or "regex" for faster but less precise
)

# Reload page to apply instrumentation
instrumentation(action="reload")

# View instrumentation logs
instrumentation(action="log", type_filter="tap_get")     # Property reads
instrumentation(action="log", type_filter="tap_method")  # Method calls
instrumentation(action="log", type_filter="tap_set")     # Property writes

# Check status
instrumentation(action="status")

# Stop instrumentation
instrumentation(action="stop")

# Method 2: Runtime interpreter hooking
# Best for: Behavior-based JSVMP (TK, JY)
hook_jsvmp_interpreter(
    mode="proxy",  # Full coverage (may break signature)
    # mode="transparent"  # Signature-safe (less coverage)
)

# Collect browser environment for Node.js补环境
compare_env()
# Returns: {
#   "navigator": {"userAgent": "...", "platform": "..."},
#   "screen": {"width": 1920, "height": 1080},
#   "webgl": {"vendor": "...", "renderer": "..."},
#   ...
# }
```

### C++ Engine-Level Property Tracing (v1.1.0)

**Requires**: [camoufox-reverse](https://github.com/WhiteNightShadow/camoufox-reverse) custom browser fork.

```python
# Launch browser with tracing enabled
launch_browser(enable_trace=True)

# Navigate to JSVMP-protected page
navigate("https://www.douyin.com/video/xxx")

# Trace property access (summary mode)
trace_property_access(
    duration=0,  # 0 = read all events since launch, >0 = start new trace window
    mode="summary",
    collect_values=True  # Fetch actual values from browser
)
# Returns: {
#   "events": [
#     {
#       "property": "navigator.userAgent",
#       "count": 15,
#       "object_type": "Navigator",
#       "value": "Mozilla/5.0...",  # Inline if small
#       "value_file": null
#     },
#     {
#       "property": "canvas.toDataURL",
#       "count": 3,
#       "value_file": "~/.cache/camoufox-reverse/values/canvas_toDataURL_abc123.txt"
#     }
#   ],
#   "metadata": {"total_events": 42, "unique_properties": 28}
# }

# Timeline view (property access rhythm)
trace_property_access(duration=0, mode="timeline", bucket_ms=500)

# Sequence view (access order)
trace_property_access(duration=0, mode="sequence", max_events=100)

# Search specific properties
trace_property_access(duration=0, mode="search", search_query="cookie")

# Filter by object type
trace_property_access(duration=0, filter_object="webgl")

# List trace files (offline analysis)
list_trace_files()

# Query historical trace file
query_trace_file(
    filename="trace_20260801_123456.json",
    filter_object="navigator"
)
```

**Difference from `compare_env`**:
- `trace_property_access`: Tracks **what JSVMP actually reads** (precise, C++ level, undetectable)
- `compare_env`: Collects **all available** environment properties (comprehensive, JS level)

### Cookie & Storage (5 tools)

```python
# Get cookies
cookies(action="get", name="session_id", domain=".example.com")

# Set cookie
cookies(action="set", cookie={
    "name": "auth",
    "value": "token123",
    "domain": ".example.com",
    "path": "/",
    "httpOnly": True
})

# Delete cookie
cookies(action="delete", name="session_id")

# Get storage
get_storage(storage_type="localStorage")
get_storage(storage_type="sessionStorage")

# Export full browser state (cookies + storage)
export_state(path="/tmp/browser_state.json")

# Import state
import_state(path="/tmp/browser_state.json")
```

### Verification & Debugging (4 tools)

```python
# Offline signature verification (character-level diff)
verify_signer_offline(
    signer_code="""
    function sign(params) {
        return myCustomSign(params.url, params.data);
    }
    """,
    samples=[
        {
            "id": "sample1",
            "input": {"url": "https://api.com/search", "data": '{"q":"test"}'},
            "expected": {"signature": "abc123xyz"}
        }
    ]
)
# Returns: {
#   "sample1": {
#     "pass": False,
#     "actual": {"signature": "abc999xyz"},
#     "expected": {"signature": "abc123xyz"},
#     "diff_position": 3,  # First char difference at index 3
#     "diff_context": "abc[1]23xyz vs abc[9]99xyz"
#   }
# }

# Check environment and dependencies
check_environment()
# Returns: {
#   "mcp_version": "1.1.0",
#   "camoufox_installed": True,
#   "camoufox_reverse_fork": True,  # Custom fork detected
#   "playwright_installed": True,
#   "browser_running": True
# }

# Reset browser state (clear hooks, routes, captures)
reset_browser_state()

# Close browser
close_browser()
```

## Common Workflows

### Workflow 1: Reverse Login Endpoint Signature

```python
# 1. Launch and setup hooks
launch_browser()
inject_hook_preset("xhr", persistent=True)
inject_hook_preset("crypto", persistent=True)

# 2. Navigate to login page
navigate("https://example.com/login")

# 3. Trigger login
type_text("#username", "testuser")
type_text("#password", "testpass")
click("#login-btn")

# 4. Find the POST request
requests = list_network_requests(method="POST", url_filter="*/login")
login_request = requests[0]

# 5. Get request details and call stack
details = get_network_request(request_id=login_request["id"])
stack = get_request_initiator(request_id=login_request["id"])

# 6. Search for sign function
search_results = search_code(keyword="sign")

# 7. Hook the sign function
hook_function(
    target="window.generateLoginSign",
    mode="trace",
    persistent=True
)

# 8. Reload and collect traces
reload()
logs = get_console_logs()
```

### Workflow 2: JSVMP Analysis (Signature-Based)

```python
# 1. Launch and start network capture
launch_browser(enable_trace=True)  # Enable C++ tracing if available
network_capture(action="start")

# 2. Navigate to target
navigate("https://target-site.com/")

# 3. Find JSVMP script
scripts_list = list_network_requests(resource_type="script")
vmp_script = [s for s in scripts_list if "vmp" in s["url"].lower()][0]

# 4. Install AST instrumentation
instrumentation(
    action="install",
    url_pattern=f"**/{vmp_script['url'].split('/')[-1]}",
    mode="ast"
)

# 5. Inject persistent hooks
inject_hook_preset("cookie", persistent=True)
inject_hook_preset("runtime_probe", persistent=True)

# 6. Reload to apply instrumentation
instrumentation(action="reload")

# 7. View what JSVMP reads
get_logs = instrumentation(action="log", type_filter="tap_get")
method_logs = instrumentation(action="log", type_filter="tap_method")

# 8. Collect environment for Node.js补环境
env = compare_env()

# 9. If camoufox-reverse fork available, get precise access trace
trace = trace_property_access(duration=0, mode="summary", collect_values=True)
# Shows only properties JSVMP actually accessed (not all available)
```

### Workflow 3: Verify Extracted Signature Function

```python
# 1. Launch browser and collect samples
launch_browser()
navigate("https://target.com")
network_capture(action="start")

# Trigger actions to collect real signed requests
click("#search-btn")
requests = list_network_requests(url_filter="*/api/search")

# Extract samples
samples = []
for req in requests[:5]:
    details = get_network_request(request_id=req["id"])
    samples.append({
        "id": req["id"],
        "input": {
            "url": details["url"],
            "params": details["postData"]
        },
        "expected": {
            "X-Sign": details["response"]["headers"]["X-Sign"]
        }
    })

# 2. Verify your extracted signer code
result = verify_signer_offline(
    signer_code="""
    function sign(input) {
        // Your extracted signing logic
        return {'X-Sign': mySignAlgorithm(input.url, input.params)};
    }
    """,
    samples=samples
)

# 3. Check results
for sample_id, result in result.items():
    if not result["pass"]:
        print(f"Failed at char {result['diff_position']}: {result['diff_context']}")
```

### Workflow 4: Intercept and Mock API Responses

```python
# 1. Start interception
launch_browser()
intercept_request(action="start", url_pattern="**/api/config")

# 2. Mock response
intercept_request(
    action="mock",
    url_pattern="**/api/config",
    mock_response={
        "status": 200,
        "headers": {"Content-Type": "application/json"},
        "body": '{"feature_enabled": true, "debug_mode": true}'
    }
)

# 3. Navigate and verify mocked response is used
navigate("https://target.com")

# 4. Stop interception
intercept_request(action="stop")
```

## Troubleshooting

### Browser won't start

```python
# Check environment
check_environment()

# If camoufox not installed, it will auto-install on first launch
# If proxy issues, try without proxy first
launch_browser(headless=False)  # Visual debugging
```

### Hooks not persisting after navigation

```python
# Ensure persistent=True
inject_hook_preset("xhr", persistent=True)
hook_function("window.sign", mode="trace", persistent=True)

# Hooks auto-reinject on navigate() calls
# For manual page.goto(), use navigate() wrapper instead
```

### JSVMP signature breaks after instrumentation

```python
# Use transparent mode (less invasive)
hook_jsvmp_interpreter(mode="transparent")

# Or use source instrumentation instead
instrumentation(action="install", mode="ast")
```

### Property tracing returns "not supported"

```python
# Requires camoufox-reverse fork
# Check if fork is installed
env = check_environment()
if not env["camoufox_reverse_fork"]:
    print("Install from: https://github.com/WhiteNightShadow/camoufox-reverse/releases")

# Ensure tracing enabled at launch
launch_browser(enable_trace=True)
```

### Network capture missing requests

```python
# Start capture BEFORE navigation
network_capture(action="start")
navigate("https://target.com")

# Check capture status
network_capture(action="status")
```

### Large response bodies truncated

```python
# Increase max_body_size (default 10000 chars)
get_network_request(request_id="r42", max_body_size=100000)
```

## Advanced Tips

### Combining Tracing Methods

```python
# For maximum coverage on signature-based JSVMP:
launch_browser(enable_trace=True)
instrumentation(action="install", url_pattern="**/vmp.js", mode="ast")
inject_hook_preset("runtime_probe", persistent=True)
instrumentation(action="reload")

# After page loads:
ast_logs = instrumentation(action="log")           # What instrumentation saw
cpp_trace = trace_property_access(duration=0)      # What C++ engine saw
env = compare_env()                                 # Full environment snapshot
```

### Debugging Hook Injection

```python
# Get console logs to see hook output
inject_hook_preset("xhr")
navigate("https://target.com")
logs = get_console_logs()

# Look for [HOOK] prefixed messages
for log in logs:
    if "[HOOK]" in log["text"]:
        print(log)
```

### Exporting and Replaying Sessions

```python
# Save browser state after manual login
navigate("https://target.com/login")
# ... manual login in GUI ...
export_state(path="/tmp/logged_in_state.json")

# Later: replay session
launch_browser()
import_state(path="/tmp/logged_in_state.json")
navigate("https://target.com/dashboard")  # Already logged in
```

## Key Differences from chrome-devtools-mcp

| Feature | chrome-devtools-mcp | camoufox-reverse-mcp |
|---------|---------------------|----------------------|
| Browser | Chrome (Puppeteer) | Firefox (Camoufox) |
| Anti-detection | None | C++ engine-level |
| Debugging | Limited | Playwright + JS hooks |
| JSVMP support | No | AST instrumentation + interpreter proxy |
| Hook persistence | No | Yes (auto-reinject) |
| Property tracing | No | C++ engine-level (camoufox-reverse fork) |

## References

- Project: https://github.com/WhiteNightShadow/camoufox-reverse-mcp
- Camoufox: https://camoufox.com/
- Custom fork: https://github.com/WhiteNightShadow/camoufox-reverse
- JSVMP Playbook: [docs/JSVMP_PLAYBOOK.md](https://github.com/WhiteNightShadow/camoufox-reverse-mcp/blob/main/docs/JSVMP_PLAYBOOK.md)
```
