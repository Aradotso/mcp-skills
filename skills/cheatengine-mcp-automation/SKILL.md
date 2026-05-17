---
name: cheatengine-mcp-automation
description: Automate Cheat Engine memory analysis, reverse engineering, and debugging using AI through MCP protocol
triggers:
  - analyze game memory with cheat engine
  - find pointers and memory structures
  - automate reverse engineering with AI
  - scan memory for values using cheat engine
  - set breakpoints and trace execution
  - disassemble functions in target process
  - create game trainer or mod
  - analyze packet structures in memory
---

# Cheat Engine MCP Automation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill enables AI agents to control Cheat Engine through the Model Context Protocol, automating memory analysis, pointer scanning, structure dissection, and reverse engineering tasks that normally take days or weeks.

## What It Does

Connect Claude, Cursor, or any MCP-compatible AI to Cheat Engine for:
- **Memory scanning**: Find values (int, float, string, AOB patterns) across gigabytes instantly
- **Pointer chain resolution**: Follow `[[base+0x10]+0x20]+0x8` paths automatically
- **Structure analysis**: Auto-detect field types and offsets in memory structures
- **Code disassembly**: Analyze functions, find references, identify C++ RTTI classes
- **Invisible debugging**: Hardware breakpoints + Ring -1 DBVM tracing
- **Code injection**: Allocate memory, inject DLLs, execute shellcode
- **Process automation**: Attach, pause, resume, create processes programmatically

## Installation

### Prerequisites
- **Windows only** (uses Named Pipes)
- Cheat Engine 7.5+ installed
- Python 3.10+

### 1. Install Python Dependencies

```bash
cd MCP_Server
pip install -r requirements.txt
```

Or manually:
```bash
pip install mcp pywin32
```

### 2. Configure MCP Server

Add to your MCP client config (location varies by client):

**Claude Desktop** (`~/AppData/Roaming/Claude/claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "cheatengine": {
      "command": "python",
      "args": ["C:/path/to/cheatengine-mcp-bridge/MCP_Server/mcp_cheatengine.py"]
    }
  }
}
```

**Cursor** (`.cursor/mcp.json` in project):
```json
{
  "servers": {
    "cheatengine": {
      "command": "python",
      "args": ["C:/absolute/path/to/MCP_Server/mcp_cheatengine.py"]
    }
  }
}
```

**Codex** (`~/.codex/config.toml`):
```toml
[mcp_servers.cheatengine]
command = "python"
args = ['C:\path\to\cheatengine-mcp-bridge\MCP_Server\mcp_cheatengine.py']
```

### 3. Load Bridge in Cheat Engine

Open Cheat Engine, then load the Lua bridge:

**Option A: Execute Script (Recommended)**
1. Go to `File` → `Execute Script`
2. Select `MCP_Server/ce_mcp_bridge.lua`
3. Click `Execute`

**Option B: Lua Engine**
1. Open `Table` → `Show Cheat Table Lua Script`
2. Paste this line:
```lua
dofile([[C:\path\to\cheatengine-mcp-bridge\MCP_Server\ce_mcp_bridge.lua]])
```
3. Execute

Look for confirmation:
```
[MCP v12.0.0] MCP Server Listening on: CE_MCP_Bridge_v99
```

### 4. Verify Connection

Restart your MCP client, then use the `ping` tool:
```
User: "Ping cheat engine"
```

Expected response:
```json
{
  "success": true,
  "version": "12.0.0",
  "message": "CE MCP Bridge Active",
  "process_id": 0
}
```

## Critical Configuration

### Prevent BSODs

**MUST DO**: Disable this setting to prevent `CLOCK_WATCHDOG_TIMEOUT` blue screens:

1. Cheat Engine → `Edit` → `Settings`
2. Go to `Extra` tab
3. **UNCHECK** "Query memory region routines"
4. Restart Cheat Engine

This prevents conflicts between memory scanning and DBVM/anti-cheat systems.

### Enable DBVM (Optional)

For Ring -1 debugging and invisible breakpoints:
1. Cheat Engine → `Kernel Tools` → `DBVM`
2. Follow prompts to install hypervisor
3. Reboot system
4. DBVM functions (`start_dbvm_watch`, etc.) now available

## Core MCP Tools

### Process Management

```lua
-- List running processes
get_process_list()
-- Returns: [{pid, name, windowTitle}, ...]

-- Attach to process
open_process({process_name = "game.exe"})
-- Or by PID: open_process({process_id = 1234})

-- Get current process info
get_process_info()
-- Returns: {name, pid, base_address, entry_point}

-- Launch new process under CE control
create_process({path = "C:/game/game.exe", parameters = "--debug"})

-- Pause/resume execution
pause_process()
unpause_process()
```

### Memory Reading

```lua
-- Read typed values
read_integer({address = "0x400000", size = 4})  -- 4-byte int
read_float({address = "0x400000", double = true})  -- 8-byte double
read_string({address = "0x400000", length = 100})  -- Read 100 chars
read_bytes({address = "0x400000", size = 16})  -- Raw bytes

-- Follow pointer chains
read_pointer_chain({
  base = "game.exe",
  offsets = [0x123456, 0x10, 0x20, 0x8]
})
-- Resolves: [[game.exe+0x123456]+0x10]+0x20]+0x8

-- Read structures
read_memory({address = "0x400000", size = 256})
```

### Memory Scanning

```lua
-- Scan for specific value
scan_all({
  value = "15000",
  value_type = 4,  -- 4-byte int
  writable = true
})
-- Returns: [{address, value}, ...]

-- Next scan (filter previous results)
scan_all({
  value = "15100",
  value_type = 4,
  next_scan = true
})

-- AOB (Array of Bytes) pattern scanning
aob_scan({
  pattern = "48 8B 05 ?? ?? ?? ?? 48 85 C0",
  writable = false,
  executable = true
})

-- Find what writes to address
find_what_writes({address = "0x400000"})
-- Returns: [{instruction, address}, ...]
```

### Code Analysis

```lua
-- Disassemble function
disassemble({
  address = "0x401000",
  count = 20  -- Instructions to disassemble
})

-- Analyze function structure
analyze_function({address = "0x401000"})
-- Returns: {prologue, epilogue, calls, jumps, stack_size}

-- Find all references to address
find_references({address = "0x500000"})

-- Find all CALL instructions to function
find_call_references({address = "0x401000"})

-- Get C++ class name via RTTI
get_rtti_classname({address = "0x600000"})
-- Returns: "CPlayerInventory"
```

### Structure Dissection

```lua
-- Auto-analyze memory structure
dissect_structure({
  address = "0x500000",
  size = 512,
  name = "PlayerData"
})
-- Returns: [
--   {offset: 0x00, type: "ptr", name: "vtable", value: "0x1234"},
--   {offset: 0x08, type: "int", name: "health", value: "100"},
--   {offset: 0x0C, type: "float", name: "x_pos", value: "123.45"}
-- ]
```

### Debugging & Breakpoints

```lua
-- Hardware breakpoint (execution)
set_breakpoint({
  address = "0x401000",
  type = 1  -- 1=execute, 2=write, 3=access
})

-- Data breakpoint (watch memory)
set_data_breakpoint({
  address = "0x500000",
  size = 4,
  on_write = true
})

-- DBVM invisible tracing (Ring -1)
start_dbvm_watch({
  address = "0x401000",
  watch_writes = true
})

-- Remove breakpoint
remove_breakpoint({address = "0x401000"})
```

### Code Injection

```lua
-- Allocate memory in target
allocate_memory({size = 4096, near = "game.exe"})
-- Returns: {address: "0x10000000"}

-- Inject DLL
inject_dll({dll_path = "C:/mods/mymod.dll"})

-- Assemble instruction
assemble_instruction({
  instruction = "mov rax, [rbx+0x10]",
  address = "0x400000"  -- For relative addressing
})
-- Returns: {bytes: "48 8B 43 10"}

-- Execute shellcode
execute_code({
  code = "90 90 C3",  -- nop nop ret
  address = "0x10000000"
})

-- Generate API hook template
generate_api_hook_script({
  function_address = "0x401000",
  function_name = "ProcessPacket"
})
```

### Symbol Management

```lua
-- Register named symbol
register_symbol({
  name = "PlayerBase",
  address = "0x500000"
})

-- Use in other commands
read_pointer_chain({
  base = "PlayerBase",
  offsets = [0x10, 0x20]
})

-- Get symbol info
get_symbol_info({name = "PlayerBase"})

-- Enable Windows PDB symbols
enable_windows_symbols()
```

## Common Patterns

### Finding Player Health

```
User: "Find player health, currently at 100"

AI workflow:
1. scan_all({value: "100", value_type: 4})
2. User changes health to 95
3. scan_all({value: "95", value_type: 4, next_scan: true})
4. find_what_writes({address: first_result})
5. disassemble({address: write_instruction})
6. analyze_function({address: function_start})
```

### Tracing Packet Encryption

```
User: "Find where network packets are encrypted"

AI workflow:
1. aob_scan({pattern: "E8 ?? ?? ?? ?? 48 8B", executable: true})
   // Common CALL pattern before crypto
2. For each result:
   - disassemble({address: result, count: 30})
   - find_call_references({address: called_function})
3. set_breakpoint({address: suspect_function, type: 1})
4. start_dbvm_watch({address: suspect_function})
```

### Understanding C++ Object

```
User: "What is the object at [[game.exe+0x123456]+0x10]?"

AI workflow:
1. read_pointer_chain({base: "game.exe", offsets: [0x123456, 0x10]})
2. get_rtti_classname({address: result})  // "CPlayerInventory"
3. dissect_structure({address: result, size: 512, name: "CPlayerInventory"})
4. For each pointer field:
   - read_pointer_chain({base: result, offsets: [field_offset]})
   - get_rtti_classname({address: pointer_value})
```

### Creating Update-Proof AOB

```
User: "Find a unique pattern for the health function"

AI workflow:
1. find_what_writes({address: health_address})
2. disassemble({address: write_instruction, count: 50})
3. Analyze for unique opcodes (avoid relative offsets)
4. aob_scan({pattern: "48 8B 05 ?? ?? ?? ?? 48 85 C0 74 ?? 8B 40 08"})
5. Verify single result across full memory space
```

### Automating Pointer Path Discovery

```lua
-- Example: Find pointer path from base to target
User: "Find the pointer chain from game.exe to 0x12345678"

AI workflow:
1. get_process_info()  // Get base address
2. scan_all({value: "0x12345678", value_type: 8})  // Find pointers to target
3. For each pointer result:
   - scan_all({value: pointer_result, value_type: 8})
4. Build chain: base -> ptr1 -> ptr2 -> target
5. register_symbol({name: "TargetPath", address: "[[game.exe+0xABC]+0x10]+0x20"})
```

## Real Example: Complete Game Trainer Workflow

```
User: "Create a god mode for MyGame.exe"

AI session:
───────────────────────────────────────────────────
You: Attach to MyGame.exe
AI: open_process({process_name: "MyGame.exe"})
    ✓ Attached to PID 5432, base: 0x140000000

You: Find my health, currently 100
AI: scan_all({value: "100", value_type: 4})
    → Found 2,847 results

You: Took damage, now at 87
AI: scan_all({value: "87", value_type: 4, next_scan: true})
    → Filtered to 3 addresses

You: What writes to the first one?
AI: find_what_writes({address: "0x20A3B4C0"})
    → MyGame.exe+0x4F2A10 - mov [rax+08],edx

You: Disassemble that function
AI: disassemble({address: "0x1404F2A00", count: 40})
    → Found: Function processes damage, compares against max health at rax+0C

You: What's the full structure?
AI: dissect_structure({address: "0x20A3B4C0", size: 256})
    → +00: vtable
    → +08: current_health (int) = 87
    → +0C: max_health (int) = 100
    → +10: shield (int) = 0
    → +14: invulnerable (byte) = 0

You: Create a cheat to set invulnerable to 1
AI: 1. register_symbol({name: "PlayerHealth", address: "0x20A3B4C0"})
    2. Generated script:
       [ENABLE]
       PlayerHealth+14:
         db 01
       [DISABLE]
       PlayerHealth+14:
         db 00

You: Make it work after game updates
AI: aob_scan({pattern: "48 8B 05 ?? ?? ?? ?? 48 85 C0 74 ?? 89 50 08"})
    → Found unique pattern at MyGame.exe+0x4F2A0E
    Updated script with AOB injection point
```

## Environment Variables

Set these before starting the MCP server:

```bash
# Increase timeout for complex operations (default: 30s)
set CE_MCP_TIMEOUT=60

# Enable shell commands (SECURITY RISK - only for trusted automation)
set CE_MCP_ALLOW_SHELL=1
```

## Troubleshooting

### "Too many local variables" error in Cheat Engine

**Solution**: Use `dofile()` instead of pasting the script:
```lua
dofile([[C:\path\to\ce_mcp_bridge.lua]])
```

The bridge uses global functions to avoid Lua's 200 local variable limit.

### MCP client can't connect

**Check:**
1. Cheat Engine shows `MCP Server Listening on: CE_MCP_Bridge_v99`
2. MCP client restarted after config change
3. Python path in config is absolute (not relative)
4. `pip install mcp pywin32` completed successfully
5. Run `ping` tool - should return `success: true`

### `process_id: 0` in ping response

**Normal** - means no process attached yet. Use `open_process()` first.

### BSOD (CLOCK_WATCHDOG_TIMEOUT)

**Cause**: "Query memory region routines" enabled + DBVM conflict

**Fix**: 
1. Cheat Engine → Settings → Extra
2. **UNCHECK** "Query memory region routines"
3. Restart Cheat Engine

### Commands return `"CE not attached"`

**Solution**: Attach to a process first:
```lua
open_process({process_name: "target.exe"})
```

### DBVM functions fail

**Check:**
1. DBVM installed: Cheat Engine → Kernel Tools → DBVM
2. System rebooted after DBVM install
3. No conflicting hypervisors (Hyper-V, VMware)

## Testing Your Setup

Run the test suite to verify all ~180 tools work:

```bash
cd MCP_Server
python test_mcp.py
```

Expected output:
```
✅ Memory Reading: 6/6 tests passed
✅ Process Info: 4/4 tests passed  
✅ Code Analysis: 8/8 tests passed
✅ Breakpoints: 4/4 tests passed
✅ DBVM Functions: 3/3 tests passed
✅ Utility Commands: 11/11 tests passed
────────────────────────────────────
Total: 36/37 PASSED (100% success)
```

## Key Value Types Reference

```lua
-- value_type parameter for scanning:
1  = byte (1 byte)
2  = 2 bytes
4  = 4 bytes (int32)
8  = 8 bytes (int64)
5  = float (4 bytes)
6  = double (8 bytes)
7  = string
8  = pointer (4 or 8 bytes depending on process)
9  = AOB (array of bytes)
```

## Architecture Overview

```
AI Agent (Claude/Cursor/Codex)
    ↓ JSON-RPC over stdio
Python MCP Server (mcp_cheatengine.py)
    ↓ Named Pipe: \\.\pipe\CE_MCP_Bridge_v99
Lua Bridge (ce_mcp_bridge.lua)
    ↓ Cheat Engine API
Target Process Memory
```

The Python server translates MCP protocol to pipe commands, the Lua bridge executes them using Cheat Engine's API, and results flow back through the same chain.

## Security Note

This tool enables **arbitrary code execution** in target processes. Only use on:
- Single-player games you own
- Your own software for testing
- CTF/research environments
- Educational reverse engineering

**Never** use on:
- Multiplayer games (violates ToS, ruins others' experience)
- Software you don't have authorization to modify
- Production systems
