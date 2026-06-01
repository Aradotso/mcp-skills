---
name: tauri-mcp-server-development
description: Build, test, and debug Tauri v2 applications using AI assistants with the Model Context Protocol server for UI automation, IPC monitoring, and mobile development
triggers:
  - "set up Tauri MCP for AI development"
  - "automate Tauri app testing with screenshots"
  - "monitor Tauri IPC calls in my app"
  - "debug my Tauri webview with AI"
  - "add MCP bridge plugin to Tauri project"
  - "capture console logs from Tauri app"
  - "interact with Tauri UI elements programmatically"
  - "use visual element picker in Tauri"
---

# Tauri MCP Server Development

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that enables AI assistants to build, test, and debug Tauri v2 applications through UI automation, IPC monitoring, log streaming, and mobile device management.

## What It Does

The Tauri MCP server provides 21 tools across four categories:

- **UI Automation**: Screenshots, element finding, clicking, typing, scrolling, visual element picker
- **IPC Monitoring**: Capture and inspect Tauri IPC calls in real-time
- **Mobile Development**: List iOS simulators and Android emulators
- **Logs**: Stream console, Android logcat, iOS, and system logs

It consists of two components:
1. **MCP Server** (Node.js) - Connects to AI assistants via stdio
2. **MCP Bridge Plugin** (Rust) - Installed in your Tauri app, exposes automation APIs via WebSocket

## Installation

### Install MCP Server for AI Assistant

Use `install-mcp` to add the server to your AI assistant:

```bash
# For Claude Code
npx -y install-mcp @hypothesi/tauri-mcp-server --client claude-code

# For Cursor
npx -y install-mcp @hypothesi/tauri-mcp-server --client cursor

# For VS Code / Copilot
npx -y install-mcp @hypothesi/tauri-mcp-server --client vscode

# For Windsurf
npx -y install-mcp @hypothesi/tauri-mcp-server --client windsurf
```

Supported clients: `claude-code`, `cursor`, `windsurf`, `vscode`, `cline`, `roo-cline`, `claude`, `zed`, `goose`, `warp`, `codex`

**Restart your AI assistant** after installation.

### CLI Usage (Optional)

Install the CLI to call tools directly from terminal:

```bash
npm install -g @hypothesi/tauri-mcp-cli

# Start driver session
tauri-mcp driver-session start --port 9223

# Take screenshot
tauri-mcp webview-screenshot --file screenshot.png

# Find element
tauri-mcp webview-find-element --selector "button.submit"

# Read console logs
tauri-mcp read-logs --type console
```

### Add MCP Bridge Plugin to Tauri App

#### Automated Setup

Ask your AI assistant:

> "Help me set up the Tauri MCP Bridge plugin"

The AI will examine your project, show required changes, and ask permission before modifying files.

Or use the `/setup` slash command in your AI assistant.

#### Manual Setup

**1. Add dependency to `Cargo.toml`:**

```toml
[dependencies]
tauri-plugin-mcp-bridge = "0.1"
```

**2. Register plugin in `src-tauri/src/lib.rs`:**

```rust
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_mcp_bridge::init())
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

**3. Add permissions to `src-tauri/capabilities/default.json`:**

```json
{
  "permissions": [
    "mcp-bridge:default"
  ]
}
```

**4. (Optional) Initialize frontend bindings:**

```typescript
import { invoke } from '@tauri-apps/api/core';
import { listen } from '@tauri-apps/api/event';

// Listen for element selection events
await listen('mcp-bridge://element-selected', (event) => {
  console.log('Selected element:', event.payload);
});

// Get backend state
const state = await invoke('plugin:mcp-bridge|get_backend_state');
console.log('App state:', state);
```

## Key Tools Reference

### Setup & Configuration

#### `get_setup_instructions`

Get instructions for setting up or updating the MCP Bridge plugin.

```typescript
// Used by AI assistant automatically
// Returns setup instructions and current project status
```

### UI Automation

#### `driver_session`

Start, stop, or check status of automation session.

```bash
# CLI
tauri-mcp driver-session start --port 9223
tauri-mcp driver-session status
tauri-mcp driver-session stop

# In AI assistant - automatically managed
```

Parameters:
- `action`: `"start"` | `"stop"` | `"status"`
- `port`: WebSocket port (default: 9223)
- `host`: Host address (default: "localhost")

#### `webview_screenshot`

Capture screenshots of webview or specific elements.

```bash
# CLI - Full window
tauri-mcp webview-screenshot --file screenshot.png

# Specific element
tauri-mcp webview-screenshot --selector ".main-content" --file content.png

# Base64 output
tauri-mcp webview-screenshot --format base64
```

Parameters:
- `selector?`: CSS selector for element to capture
- `format?`: `"png"` | `"base64"` (default: "png")
- `file?`: Output file path (CLI only)
- `windowId?`: Target specific window

#### `webview_find_element`

Find elements using CSS selectors.

```bash
# CLI
tauri-mcp webview-find-element --selector "button.submit"
tauri-mcp webview-find-element --selector "input[type='text']" --all
```

Returns:
```json
{
  "selector": "button.submit",
  "found": true,
  "count": 1,
  "elements": [{
    "tagName": "BUTTON",
    "text": "Submit",
    "visible": true,
    "bounds": {"x": 100, "y": 200, "width": 80, "height": 40}
  }]
}
```

Parameters:
- `selector`: CSS selector
- `all?`: Return all matches (default: false)
- `windowId?`: Target specific window

#### `webview_interact`

Click, scroll, swipe, focus, or long-press elements.

```bash
# Click
tauri-mcp webview-interact --action click --selector "button#login"

# Scroll
tauri-mcp webview-interact --action scroll --selector ".content" --x 0 --y 100

# Swipe (mobile)
tauri-mcp webview-interact --action swipe --direction up --duration 300

# Long press
tauri-mcp webview-interact --action longPress --selector ".context-menu-trigger"
```

Parameters:
- `action`: `"click"` | `"scroll"` | `"swipe"` | `"focus"` | `"longPress"`
- `selector?`: CSS selector (required for click, scroll, focus, longPress)
- `x?`, `y?`: Coordinates for scroll
- `direction?`: `"up"` | `"down"` | `"left"` | `"right"` (for swipe)
- `duration?`: Duration in ms (for swipe, longPress)
- `windowId?`: Target specific window

#### `webview_keyboard`

Type text or send key events.

```bash
# Type text
tauri-mcp webview-keyboard --action type --text "Hello, World!"

# Press key
tauri-mcp webview-keyboard --action press --key Enter

# Key combination
tauri-mcp webview-keyboard --action press --key c --modifiers '["Control"]'
```

Parameters:
- `action`: `"type"` | `"press"`
- `text?`: Text to type (for type action)
- `key?`: Key name (for press action)
- `modifiers?`: Array of `"Control"` | `"Shift"` | `"Alt"` | `"Meta"`
- `windowId?`: Target specific window

#### `webview_wait_for`

Wait for elements, text, or events.

```bash
# Wait for element
tauri-mcp webview-wait-for --type element --selector ".loaded" --timeout 5000

# Wait for text
tauri-mcp webview-wait-for --type text --text "Success"

# Wait for event
tauri-mcp webview-wait-for --type event --eventName "data-loaded"
```

Parameters:
- `type`: `"element"` | `"text"` | `"event"`
- `selector?`: CSS selector (for element)
- `text?`: Text to wait for
- `eventName?`: Event name to listen for
- `timeout?`: Timeout in ms (default: 5000)
- `windowId?`: Target specific window

#### `webview_execute_js`

Execute JavaScript in webview.

```bash
# CLI
tauri-mcp webview-execute-js --code "document.title"
tauri-mcp webview-execute-js --code "document.querySelector('.count').textContent"
```

Parameters:
- `code`: JavaScript code to execute
- `windowId?`: Target specific window

#### `webview_dom_snapshot`

Get structured accessibility tree snapshot.

```bash
# CLI
tauri-mcp webview-dom-snapshot --selector ".main-content"
tauri-mcp webview-dom-snapshot --max-depth 3
```

Returns structured DOM tree with accessibility information.

Parameters:
- `selector?`: Root selector (default: "body")
- `maxDepth?`: Maximum tree depth
- `windowId?`: Target specific window

#### `webview_select_element`

Visual element picker - user clicks element in app, returns metadata and screenshot.

```bash
# CLI
tauri-mcp webview-select-element --timeout 30000
```

Process:
1. App enters selection mode
2. User hovers/clicks element
3. Returns element metadata + screenshot with highlight

Parameters:
- `timeout?`: Selection timeout in ms (default: 30000)
- `windowId?`: Target specific window

#### `webview_get_pointed_element`

Get metadata for element user Alt+Shift+Clicked.

```bash
# CLI
tauri-mcp webview-get-pointed-element
```

Returns most recently pointed element metadata.

#### `manage_window`

List windows, get info, or resize.

```bash
# List all windows
tauri-mcp manage-window --action list

# Get window info
tauri-mcp manage-window --action info --window-id "main"

# Resize window
tauri-mcp manage-window --action resize --window-id "main" --width 800 --height 600
```

Parameters:
- `action`: `"list"` | `"info"` | `"resize"`
- `windowId?`: Window ID (for info, resize)
- `width?`, `height?`: Dimensions (for resize)

### IPC & Plugin Tools

#### `ipc_execute_command`

Execute Tauri IPC commands.

```bash
# CLI
tauri-mcp ipc-execute-command --command "get_app_version"
tauri-mcp ipc-execute-command --command "save_data" --args '{"key": "value"}'
```

Parameters:
- `command`: Command name
- `args?`: Command arguments (JSON object)

#### `ipc_get_backend_state`

Get app metadata and backend state.

```bash
# CLI
tauri-mcp ipc-get-backend-state
```

Returns:
```json
{
  "metadata": {
    "name": "my-app",
    "version": "1.0.0",
    "os": "macos",
    "arch": "aarch64"
  },
  "capabilities": {
    "windows": ["main"],
    "commands": ["get_app_version", "save_data"]
  }
}
```

#### `ipc_monitor`

Start or stop IPC monitoring.

```bash
# Start monitoring
tauri-mcp ipc-monitor --action start --events '["command", "event"]'

# Stop monitoring
tauri-mcp ipc-monitor --action stop
```

Parameters:
- `action`: `"start"` | `"stop"`
- `events?`: Array of event types to capture

#### `ipc_get_captured`

Get captured IPC traffic.

```bash
# CLI
tauri-mcp ipc-get-captured --clear
```

Parameters:
- `clear?`: Clear captured events after reading (default: false)

#### `ipc_emit_event`

Emit custom events to Tauri app.

```bash
# CLI
tauri-mcp ipc-emit-event --event "data-updated" --payload '{"id": 123}'
```

Parameters:
- `event`: Event name
- `payload?`: Event payload (any JSON value)

### Logs

#### `read_logs`

Read console, Android, iOS, or system logs.

```bash
# Console logs
tauri-mcp read-logs --type console

# Android logcat
tauri-mcp read-logs --type android --device "emulator-5554"

# iOS logs
tauri-mcp read-logs --type ios --device "iPhone 15 Pro"

# System logs (backend)
tauri-mcp read-logs --type system
```

Parameters:
- `type`: `"console"` | `"android"` | `"ios"` | `"system"`
- `device?`: Device ID (for android, ios)
- `follow?`: Stream logs continuously
- `windowId?`: Target specific window (for console logs)

### Mobile Development

#### `list_devices`

List Android devices and iOS simulators.

```bash
# CLI
tauri-mcp list-devices --platform android
tauri-mcp list-devices --platform ios
tauri-mcp list-devices --platform all
```

Returns:
```json
{
  "android": [
    {"id": "emulator-5554", "name": "Pixel 7", "status": "booted"}
  ],
  "ios": [
    {"id": "...", "name": "iPhone 15 Pro", "status": "Booted"}
  ]
}
```

## Configuration

### MCP Server Config

If you manually configure (not using `install-mcp`), add to your AI assistant's MCP config:

**Claude Desktop (`~/Library/Application Support/Claude/claude_desktop_config.json`):**

```json
{
  "mcpServers": {
    "tauri": {
      "command": "npx",
      "args": ["-y", "@hypothesi/tauri-mcp-server"]
    }
  }
}
```

**Cursor (`~/.cursor/config.json` or similar):**

```json
{
  "mcp": {
    "servers": {
      "tauri": {
        "command": "npx",
        "args": ["-y", "@hypothesi/tauri-mcp-server"]
      }
    }
  }
}
```

### Environment Variables

Set via MCP server config or CLI:

- `TAURI_MCP_PORT`: Default WebSocket port (default: 9223)
- `TAURI_MCP_HOST`: Default host (default: "localhost")
- `DEBUG`: Enable debug logging (e.g., `DEBUG=tauri-mcp:*`)

### Multi-Window Support

All webview tools accept optional `windowId` parameter:

```typescript
// List windows first
const windows = await useMcpTool('manage_window', { action: 'list' });
// Returns: [{ id: 'main', title: 'Main Window' }, { id: 'settings', title: 'Settings' }]

// Target specific window
await useMcpTool('webview_screenshot', {
  windowId: 'settings',
  file: 'settings.png'
});
```

## Common Patterns

### Debug Webview Errors

Use the `/fix-webview-errors` slash command, or manually:

```typescript
// 1. Read console logs
const logs = await useMcpTool('read_logs', { type: 'console' });

// 2. Take screenshot to see UI state
const screenshot = await useMcpTool('webview_screenshot', {});

// 3. Get DOM snapshot
const dom = await useMcpTool('webview_dom_snapshot', {});

// 4. Execute diagnostic JS
const result = await useMcpTool('webview_execute_js', {
  code: 'window.lastError || "no errors"'
});
```

### Visual Element Selection Workflow

Use `/select` slash command or:

```typescript
// 1. Start element selection
const selection = await useMcpTool('webview_select_element', {
  timeout: 30000
});

// Returns:
// {
//   selector: '.submit-button',
//   tagName: 'BUTTON',
//   text: 'Submit',
//   bounds: { x: 100, y: 200, width: 80, height: 40 },
//   screenshot: 'base64...'
// }

// 2. Use returned selector for interactions
await useMcpTool('webview_interact', {
  action: 'click',
  selector: selection.selector
});
```

### Monitor IPC Traffic

```typescript
// 1. Start monitoring
await useMcpTool('ipc_monitor', {
  action: 'start',
  events: ['command', 'event']
});

// 2. Perform actions in app
// ...

// 3. Get captured traffic
const traffic = await useMcpTool('ipc_get_captured', { clear: true });

// Returns:
// {
//   commands: [
//     { name: 'save_data', args: {...}, timestamp: ... }
//   ],
//   events: [
//     { name: 'data-updated', payload: {...}, timestamp: ... }
//   ]
// }
```

### Mobile Testing

```typescript
// 1. List devices
const devices = await useMcpTool('list_devices', { platform: 'all' });

// 2. Start app on device
// (use Tauri CLI: tauri android dev -d emulator-5554)

// 3. Read Android logs
const logs = await useMcpTool('read_logs', {
  type: 'android',
  device: 'emulator-5554',
  follow: true
});
```

### Automated Testing Workflow

```typescript
// 1. Start session
await useMcpTool('driver_session', { action: 'start', port: 9223 });

// 2. Wait for app to load
await useMcpTool('webview_wait_for', {
  type: 'element',
  selector: '.app-loaded',
  timeout: 5000
});

// 3. Fill form
await useMcpTool('webview_interact', {
  action: 'click',
  selector: 'input[name="username"]'
});
await useMcpTool('webview_keyboard', {
  action: 'type',
  text: 'testuser'
});

// 4. Submit
await useMcpTool('webview_interact', {
  action: 'click',
  selector: 'button[type="submit"]'
});

// 5. Wait for result
await useMcpTool('webview_wait_for', {
  type: 'text',
  text: 'Login successful'
});

// 6. Verify
const screenshot = await useMcpTool('webview_screenshot', {});
```

## Troubleshooting

### Connection Issues

**Problem**: `Failed to connect to WebSocket`

**Solution**:
```bash
# 1. Check if app is running
ps aux | grep tauri

# 2. Verify plugin is registered
# Check src-tauri/src/lib.rs for:
# .plugin(tauri_plugin_mcp_bridge::init())

# 3. Check port
tauri-mcp driver-session status

# 4. Try different port
tauri-mcp driver-session start --port 9224
```

### Element Not Found

**Problem**: `Element not found` despite element being visible

**Solution**:
```typescript
// 1. Wait for element to appear
await useMcpTool('webview_wait_for', {
  type: 'element',
  selector: '.my-element',
  timeout: 5000
});

// 2. Use visual selector to verify correct selector
const element = await useMcpTool('webview_select_element', {});
console.log('Correct selector:', element.selector);

// 3. Check DOM snapshot to see what's available
const dom = await useMcpTool('webview_dom_snapshot', {});
```

### Screenshots Empty or Black

**Problem**: Screenshots are blank

**Solution**:
```typescript
// 1. Wait for content to render
await useMcpTool('webview_wait_for', {
  type: 'element',
  selector: 'body.loaded' // or your app's ready class
});

// 2. Check window visibility
const windows = await useMcpTool('manage_window', { action: 'list' });
console.log('Windows:', windows);

// 3. Target specific window
await useMcpTool('webview_screenshot', {
  windowId: 'main'
});
```

### Permission Errors

**Problem**: `Command not found: plugin:mcp-bridge|...`

**Solution**:
```json
// Add to src-tauri/capabilities/default.json
{
  "permissions": [
    "mcp-bridge:default",
    "mcp-bridge:allow-get-backend-state",
    "mcp-bridge:allow-execute-command"
  ]
}
```

### Mobile Logs Not Appearing

**Problem**: Android/iOS logs are empty

**Solution**:
```bash
# Android - check device connection
adb devices

# Android - verify logcat
adb -s emulator-5554 logcat

# iOS - verify simulator is booted
xcrun simctl list | grep Booted

# iOS - check system logs
xcrun simctl spawn <device-id> log stream
```

## Real-World Examples

### E2E Test Automation

```typescript
async function testLoginFlow() {
  // Start automation
  await useMcpTool('driver_session', { action: 'start' });
  
  // Navigate to login
  await useMcpTool('webview_interact', {
    action: 'click',
    selector: 'a[href="/login"]'
  });
  
  // Fill credentials
  await useMcpTool('webview_keyboard', {
    action: 'type',
    text: 'user@example.com'
  });
  await useMcpTool('webview_keyboard', {
    action: 'press',
    key: 'Tab'
  });
  await useMcpTool('webview_keyboard', {
    action: 'type',
    text: 'password123'
  });
  
  // Submit
  await useMcpTool('webview_keyboard', {
    action: 'press',
    key: 'Enter'
  });
  
  // Verify success
  await useMcpTool('webview_wait_for', {
    type: 'element',
    selector: '.dashboard',
    timeout: 5000
  });
  
  const screenshot = await useMcpTool('webview_screenshot', {
    file: 'dashboard.png'
  });
  
  console.log('✓ Login successful');
}
```

### Debug Performance Issue

```typescript
async function debugSlowRender() {
  // Start monitoring
  await useMcpTool('ipc_monitor', { action: 'start' });
  
  // Trigger slow operation
  await useMcpTool('webview_interact', {
    action: 'click',
    selector: '.load-data'
  });
  
  // Capture console logs
  const logs = await useMcpTool('read_logs', {
    type: 'console'
  });
  
  // Get IPC traffic
  const traffic = await useMcpTool('ipc_get_captured', {});
  
  // Analyze timing
  console.log('IPC Commands:', traffic.commands.length);
  console.log('Console Logs:', logs.length);
  
  // Take screenshot of result
  await useMcpTool('webview_screenshot', {
    file: 'after-load.png'
  });
}
```

### Mobile Screenshot Testing

```typescript
async function captureScreensOnAllDevices() {
  const devices = await useMcpTool('list_devices', { platform: 'all' });
  
  for (const device of devices.android) {
    console.log(`Testing on ${device.name}...`);
    
    // Launch app on device (via Tauri CLI externally)
    // tauri android dev -d ${device.id}
    
    // Wait for app
    await useMcpTool('webview_wait_for', {
      type: 'element',
      selector: '.app-ready',
      timeout: 10000
    });
    
    // Capture screen
    await useMcpTool('webview_screenshot', {
      file: `screenshots/${device.name.replace(/\s/g, '-')}.png`
    });
    
    // Read logs
    const logs = await useMcpTool('read_logs', {
      type: 'android',
      device: device.id
    });
    
    console.log(`✓ Captured ${device.name}`);
  }
}
```

## Additional Resources

- [Full Documentation](https://hypothesi.github.io/mcp-server-tauri/)
- [GitHub Repository](https://github.com/hypothesi/mcp-server-tauri)
- [MCP Server Package](https://www.npmjs.com/package/@hypothesi/tauri-mcp-server)
- [MCP Bridge Plugin (crates.io)](https://crates.io/crates/tauri-plugin-mcp-bridge)
- [Tauri v2 Documentation](https://v2.tauri.app)
