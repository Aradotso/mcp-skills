---
name: unity-mcp-server
description: Control Unity Editor and Unity Hub from AI assistants via MCP — 288 tools for scene management, GameObjects, components, builds, profiling, Shader Graph, terrain, physics, and more
triggers:
  - manipulate Unity scenes and GameObjects through AI
  - automate Unity Editor workflows with MCP
  - build and profile Unity projects via AI assistant
  - create and manage Unity components programmatically
  - integrate Claude or Cursor with Unity development
  - control Unity Editor remotely through MCP server
  - manage Unity Hub and editor versions with AI
  - automate Unity terrain, shader, and NavMesh tasks
---

# unity-mcp-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**unity-mcp-server** is a comprehensive Model Context Protocol (MCP) server that connects AI assistants (Claude Desktop, Cursor, Windsurf) to Unity Editor and Unity Hub. It provides 288 tools across 30+ categories for AI-assisted game development, including scene management, GameObject manipulation, component wiring, builds, profiling, Shader Graph editing, terrain sculpting, NavMesh baking, and multiplayer scenarios.

The server uses a two-tier architecture: ~70 core tools exposed directly, and ~130+ advanced tools accessed via `unity_advanced_tool` proxy with lazy loading to avoid overwhelming MCP clients.

**Architecture:**
```
AI Assistant ↔ MCP Server (Node.js) ↔ Unity Editor Plugin (HTTP bridge) ↔ Unity Editor
                    ↕
                Unity Hub CLI
```

## Installation

### 1. Install Unity Plugin

In Unity Editor:
1. Open **Window > Package Manager**
2. Click **+ > Add package from git URL**
3. Enter: `https://github.com/AnkleBreaker-Studio/unity-mcp-plugin.git`
4. Wait for installation to complete
5. Verify plugin is running: **Window > MCP Dashboard**

### 2. Install MCP Server

```bash
git clone https://github.com/AnkleBreaker-Studio/unity-mcp-server.git
cd unity-mcp-server
npm install
```

### 3. Configure Claude Desktop

Edit Claude Desktop config (`~/.config/Claude/claude_desktop_config.json` on macOS/Linux, `%APPDATA%\Claude\claude_desktop_config.json` on Windows):

```json
{
  "mcpServers": {
    "unity": {
      "command": "node",
      "args": ["/absolute/path/to/unity-mcp-server/src/index.js"],
      "env": {
        "UNITY_HUB_PATH": "/Applications/Unity Hub.app/Contents/MacOS/Unity Hub",
        "UNITY_BRIDGE_PORT": "7890"
      }
    }
  }
}
```

**Windows example:**
```json
{
  "mcpServers": {
    "unity": {
      "command": "node",
      "args": ["C:\\Dev\\unity-mcp-server\\src\\index.js"],
      "env": {
        "UNITY_HUB_PATH": "C:\\Program Files\\Unity Hub\\Unity Hub.exe",
        "UNITY_BRIDGE_PORT": "7890"
      }
    }
  }
}
```

### 4. Configure Cursor/Windsurf

Add to `.cursor/mcp.json` or Windsurf settings:

```json
{
  "mcpServers": {
    "unity": {
      "command": "node",
      "args": ["/absolute/path/to/unity-mcp-server/src/index.js"],
      "env": {
        "UNITY_HUB_PATH": "/Applications/Unity Hub.app/Contents/MacOS/Unity Hub",
        "UNITY_BRIDGE_PORT": "7890"
      }
    }
  }
}
```

Restart your AI assistant after configuration.

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `UNITY_HUB_PATH` | (OS-dependent) | Path to Unity Hub executable |
| `UNITY_BRIDGE_HOST` | `127.0.0.1` | Editor plugin HTTP host |
| `UNITY_BRIDGE_PORT` | `7890` | Editor plugin HTTP port (auto-discovered in multi-instance mode) |
| `UNITY_BRIDGE_TIMEOUT` | `30000` | Request timeout in milliseconds |
| `UNITY_PORT_RANGE_START` | `7890` | Port scan range start for multi-instance discovery |
| `UNITY_PORT_RANGE_END` | `7899` | Port scan range end |
| `UNITY_REGISTRY_STALENESS_TIMEOUT` | `300000` | Registry entry staleness timeout (crash detection) |
| `UNITY_RESPONSE_SOFT_LIMIT` | `2097152` | Response size soft limit (2MB, warning only) |
| `UNITY_RESPONSE_HARD_LIMIT` | `4194304` | Response size hard limit (4MB, truncates) |
| `UNITY_MCP_DEBUG` | `false` | Enable debug logging |

### Unity Plugin Settings

Access via **Window > MCP Dashboard** in Unity Editor:
- **HTTP Port**: Change the server port (default: 7890)
- **Auto-Start Server**: Start HTTP server on editor launch
- **Category Toggles**: Enable/disable specific tool categories (Shader Graph, Amplify, MPPM, etc.)

## Core Tools

### Unity Hub Management

**List installed Unity editors:**
```
unity_hub_list_editors
```

**Install a Unity editor version:**
```
unity_hub_install_editor
{
  "version": "2022.3.20f1",
  "modules": ["android", "ios", "webgl"]
}
```

**Set Unity Hub install path:**
```
unity_hub_set_install_path
{
  "path": "/Applications/Unity/Hub/Editor"
}
```

### Scene Management

**Get scene hierarchy (with pagination):**
```
unity_get_scene_hierarchy
{
  "page": 1,
  "pageSize": 50
}
```

**Open scene:**
```
unity_open_scene
{
  "scenePath": "Assets/Scenes/MainMenu.unity"
}
```

**Create new scene:**
```
unity_create_scene
{
  "sceneName": "Level_01",
  "savePath": "Assets/Scenes/Level_01.unity",
  "additive": false
}
```

### GameObject Manipulation

**Create primitive GameObject:**
```
unity_create_gameobject
{
  "name": "RedCube",
  "primitiveType": "Cube",
  "position": [0, 2, 0],
  "parent": null
}
```

**Delete GameObject:**
```
unity_delete_gameobject
{
  "objectPath": "RedCube"
}
```

**Set transform (local):**
```
unity_set_transform
{
  "objectPath": "RedCube",
  "position": [0, 5, 0],
  "rotation": [0, 45, 0],
  "scale": [2, 2, 2],
  "space": "Local"
}
```

**Reparent GameObject:**
```
unity_set_parent
{
  "objectPath": "RedCube",
  "parentPath": "GameObjects/Level",
  "worldPositionStays": true
}
```

### Component Management

**Add component:**
```
unity_add_component
{
  "objectPath": "RedCube",
  "componentType": "Rigidbody"
}
```

**Set component property:**
```
unity_set_component_property
{
  "objectPath": "RedCube",
  "componentType": "Rigidbody",
  "propertyPath": "mass",
  "value": 10.5
}
```

**Get component properties:**
```
unity_get_component
{
  "objectPath": "RedCube",
  "componentType": "Rigidbody"
}
```

**Wire object reference:**
```
unity_wire_object_reference
{
  "sourceObjectPath": "Player",
  "sourceComponentType": "PlayerController",
  "propertyPath": "target",
  "targetObjectPath": "Enemy/Boss"
}
```

**Batch wire references:**
```
unity_batch_wire_references
{
  "operations": [
    {
      "sourceObjectPath": "UI/HealthBar",
      "sourceComponentType": "HealthDisplay",
      "propertyPath": "player",
      "targetObjectPath": "Player"
    },
    {
      "sourceObjectPath": "Camera",
      "sourceComponentType": "CameraFollow",
      "propertyPath": "target",
      "targetObjectPath": "Player"
    }
  ]
}
```

### Asset Management

**List assets:**
```
unity_list_assets
{
  "path": "Assets/Materials",
  "recursive": true
}
```

**Create material:**
```
unity_create_material
{
  "name": "RedMetal",
  "savePath": "Assets/Materials/RedMetal.mat",
  "shader": "Standard",
  "properties": {
    "_Color": [1, 0, 0, 1],
    "_Metallic": 0.8,
    "_Glossiness": 0.6
  }
}
```

**Assign material:**
```
unity_assign_material
{
  "objectPath": "RedCube",
  "materialPath": "Assets/Materials/RedMetal.mat"
}
```

**Create prefab:**
```
unity_create_prefab
{
  "objectPath": "RedCube",
  "savePath": "Assets/Prefabs/RedCube.prefab"
}
```

### Script Management

**Create C# script:**
```
unity_create_script
{
  "scriptName": "PlayerController",
  "savePath": "Assets/Scripts/PlayerController.cs",
  "template": "MonoBehaviour"
}
```

**Read script:**
```
unity_read_script
{
  "scriptPath": "Assets/Scripts/PlayerController.cs"
}
```

**Update script:**
```
unity_update_script
{
  "scriptPath": "Assets/Scripts/PlayerController.cs",
  "content": "using UnityEngine;\n\npublic class PlayerController : MonoBehaviour\n{\n    public float speed = 5f;\n    \n    void Update()\n    {\n        float h = Input.GetAxis(\"Horizontal\");\n        float v = Input.GetAxis(\"Vertical\");\n        transform.Translate(new Vector3(h, 0, v) * speed * Time.deltaTime);\n    }\n}"
}
```

### Build Management

**Build for Windows:**
```
unity_build
{
  "target": "StandaloneWindows64",
  "outputPath": "Builds/Windows/Game.exe",
  "developmentBuild": false
}
```

**Build for WebGL:**
```
unity_build
{
  "target": "WebGL",
  "outputPath": "Builds/WebGL",
  "developmentBuild": true
}
```

**Supported platforms:** `StandaloneWindows64`, `StandaloneOSX`, `StandaloneLinux64`, `Android`, `iOS`, `WebGL`

### Console & Compilation

**Get console logs:**
```
unity_get_console_logs
{
  "includeErrors": true,
  "includeWarnings": true,
  "includeInfo": false,
  "maxCount": 50
}
```

**Clear console:**
```
unity_clear_console
```

**Get compilation errors (independent of console):**
```
unity_get_compilation_errors
```

### Play Mode Control

**Enter play mode:**
```
unity_play
```

**Pause play mode:**
```
unity_pause
```

**Stop play mode:**
```
unity_stop
```

### Testing

**List available tests:**
```
unity_list_tests
{
  "testMode": "EditMode"
}
```

**Run tests:**
```
unity_run_tests
{
  "testMode": "PlayMode",
  "testFilter": "PlayerTests"
}
```

**Get test results:**
```
unity_get_test_results
```

## Advanced Tools

Access advanced tools via the proxy:

```
unity_advanced_tool
{
  "toolName": "unity_bake_navmesh",
  "arguments": {}
}
```

**List all advanced tools:**
```
unity_list_advanced_tools
```

### Common Advanced Tool Examples

**Terrain sculpting:**
```
unity_advanced_tool
{
  "toolName": "unity_paint_terrain_heightmap",
  "arguments": {
    "terrainPath": "Terrain",
    "brushSize": 10,
    "brushStrength": 0.5,
    "centerX": 128,
    "centerZ": 128,
    "raise": true
  }
}
```

**Bake NavMesh:**
```
unity_advanced_tool
{
  "toolName": "unity_bake_navmesh",
  "arguments": {}
}
```

**Start profiling:**
```
unity_advanced_tool
{
  "toolName": "unity_profiler_start",
  "arguments": {
    "deepProfile": true
  }
}
```

**Get profiler stats:**
```
unity_advanced_tool
{
  "toolName": "unity_profiler_get_stats",
  "arguments": {}
}
```

**List Shader Graphs:**
```
unity_advanced_tool
{
  "toolName": "unity_shadergraph_list",
  "arguments": {}
}
```

**Create Shader Graph:**
```
unity_advanced_tool
{
  "toolName": "unity_shadergraph_create",
  "arguments": {
    "name": "MyCustomShader",
    "savePath": "Assets/Shaders/MyCustomShader.shadergraph",
    "graphType": "PBR"
  }
}
```

**Capture scene view:**
```
unity_advanced_tool
{
  "toolName": "unity_capture_scene_view",
  "arguments": {
    "width": 1920,
    "height": 1080
  }
}
```

## Multi-Instance Workflows

When multiple Unity Editor instances are running (e.g., main editor + ParrelSync clones), the server auto-discovers all instances on startup.

**Discovery flow:**
1. Server scans ports 7890-7899
2. Queries each active port for instance info
3. If one instance found: auto-connects
4. If multiple instances found: prompts user to select

**Switch instances:**
```
unity_switch_instance
{
  "port": 7891
}
```

**List discovered instances:**
```
unity_list_instances
```

The server validates instance identity (project name + path) when restoring saved connections to prevent port conflicts.

## Common Patterns

### Building a Complete Scene

```javascript
// AI workflow example (natural language → MCP tools)

// 1. Create new scene
unity_create_scene { sceneName: "GameLevel", savePath: "Assets/Scenes/GameLevel.unity" }

// 2. Add terrain
unity_advanced_tool { toolName: "unity_create_terrain", arguments: { name: "Terrain", position: [0,0,0] } }

// 3. Create player GameObject
unity_create_gameobject { name: "Player", primitiveType: "Capsule", position: [0,1,0] }
unity_add_component { objectPath: "Player", componentType: "Rigidbody" }
unity_add_component { objectPath: "Player", componentType: "CapsuleCollider" }
unity_create_script { scriptName: "PlayerController", savePath: "Assets/Scripts/PlayerController.cs" }
unity_add_component { objectPath: "Player", componentType: "PlayerController" }

// 4. Create enemy
unity_create_gameobject { name: "Enemy", primitiveType: "Cube", position: [10,1,10] }
unity_create_material { name: "EnemyRed", savePath: "Assets/Materials/EnemyRed.mat", shader: "Standard", properties: { "_Color": [1,0,0,1] } }
unity_assign_material { objectPath: "Enemy", materialPath: "Assets/Materials/EnemyRed.mat" }

// 5. Wire references
unity_wire_object_reference { sourceObjectPath: "Player", sourceComponentType: "PlayerController", propertyPath: "enemy", targetObjectPath: "Enemy" }

// 6. Save scene
unity_save_scene { scenePath: "Assets/Scenes/GameLevel.unity" }
```

### Profiling and Optimization

```javascript
// Start profiling
unity_advanced_tool { toolName: "unity_profiler_start", arguments: { deepProfile: false } }

// Enter play mode
unity_play

// Wait for gameplay...
// (AI can observe frame time from stats)

// Get profiler stats
unity_advanced_tool { toolName: "unity_profiler_get_stats", arguments: {} }

// Stop profiling
unity_advanced_tool { toolName: "unity_profiler_stop", arguments: {} }

// Get memory breakdown
unity_advanced_tool { toolName: "unity_memory_get_breakdown", arguments: {} }

// Stop play mode
unity_stop
```

### Automated Testing

```javascript
// List available tests
unity_list_tests { testMode: "PlayMode" }

// Run specific test suite
unity_run_tests { testMode: "PlayMode", testFilter: "CombatTests" }

// Poll for results
unity_get_test_results

// If failures, check console
unity_get_console_logs { includeErrors: true, includeWarnings: false }
```

### Multi-Platform Builds

```javascript
// Build for Windows
unity_build { target: "StandaloneWindows64", outputPath: "Builds/Windows/Game.exe" }

// Build for macOS
unity_build { target: "StandaloneOSX", outputPath: "Builds/macOS/Game.app" }

// Build for WebGL (development)
unity_build { target: "WebGL", outputPath: "Builds/WebGL", developmentBuild: true }

// Build for Android
unity_build { target: "Android", outputPath: "Builds/Android/Game.apk" }
```

## Troubleshooting

### Server Won't Connect to Unity Editor

**Check plugin installation:**
- Open **Window > MCP Dashboard** in Unity
- Verify HTTP server is running (green indicator)
- Check port number matches `UNITY_BRIDGE_PORT`

**Verify port availability:**
```bash
# macOS/Linux
lsof -i :7890

# Windows
netstat -ano | findstr :7890
```

**Enable debug logging:**
```json
{
  "env": {
    "UNITY_MCP_DEBUG": "true"
  }
}
```

### Multi-Instance Port Conflicts

If Unity Editor instances conflict on ports:

1. Open **Window > MCP Dashboard** in each instance
2. Manually assign different ports (7890, 7891, 7892, etc.)
3. Restart MCP server to re-discover instances
4. Use `unity_switch_instance` to select correct instance

### Response Size Errors

If responses exceed limits (common with large hierarchies):

**Use pagination:**
```
unity_get_scene_hierarchy { page: 1, pageSize: 25 }
```

**Increase limits (if needed):**
```json
{
  "env": {
    "UNITY_RESPONSE_SOFT_LIMIT": "4194304",
    "UNITY_RESPONSE_HARD_LIMIT": "8388608"
  }
}
```

### Compilation Errors Not Clearing

Unity's console buffer is separate from `CompilationPipeline` API:

- `unity_get_console_logs` → console buffer (may be truncated)
- `unity_get_compilation_errors` → direct from CompilationPipeline (accurate for C# errors)

Always use `unity_get_compilation_errors` for accurate compilation status.

### Unity Hub Not Found

**macOS:**
```json
{
  "UNITY_HUB_PATH": "/Applications/Unity Hub.app/Contents/MacOS/Unity Hub"
}
```

**Windows:**
```json
{
  "UNITY_HUB_PATH": "C:\\Program Files\\Unity Hub\\Unity Hub.exe"
}
```

**Linux:**
```json
{
  "UNITY_HUB_PATH": "/usr/bin/unityhub"
}
```

### Package-Specific Tools Not Available

Some tools require Unity packages:

| Package | Install Command |
|---------|----------------|
| Shader Graph | Window > Package Manager > Shader Graph > Install |
| Memory Profiler | Window > Package Manager > Memory Profiler > Install |
| Input System | Window > Package Manager > Input System > Install |
| Multiplayer Playmode | Window > Package Manager > Multiplayer Playmode > Install |

After installing, restart Unity Editor and MCP server.

## Best Practices

1. **Always save scenes** after significant changes using `unity_save_scene`
2. **Use pagination** for large hierarchies to avoid response size limits
3. **Check compilation status** with `unity_get_compilation_errors` after creating/modifying scripts
4. **Stop play mode** before making structural changes to scenes
5. **Use batch operations** (e.g., `unity_batch_wire_references`) for efficiency
6. **Enable deep profiling selectively** — it has performance overhead
7. **Verify object paths** with `unity_get_scene_hierarchy` before manipulation
8. **Use world space** for absolute positioning, local space for relative transforms
9. **Create materials before assignment** to avoid missing material warnings
10. **Test builds incrementally** — don't wait until final scene completion

## Resources

- **GitHub Repository**: https://github.com/AnkleBreaker-Studio/unity-mcp-server
- **Unity Plugin**: https://github.com/AnkleBreaker-Studio/unity-mcp-plugin
- **MCP Protocol**: https://modelcontextprotocol.io
- **AnkleBreaker Studio**: https://anklebreaker-consulting.com/en/projects/unity-mcp
