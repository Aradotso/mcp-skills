---
name: sts2mcp-slay-the-spire-2-ai-agent
description: Control and automate Slay the Spire 2 gameplay through REST API or MCP server for AI agents
triggers:
  - help me set up AI agent for Slay the Spire 2
  - how do I use the STS2MCP mod
  - connect AI to Slay the Spire 2
  - automate Slay the Spire 2 runs
  - integrate Claude with STS2
  - build an agent to play Slay the Spire
  - configure STS2MCP server
  - access Slay the Spire 2 game state
---

# STS2MCP: Slay the Spire 2 AI Agent Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

STS2MCP is a mod for Slay the Spire 2 that exposes game state and actions via a localhost REST API, with an optional MCP server for AI agent integration. It enables fully automated gameplay for singleplayer and multiplayer runs, including menu navigation, character selection, lobby control, and in-game decision making.

## What It Does

- **Game State Access**: Read current game state (cards, relics, potions, enemies, map, etc.)
- **Action Control**: Play cards, use potions, make choices, navigate menus
- **Profile Management**: Switch profiles, access compendium data, search wiki entries
- **Multiplayer Support**: Host/join co-op games, control both singleplayer and multiplayer flows
- **MCP Integration**: Built-in MCP server for seamless Claude Desktop/Code integration

## Installation

### 1. Install the Game Mod

Download the latest release from [GitHub](https://github.com/Gennadiyev/STS2MCP/releases/latest).

**Windows/Linux:**
```bash
# Copy to game mods directory
cp STS2_MCP.dll "<game_install>/mods/"
cp STS2_MCP.json "<game_install>/mods/"
```

**macOS:**
```bash
GAME_DIR="$HOME/Library/Application Support/Steam/steamapps/common/Slay the Spire 2"
MODS_DIR="$GAME_DIR/SlayTheSpire2.app/Contents/MacOS/mods"
mkdir -p "$MODS_DIR"
cp STS2_MCP.dll "$MODS_DIR/"
cp STS2_MCP.json "$MODS_DIR/"
```

Launch the game, enable mods in settings, and verify the server is running:

```bash
curl http://localhost:15526/
# Expected: {"message": "Hello from STS2 MCP v0.3.4", "status": "ok"}
```

### 2. Set Up the MCP Server (for Claude Integration)

Install [uv](https://docs.astral.sh/uv/) package manager:

```bash
# macOS
brew install uv

# Or use pip
pip install uv
```

Clone the repository and test the server:

```bash
git clone https://github.com/Gennadiyev/STS2MCP.git
cd STS2MCP
uv run --directory mcp python server.py --help
```

Add to your MCP configuration:

**Claude Code (`.mcp.json`):**
```json
{
  "mcpServers": {
    "sts2": {
      "command": "uv",
      "args": ["run", "--directory", "/absolute/path/to/STS2MCP/mcp", "python", "server.py"]
    }
  }
}
```

**Claude Desktop (`claude_desktop_config.json`):**
```json
{
  "mcpServers": {
    "sts2": {
      "command": "/opt/homebrew/bin/uv",
      "args": ["run", "--directory", "/absolute/path/to/STS2MCP/mcp", "python", "server.py"]
    }
  }
}
```

**Note:** On macOS GUI apps, use absolute path to `uv` (find with `which uv`).

Server options:
```bash
# Custom host/port
uv run --directory mcp python server.py --host 127.0.0.1 --port 8080

# Disable proxy environment variables
uv run --directory mcp python server.py --no-trust-env
```

## REST API Reference

Base URL: `http://localhost:15526`

### Core Endpoints

#### Get Game State
```bash
GET /api/v1/state
```

Returns comprehensive game state including:
- Current screen/room type
- Player deck, relics, potions, gold, HP
- Combat state (enemies, cards in hand, energy)
- Map state and available paths
- Rewards and choices

Example response structure:
```json
{
  "screen": "COMBAT",
  "room_type": "ELITE",
  "player": {
    "hp": 68,
    "max_hp": 80,
    "gold": 142,
    "deck": [...],
    "relics": [...],
    "potions": [...]
  },
  "combat": {
    "turn": 3,
    "energy": 3,
    "hand": [...],
    "enemies": [...]
  }
}
```

#### Execute Action
```bash
POST /api/v1/action
Content-Type: application/json

{
  "action": "play_card",
  "card_id": 123,
  "target_index": 0
}
```

Common actions:
- `play_card`: Play a card (requires `card_id`, optional `target_index`)
- `end_turn`: End combat turn
- `use_potion`: Use a potion (requires `potion_index`, optional `target_index`)
- `choose_option`: Make a choice (requires `option_index`)
- `proceed`: Click proceed/continue button
- `skip_card_reward`: Skip card reward screen
- `take_card_reward`: Take a card (requires `card_index`)
- `take_relic_reward`: Take a relic (requires `relic_index`)
- `take_gold_reward`: Take gold
- `upgrade_card`: Upgrade a card at rest site (requires `card_id`)
- `rest`: Rest at campfire
- `navigate_map`: Move on map (requires `node_index`)

### Profile & Compendium

#### Get Active Profile
```bash
GET /api/v1/profile
```

Returns profile progress: discoveries, achievements, character stats, run totals.

#### Get Compendium Data
```bash
GET /api/v1/compendium
```

Returns organized compendium data: Card Library, Relic Collection, Potion Lab, Bestiary, Character Stats, Run History (last 20 runs).

#### Search Wiki
```bash
GET /api/v1/wiki?query=strike&item_type=card&limit=5
```

Fuzzy search for discovered cards/relics. Parameters:
- `query`: Search term (required)
- `item_type`: Filter by `card` or `relic` (optional)
- `limit`: Max results (default 10)

#### List Profiles
```bash
GET /api/v1/profiles
```

#### Switch/Delete Profile
```bash
POST /api/v1/profiles
Content-Type: application/json

{
  "action": "switch",
  "profile_id": 2
}

# Or delete
{
  "action": "delete",
  "profile_id": 1
}
```

## MCP Server Functions

When using the MCP server with Claude, the following tools are available:

### Game State Functions

```python
# Get current game state
get_game_state()

# Execute action
execute_action(action_data: dict)
# Example: execute_action({"action": "play_card", "card_id": 42, "target_index": 1})
```

### Profile Functions

```python
# Get active profile data
get_profile()

# Get compendium (organized progress data)
get_compendium()

# Search wiki entries
search_wiki(query: str, item_type: str = None, limit: int = 10)
# Example: search_wiki("strike", item_type="card", limit=5)

# List all profiles
list_profiles()

# Switch profile
switch_profile(profile_id: int)

# Delete profile
delete_profile(profile_id: int)
```

## Building the Mod from Source

Requires [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet/9.0).

**Windows (PowerShell):**
```powershell
# Set game directory
$env:STS2_GAME_DIR = "D:\SteamLibrary\steamapps\common\Slay the Spire 2"
.\build.ps1
```

**macOS/Linux:**
```bash
# Install .NET 9
brew install dotnet@9  # macOS
export DOTNET_ROOT="/opt/homebrew/opt/dotnet@9/libexec"
export PATH="$DOTNET_ROOT:$PATH"

# Build
dotnet build STS2_MCP.csproj -c Release -o out/STS2_MCP \
  -p:STS2GameDir="$HOME/Library/Application Support/Steam/steamapps/common/Slay the Spire 2"

# Install
MODS_DIR="$HOME/Library/Application Support/Steam/steamapps/common/Slay the Spire 2/SlayTheSpire2.app/Contents/MacOS/mods"
mkdir -p "$MODS_DIR"
cp out/STS2_MCP/STS2_MCP.dll "$MODS_DIR/"
cp mod_manifest.json "$MODS_DIR/STS2_MCP.json"
```

## Common Usage Patterns

### Basic Agent Loop (Python)

```python
import requests
import time

BASE_URL = "http://localhost:15526"

def get_state():
    response = requests.get(f"{BASE_URL}/api/v1/state")
    return response.json()

def execute_action(action_data):
    response = requests.post(
        f"{BASE_URL}/api/v1/action",
        json=action_data,
        headers={"Content-Type": "application/json"}
    )
    return response.json()

def play_combat_turn(state):
    """Example: Play all playable cards on first enemy"""
    if state["screen"] != "COMBAT":
        return
    
    combat = state["combat"]
    hand = combat["hand"]
    energy = combat["energy"]
    
    for card in hand:
        if card["cost"] <= energy and card["playable"]:
            execute_action({
                "action": "play_card",
                "card_id": card["id"],
                "target_index": 0
            })
            time.sleep(0.5)  # Allow game to process
            
            # Refresh state
            state = get_state()
            energy = state["combat"]["energy"]
    
    # End turn when done
    execute_action({"action": "end_turn"})

# Main agent loop
while True:
    state = get_state()
    
    if state["screen"] == "COMBAT":
        play_combat_turn(state)
    elif state["screen"] == "CARD_REWARD":
        # Skip card rewards for simplicity
        execute_action({"action": "skip_card_reward"})
    elif state["screen"] == "MAP":
        # Take first available path
        execute_action({"action": "navigate_map", "node_index": 0})
    else:
        # Try to proceed
        execute_action({"action": "proceed"})
    
    time.sleep(1)
```

### Using MCP with Claude

When Claude has the MCP server connected, you can give natural language instructions:

```
"Start a new run with Ironclad and play through the first combat"

"Check my compendium and tell me which cards I've discovered"

"Search for all strike cards in the wiki"

"Look at the current game state and suggest the best card to play"
```

Claude will use the appropriate MCP tools to fulfill these requests.

### Profile Management Example

```python
import requests

BASE_URL = "http://localhost:15526"

def switch_to_profile(profile_id):
    """Switch to a specific profile"""
    response = requests.post(
        f"{BASE_URL}/api/v1/profiles",
        json={"action": "switch", "profile_id": profile_id}
    )
    return response.json()

def get_all_discovered_cards():
    """Get all discovered cards from compendium"""
    response = requests.get(f"{BASE_URL}/api/v1/compendium")
    compendium = response.json()
    return compendium.get("card_library", {}).get("discovered_cards", [])

def search_for_card(card_name):
    """Search wiki for specific card"""
    response = requests.get(
        f"{BASE_URL}/api/v1/wiki",
        params={"query": card_name, "item_type": "card", "limit": 1}
    )
    return response.json()

# Example usage
switch_to_profile(1)
cards = get_all_discovered_cards()
print(f"Discovered {len(cards)} cards")

strike_info = search_for_card("strike")
print(f"Strike card info: {strike_info}")
```

### Combat Decision Making

```python
def choose_best_target(state):
    """Example: Target enemy with lowest HP"""
    enemies = state["combat"]["enemies"]
    if not enemies:
        return None
    
    # Find enemy with lowest HP
    min_hp = float('inf')
    target_idx = 0
    
    for idx, enemy in enumerate(enemies):
        if enemy["hp"] < min_hp:
            min_hp = enemy["hp"]
            target_idx = idx
    
    return target_idx

def play_damage_cards(state):
    """Play damage-dealing cards on weakest enemy"""
    hand = state["combat"]["hand"]
    energy = state["combat"]["energy"]
    target_idx = choose_best_target(state)
    
    if target_idx is None:
        return
    
    for card in hand:
        if card["cost"] <= energy and card["playable"]:
            # Check if card deals damage (simplified)
            if "damage" in card.get("description", "").lower():
                execute_action({
                    "action": "play_card",
                    "card_id": card["id"],
                    "target_index": target_idx
                })
                time.sleep(0.5)
                state = get_state()
                energy = state["combat"]["energy"]
```

## Troubleshooting

### Mod Not Loading

1. **Check mod files**: Ensure `STS2_MCP.dll` and `STS2_MCP.json` are in the correct mods directory
2. **Enable mods**: Go to Settings → Mods in-game and enable mod support
3. **Check consent dialog**: Accept the mod consent dialog on first launch
4. **Verify server**: Run `curl http://localhost:15526/` to check if server is running

### Connection Refused

```bash
# Test if game is running and mod is loaded
curl http://localhost:15526/

# If failed, check game logs for errors
# Windows: <game_install>/logs/
# macOS: ~/Library/Application Support/Steam/steamapps/common/Slay the Spire 2/SlayTheSpire2.app/Contents/MacOS/logs/
```

### MCP Server Not Starting

1. **Check uv installation**: `uv --version`
2. **Use absolute paths**: GUI apps may not inherit shell PATH
3. **Test server manually**: `uv run --directory /path/to/STS2MCP/mcp python server.py`
4. **Check logs**: Claude Desktop logs are in `~/Library/Logs/Claude/` (macOS) or `%APPDATA%/Claude/logs/` (Windows)

### Actions Not Working

- **Wait between actions**: Add 0.5-1s delays to allow game to process
- **Verify game state**: Always refresh state after executing actions
- **Check action validity**: Ensure the action is valid for current screen (e.g., can't play cards outside combat)
- **Check IDs**: Card/potion IDs are instance-specific, refresh state to get current IDs

### Proxy Issues

If running in a containerized environment with proxy settings:

```bash
# Disable proxy for MCP server
uv run --directory mcp python server.py --no-trust-env
```

### Multiplayer Issues

Multiplayer support is in beta. If you encounter issues:

1. Disable the mod and verify issue persists
2. Report bugs only if they occur with mod disabled
3. Check that multiplayer actions are properly sequenced (host first, then client)

## Performance Notes

- **Token Usage**: A full Ironclad run with Claude Sonnet uses ~8M tokens (input + output + tool responses)
- **API Latency**: Local API calls are <10ms, but game animation/processing adds 200-500ms per action
- **Rate Limiting**: No built-in rate limiting, but game needs time to process actions
- **State Refresh**: Poll game state at 1Hz or less to avoid overwhelming the game

## License

MIT License - See repository for full text.
