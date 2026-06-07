---
name: ios-mcp-jailbreak-automation
description: Control jailbroken iOS devices via MCP tools for AI agents to inspect, automate, and manage iPhone applications and system functions
triggers:
  - control my jailbroken iPhone with MCP
  - automate iOS device interactions
  - take screenshots and inspect UI on iOS
  - launch apps and simulate touches on iPhone
  - get device information from jailbroken iOS
  - run shell commands on jailbroken iPhone
  - manage clipboard and input text on iOS
  - install or uninstall apps on jailbroken device
---

# iOS MCP Jailbreak Automation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

iOS MCP is a Model Context Protocol (MCP) server that runs on jailbroken iPhones, enabling AI agents to directly control and inspect iOS devices through a comprehensive set of 34 automation tools.

## What It Does

iOS MCP provides AI agents with the ability to:

- **Touch & Gestures**: Tap, swipe, long press, double tap, drag and drop at precise screen coordinates
- **Hardware Buttons**: Simulate physical buttons (Home, Power, Volume, Mute) via HID
- **Text Input**: Type text via clipboard injection or HID simulation, press special keys
- **Screenshots**: Capture screen as Base64 JPEG, get screen dimensions and orientation
- **App Management**: Launch, kill, install, uninstall apps; list running and installed apps
- **Accessibility**: Query UI element tree, get elements at specific coordinates
- **Clipboard**: Read and write clipboard content
- **Device Control**: Adjust brightness and volume
- **Device Info**: Get model, iOS version, battery, storage, memory, jailbreak type
- **URL Handling**: Open URLs or URL schemes
- **Shell**: Execute arbitrary shell commands

## Installation

### Requirements

- Jailbroken iOS device (iOS 13-18)
- Appropriate jailbreak type:
  - **rootful** (iOS 13-18): `iphoneos-arm`
  - **rootless** (iOS 15-18): `iphoneos-arm64`
  - **roothide** (iOS 15-18): `iphoneos-arm64e`

### Install via Package Manager

1. Open Cydia or Sileo on your jailbroken device
2. Search for `iOS MCP`
3. Install the package (dependencies: `mobilesubstrate/ElleKit`, `preferenceloader`)
4. Respring SpringBoard

### Verify Installation

After installation, verify the server is running:

```bash
# From your computer or the device itself
curl http://<DEVICE_IP>:8090/health
```

Expected response:

```json
{"status":"ok","server":"ios-mcp","version":"1.1.1"}
```

### Configuration

1. Open **Settings** → **iOS MCP** on your device
2. Enable the MCP service
3. Tap **Copy MCP Prompt Snippet** to get the connection configuration
4. Paste the snippet into your AI agent's configuration

The server runs on port `8090` by default.

## MCP Configuration

Add to your MCP settings (e.g., `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "ios-mcp": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-fetch",
        "http://<YOUR_IOS_DEVICE_IP>:8090/mcp"
      ]
    }
  }
}
```

Replace `<YOUR_IOS_DEVICE_IP>` with your device's local IP address.

## Available MCP Tools

### Touch & Gestures

**tap_screen**: Tap at specific coordinates
```typescript
// Parameters: x, y
await mcp.call_tool("tap_screen", { x: 200, y: 400 });
```

**swipe_screen**: Swipe from one point to another
```typescript
// Parameters: from_x, from_y, to_x, to_y, duration (optional)
await mcp.call_tool("swipe_screen", {
  from_x: 200,
  from_y: 600,
  to_x: 200,
  to_y: 200,
  duration: 0.3
});
```

**long_press**: Long press at coordinates
```typescript
// Parameters: x, y, duration (optional)
await mcp.call_tool("long_press", { x: 200, y: 400, duration: 1.0 });
```

**double_tap**: Double tap at coordinates
```typescript
// Parameters: x, y
await mcp.call_tool("double_tap", { x: 200, y: 400 });
```

**drag_and_drop**: Drag from one point to another
```typescript
// Parameters: from_x, from_y, to_x, to_y, duration (optional)
await mcp.call_tool("drag_and_drop", {
  from_x: 100,
  from_y: 300,
  to_x: 300,
  to_y: 500,
  duration: 0.5
});
```

### Hardware Buttons

**press_home**: Press Home button
```typescript
await mcp.call_tool("press_home", {});
```

**press_power**: Press Power button
```typescript
await mcp.call_tool("press_power", {});
```

**press_volume_up** / **press_volume_down**: Press volume buttons
```typescript
await mcp.call_tool("press_volume_up", {});
await mcp.call_tool("press_volume_down", {});
```

**wake_and_home**: Wake device and press Home (useful when screen is locked)
```typescript
await mcp.call_tool("wake_and_home", {});
```

### Text Input

**input_text**: Fast text input via clipboard
```typescript
// Parameters: text
await mcp.call_tool("input_text", { text: "Hello from AI agent" });
```

**type_text**: Character-by-character typing via HID
```typescript
// Parameters: text
await mcp.call_tool("type_text", { text: "Slow typing simulation" });
```

**press_key**: Press special keys
```typescript
// Parameters: key (e.g., "return", "delete", "space")
await mcp.call_tool("press_key", { key: "return" });
```

### Screenshots & Screen Info

**screenshot**: Capture screen as Base64 JPEG
```typescript
// Returns: { image: "base64_jpeg_data", width: 1170, height: 2532 }
const result = await mcp.call_tool("screenshot", {});
// Decode base64 to save as file if needed
```

**get_screen_info**: Get screen dimensions and orientation
```typescript
// Returns: { width: 1170, height: 2532, orientation: "portrait" }
await mcp.call_tool("get_screen_info", {});
```

### App Management

**launch_app**: Launch app by bundle ID
```typescript
// Parameters: bundle_id
await mcp.call_tool("launch_app", { bundle_id: "com.apple.mobilesafari" });
```

**kill_app**: Force quit app
```typescript
// Parameters: bundle_id
await mcp.call_tool("kill_app", { bundle_id: "com.apple.mobilesafari" });
```

**list_apps**: List all installed apps
```typescript
// Returns array of apps with bundle_id, name, version
const apps = await mcp.call_tool("list_apps", {});
```

**list_running_apps**: List currently running apps
```typescript
const running = await mcp.call_tool("list_running_apps", {});
```

**get_frontmost_app**: Get currently focused app
```typescript
// Returns: { bundle_id: "com.example.app", name: "App Name" }
const frontmost = await mcp.call_tool("get_frontmost_app", {});
```

**install_app**: Install IPA file
```typescript
// Parameters: ipa_path (absolute path on device)
await mcp.call_tool("install_app", { ipa_path: "/var/mobile/app.ipa" });
```

**uninstall_app**: Uninstall app
```typescript
// Parameters: bundle_id
await mcp.call_tool("uninstall_app", { bundle_id: "com.example.app" });
```

### UI Accessibility

**get_ui_elements**: Get UI element tree (accessibility hierarchy)
```typescript
// Returns nested structure of UI elements with labels, types, frames
const uiTree = await mcp.call_tool("get_ui_elements", {});
```

**get_element_at_point**: Get UI element at specific coordinates
```typescript
// Parameters: x, y
const element = await mcp.call_tool("get_element_at_point", { x: 200, y: 400 });
```

### Clipboard

**get_clipboard**: Read clipboard content
```typescript
const content = await mcp.call_tool("get_clipboard", {});
```

**set_clipboard**: Write to clipboard
```typescript
// Parameters: text
await mcp.call_tool("set_clipboard", { text: "New clipboard content" });
```

### Device Control

**get_brightness** / **set_brightness**: Control screen brightness
```typescript
const brightness = await mcp.call_tool("get_brightness", {});
// Parameters: level (0.0 to 1.0)
await mcp.call_tool("set_brightness", { level: 0.5 });
```

**get_volume** / **set_volume**: Control system volume
```typescript
const volume = await mcp.call_tool("get_volume", {});
// Parameters: level (0.0 to 1.0)
await mcp.call_tool("set_volume", { level: 0.7 });
```

### Device Information

**get_device_info**: Get comprehensive device information
```typescript
// Returns: model, iOS version, battery, storage, memory, jailbreak type
const info = await mcp.call_tool("get_device_info", {});
// Example response:
// {
//   "model": "iPhone 14 Pro",
//   "ios_version": "16.1.2",
//   "battery_level": 85,
//   "battery_state": "charging",
//   "storage_total": 256000000000,
//   "storage_free": 128000000000,
//   "memory_total": 6000000000,
//   "jailbreak_type": "rootless"
// }
```

### URL & Shell

**open_url**: Open URL or URL scheme
```typescript
// Parameters: url
await mcp.call_tool("open_url", { url: "https://example.com" });
await mcp.call_tool("open_url", { url: "shortcuts://run-shortcut?name=MyShortcut" });
```

**run_command**: Execute shell command
```typescript
// Parameters: command
// WARNING: Use with caution - can execute any shell command
const output = await mcp.call_tool("run_command", { command: "ls -la /var/mobile" });
```

## Common Patterns

### Workflow: Automate Safari Navigation

```typescript
// 1. Wake device and unlock if needed
await mcp.call_tool("wake_and_home", {});

// 2. Launch Safari
await mcp.call_tool("launch_app", { bundle_id: "com.apple.mobilesafari" });

// 3. Wait a moment for app to load
await new Promise(resolve => setTimeout(resolve, 1000));

// 4. Take screenshot to see current state
const screen = await mcp.call_tool("screenshot", {});

// 5. Tap address bar (coordinates depend on device)
await mcp.call_tool("tap_screen", { x: 200, y: 100 });

// 6. Input URL
await mcp.call_tool("input_text", { text: "https://example.com" });

// 7. Press return
await mcp.call_tool("press_key", { key: "return" });
```

### Workflow: UI Inspection and Interaction

```typescript
// 1. Get UI element tree
const uiTree = await mcp.call_tool("get_ui_elements", {});

// 2. Parse tree to find button with specific label
function findButton(tree, label) {
  // Recursive search implementation
  if (tree.label === label && tree.type === "Button") {
    return tree;
  }
  for (const child of tree.children || []) {
    const found = findButton(child, label);
    if (found) return found;
  }
  return null;
}

const button = findButton(uiTree, "Submit");

// 3. Tap button at its center
if (button && button.frame) {
  const x = button.frame.x + button.frame.width / 2;
  const y = button.frame.y + button.frame.height / 2;
  await mcp.call_tool("tap_screen", { x, y });
}
```

### Workflow: Install and Test App

```typescript
// 1. Check device info
const deviceInfo = await mcp.call_tool("get_device_info", {});
console.log(`Device: ${deviceInfo.model}, iOS: ${deviceInfo.ios_version}`);

// 2. Install IPA (assume already uploaded to device)
await mcp.call_tool("install_app", { ipa_path: "/var/mobile/Documents/MyApp.ipa" });

// 3. Wait for installation
await new Promise(resolve => setTimeout(resolve, 5000));

// 4. Launch the app
await mcp.call_tool("launch_app", { bundle_id: "com.example.myapp" });

// 5. Take screenshot for verification
const screenshot = await mcp.call_tool("screenshot", {});

// 6. Interact with app...
await mcp.call_tool("tap_screen", { x: 200, y: 400 });
```

### Workflow: Automated Testing Loop

```typescript
// Automated regression test
async function runTest() {
  // Launch app
  await mcp.call_tool("launch_app", { bundle_id: "com.example.testapp" });
  
  // Perform test actions
  await mcp.call_tool("tap_screen", { x: 100, y: 200 });
  await mcp.call_tool("input_text", { text: "Test Data" });
  await mcp.call_tool("tap_screen", { x: 200, y: 500 });
  
  // Capture result
  const result = await mcp.call_tool("screenshot", {});
  
  // Verify UI state via get_ui_elements
  const ui = await mcp.call_tool("get_ui_elements", {});
  
  // Kill app
  await mcp.call_tool("kill_app", { bundle_id: "com.example.testapp" });
  
  return { screenshot: result, ui };
}
```

## Security & Best Practices

### Important Security Notes

- **No Authentication**: The MCP server has no built-in authentication. Only use on trusted local networks.
- **Lockscreen Protection**: When device is locked or screen is off, interactive tools (tap, swipe, input, launch app, shell) are blocked. Only status queries, screenshots, and wake commands are allowed.
- **Shell Command Risk**: `run_command` can execute arbitrary shell commands with device privileges. Use with extreme caution.
- **Root Privilege**: The `mcp-root` helper provides root escalation for specific tools only.

### Recommended Practices

1. **Network Security**: Keep iOS MCP on a private, trusted network
2. **Firewall**: Consider device firewall rules to limit access to port 8090
3. **Monitoring**: Check server logs if unexpected behavior occurs
4. **Testing**: Test automation workflows on non-production devices first
5. **Coordinates**: Screen coordinates vary by device model and orientation - always verify with `get_screen_info`

## Troubleshooting

### Server Not Responding

**Problem**: `curl http://<DEVICE_IP>:8090/health` fails

**Solutions**:
- Verify device IP address (Settings → Wi-Fi → (i) button)
- Ensure iOS MCP service is enabled in Settings → iOS MCP
- Respring SpringBoard after installation
- Check if firewall is blocking port 8090
- Ensure device and computer are on same network

### Tools Return "Device Locked" Error

**Problem**: Tap, swipe, or input commands fail with lock error

**Solutions**:
- Device screen is locked or off
- Use `wake_and_home` to unlock device first
- Status queries and screenshots still work when locked

### Screenshot Returns Invalid Base64

**Problem**: Screenshot data cannot be decoded

**Solutions**:
- Check if screen is on (may return blank if off)
- Verify `screenshot` tool returns valid JSON with `image` field
- Try `get_screen_info` first to verify screen state

### App Launch Fails

**Problem**: `launch_app` doesn't start the app

**Solutions**:
- Verify bundle ID is correct: use `list_apps` to find exact bundle ID
- Check if app is already running: use `get_frontmost_app`
- Ensure device is unlocked
- Some system apps may have restrictions

### UI Elements Tree is Empty

**Problem**: `get_ui_elements` returns no elements

**Solutions**:
- App may not expose accessibility information
- Try on a different app (e.g., Settings, Safari)
- Ensure app is in foreground: use `get_frontmost_app` to verify

### Permission Errors with Shell Commands

**Problem**: `run_command` returns permission denied

**Solutions**:
- Check jailbreak type and permissions
- Some commands require root (automatically escalated via `mcp-root`)
- Verify command syntax and paths are correct

### Install App Fails

**Problem**: `install_app` returns error

**Solutions**:
- Verify IPA file exists at specified path
- Check IPA is valid and signed appropriately for jailbroken device
- Ensure sufficient storage space
- Path must be absolute (e.g., `/var/mobile/Documents/app.ipa`)

---

This skill enables AI agents to fully automate jailbroken iOS devices through MCP, providing comprehensive control over UI interactions, app management, device state, and shell access for advanced automation and testing workflows.
