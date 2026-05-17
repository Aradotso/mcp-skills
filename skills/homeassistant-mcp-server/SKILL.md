---
name: homeassistant-mcp-server
description: Control Home Assistant smart home devices through AI assistants using the Model Context Protocol
triggers:
  - control my home assistant devices
  - set up home assistant mcp server
  - automate my smart home with ai
  - connect claude to home assistant
  - use mcp with home assistant
  - control lights and climate through ai
  - query home assistant device states
  - create home assistant automations with ai
---

# Home Assistant MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection

## Overview

The Home Assistant MCP Server enables AI assistants (Claude, GPT-4, Cursor, etc.) to control smart home devices through the Model Context Protocol. It provides 40 tools covering lights, climate, covers, locks, media players, vacuums, and more, with BM25 tool search to keep context lean and MCP resources for read-only data access.

Built with Python and FastMCP, it uses async operations with TTL-based caching to minimize API load.

## Installation

### uvx (Recommended - No Installation Required)

Configure your MCP client with:

```json
{
  "mcpServers": {
    "homeassistant": {
      "command": "uvx",
      "args": ["homeassistant-mcp"],
      "env": {
        "HASS_HOST": "http://homeassistant.local:8123",
        "HASS_TOKEN": "your_long_lived_access_token_here"
      }
    }
  }
}
```

### pip Installation

```bash
pip install homeassistant-mcp
```

Run with:
```bash
export HASS_HOST="http://homeassistant.local:8123"
export HASS_TOKEN="your_token_here"
homeassistant-mcp
```

### From Source

```bash
git clone https://github.com/robbrad/homeassistant-mcp.git
cd homeassistant-mcp
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -e ".[dev]"
homeassistant-mcp
```

## Configuration

### Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `HASS_HOST` | Home Assistant URL | `http://homeassistant.local:8123` |
| `HASS_TOKEN` | Long-lived access token | Generate from HA profile page |

### Optional Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CACHE_TTL_STATES` | `30` | Cache TTL for bulk state queries (seconds) |
| `CACHE_TTL_ENTITY` | `10` | Cache TTL for individual entity queries (seconds) |
| `LOG_LEVEL` | `INFO` | Logging level (DEBUG, INFO, WARNING, ERROR) |

### Claude Desktop Configuration

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`  
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "homeassistant": {
      "command": "uvx",
      "args": ["homeassistant-mcp"],
      "env": {
        "HASS_HOST": "http://homeassistant.local:8123",
        "HASS_TOKEN": "eyJ0eXAiOiJKV1QiLCJhbGc..."
      }
    }
  }
}
```

### Cursor Configuration

`.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "homeassistant": {
      "command": "uvx",
      "args": ["homeassistant-mcp"],
      "env": {
        "HASS_HOST": "${HASS_HOST}",
        "HASS_TOKEN": "${HASS_TOKEN}"
      }
    }
  }
}
```

## Tool Discovery System

The server exposes 8 core tools for tool discovery instead of all 40 upfront:

### Core Discovery Tools

```python
# 1. Discover all available tools
discover_tools()
# Returns: Full catalog of 40 tools organized by category

# 2. Search for specific tools by description
search_tools(query="lights brightness")
# Returns: BM25-ranked tool schemas matching query

# 3. Call a discovered tool by name
call_tool(tool_name="lights_control", arguments={...})
# Executes the specified tool with given arguments
```

### Example Tool Discovery Flow

```python
# Step 1: Discover what tools are available
tools = discover_tools()
# Returns categories: Device Control, Automation & Scenes, Input Helpers, etc.

# Step 2: Search for specific functionality
results = search_tools(query="control lights and brightness")
# Returns: lights_control schema with full parameters

# Step 3: Use the tool directly
lights_control(
    action="turn_on",
    entity_id="light.living_room",
    brightness=80,
    color_temp=370
)
```

## Key Tools by Category

### Device Control Tools (16 total)

#### Light Control

```python
# Turn on lights with brightness and color
lights_control(
    action="turn_on",
    entity_id="light.bedroom",
    brightness=100,  # 0-100
    rgb_color=[255, 0, 0]  # Red
)

# Set color temperature
lights_control(
    action="turn_on",
    entity_id="light.kitchen",
    color_temp=370  # Warm white (153-500 mireds)
)

# Turn off
lights_control(action="turn_off", entity_id="light.bedroom")

# Bulk operations
lights_control(
    action="turn_on",
    entity_id="light.all_lights",
    brightness=50
)
```

#### Climate Control

```python
# Set temperature and mode
climate_control(
    action="set_temperature",
    entity_id="climate.living_room",
    temperature=22,  # Celsius
    hvac_mode="heat"  # Options: heat, cool, auto, off, heat_cool
)

# Set fan mode
climate_control(
    action="set_fan_mode",
    entity_id="climate.bedroom",
    fan_mode="auto"  # Options: auto, low, medium, high
)

# Get current state
climate_control(action="get", entity_id="climate.living_room")
```

#### Cover Control

```python
# Set position
cover_control(
    action="set_position",
    entity_id="cover.garage_door",
    position=50  # 0-100 (0=closed, 100=open)
)

# Open/close
cover_control(action="open", entity_id="cover.living_room_blinds")
cover_control(action="close", entity_id="cover.bedroom_blinds")

# Set tilt
cover_control(
    action="set_tilt",
    entity_id="cover.venetian_blinds",
    tilt=45
)
```

#### Lock Control

```python
# Lock/unlock
lock_control(action="lock", entity_id="lock.front_door")
lock_control(action="unlock", entity_id="lock.back_door", code="1234")

# Get state
lock_control(action="get", entity_id="lock.front_door")
```

#### Media Player Control

```python
# Play/pause/stop
media_player_control(action="play", entity_id="media_player.living_room_tv")
media_player_control(action="pause", entity_id="media_player.spotify")

# Volume control
media_player_control(
    action="volume_set",
    entity_id="media_player.speakers",
    volume=0.5  # 0.0-1.0
)

# Select source
media_player_control(
    action="select_source",
    entity_id="media_player.receiver",
    source="HDMI 1"
)
```

#### Vacuum Control

```python
# Start cleaning
vacuum_control(action="start", entity_id="vacuum.roborock")

# Return to dock
vacuum_control(action="dock", entity_id="vacuum.roborock")

# Set fan speed
vacuum_control(
    action="set_fan_speed",
    entity_id="vacuum.roborock",
    fan_speed="turbo"
)
```

### State Management

```python
# List all entities
states_control(action="list")

# List entities by domain
states_control(action="list", domain="light")

# Get specific entity
states_control(action="get", entity_id="sensor.temperature")

# Set entity state
states_control(
    action="set",
    entity_id="sensor.custom_sensor",
    state="active",
    attributes={"battery": 95}
)

# Delete entity state
states_control(action="delete", entity_id="sensor.old_sensor")
```

### Device Discovery

```python
# List all devices
list_devices()

# Filter by domain
list_devices(domain="light")

# Filter by area
list_devices(area="living_room")

# Filter by floor
list_devices(floor="ground_floor")

# Combine filters
list_devices(domain="switch", area="bedroom")
```

### Service Calls

```python
# Call any Home Assistant service directly
call_service(
    domain="light",
    service="turn_on",
    service_data={
        "entity_id": "light.bedroom",
        "brightness": 80,
        "rgb_color": [255, 120, 0]
    }
)

# Notify service
call_service(
    domain="notify",
    service="mobile_app_phone",
    service_data={
        "message": "Motion detected in garage",
        "title": "Security Alert"
    }
)

# TTS service
call_service(
    domain="tts",
    service="google_translate_say",
    service_data={
        "entity_id": "media_player.kitchen_speaker",
        "message": "Dinner is ready"
    }
)
```

### Automation Control

```python
# List all automations
automation_control(action="list")

# Trigger automation
automation_control(action="trigger", automation_id="automation.morning_routine")

# Enable/disable
automation_control(action="enable", automation_id="automation.night_mode")
automation_control(action="disable", automation_id="automation.away_mode")

# Reload automations
automation_control(action="reload")
```

### Scene Control

```python
# List scenes
scene_control(action="list")

# Activate scene
scene_control(action="activate", scene_id="scene.movie_time")
```

### Script Control

```python
# List scripts
script_control(action="list")

# Execute script with variables
script_control(
    action="execute",
    script_id="script.notification_routine",
    variables={"message": "Door left open", "priority": "high"}
)

# Reload scripts
script_control(action="reload")
```

### Template Rendering

```python
# Render Jinja2 templates
template_render(
    template="{{ states('sensor.temperature') | float + 5 }}"
)

# Complex template with entity states
template_render(
    template="""
    {% set lights = states.light | selectattr('state', 'eq', 'on') | list %}
    {{ lights | length }} lights are currently on
    """
)
```

### History & Logbook

```python
# Get entity history (last N hours)
history_query(
    entity_id="sensor.temperature",
    hours=24
)

# Get logbook entries
logbook_query(
    entity_id="binary_sensor.front_door",
    hours=12
)

# Get error log summary (parsed and deduplicated)
error_log_get()
```

### Calendar Access

```python
# List all calendars
calendar_access(action="list")

# Get events for date range
calendar_access(
    action="get_events",
    calendar_id="calendar.family",
    start_date="2024-05-01",
    end_date="2024-05-31"
)
```

### Camera Operations

```python
# Get camera snapshot
camera_proxy_get(
    entity_id="camera.front_door",
    width=640,
    height=480
)

# Enable motion detection
camera_control(
    action="enable_motion_detection",
    entity_id="camera.garage"
)

# Take snapshot
camera_control(
    action="snapshot",
    entity_id="camera.backyard",
    filename="/config/www/snapshots/backyard.jpg"
)
```

## MCP Resources

Resources provide read-only access to Home Assistant data via URIs.

### Entity Resource

```python
# Access entity state and attributes
# URI: hass://entity/{entity_id}
hass://entity/light.living_room
# Returns: {"state": "on", "attributes": {"brightness": 100, ...}}
```

### Area Resource

```python
# Get all entities in an area (compact summaries)
# URI: hass://area/{area_id}
hass://area/living_room
# Returns: List of entities with state and key attributes
```

### Device Resource

```python
# Get all entities for a device (compact summaries)
# URI: hass://device/{device_id}
hass://device/abc123def456
# Returns: Device entities with states
```

### Services Resource

```python
# Get all available services organized by domain
# URI: hass://services
hass://services
# Returns: All service definitions with parameters
```

### Entity History Resource

```python
# Get entity history with pagination
# URI: hass://entity/{entity_id}/history?hours=24&limit=100
hass://entity/sensor.temperature/history?hours=24
# Returns: Paginated history with state changes
```

## Common Patterns

### Morning Routine Automation

```python
# 1. Discover automation tools
tools = discover_tools()

# 2. Create morning scene
scene_control(action="activate", scene_id="scene.morning")

# 3. Set climate
climate_control(
    action="set_temperature",
    entity_id="climate.bedroom",
    temperature=21,
    hvac_mode="heat"
)

# 4. Open blinds
cover_control(action="open", entity_id="cover.bedroom_blinds")

# 5. Turn on lights gradually
lights_control(
    action="turn_on",
    entity_id="light.bedroom",
    brightness=30,
    transition=60  # Seconds
)
```

### Security Check Routine

```python
# Check all locks
locks = list_devices(domain="lock")
for lock in locks:
    state = lock_control(action="get", entity_id=lock["entity_id"])
    if state["state"] == "unlocked":
        send_notification(
            message=f"{lock['name']} is unlocked!",
            title="Security Alert"
        )

# Check door sensors
doors = list_devices(domain="binary_sensor", area="entry")
for door in doors:
    if "door" in door["entity_id"]:
        state = states_control(action="get", entity_id=door["entity_id"])
        # Process door states
```

### Energy Optimization

```python
# Turn off unused devices
lights = states_control(action="list", domain="light")
for light in lights:
    if light["state"] == "on":
        # Check if anyone home using presence sensors
        presence = states_control(action="get", entity_id="binary_sensor.occupancy")
        if presence["state"] == "off":
            lights_control(action="turn_off", entity_id=light["entity_id"])

# Optimize climate
climate_control(
    action="set_temperature",
    entity_id="climate.all",
    temperature=19,  # Lower when away
    hvac_mode="heat"
)
```

### Media Room Scene

```python
# Dim lights
lights_control(
    action="turn_on",
    entity_id="light.media_room",
    brightness=10,
    rgb_color=[255, 100, 0]
)

# Close blinds
cover_control(action="close", entity_id="cover.media_room_blinds")

# Start media player
media_player_control(
    action="select_source",
    entity_id="media_player.tv",
    source="Netflix"
)
media_player_control(action="play", entity_id="media_player.tv")
```

### Conditional Device Control

```python
# Only turn on if temperature is low
temp_state = states_control(action="get", entity_id="sensor.temperature")
current_temp = float(temp_state["state"])

if current_temp < 18:
    climate_control(
        action="set_temperature",
        entity_id="climate.living_room",
        temperature=22,
        hvac_mode="heat"
    )

# Turn on lights only if dark
lux_state = states_control(action="get", entity_id="sensor.illuminance")
if float(lux_state["state"]) < 50:
    lights_control(action="turn_on", entity_id="light.all_lights")
```

## Development

### Running Tests

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run tests
pytest

# Run with coverage
pytest --cov=src --cov-report=html
```

### Code Quality

```bash
# Format code
black src/ tests/

# Lint
ruff check src/ tests/

# Type checking
mypy src/
```

### Local Development

```bash
# Clone and setup
git clone https://github.com/robbrad/homeassistant-mcp.git
cd homeassistant-mcp
python -m venv venv
source venv/bin/activate
pip install -e ".[dev]"

# Create .env file
cat > .env << EOF
HASS_HOST=http://homeassistant.local:8123
HASS_TOKEN=your_token_here
LOG_LEVEL=DEBUG
EOF

# Run server
homeassistant-mcp
```

### Adding New Tools

```python
from fastmcp import FastMCP

mcp = FastMCP("homeassistant")

@mcp.tool(
    description="Control custom device",
    tags=["device", "custom"],
    timeout=30.0
)
async def custom_device_control(
    action: str,
    entity_id: str,
    value: int = 0
) -> dict:
    """
    Control custom device
    
    Args:
        action: Action to perform (on, off, set)
        entity_id: Entity ID
        value: Optional value parameter
    """
    client = get_hass_client()
    return await client.call_service(
        domain="custom_domain",
        service=action,
        service_data={"entity_id": entity_id, "value": value}
    )
```

## Troubleshooting

### Connection Issues

```python
# Check API info
api_info()
# Returns HA version, loaded components, and connection status

# Verify host and token
# Ensure HASS_HOST includes protocol (http:// or https://)
# Verify token has not expired
```

### Entity Not Found

```python
# List all entities to find correct entity_id
all_entities = states_control(action="list")

# Search by domain
lights = states_control(action="list", domain="light")

# List devices in area
devices = list_devices(area="bedroom")
```

### Service Call Failures

```python
# List available services
services = services_control(action="list", domain="light")

# Check service schema
schema = services_control(action="get", domain="light", service="turn_on")

# Validate configuration
config_check()
```

### Tool Not Available

```python
# Discover all tools
all_tools = discover_tools()

# Search for specific functionality
matching_tools = search_tools(query="your search query")

# Call tool by name if not directly available
call_tool(tool_name="tool_name", arguments={...})
```

### Cache Issues

```bash
# Adjust cache TTL for more frequent updates
export CACHE_TTL_STATES=5  # 5 seconds instead of default 30
export CACHE_TTL_ENTITY=2  # 2 seconds instead of default 10

# Enable debug logging to see cache hits/misses
export LOG_LEVEL=DEBUG
```

### Error Log Analysis

```python
# Get parsed error log with deduplication
errors = error_log_get()
# Returns: Deduplicated errors with component counts and timestamps
```

### Authentication Errors

- Verify `HASS_TOKEN` is a long-lived access token (not temporary)
- Create new token: HA Profile → Long-Lived Access Tokens → Create Token
- Check token has required permissions for device domains

### Rate Limiting

The server uses TTL-based caching to reduce API load:
- Bulk state queries cached for 30s (configurable via `CACHE_TTL_STATES`)
- Individual entity queries cached for 10s (configurable via `CACHE_TTL_ENTITY`)
- Adjust cache TTLs based on your update frequency needs
