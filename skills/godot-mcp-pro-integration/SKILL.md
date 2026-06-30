---
name: godot-mcp-pro-integration
description: AI-powered Godot 4 development with 175 MCP tools for scene manipulation, animation, 3D, physics, shaders, and runtime testing via WebSocket connection
triggers:
  - help me build a Godot game with AI
  - connect Claude to Godot editor
  - create Godot scenes with AI assistance
  - automate Godot development tasks
  - set up MCP for Godot 4
  - add nodes to Godot scene programmatically
  - test Godot game with AI tools
  - simulate input in Godot game
---

# Godot MCP Pro Integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Godot MCP Pro connects AI assistants (Claude, Cursor, Copilot) to your Godot 4 editor through WebSocket, providing 175 tools for real-time scene manipulation, script editing, input simulation, runtime analysis, and more.

## Architecture

```
AI Assistant ←—MCP/stdio—→ Node.js Server ←—WebSocket:6505—→ Godot Editor Plugin
```

- **Real-time**: WebSocket connection for instant feedback
- **Editor Integration**: Full access to Godot's editor API and UndoRedo system
- **JSON-RPC 2.0**: Standard protocol with error codes and suggestions
- **Four Modes**: Full (175), 3D (103), Lite (84), Minimal (35) tools to fit client limits

## Installation

### 1. Install Godot Plugin (Free)

```bash
# Clone or download the free addon
git clone https://github.com/youichi-uda/godot-mcp-pro.git
```

Copy `addons/godot_mcp/` into your Godot project's `addons/` directory.

Enable in Godot: **Project → Project Settings → Plugins → Godot MCP Pro → Enable**

### 2. Install MCP Server (Paid)

The Node.js server is distributed separately as a paid package ($15 one-time):
- Buy Me a Coffee: https://buymeacoffee.com/y1uda/extras
- itch.io: https://y1uda.itch.io/godot-mcp-pro

After purchasing and extracting:

```bash
cd server
npm install
npm run build
```

### 3. Configure Your AI Client

#### Claude Code (.mcp.json)

```json
{
  "mcpServers": {
    "godot-mcp-pro": {
      "command": "node",
      "args": ["/absolute/path/to/server/build/index.js"],
      "env": {
        "GODOT_MCP_PORT": "6505"
      }
    }
  }
}
```

#### Cursor / VS Code Copilot

Full mode (175 tools) recommended — both support large tool counts.

#### Windsurf / JetBrains Junie (100 tool limit)

```json
{
  "mcpServers": {
    "godot-mcp-pro": {
      "command": "node",
      "args": ["/absolute/path/to/server/build/index.js", "--lite"]
    }
  }
}
```

#### OpenCode / Local LLMs

```json
{
  "mcpServers": {
    "godot-mcp-pro": {
      "command": "node",
      "args": ["/absolute/path/to/server/build/index.js", "--minimal"]
    }
  }
}
```

### 4. Start Development

1. Launch Godot editor with your project and plugin enabled
2. Start your AI assistant (e.g., Claude Code)
3. The MCP server auto-connects to Godot via WebSocket

## Tool Categories & Key Commands

### Project Management (7 tools)

```gdscript
# Get project info
get_project_info()
# Returns: {name, version, viewport_size, autoloads, main_scene}

# Search files
search_files(pattern: "*.gd", use_glob: true)

# Get filesystem tree
get_filesystem_tree(path: "res://", max_depth: 3, include_extensions: [".gd", ".tscn"])

# UID conversion
uid_to_project_path(uid: "uid://abc123")
project_path_to_uid(path: "res://scenes/player.tscn")
```

### Scene Manipulation (9 tools)

```gdscript
# Get scene tree
get_scene_tree()
# Returns hierarchical node structure with properties

# Create new scene
create_scene(
  path: "res://scenes/enemy.tscn",
  root_type: "CharacterBody3D"
)

# Open scene
open_scene(scene_path: "res://scenes/level_01.tscn")

# Save current scene
save_scene()

# Add scene instance
add_scene_instance(
  scene_path: "res://actors/player.tscn",
  parent_path: "Level/Entities",
  instance_name: "Player"
)

# Play/stop scene
play_scene(mode: "current")  # "main", "current", or custom path
stop_scene()
```

### Node Operations (17 tools)

```gdscript
# Add node
add_node(
  parent_path: "Player",
  node_type: "CollisionShape3D",
  node_name: "BodyCollision",
  properties: {
    "position": {"x": 0, "y": 1, "z": 0}
  }
)

# Update property (auto type parsing)
update_property(
  node_path: "Player",
  property: "speed",
  value: "10.0"  # Auto-converts to float
)

update_property(
  node_path: "Player/Sprite",
  property: "modulate",
  value: "#ff0000"  # Auto-converts to Color
)

# Get node properties
get_node_properties(
  node_path: "Player",
  include_readonly: false
)

# Move/reparent node
move_node(
  node_path: "Enemy",
  new_parent_path: "Level/Enemies",
  new_index: 0
)

# Duplicate node
duplicate_node(
  node_path: "Enemy",
  new_name: "Enemy2"
)

# Delete node
delete_node(node_path: "OldNode")

# Connect signal
connect_signal(
  source_path: "Button",
  signal_name: "pressed",
  target_path: "Game",
  method_name: "_on_button_pressed"
)

# Groups
set_node_groups(node_path: "Enemy", groups: ["enemies", "ai"])
find_nodes_in_group(group_name: "enemies")

# Selection
select_nodes(node_paths: ["Player", "Player/Camera3D"], focus_first: true)
get_editor_selection()
clear_editor_selection()
```

### Script Management (8 tools)

```gdscript
# List all scripts
list_scripts()
# Returns: [{path, class_name, extends, methods, signals}]

# Read script
read_script(script_path: "res://scripts/player.gd")

# Create script
create_script(
  path: "res://scripts/enemy.gd",
  template: "CharacterBody3D",
  class_name: "Enemy"
)

# Edit script (search/replace)
edit_script(
  script_path: "res://scripts/player.gd",
  mode: "search_replace",
  search: "var speed = 5.0",
  replace: "var speed = 10.0"
)

# Full replacement
edit_script(
  script_path: "res://scripts/player.gd",
  mode: "replace",
  content: "extends CharacterBody3D\n\nfunc _ready():\n\tpass"
)

# Attach script to node
attach_script(
  node_path: "Enemy",
  script_path: "res://scripts/enemy.gd"
)

# Validate syntax
validate_script(script_path: "res://scripts/player.gd")

# Search in files
search_in_files(
  pattern: "func _process",
  extensions: [".gd"],
  case_sensitive: false
)
```

### Runtime Testing (19 tools)

```gdscript
# Get running game scene tree
get_game_scene_tree()

# Get node properties in running game
get_game_node_properties(node_path: "/root/Level/Player")

# Set property in running game
set_game_node_property(
  node_path: "/root/Level/Player",
  property: "position",
  value: {"x": 100, "y": 200, "z": 0}
)

# Execute code in game context
execute_game_script(code: """
var player = get_node('/root/Level/Player')
player.health = 100
print('Player health reset')
""")

# Input simulation
simulate_key(key: "Space", pressed: true)
simulate_key(key: "Space", pressed: false)

simulate_mouse_click(
  position: {"x": 512, "y": 300},
  button: "left"
)

simulate_action(action: "jump", strength: 1.0)

simulate_sequence(events: [
  {"type": "key", "key": "Right", "pressed": true},
  {"type": "wait", "frames": 30},
  {"type": "key", "key": "Space", "pressed": true},
  {"type": "wait", "frames": 5},
  {"type": "key", "key": "Space", "pressed": false},
  {"type": "key", "key": "Right", "pressed": false}
])

# Capture frames
capture_frames(
  duration_frames: 60,
  interval_frames: 10,
  output_dir: "res://test_captures"
)

# Monitor properties
monitor_properties(
  node_path: "/root/Level/Player",
  properties: ["position", "velocity"],
  duration_frames: 120,
  interval_frames: 5
)

# Recording/replay
start_recording()
stop_recording()  # Returns recording data
replay_recording(recording_data: {...})

# Find nodes
find_nodes_by_script(script_path: "res://scripts/enemy.gd")
find_ui_elements(node_type: "Button")
find_nearby_nodes(
  position: {"x": 100, "y": 200, "z": 0},
  radius: 50.0,
  node_type: "Area3D"
)

# UI interaction
click_button_by_text(text: "Start Game")

# Wait for condition
wait_for_node(
  node_path: "/root/Level/Boss",
  timeout_seconds: 5.0
)
```

### Animation (6 tools)

```gdscript
# List animations
list_animations(player_path: "Player/AnimationPlayer")

# Create animation
create_animation(
  player_path: "Player/AnimationPlayer",
  animation_name: "run",
  length: 1.0,
  loop: true
)

# Add track
add_animation_track(
  player_path: "Player/AnimationPlayer",
  animation_name: "run",
  track_type: "value",  # "value", "position", "rotation", "method", "bezier"
  node_path: "Player/Sprite",
  property: "frame"
)

# Set keyframe
set_animation_keyframe(
  player_path: "Player/AnimationPlayer",
  animation_name: "run",
  track_index: 0,
  time: 0.0,
  value: 0
)

set_animation_keyframe(
  player_path: "Player/AnimationPlayer",
  animation_name: "run",
  track_index: 0,
  time: 1.0,
  value: 8
)

# Get animation info
get_animation_info(
  player_path: "Player/AnimationPlayer",
  animation_name: "run"
)

# Remove animation
remove_animation(
  player_path: "Player/AnimationPlayer",
  animation_name: "old_anim"
)
```

### 3D & Physics

```gdscript
# Add collision shape resource
add_resource(
  node_path: "Player/CollisionShape3D",
  resource_type: "BoxShape3D",
  properties: {
    "size": {"x": 1, "y": 2, "z": 1}
  }
)

# Add mesh resource
add_resource(
  node_path: "Enemy/MeshInstance3D",
  resource_type: "SphereMesh",
  properties: {
    "radius": 0.5,
    "height": 1.0
  }
)

# Set 3D node properties
update_property(
  node_path: "Player",
  property: "position",
  value: {"x": 0, "y": 0, "z": 0}
)

update_property(
  node_path: "Player",
  property: "rotation_degrees",
  value: {"x": 0, "y": 90, "z": 0}
)

# Add light
add_node(
  parent_path: "Level",
  node_type: "DirectionalLight3D",
  node_name: "Sun",
  properties: {
    "light_energy": 1.0,
    "rotation_degrees": {"x": -45, "y": 0, "z": 0}
  }
)

# Add camera
add_node(
  parent_path: "Player",
  node_type: "Camera3D",
  node_name: "Camera",
  properties: {
    "position": {"x": 0, "y": 2, "z": 5},
    "current": true
  }
)
```

### Shader Tools (6 tools)

```gdscript
# Create shader
create_shader(
  path: "res://shaders/water.gdshader",
  template: "canvas_item"  # "spatial", "canvas_item", "sky", "fog"
)

# Edit shader
edit_shader(
  shader_path: "res://shaders/water.gdshader",
  mode: "replace",
  content: """
shader_type canvas_item;

uniform float wave_speed : hint_range(0.0, 10.0) = 2.0;

void fragment() {
    COLOR = vec4(0.0, 0.5, 1.0, 1.0);
}
"""
)

# Assign shader material
assign_shader_material(
  node_path: "Water",
  shader_path: "res://shaders/water.gdshader"
)

# Set shader parameter
set_shader_param(
  node_path: "Water",
  param_name: "wave_speed",
  value: 3.5
)

# Get shader parameters
get_shader_params(node_path: "Water")
```

### TileMap (6 tools)

```gdscript
# Set single cell
tilemap_set_cell(
  tilemap_path: "Level/TileMapLayer",
  coords: {"x": 5, "y": 3},
  source_id: 0,
  atlas_coords: {"x": 2, "y": 1}
)

# Fill rectangle
tilemap_fill_rect(
  tilemap_path: "Level/TileMapLayer",
  start_coords: {"x": 0, "y": 0},
  end_coords: {"x": 10, "y": 10},
  source_id: 0,
  atlas_coords: {"x": 1, "y": 0}
)

# Get cell info
tilemap_get_cell(
  tilemap_path: "Level/TileMapLayer",
  coords: {"x": 5, "y": 3}
)

# Get used cells
tilemap_get_used_cells(tilemap_path: "Level/TileMapLayer")

# Clear all
tilemap_clear(tilemap_path: "Level/TileMapLayer")
```

### Editor Integration

```gdscript
# Get editor errors
get_editor_errors()
# Returns: [{type, message, file, line, stack_trace}]

# Execute editor script (runs in EditorScript context)
execute_editor_script(code: """
var editor = get_editor_interface()
print('Current scene: ', editor.get_edited_scene_root().name)
""")

# Capture editor viewport
get_editor_screenshot(
  output_path: "res://editor_screenshot.png",
  viewport: "3D"  # "2D", "3D", "Script"
)

# Capture running game
get_game_screenshot(output_path: "res://game_screenshot.png")

# Clear output panel
clear_output()

# Get output log
get_output_log(max_lines: 100)

# Reload plugin (auto-reconnects)
reload_plugin()

# Reload project (rescan filesystem)
reload_project()

# Get signals
get_signals(node_path: "Player")
```

## Common Patterns

### Creating a Character Controller

```gdscript
# 1. Create scene
create_scene(
  path: "res://actors/player.tscn",
  root_type: "CharacterBody3D"
)

open_scene(scene_path: "res://actors/player.tscn")

# 2. Add collision
add_node(
  parent_path: ".",
  node_type: "CollisionShape3D",
  node_name: "Collision"
)

add_resource(
  node_path: "Collision",
  resource_type: "CapsuleShape3D",
  properties: {"radius": 0.5, "height": 2.0}
)

# 3. Add mesh
add_node(
  parent_path: ".",
  node_type: "MeshInstance3D",
  node_name: "Mesh"
)

add_resource(
  node_path: "Mesh",
  resource_type: "CapsuleMesh",
  properties: {"radius": 0.5, "height": 2.0}
)

# 4. Add camera
add_node(
  parent_path: ".",
  node_type: "Camera3D",
  node_name: "Camera",
  properties: {
    "position": {"x": 0, "y": 1.6, "z": 0}
  }
)

# 5. Attach script
create_script(
  path: "res://scripts/player.gd",
  template: "CharacterBody3D",
  class_name: "Player"
)

attach_script(
  node_path: ".",
  script_path: "res://scripts/player.gd"
)

# 6. Edit script
edit_script(
  script_path: "res://scripts/player.gd",
  mode: "replace",
  content: """
extends CharacterBody3D

const SPEED = 5.0
const JUMP_VELOCITY = 4.5

var gravity = ProjectSettings.get_setting("physics/3d/default_gravity")

func _physics_process(delta):
    if not is_on_floor():
        velocity.y -= gravity * delta
    
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = JUMP_VELOCITY
    
    var input_dir = Input.get_vector("left", "right", "forward", "back")
    var direction = (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()
    
    if direction:
        velocity.x = direction.x * SPEED
        velocity.z = direction.z * SPEED
    else:
        velocity.x = move_toward(velocity.x, 0, SPEED)
        velocity.z = move_toward(velocity.z, 0, SPEED)
    
    move_and_slide()
"""
)

save_scene()
```

### Automated Testing Workflow

```gdscript
# 1. Start game
play_scene(mode: "main")

# Wait for scene to load
wait_for_node(node_path: "/root/Level", timeout_seconds: 3.0)

# 2. Execute test setup
execute_game_script(code: """
var player = get_node('/root/Level/Player')
player.position = Vector3(0, 1, 0)
player.health = 100
""")

# 3. Record input sequence
start_recording()

simulate_sequence(events: [
  {"type": "key", "key": "Right", "pressed": true},
  {"type": "wait", "frames": 60},
  {"type": "key", "key": "Space", "pressed": true},
  {"type": "wait", "frames": 5},
  {"type": "key", "key": "Space", "pressed": false},
  {"type": "wait", "frames": 30},
  {"type": "key", "key": "Right", "pressed": false}
])

var recording = stop_recording()

# 4. Monitor result
monitor_properties(
  node_path: "/root/Level/Player",
  properties: ["position", "velocity", "is_on_floor"],
  duration_frames: 120,
  interval_frames: 10
)

# 5. Capture evidence
capture_frames(
  duration_frames: 120,
  interval_frames: 20,
  output_dir: "res://test_results"
)

# 6. Stop game
stop_scene()

# 7. Verify results
get_output_log(max_lines: 50)
get_editor_errors()
```

### Batch Refactoring

```gdscript
# Find all CharacterBody3D nodes
find_nodes_by_type(node_type: "CharacterBody3D")

# Set gravity scale on all
batch_set_property(
  node_type: "CharacterBody3D",
  property: "motion_mode",
  value: 0  # MOTION_MODE_GROUNDED
)

# Find signal connections
find_signal_connections(scene_path: "res://scenes/level.tscn")

# Find script usage
find_script_references(resource_path: "res://scripts/old_player.gd")

# Cross-scene property update
cross_scene_set_property(
  node_type: "DirectionalLight3D",
  property: "light_energy",
  value: 1.5,
  scene_pattern: "res://scenes/*.tscn"
)

# Detect circular dependencies
detect_circular_dependencies()
```

## CLI Mode (Alternative to MCP)

For clients without MCP support or to minimize context overhead:

```bash
# Top-level help
node /path/to/server/build/cli.js --help

# Group help
node /path/to/server/build/cli.js scene --help
node /path/to/server/build/cli.js node --help

# Execute commands
node /path/to/server/build/cli.js project info
node /path/to/server/build/cli.js scene play --mode current
node /path/to/server/build/cli.js node add --type CharacterBody3D --name Player --parent .
node /path/to/server/build/cli.js script create --path res://player.gd --template CharacterBody3D
node /path/to/server/build/cli.js input simulate-key --key Space --pressed true
```

CLI requires:
- Godot editor running with plugin enabled
- Server built (`npm run build`)
- Available port 6510-6514

## Troubleshooting

### WebSocket Connection Fails

```
Error: Could not connect to Godot (tried ports 6505-6509)
```

**Solutions:**
1. Verify Godot editor is running with plugin enabled
2. Check plugin status: **Project → Project Settings → Plugins**
3. Restart Godot editor
4. Check if port 6505 is blocked by firewall
5. Try reloading plugin: `reload_plugin()`

### "Tool not available" in Lite/Minimal mode

Different modes expose different tool sets:

- **Full** (175): All tools
- **3D** (103): 3D-focused subset (excludes 2D-specific)
- **Lite** (84): Essential tools (no profiling, advanced batch operations)
- **Minimal** (35): Core CRUD only

Switch modes in your MCP config:

```json
"args": ["/path/to/server/build/index.js", "--lite"]
```

### Type Conversion Errors

`update_property` auto-parses strings to correct types:

```gdscript
# ✓ Correct
update_property(node_path: "Player", property: "speed", value: "10.0")
update_property(node_path: "Sprite", property: "modulate", value: "#ff0000")
update_property(node_path: "Node", property: "position", value: {"x": 1, "y": 2, "z": 3})

# ✗ Wrong (will fail)
update_property(node_path: "Player", property: "speed", value: 10)  # Pass as string
```

### Script Validation Fails

```gdscript
# Always validate before saving
validate_script(script_path: "res://scripts/player.gd")

# Check editor errors
get_editor_errors()
```

### Runtime Property Changes Don't Persist

Runtime changes via `set_game_node_property` only affect the running game instance. To persist:

```gdscript
# 1. Change in running game (for testing)
set_game_node_property(
  node_path: "/root/Level/Player",
  property: "speed",
  value: 15.0
)

# 2. Stop game
stop_scene()

# 3. Update scene file
update_property(
  node_path: "Player",
  property: "speed",
  value: "15.0"
)

save_scene()
```

### Node Path Not Found

Always use full paths from scene root:

```gdscript
# ✓ Correct
update_property(node_path: "Level/Entities/Player", ...)

# ✗ Wrong
update_property(node_path: "Player", ...)  # If not at root

# Get current tree structure
get_scene_tree()
```

### Performance Issues with Capture/Monitor

```gdscript
# Reduce frequency
capture_frames(
  duration_frames: 60,
  interval_frames: 30  # Every 30 frames instead of 10
)

monitor_properties(
  node_path: "/root/Player",
  properties: ["position"],  # Fewer properties
  duration_frames: 60,
  interval_frames: 15
)
```

## Environment Variables

```bash
# WebSocket port (default: 6505)
GODOT_MCP_PORT=6505

# Server timeout (default: 5000ms)
GODOT_MCP_TIMEOUT=5000

# Debug logging
GODOT_MCP_DEBUG=true
```

## Client Compatibility Reference

| Client | Mode | Tool Limit | Notes |
|--------|------|-----------|-------|
| Claude Code | Full | Unlimited | Deferred tool loading |
| VS Code Copilot | Full | Unlimited | Virtual Tools grouping |
| Cursor | Full | Unlimited | Dynamic Context Discovery |
| Cline | Full | Unlimited | Use `enabledTools` for whitelist |
| Roo Code | Full | Unlimited | No hard limit |
| Windsurf | Lite | 100 | Hard limit |
| JetBrains Junie | Lite | 100 | Hard limit |
| Gemini CLI | Lite | ~100 | Use `excludeTools` to tune |
| Antigravity | 3D | 100 | 3D-focused subset |
| OpenCode | Minimal/CLI | ~40 | Model degrades past 40 |
| LM Studio | Minimal/CLI | Varies | Context is bottleneck |

## Additional Resources

- Homepage: https://godot-mcp.abyo.net/
- Purchase: https://buymeacoffee.com/y1uda/extras or https://y1uda.itch.io/godot-mcp-pro
- GitHub (free plugin only): https://github.com/youichi-uda/godot-mcp-pro
- Godot Documentation: https://docs.godotengine.org/
