---
name: godot-mcp-native
description: Control Godot Engine projects via MCP - create nodes, edit scripts, manage scenes, and debug games through AI assistants
triggers:
  - help me build a Godot scene
  - create a new node in my Godot project
  - analyze my GDScript code
  - modify my Godot scene structure
  - debug my Godot game
  - set up a camera in Godot
  - add a script to this node
  - inspect my Godot scene tree
---

# Godot MCP Native

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Godot MCP Native is an MCP server plugin for Godot Engine 4.x that enables AI assistants to directly read and modify Godot projects through natural language. It provides 154 tools for manipulating nodes, scripts, scenes, editor state, and runtime debugging—all implemented natively in GDScript without external dependencies.

## Installation

### Method 1: Asset Library (Recommended)

1. Open your Godot project
2. Go to **AssetLib** tab in the editor
3. Search for "Godot MCP Native"
4. Click **Download** then **Install**

### Method 2: Manual Installation

```bash
# Clone into your project's addons directory
cd your-godot-project
mkdir -p addons
cd addons
git clone https://github.com/yurineko73/Godot-MCP-Native.git godot_mcp
```

### Enable the Plugin

1. Open **Project > Project Settings > Plugins**
2. Find "Godot MCP Native" in the list
3. Set status to **Enable**

## Configuration

The plugin runs an HTTP MCP server on port 9080 by default. Configure in **Project > Project Settings > Godot MCP**:

```gdscript
# Default configuration
transport_mode = "http"  # HTTP mode for remote access
http_port = 9080         # Server port
auth_enabled = false     # Enable authentication
auth_token = ""          # Bearer token (if auth enabled)
```

### Connecting with Claude Desktop

Install `mcp-remote`:

```bash
npm install mcp-remote
```

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "godot-mcp": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://localhost:9080/mcp"
      ]
    }
  }
}
```

### Connecting with Cursor / Trae

Add to `.cursor/mcp.json` or `.trae/mcp.json`:

```json
{
  "mcpServers": {
    "godot-mcp": {
      "url": "http://localhost:9080/mcp"
    }
  }
}
```

With authentication:

```json
{
  "mcpServers": {
    "godot-mcp": {
      "url": "http://localhost:9080/mcp",
      "headers": {
        "Authorization": "Bearer ${GODOT_MCP_TOKEN}"
      }
    }
  }
}
```

### Connecting with Cline

Add to MCP settings:

```json
{
  "mcpServers": {
    "godot-mcp": {
      "url": "http://localhost:9080/mcp",
      "type": "streamableHttp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### Connecting with OpenCode

```json
{
  "mcp": {
    "godot-mcp": {
      "type": "remote",
      "url": "http://localhost:9080/mcp"
    }
  }
}
```

### Connecting with Codex

```toml
[mcp_servers]

[mcp_servers.godot-mcp]
type = "streamableHttp"
url = "http://localhost:9080/mcp"
```

## Core Tools & Usage

### Node Operations

#### Get Scene Tree

```
@mcp godot-mcp get-scene-tree

Show me the current scene structure
```

Returns hierarchical scene tree with node types and properties.

#### Create Node

```
@mcp godot-mcp create-node

Create a Node3D called "Player" as a child of the root node
```

Parameters:
- `node_name`: Name for the new node
- `node_type`: Godot class (e.g. "Node3D", "Sprite2D", "CharacterBody3D")
- `parent_path`: Parent node path (e.g. ".", "/root/Main")

#### Update Node Property

```
@mcp godot-mcp update-node-property

Set the Player's position to (0, 5, 0)
```

Parameters:
- `node_path`: Path to the node (e.g. "Player", "/root/Main/Player")
- `property_name`: Property to modify (e.g. "position", "rotation", "scale")
- `property_value`: New value as JSON (e.g. `{"x": 0, "y": 5, "z": 0}`)

#### Delete Node

```
@mcp godot-mcp delete-node

Remove the Enemy node from the scene
```

#### Duplicate Node

```
@mcp godot-mcp duplicate-node

Duplicate the Coin node and place the copy under Collectibles
```

Parameters:
- `node_path`: Source node to duplicate
- `new_name`: Name for the duplicate
- `new_parent_path`: Optional new parent

#### Move Node

```
@mcp godot-mcp move-node

Move the UI node to be a child of CanvasLayer
```

#### Rename Node

```
@mcp godot-mcp rename-node

Rename "Player1" to "MainPlayer"
```

### Script Operations

#### List Project Scripts

```
@mcp godot-mcp list-project-scripts

Show me all GDScript files in the project
```

#### Read Script

```
@mcp godot-mcp read-script

Read the player.gd script
```

Parameters:
- `script_path`: Path to script (e.g. "res://scripts/player.gd")

#### Modify Script

```
@mcp godot-mcp modify-script

Update the player movement script to include jump logic
```

Parameters:
- `script_path`: Path to script
- `content`: New script content

#### Create Script

```
@mcp godot-mcp create-script

Create a new script res://scripts/enemy.gd
```

Parameters:
- `script_path`: Path for new script
- `content`: Script content
- `base_class`: Optional base class (default: "Node")

#### Attach Script

```
@mcp godot-mcp attach-script

Attach res://scripts/player.gd to the Player node
```

Parameters:
- `node_path`: Target node
- `script_path`: Script to attach

#### Execute Script

```
@mcp godot-mcp execute-script

Execute: print(get_tree().root.get_child_count())
```

Executes GDScript expressions in the editor context.

#### Analyze Script

```
@mcp godot-mcp analyze-script

Analyze the structure of player.gd
```

Returns classes, functions, variables, signals, and dependencies.

#### Validate Script

```
@mcp godot-mcp validate-script

Check if enemy.gd has syntax errors
```

### Scene Operations

#### Create Scene

```
@mcp godot-mcp create-scene

Create a new scene with a CharacterBody3D root node at res://scenes/player.tscn
```

Parameters:
- `scene_path`: Path for new scene
- `root_node_type`: Root node class

#### Save Scene

```
@mcp godot-mcp save-scene

Save the current scene
```

#### Open Scene

```
@mcp godot-mcp open-scene

Open res://scenes/main_menu.tscn
```

#### Get Current Scene

```
@mcp godot-mcp get-current-scene

What scene am I editing?
```

#### List Project Scenes

```
@mcp godot-mcp list-project-scenes

Show all scene files in the project
```

### Editor Operations

#### Get Editor State

```
@mcp godot-mcp get-editor-state

What's the current editor status?
```

Returns active scene, selected nodes, running project state.

#### Run Project

```
@mcp godot-mcp run-project

Run the game
```

#### Stop Project

```
@mcp godot-mcp stop-project

Stop the running game
```

#### Get Editor Screenshot

```
@mcp godot-mcp get-editor-screenshot

Take a screenshot of the 3D viewport
```

Parameters:
- `viewport_index`: Which viewport (0 = 3D, 1 = 2D, 2 = Script)

#### Select Node

```
@mcp godot-mcp select-node

Select the Player node and show it in the Inspector
```

#### Get Inspector Properties

```
@mcp godot-mcp get-inspector-properties

Show me all editable properties for the Camera3D node
```

### Advanced Node Tools

#### Set Anchor Preset

```
@mcp godot-mcp set-anchor-preset

Set the UI panel to use the full rect anchor preset
```

Parameters:
- `node_path`: Control node path
- `preset_name`: Preset name (e.g. "FullRect", "CenterTop", "BottomRight")

#### Connect Signal

```
@mcp godot-mcp connect-signal

Connect the Button's pressed signal to the Main node's _on_button_pressed method
```

Parameters:
- `signal_node_path`: Node emitting the signal
- `signal_name`: Signal name
- `target_node_path`: Node receiving the signal
- `method_name`: Method to call

#### Batch Update Properties

```
@mcp godot-mcp batch-update-node-properties

Set multiple properties on Player: position (0,0,0), rotation (0,0,0), scale (1,1,1)
```

Parameters:
- `node_path`: Target node
- `properties`: Dictionary of property names to values

### Debug & Runtime Tools

#### Get Editor Logs

```
@mcp godot-mcp get-editor-logs

Show me the last 50 log entries
```

#### Debug Print

```
@mcp godot-mcp debug-print

Print debug message: "Player health initialized"
```

#### Get Performance Metrics

```
@mcp godot-mcp get-performance-metrics

Show current FPS and memory usage
```

#### Install Runtime Probe

```
@mcp godot-mcp install-runtime-probe

Add the MCP runtime probe to the current scene
```

Installs a bridge node that enables runtime inspection and modification.

#### Get Runtime Scene Tree

```
@mcp godot-mcp get-runtime-scene-tree

Show the live scene tree of the running game
```

#### Inspect Runtime Node

```
@mcp godot-mcp inspect-runtime-node

Inspect the Player node in the running game
```

Parameters:
- `node_path`: Runtime node path

#### Update Runtime Node Property

```
@mcp godot-mcp update-runtime-node-property

Set the player's health to 100 in the running game
```

Parameters:
- `node_path`: Runtime node path
- `property_name`: Property to modify
- `property_value`: New value

#### Evaluate Runtime Expression

```
@mcp godot-mcp evaluate-runtime-expression

Evaluate: get_tree().paused = true
```

Executes GDScript in the running game context.

### Debugging Sessions

#### Get Debugger Sessions

```
@mcp godot-mcp get-debugger-sessions

Show active debugger sessions
```

#### Set Debugger Breakpoint

```
@mcp godot-mcp set-debugger-breakpoint

Set a breakpoint at player.gd:45
```

Parameters:
- `script_path`: Script file
- `line`: Line number
- `enabled`: true/false

#### Get Debug Stack Frames

```
@mcp godot-mcp get-debug-stack-frames

Show the call stack when breaked
```

#### Get Debug Stack Variables

```
@mcp godot-mcp get-debug-stack-variables

Show local variables for frame 0
```

Parameters:
- `frame_index`: Stack frame index

#### Debug Step Over / Into / Out / Continue

```
@mcp godot-mcp debug-step-over

Step over the current line
```

```
@mcp godot-mcp debug-continue

Resume execution
```

## Common Patterns

### Creating a 3D Player Character

```
1. Create the character body:
   @mcp godot-mcp create-node
   Create a CharacterBody3D called "Player" as a child of root

2. Add a mesh:
   @mcp godot-mcp create-node
   Create a MeshInstance3D called "PlayerMesh" as a child of Player
   
   @mcp godot-mcp update-node-property
   Set PlayerMesh's mesh to a new BoxMesh

3. Add collision:
   @mcp godot-mcp create-node
   Create a CollisionShape3D called "Collision" as a child of Player
   
   @mcp godot-mcp add-resource
   Add a BoxShape3D resource to the Collision node

4. Create and attach script:
   @mcp godot-mcp create-script
   Create res://scripts/player.gd with movement logic
   
   @mcp godot-mcp attach-script
   Attach the player script to the Player node
```

### Building a UI Menu

```
1. Create CanvasLayer container:
   @mcp godot-mcp create-node
   Create a CanvasLayer called "UI" as a child of root

2. Add panel:
   @mcp godot-mcp create-node
   Create a Panel called "MenuPanel" as a child of UI
   
   @mcp godot-mcp set-anchor-preset
   Set MenuPanel to CenterTop preset

3. Add buttons:
   @mcp godot-mcp create-node
   Create a Button called "PlayButton" as a child of MenuPanel
   
   @mcp godot-mcp update-node-property
   Set PlayButton's text to "Play Game"

4. Connect signals:
   @mcp godot-mcp connect-signal
   Connect PlayButton's pressed signal to UI's _on_play_pressed method
```

### Debugging Runtime Issues

```
1. Install probe:
   @mcp godot-mcp install-runtime-probe
   Add the MCP runtime probe to the scene

2. Run game:
   @mcp godot-mcp run-project
   Start the game

3. Inspect runtime state:
   @mcp godot-mcp get-runtime-scene-tree
   Show live scene structure
   
   @mcp godot-mcp inspect-runtime-node
   Inspect the Player node's current property values

4. Modify at runtime:
   @mcp godot-mcp update-runtime-node-property
   Set player health to 50 in the running game

5. Evaluate expressions:
   @mcp godot-mcp evaluate-runtime-expression
   Execute: print(get_tree().get_nodes_in_group("enemies").size())
```

### Scene Inheritance & Instancing

```
1. Audit scene structure:
   @mcp godot-mcp audit-scene-inheritance
   Show inherited/instanced nodes in the current scene

2. Check node persistence:
   @mcp godot-mcp audit-scene-node-persistence
   Verify node ownership and save state
```

### Batch Scene Edits

```
@mcp godot-mcp batch-scene-node-edits

Apply multiple edits in one undo action:
- Create a Node3D called "Enemies"
- Create 3 CharacterBody3D children
- Delete the old "Enemy" node
```

## Troubleshooting

### Server Not Responding

- Verify Godot project is open with plugin enabled
- Check server is running: look for "MCP Server started on port 9080" in Output
- Ensure port 9080 is not blocked by firewall
- Test with: `curl http://localhost:9080/mcp`

### Authentication Errors

```gdscript
# In Project Settings > Godot MCP
auth_enabled = true
auth_token = "your-secret-token"
```

Then in MCP client config:

```json
{
  "headers": {
    "Authorization": "Bearer your-secret-token"
  }
}
```

### Node Paths Not Found

- Use `get-scene-tree` to verify exact node paths
- Paths are relative to scene root unless starting with `/root/`
- Example: `"Player"` not `"/Player"` for root children

### Script Execution Errors

- Use `validate-script` before `modify-script`
- Check editor Output panel for GDScript errors
- Ensure script paths start with `res://`

### Runtime Probe Not Working

- Install probe BEFORE running project: `install-runtime-probe`
- Verify probe exists: `get-scene-tree` should show "MCPRuntimeProbe"
- Remove and reinstall if corrupted: `remove-runtime-probe` then `install-runtime-probe`

### Property Updates Failing

- Check property name spelling (case-sensitive)
- Use `get-node-properties` or `get-inspector-properties` to see available properties
- Ensure value type matches property (Vector3 for position, String for text, etc.)
- Complex types use JSON: `{"x": 1, "y": 2, "z": 3}` for Vector3

### Export/Build Issues

```
@mcp godot-mcp list-export-presets
Show configured export presets

@mcp godot-mcp validate-export-preset
Check if "Linux/X11" preset is valid

@mcp godot-mcp inspect-export-templates
Verify export templates are installed
```

## Resource URI Schemes

The plugin uses custom URI schemes for resource addressing:

- `godot://script/current` - Currently editing script
- `godot://scene/current` - Currently editing scene
- `res://path/to/file.gd` - Project resource paths

Example:

```
@mcp godot-mcp read godot://script/current

Show me the script I'm currently editing
```
