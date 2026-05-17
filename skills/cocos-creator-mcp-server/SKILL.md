---
name: cocos-creator-mcp-server
description: MCP server plugin for Cocos Creator 3.8+ that enables AI assistants to control the editor through 50 powerful tools for scenes, nodes, components, prefabs, and assets.
triggers:
  - how do I control Cocos Creator with AI
  - integrate AI with Cocos Creator editor
  - automate Cocos Creator scene creation
  - manipulate Cocos Creator nodes programmatically
  - create prefabs in Cocos Creator with MCP
  - use AI to build Cocos Creator games
  - connect Claude to Cocos Creator
  - automate Cocos Creator workflows
---

# Cocos Creator MCP Server Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A comprehensive MCP (Model Context Protocol) server plugin for Cocos Creator 3.8+ that enables AI assistants to interact with the Cocos Creator editor through a standardized protocol. Provides 50 powerful tools covering 99% of editor operations including scenes, nodes, components, prefabs, assets, project management, debugging, and preferences.

## Installation

### Prerequisites

- Cocos Creator 3.8.6 or higher
- MCP-compatible client (Claude Desktop, Claude CLI, Cursor, etc.)

### Plugin Installation

1. **From Cocos Store** (Recommended):
   - Visit https://store.cocos.com/app/detail/7941
   - Click install and follow prompts in Cocos Creator

2. **Manual Installation**:
   ```bash
   # Clone the repository
   git clone https://github.com/DaxianLee/cocos-mcp-server.git
   
   # Install in Cocos Creator extensions folder
   # Windows: %USERPROFILE%/.CocosCreator/extensions/
   # macOS: ~/.CocosCreator/extensions/
   # Copy the plugin folder to the extensions directory
   ```

3. **Enable in Cocos Creator**:
   - Open Cocos Creator
   - Go to **Extensions → Extension Manager**
   - Find "Cocos Creator MCP Server" and enable it
   - Configure server port (default: 3000) in the MCP panel

### Client Configuration

**Claude Desktop** (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "cocos-creator": {
      "type": "http",
      "url": "http://127.0.0.1:3000/mcp"
    }
  }
}
```

**Claude CLI**:
```bash
claude mcp add --transport http cocos-creator http://127.0.0.1:3000/mcp
```

**Cursor** (`.cursor/mcp.json` or settings):
```json
{
  "mcpServers": {
    "cocos-creator": {
      "url": "http://localhost:3000/mcp"
    }
  }
}
```

## Core Concepts

### Action-Based Tool System

All 50 tools follow a unified "action + parameters" pattern:

```typescript
{
  "tool": "category_operation",
  "arguments": {
    "action": "specific_action",
    // ... action-specific parameters
  }
}
```

This design reduces token consumption by ~50% and increases AI call success rates.

### Tool Categories

- **scene_*** - Scene management and hierarchy
- **node_*** - Node lifecycle, transforms, and hierarchy
- **component_*** - Component management and scripting
- **prefab_*** - Prefab browsing, creation, and instantiation
- **asset_*** - Asset management and analysis
- **project_*** - Project control and build system
- **debug_*** - Console, logs, and system debugging
- **preferences_*** - Editor preferences
- **server_*** - Server information
- **broadcast_*** - Message broadcasting

## Key Tools & Usage

### Scene Management

#### Get Current Scene
```typescript
// Tool: scene_management
{
  "action": "get_current_scene"
}
// Returns: { uuid: string, name: string, path: string }
```

#### Open Scene
```typescript
{
  "action": "open_scene",
  "sceneUuid": "scene-uuid-here"
}
```

#### Create New Scene
```typescript
{
  "action": "create_scene",
  "name": "MyNewScene",
  "savePath": "db://assets/scenes/"
}
```

#### Save Current Scene
```typescript
{
  "action": "save_scene"
}
```

### Node Operations

#### Create Node
```typescript
// Tool: node_lifecycle
{
  "action": "create",
  "name": "PlayerNode",
  "parentUuid": "parent-node-uuid",  // Optional, defaults to scene root
  "nodeType": "2DNode",  // or "3DNode"
  "components": [  // Optional pre-installed components
    {
      "type": "cc.Sprite",
      "properties": {
        "spriteFrame": "texture-uuid"
      }
    }
  ]
}
```

#### Query Nodes
```typescript
// Tool: node_query
{
  "action": "find_by_name",
  "name": "Player",
  "exactMatch": false  // Use pattern matching
}

// Or find all nodes
{
  "action": "find_all",
  "includeComponents": true
}
```

#### Delete Node
```typescript
// Tool: node_lifecycle
{
  "action": "delete",
  "uuid": "node-uuid-to-delete"
}
```

#### Transform Node
```typescript
// Tool: node_transform
{
  "action": "set_property",
  "uuid": "node-uuid",
  "property": "position",
  "value": { "x": 100, "y": 200, "z": 0 }
}

// Set rotation
{
  "action": "set_property",
  "uuid": "node-uuid",
  "property": "rotation",
  "value": { "x": 0, "y": 0, "z": 45 }  // Euler angles
}

// Set scale
{
  "action": "set_property",
  "uuid": "node-uuid",
  "property": "scale",
  "value": { "x": 2, "y": 2, "z": 1 }
}
```

#### Move Node in Hierarchy
```typescript
// Tool: node_hierarchy
{
  "action": "move",
  "uuid": "node-uuid",
  "newParentUuid": "new-parent-uuid",
  "siblingIndex": 0  // Optional position among siblings
}
```

### Component Management

#### Add Engine Component
```typescript
// Tool: component_manage
{
  "action": "add",
  "nodeUuid": "node-uuid",
  "componentType": "cc.Sprite",
  "properties": {
    "spriteFrame": "texture-uuid",
    "sizeMode": 0
  }
}

// Common component types:
// cc.Sprite, cc.Label, cc.Button, cc.RichText,
// cc.UITransform, cc.Canvas, cc.Widget,
// cc.BoxCollider2D, cc.RigidBody2D, etc.
```

#### Attach Custom Script
```typescript
// Tool: component_script
{
  "action": "add_script",
  "nodeUuid": "node-uuid",
  "scriptName": "PlayerController",  // Script file name without .ts
  "properties": {
    "speed": 100,
    "jumpForce": 500
  }
}
```

#### Get Component Information
```typescript
// Tool: component_query
{
  "action": "get_components",
  "nodeUuid": "node-uuid"
}
// Returns array with type (cid) and properties for each component
```

#### Remove Component (IMPORTANT)
```typescript
// Tool: component_manage
// MUST use component's cid (type field), NOT script name!
// First, get component info:
{
  "action": "get_components",
  "nodeUuid": "node-uuid"
}
// Returns: [{ type: "comp.PlayerController!1234abcd", ... }]

// Then remove using exact type (cid):
{
  "action": "remove",
  "nodeUuid": "node-uuid",
  "componentType": "comp.PlayerController!1234abcd"  // Use exact cid
}
```

#### Set Component Properties
```typescript
// Tool: set_component_property
{
  "nodeUuid": "node-uuid",
  "componentType": "cc.Sprite",
  "properties": {
    "color": { "r": 255, "g": 0, "b": 0, "a": 255 }
  }
}

// For custom scripts, use full cid from get_components
{
  "nodeUuid": "node-uuid",
  "componentType": "comp.PlayerController!1234abcd",
  "properties": {
    "health": 100,
    "maxSpeed": 200
  }
}
```

### Prefab Operations

#### List Prefabs
```typescript
// Tool: prefab_browse
{
  "action": "list",
  "folderPath": "db://assets/prefabs/"  // Optional
}
```

#### Create Prefab from Node
```typescript
// Tool: prefab_lifecycle
{
  "action": "create",
  "nodeUuid": "source-node-uuid",
  "savePath": "db://assets/prefabs/MyPrefab.prefab"
}
```

#### Instantiate Prefab
```typescript
// Tool: prefab_instance
{
  "action": "instantiate",
  "prefabUuid": "prefab-uuid",
  "parentUuid": "parent-node-uuid",  // Optional
  "position": { "x": 0, "y": 0, "z": 0 }  // Optional
}
```

#### Apply Instance Changes to Prefab
```typescript
// Tool: prefab_instance
{
  "action": "apply",
  "nodeUuid": "prefab-instance-uuid"
}
```

#### Revert Instance to Original
```typescript
// Tool: prefab_instance
{
  "action": "revert",
  "nodeUuid": "prefab-instance-uuid"
}
```

### Asset Management

#### Import Assets
```typescript
// Tool: asset_manage
{
  "action": "import",
  "paths": [
    "/path/to/texture.png",
    "/path/to/audio.mp3"
  ]
}
```

#### Query Assets by Type
```typescript
// Tool: asset_query
{
  "action": "query_by_type",
  "type": "cc.Texture2D",  // cc.Texture2D, cc.SpriteFrame, cc.Prefab, cc.AudioClip, etc.
  "folder": "db://assets/textures/"  // Optional
}
```

#### Get Asset Dependencies
```typescript
// Tool: asset_analyze
{
  "action": "get_dependencies",
  "uuid": "asset-uuid"
}
```

#### Delete Asset
```typescript
// Tool: asset_operations
{
  "action": "delete",
  "uuid": "asset-uuid"
}
```

### Project Control

#### Run Project
```typescript
// Tool: project_manage
{
  "action": "run",
  "preview": true  // false for simulator
}
```

#### Build Project
```typescript
// Tool: project_build_system
{
  "action": "build",
  "platform": "web-mobile",  // web-mobile, android, ios, windows, mac
  "buildPath": "/path/to/build/output"
}
```

#### Get Project Info
```typescript
// Tool: project_manage
{
  "action": "get_info"
}
// Returns project name, version, path, settings
```

### Debugging

#### Get Console Logs
```typescript
// Tool: debug_console
{
  "action": "get_logs",
  "filter": "error",  // Optional: log, warn, error
  "limit": 50  // Optional
}
```

#### Clear Console
```typescript
// Tool: debug_console
{
  "action": "clear"
}
```

#### Search Log Files
```typescript
// Tool: debug_logs
{
  "action": "search",
  "pattern": "Error:",
  "maxLines": 100
}
```

## Common Patterns

### Creating a Complete Game Object

```typescript
// 1. Create node
const createNode = {
  tool: "node_lifecycle",
  arguments: {
    action: "create",
    name: "Player",
    nodeType: "2DNode"
  }
};
// Response includes: { uuid: "player-node-uuid" }

// 2. Add sprite component
const addSprite = {
  tool: "component_manage",
  arguments: {
    action: "add",
    nodeUuid: "player-node-uuid",
    componentType: "cc.Sprite",
    properties: {
      spriteFrame: "player-texture-uuid"
    }
  }
};

// 3. Attach script
const addScript = {
  tool: "component_script",
  arguments: {
    action: "add_script",
    nodeUuid: "player-node-uuid",
    scriptName: "PlayerController",
    properties: {
      speed: 300,
      health: 100
    }
  }
};

// 4. Position node
const setPosition = {
  tool: "node_transform",
  arguments: {
    action: "set_property",
    uuid: "player-node-uuid",
    property: "position",
    value: { x: 0, y: 0, z: 0 }
  }
};
```

### Working with Prefabs

```typescript
// 1. Find a node to convert
const findNode = {
  tool: "node_query",
  arguments: {
    action: "find_by_name",
    name: "Enemy"
  }
};

// 2. Create prefab from node
const createPrefab = {
  tool: "prefab_lifecycle",
  arguments: {
    action: "create",
    nodeUuid: "enemy-node-uuid",
    savePath: "db://assets/prefabs/Enemy.prefab"
  }
};

// 3. Instantiate multiple times
const spawn1 = {
  tool: "prefab_instance",
  arguments: {
    action: "instantiate",
    prefabUuid: "enemy-prefab-uuid",
    position: { x: 100, y: 0, z: 0 }
  }
};

const spawn2 = {
  tool: "prefab_instance",
  arguments: {
    action: "instantiate",
    prefabUuid: "enemy-prefab-uuid",
    position: { x: -100, y: 0, z: 0 }
  }
};
```

### Batch Node Operations

```typescript
// 1. Find all enemy nodes
const findEnemies = {
  tool: "node_query",
  arguments: {
    action: "find_by_name",
    name: "Enemy",
    exactMatch: false  // Pattern match
  }
};
// Returns: [{ uuid: "enemy1-uuid" }, { uuid: "enemy2-uuid" }, ...]

// 2. Delete all enemies
// For each enemy uuid:
const deleteEnemy = {
  tool: "node_lifecycle",
  arguments: {
    action: "delete",
    uuid: "enemy-uuid"
  }
};
```

### Scene Setup Workflow

```typescript
// 1. Create new scene
const newScene = {
  tool: "scene_management",
  arguments: {
    action: "create_scene",
    name: "Level1",
    savePath: "db://assets/scenes/"
  }
};

// 2. Create background node
const background = {
  tool: "node_lifecycle",
  arguments: {
    action: "create",
    name: "Background",
    nodeType: "2DNode",
    components: [{
      type: "cc.Sprite",
      properties: {
        spriteFrame: "bg-texture-uuid"
      }
    }]
  }
};

// 3. Create UI canvas
const canvas = {
  tool: "node_lifecycle",
  arguments: {
    action: "create",
    name: "Canvas",
    nodeType: "2DNode",
    components: [{
      type: "cc.Canvas"
    }]
  }
};

// 4. Save scene
const saveScene = {
  tool: "scene_management",
  arguments: {
    action: "save_scene"
  }
};
```

## Configuration

### Server Settings

Access via MCP panel in Cocos Creator:

- **Port**: Default 3000, customizable
- **Auto-start**: Enable to launch server with editor
- **Tool Management**: Selectively enable/disable specific tools

### Tool Configuration

The plugin saves tool configurations in editor preferences. Enable/disable tools through the MCP panel's tool management tab.

### Environment Variables

For external integrations, you can configure:

```bash
# MCP server endpoint
COCOS_MCP_URL=http://127.0.0.1:3000/mcp

# Project path (if automating multiple projects)
COCOS_PROJECT_PATH=/path/to/project
```

## Troubleshooting

### Server Won't Start

**Issue**: MCP server fails to start in Cocos Creator

**Solutions**:
1. Check port is not in use: `netstat -an | grep 3000`
2. Try different port in MCP panel settings
3. Restart Cocos Creator
4. Check extension is enabled in Extension Manager

### Tool Calls Failing

**Issue**: AI can't execute tools or gets errors

**Solutions**:
1. Verify server is running (check MCP panel status)
2. Ensure scene is open (many tools require active scene)
3. Check UUIDs are valid (use query tools first)
4. Enable debug logging in console to see detailed errors

### Component Removal Fails

**Issue**: Cannot remove component or script

**Solution**:
Always use the exact `cid` (type field) from `component_query`:

```typescript
// ✅ CORRECT
// First get component info
{ action: "get_components", nodeUuid: "node-uuid" }
// Returns: [{ type: "comp.MyScript!abc123", ... }]

// Then remove with exact cid
{
  action: "remove",
  nodeUuid: "node-uuid",
  componentType: "comp.MyScript!abc123"
}

// ❌ WRONG - Don't use script name
{
  action: "remove",
  nodeUuid: "node-uuid",
  componentType: "MyScript"  // Will fail!
}
```

### Prefab References Lost

**Issue**: Prefab loses component or asset references

**Solutions**:
- This was fixed in v1.4.0+
- Ensure using latest plugin version
- Internal references use `{"__id__": x}` format
- External references set to `null` (as expected)
- Asset references preserve UUID format

### Connection Issues

**Issue**: MCP client can't connect to server

**Solutions**:
1. Verify URL in client config matches server port
2. Check firewall isn't blocking localhost connections
3. Restart both Cocos Creator and MCP client
4. Test with: `curl http://127.0.0.1:3000/mcp`

### Performance Degradation

**Issue**: Editor becomes slow with MCP server

**Solutions**:
1. Disable unused tools in tool management
2. Reduce log levels in debug tools
3. Limit query result counts
4. Close unnecessary editor panels

### Cross-Platform Path Issues

**Issue**: Asset paths don't work on different OS

**Solution**:
Always use Cocos `db://` protocol for asset paths:
```typescript
// ✅ CORRECT - Cross-platform
"savePath": "db://assets/prefabs/MyPrefab.prefab"

// ❌ WRONG - Platform-specific
"savePath": "/Users/me/project/assets/prefabs/MyPrefab.prefab"
```

## Advanced Usage

### Custom Message Broadcasting

```typescript
// Listen for custom events
const listen = {
  tool: "broadcast_message",
  arguments: {
    action: "listen",
    channel: "game-events"
  }
};

// Send custom messages
const broadcast = {
  tool: "broadcast_message",
  arguments: {
    action: "broadcast",
    channel: "game-events",
    message: { event: "player_died", score: 100 }
  }
};
```

### Preferences Management

```typescript
// Get editor preferences
const getPrefs = {
  tool: "preferences_manage",
  arguments: {
    action: "get",
    key: "editor.grid.snap"
  }
};

// Set preferences
const setPrefs = {
  tool: "preferences_manage",
  arguments: {
    action: "set",
    key: "editor.grid.snap",
    value: true
  }
};
```

### System Information

```typescript
// Get detailed system info
const systemInfo = {
  tool: "debug_system",
  arguments: {
    action: "get_info"
  }
};
// Returns: editor version, OS, memory, performance stats
```

## Best Practices

1. **Always Query Before Modify**: Use query tools to get UUIDs before operations
2. **Use Exact CIDs for Components**: Never use script names for removal
3. **Save After Major Changes**: Call `save_scene` after structural changes
4. **Validate UUIDs**: Check UUIDs exist before referencing
5. **Use db:// Paths**: Always use Cocos protocol for cross-platform compatibility
6. **Enable Only Needed Tools**: Disable unused tools for better performance
7. **Test Prefabs After Creation**: Instantiate and verify before distributing
8. **Handle Errors Gracefully**: Check tool responses for success/failure

## Resources

- **GitHub**: https://github.com/DaxianLee/cocos-mcp-server
- **Cocos Store**: https://store.cocos.com/app/detail/7941
- **Video Tutorial**: https://www.bilibili.com/video/BV1mB8dzfEw8
- **Cocos Creator Docs**: https://docs.cocos.com/creator/3.8/manual/

## Version Compatibility

- **Plugin Version**: 1.5.0+ recommended (1.4.0+ on GitHub)
- **Cocos Creator**: 3.8.6 or higher
- **MCP Clients**: Claude Desktop, Claude CLI, Cursor, VS Code (with MCP extensions)
- **Node.js**: Not required (plugin runs in editor runtime)
