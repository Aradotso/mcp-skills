---
name: drissionpage-mcp-browser-automation
description: Professional browser automation for Claude Code, Codex, and MCP clients powered by DrissionPage MCP Server
triggers:
  - "automate a web browser"
  - "scrape this website"
  - "fill out a form on this page"
  - "take a screenshot of this website"
  - "extract data from this webpage"
  - "navigate to a URL and click"
  - "test this web application"
  - "interact with browser elements"
---

# DrissionPage MCP Browser Automation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

DrissionPage MCP Server brings professional browser automation to Claude Code, Codex, Cursor, and other MCP clients. It provides 52 tools for deterministic web automation through the Model Context Protocol, leveraging DrissionPage's efficient engine for structured, LLM-optimized browser control.

## Installation

```bash
# Install from PyPI
python -m pip install -U drissionpage-mcp

# Verify installation
drissionpage-mcp --version
drissionpage-mcp doctor
```

## Configuration

### Codex CLI/IDE (Recommended)

Add to `~/.codex/config.toml` or `.codex/config.toml` in your project:

```toml
[mcp_servers.drissionpage]
command = "drissionpage-mcp"
startup_timeout_sec = 20
tool_timeout_sec = 60

# Optional environment variables
[mcp_servers.drissionpage.env]
# CHROME_PATH = "/custom/path/to/chrome"
# DP_HEADLESS = "1"
# DP_MCP_SCREENSHOT_ROOT = "/path/to/screenshots"
# DP_MCP_UPLOAD_ROOT = "/path/to/uploads"
```

Or via CLI:

```bash
codex mcp add drissionpage -- drissionpage-mcp
```

### Claude Desktop / Cursor

Add to MCP settings JSON (e.g., `~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "drissionpage": {
      "command": "drissionpage-mcp"
    }
  }
}
```

For GUI-launched clients that don't inherit shell PATH:

```json
{
  "mcpServers": {
    "drissionpage": {
      "command": "/absolute/path/to/python",
      "args": ["-m", "drissionpage_mcp.cli"]
    }
  }
}
```

### Environment Variables

- `CHROME_PATH` - Custom Chrome/Chromium executable path
- `DP_HEADLESS` - Set to `"1"` for headless mode
- `DP_MCP_SCREENSHOT_ROOT` - Directory for saved screenshots
- `DP_MCP_UPLOAD_ROOT` - Directory for file uploads (security boundary)
- `LOG_LEVEL` - Logging level (DEBUG, INFO, WARNING, ERROR)

## Core Tool Categories

### Navigation Tools

**page_navigate** - Navigate to URLs with optional tab control:

```python
# Basic navigation
page_navigate(url="https://example.com")

# Open in new tab
page_navigate(url="https://github.com", new_tab=True)

# Navigate and get page summary
page_navigate(url="https://news.ycombinator.com", observe=True)
```

**page_go_back / page_go_forward** - Browser history navigation:

```python
page_go_back()
page_go_forward()
```

**page_refresh** - Reload current page:

```python
page_refresh()
```

### Element Discovery & Interaction

**element_find** - Find single elements (CSS or XPath):

```python
# CSS selector (default)
element_find(selector="button.submit")

# XPath
element_find(selector="//button[@type='submit']")

# Get element details
result = element_find(selector="input[name='username']")
# Returns: tag, text, attributes, recommended_selector
```

**element_find_all** - Extract multiple elements:

```python
# Find all article links
element_find_all(
    selector="article h2 a",
    limit=10
)

# Returns bounded list with text, href, and recommended selectors
```

**element_click** - Click elements:

```python
# Click by CSS selector
element_click(selector="button#submit")

# Click with scroll into view
element_click(selector=".modal-close", scroll_into_view=True)
```

**element_type** - Input text:

```python
# Type into input field
element_type(selector="input[name='email']", text="user@example.com")

# Clear and type
element_type(selector="#search", text="Python tutorials", clear=True)
```

**element_upload_file** - Upload files:

```python
# Upload file (must be under DP_MCP_UPLOAD_ROOT)
element_upload_file(
    selector="input[type='file']",
    file_path="document.pdf"
)
```

**element_select** - Handle dropdowns:

```python
# Select by visible text
element_select(selector="select[name='country']", value="United States")

# Select by index
element_select(selector="select#size", index=2)
```

**element_check** - Toggle checkboxes/radios:

```python
# Check a checkbox
element_check(selector="input#agree-terms", check=True)

# Uncheck
element_check(selector="input[name='newsletter']", check=False)
```

### Data Extraction

**element_get_text** - Extract text content:

```python
# Get element text
element_get_text(selector="article .content")

# Get all page text
element_get_text()
```

**element_get_attribute** - Get HTML attributes:

```python
# Get href attribute
element_get_attribute(selector="a.download", attribute="href")

# Get data attribute
element_get_attribute(selector="div.widget", attribute="data-id")
```

**element_get_property** - Get DOM properties:

```python
# Get input value
element_get_property(selector="input#username", property="value")

# Check if checkbox is checked
element_get_property(selector="input[type='checkbox']", property="checked")
```

**element_get_html** - Get HTML content:

```python
# Get element HTML
element_get_html(selector="div.article")

# Get entire page HTML
element_get_html()
```

### Page Understanding

**page_snapshot** - Get structured page overview:

```python
page_snapshot()
# Returns: headings, links, buttons, inputs, forms with recommended selectors
```

**page_observe** - Get page fingerprint:

```python
page_observe()
# Returns: URL, title, element counts, visible text samples, active element, console summary
```

**form_inspect** - Analyze forms:

```python
form_inspect(selector="form#checkout")
# Returns: action, method, controls with labels, types, requirements, options
```

### Screenshots & Visual

**page_screenshot** - Capture inline screenshot:

```python
# Full page screenshot (base64)
page_screenshot(full_page=True)

# Viewport only
page_screenshot(full_page=False)
```

**page_screenshot_save** - Save screenshot to disk:

```python
# Save with auto-generated filename
page_screenshot_save(full_page=True)

# Custom filename (saved under DP_MCP_SCREENSHOT_ROOT)
page_screenshot_save(filename="homepage.png", full_page=True)
```

### Tab Management

**tab_list** - List open tabs:

```python
tab_list()
# Returns: list of tabs with MCP tab IDs, titles, URLs
```

**tab_switch** - Switch to tab:

```python
# Switch using tab_id from tab_list
tab_switch(tab_id="tab_0")
```

**tab_close** - Close tab:

```python
# Close specific tab
tab_close(tab_id="tab_1")
```

### Frame & Shadow DOM

**frame_list** - List iframes:

```python
frame_list()
# Returns: iframe details without changing frame context
```

**frame_snapshot** - Inspect iframe content:

```python
frame_snapshot(selector="iframe#payment-form")
# Returns: bounded outline of iframe content
```

**frame_find** - Find element in iframe:

```python
frame_find(
    frame_selector="iframe#widget",
    selector="button.submit"
)
```

**shadow_find** - Find in shadow DOM:

```python
shadow_find(
    host_selector="custom-element",
    selector=".internal-button"
)
```

**shadow_find_all** - Extract from shadow DOM:

```python
shadow_find_all(
    host_selector="product-list",
    selector=".item",
    limit=10
)
```

### Wait Operations

**wait_for_element** - Wait for element to appear:

```python
wait_for_element(selector="div.results", timeout=10)
```

**wait_for_url** - Wait for URL change:

```python
wait_for_url(text="confirmation", timeout=15)
```

**wait_until** - Wait for conditions:

```python
# Wait until element is clickable
wait_until(
    selector="button.submit",
    condition="clickable",
    timeout=10
)

# Available conditions: clickable, hidden, visible, stable, text_match, url_contains
```

**wait_time** - Simple delay:

```python
wait_time(seconds=2)
```

### Cookies & Storage

**browser_cookies_get** - Read cookies:

```python
# Get all cookies (values redacted by default)
browser_cookies_get()

# Get specific cookie with value
browser_cookies_get(name="session_id", show_value=True)
```

**storage_get** - Read localStorage/sessionStorage:

```python
# Get specific key
storage_get(storage_type="local", key="user_preferences")

# Get all storage as map
storage_get(storage_type="session")
```

**storage_set** - Set storage value:

```python
storage_set(
    storage_type="local",
    key="theme",
    value="dark"
)
```

**storage_clear** - Clear storage:

```python
# Clear specific key
storage_clear(storage_type="local", key="cache")

# Clear all localStorage
storage_clear(storage_type="local")
```

### JavaScript Execution

**page_evaluate** - Run JavaScript:

```python
# Execute JS and get result
page_evaluate(script="return document.title")

# Complex JS with bounded result
page_evaluate(script="""
return Array.from(document.querySelectorAll('a'))
  .slice(0, 10)
  .map(a => ({ text: a.textContent, href: a.href }))
""")
```

### Page Actions

**page_scroll** - Scroll page:

```python
# Scroll down
page_scroll(direction="down", amount=500)

# Scroll to specific position
page_scroll(y=1000)
```

**keyboard_press** - Send keyboard input:

```python
# Press Enter
keyboard_press(key="Enter")

# Press Escape
keyboard_press(key="Escape")
```

**page_click_xy** - Click by coordinates:

```python
page_click_xy(x=100, y=200)
```

**page_resize** - Resize browser window:

```python
page_resize(width=1920, height=1080)
```

### Debug & Diagnostics

**page_console_logs** - Read browser console:

```python
# Get recent console messages
page_console_logs()

# Filter by level
page_console_logs(level="error", limit=20)

# Paginate with cursor
page_console_logs(cursor="msg_50", limit=10)
```

**page_get_url** - Get current URL:

```python
page_get_url()
```

**page_close** - Close browser:

```python
page_close()
```

## Common Patterns

### Web Scraping Pattern

```python
# 1. Navigate to page
page_navigate(url="https://news.ycombinator.com")

# 2. Get page structure
snapshot = page_snapshot()

# 3. Extract data
stories = element_find_all(
    selector=".storylink",
    limit=30
)

# 4. Get details for each item
for story in stories["elements"]:
    title = element_get_text(selector=story["recommended_selector"])
    link = element_get_attribute(
        selector=story["recommended_selector"],
        attribute="href"
    )
```

### Form Automation Pattern

```python
# 1. Inspect form structure
form_info = form_inspect(selector="form#login")

# 2. Fill required fields
element_type(selector="input[name='username']", text="user@example.com")
element_type(selector="input[name='password']", text="$SECURE_PASSWORD")

# 3. Handle optional fields
element_check(selector="input#remember-me", check=True)

# 4. Submit
element_click(selector="button[type='submit']")

# 5. Wait for redirect
wait_for_url(text="dashboard", timeout=10)
```

### Multi-Tab Pattern

```python
# Open multiple pages
page_navigate(url="https://site1.com")
page_navigate(url="https://site2.com", new_tab=True)
page_navigate(url="https://site3.com", new_tab=True)

# List all tabs
tabs = tab_list()

# Switch and extract data from each
for tab in tabs["tabs"]:
    tab_switch(tab_id=tab["tab_id"])
    data = element_get_text(selector=".main-content")
    # Process data
```

### Dynamic Content Pattern

```python
# Navigate and wait for dynamic content
page_navigate(url="https://spa-app.com/products")

# Wait for elements to load
wait_for_element(selector=".product-card", timeout=10)

# Wait for content to stabilize
wait_until(selector=".price", condition="stable", timeout=5)

# Extract data
products = element_find_all(selector=".product-card", limit=20)
```

### Error Recovery Pattern

```python
# Try primary selector
try:
    element_click(selector="button.primary-submit")
except:
    # Fallback to alternate selector
    element_click(selector="input[type='submit']")

# Wait and retry pattern
for attempt in range(3):
    try:
        wait_for_element(selector=".success-message", timeout=5)
        break
    except:
        if attempt < 2:
            wait_time(seconds=2)
            page_refresh()
```

### Screenshot Debugging Pattern

```python
# Navigate to page
page_navigate(url="https://example.com/form")

# Take before screenshot
page_screenshot_save(filename="before.png")

# Perform actions
element_type(selector="input#name", text="Test User")
element_click(selector="button.submit")

# Wait for result
wait_time(seconds=2)

# Take after screenshot
page_screenshot_save(filename="after.png")

# Check console for errors
logs = page_console_logs(level="error")
```

## Troubleshooting

### Browser Not Starting

```bash
# Run diagnostics
drissionpage-mcp doctor

# Check Chrome path
which google-chrome-stable

# Set custom Chrome path
export CHROME_PATH="/usr/bin/chromium"
```

### Element Not Found

```python
# Use page_snapshot to see available elements
snapshot = page_snapshot()

# Try multiple selector strategies
# CSS
element_find(selector="button.submit")

# XPath
element_find(selector="//button[@type='submit']")

# Wait for element to appear
wait_for_element(selector="button.submit", timeout=10)
```

### Form Submission Issues

```python
# Inspect form first
form_info = form_inspect(selector="form")

# Check required fields
# Ensure all required inputs are filled

# Wait for element to be clickable
wait_until(selector="button[type='submit']", condition="clickable")

# Alternative: use Enter key
element_type(selector="input.last-field", text="value")
keyboard_press(key="Enter")
```

### Screenshot Path Issues

```bash
# Set screenshot directory
export DP_MCP_SCREENSHOT_ROOT="$HOME/screenshots"

# Create directory
mkdir -p "$HOME/screenshots"

# Use relative paths in page_screenshot_save
page_screenshot_save(filename="test.png")
# Saves to: $DP_MCP_SCREENSHOT_ROOT/test.png
```

### MCP Connection Issues

For Codex:

```bash
# Check MCP status
codex mcp list

# View logs
codex mcp logs drissionpage

# Restart MCP
codex mcp restart dressionpage
```

For Claude Desktop:

```bash
# Check logs (macOS)
tail -f ~/Library/Logs/Claude/mcp*.log

# Restart Claude Desktop completely
```

### Headless Mode Issues

```bash
# Disable headless for debugging
unset DP_HEADLESS

# Or set explicitly
export DP_HEADLESS="0"
```

## MCP Resources

DrissionPage MCP exposes resources via the MCP protocol:

- `drissionpage://session/summary` - Current session overview
- `drissionpage://session/history` - Navigation history
- `drissionpage://session/state` - Browser state
- `drissionpage://page/current` - Current page details
- `drissionpage://tools/catalog` - Available tools reference
- `drissionpage://guide/model-usage` - LLM usage guide
- `dressionpage://policy/summary` - Security policies

## MCP Prompts

Built-in prompts for common workflows:

- `drissionpage_mcp_usage_playbook` - Complete usage guide
- `browser_navigate_and_summarize` - Navigate and extract summary
- `browser_extract_structured_data` - Structured data extraction
- `browser_fill_form_safely` - Safe form automation
- `browser_debug_page_issue` - Debug page problems

## Best Practices

1. **Use `page_snapshot` first** - Understand page structure before element selection
2. **Prefer CSS selectors** - More readable and LLM-friendly than XPath
3. **Use recommended selectors** - Returned by `element_find_all` and `page_snapshot`
4. **Wait appropriately** - Use `wait_for_element` or `wait_until` for dynamic content
5. **Handle errors gracefully** - Check form requirements with `form_inspect`
6. **Leverage `page_observe`** - Get quick page context without full snapshot
7. **Use tabs wisely** - Switch between tabs with `tab_list` and `tab_switch`
8. **Check console logs** - Use `page_console_logs` for debugging JavaScript errors
9. **Secure file uploads** - Ensure files are under `DP_MCP_UPLOAD_ROOT`
10. **Close resources** - Use `page_close` when done to free browser instances

## References

- [GitHub Repository](https://github.com/jumodada/Drissionpage-MCP-Server)
- [Tool Contract Documentation](https://github.com/jumodada/Drissionpage-MCP-Server/blob/main/docs/tool-contract.md)
- [Compatibility Guide](https://github.com/jumodada/Drissionpage-MCP-Server/blob/main/docs/compatibility.md)
- [Troubleshooting Guide](https://github.com/jumodada/Drissionpage-MCP-Server/blob/main/docs/troubleshooting.md)
- [DrissionPage Library](https://github.com/g1879/DrissionPage)
