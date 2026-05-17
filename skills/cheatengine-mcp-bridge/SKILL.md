---
name: cheatengine-mcp-bridge
description: Connect AI agents to Cheat Engine for automated memory analysis, reverse engineering, and debugging via MCP
triggers:
  - analyze game memory with cheat engine
  - automate reverse engineering with AI
  - scan process memory using natural language
  - find pointer chains in cheat engine
  - disassemble functions with AI assistance
  - create game trainer using cheat engine
  - debug memory structures with AI
  - automate pointer scanning and AOB patterns
---

# Cheat Engine MCP Bridge

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Connect Claude, Cursor, Copilot and other AI agents directly to Cheat Engine via the Model Context Protocol. Automate reverse engineering, memory analysis, pointer scanning, and debugging using natural language queries instead of manual clicking through hex dumps.

## What This Does

The Cheat Engine MCP Bridge exposes ~180 Cheat Engine functions as MCP tools, allowing AI agents to:

- Read/write memory (integers, floats, strings, pointers)
- Follow pointer chains: `[[base+0x10]+0x20]+0x8`
- Scan for values and AOB (Array of Bytes) patterns
- Disassemble and analyze functions
- Set hardware breakpoints and debug invisibly with DBVM (Ring -1 hypervisor)
- Identify C++ objects via RTTI
- Auto-analyze memory structures
- Generate update-resistant AOB signatures
- Inject DLLs and execute shellcode

**Architecture:**
```
AI Agent (Claude/Cursor) 
  ↕ MCP Protocol (JSON-RPC over stdio)
Python MCP Server (mcp_cheatengine.py)
  ↕ Named Pipe (async)
Cheat Engine Lua Bridge (ce_mcp_bridge.lua)
  ↕ CE API
Target Process Memory
```

## Installation

### Prerequisites

- **Windows only** (uses Named Pipes via `pywin32`)
- Cheat Engine 7.4+ installed
- Python 3.10+

### Python Dependencies

```bash
cd MCP_Server
pip install -r requirements.txt
```

Or manually:
```bash
pip install mcp pywin32
```

### Load Bridge in Cheat Engine

1. **Enable DBVM** (optional, for advanced debugging):
   - Cheat Engine → Settings → Kernel → Enable DBVM

2. **Critical: Disable Memory Query Routines** (prevents BSODs):
   - Settings → Extra → **UNCHECK** "Query memory region routines"

3. **Load the Lua bridge**:
   - Method A: `File` → `Execute Script` → browse to `MCP_Server/ce_mcp_bridge.lua` → `Execute`
   - Method B: `Table` → `Show Cheat Table Lua Script` → paste:

```lua
dofile([[C:\path\to\cheatengine-mcp-bridge\MCP_Server\ce_mcp_bridge.lua]])
```

**Verify:** Console shows `[MCP v12.0.0] MCP Server Listening on: CE_MCP_Bridge_v99`

### Configure MCP Client

Add to your MCP configuration file:

**Claude Desktop** (`~/.config/claude/claude_desktop_config.json`):
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

**Cursor** (`.cursorrules` or workspace settings):
```json
{
  "mcp": {
    "servers": {
      "cheatengine": {
        "command": "python",
        "args": ["C:/path/to/cheatengine-mcp-bridge/MCP_Server/mcp_cheatengine.py"]
      }
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

**Important:** Use forward slashes or escaped backslashes in JSON. Use single quotes in TOML.

Restart your IDE to load the MCP server.

## Core MCP Tools

### Process Management

**Attach to a process:**
```
User: "Attach to notepad.exe"
Agent uses: open_process
Args: {"process_name": "notepad.exe"}
```

**List running processes:**
```
Tool: get_process_list
Returns: [{"pid": 1234, "name": "game.exe"}, ...]
```

**Launch a new process:**
```
Tool: create_process
Args: {"path": "C:\\Games\\game.exe"}
```

**Get current process info:**
```
Tool: get_process_info
Returns: {"pid": 5678, "name": "game.exe", "base_address": "0x400000"}
```

### Memory Reading

**Read integer (4 bytes):**
```
Tool: read_integer
Args: {"address": "0x12345678"}
Returns: {"value": 15000}
```

**Read float:**
```
Tool: read_float
Args: {"address": "game.exe+0x1234", "is_double": false}
Returns: {"value": 100.5}
```

**Read string:**
```
Tool: read_string
Args: {"address": "0x400000", "length": 64}
Returns: {"value": "PlayerName"}
```

**Read pointer chain:**
```
Tool: read_pointer_chain
Args: {"base": "game.exe+0x1000", "offsets": [0x10, 0x20, 0x8]}
Returns: {"final_address": "0x789ABC", "value": 42}
```

**Read bytes:**
```
Tool: read_memory
Args: {"address": "0x400000", "size": 16}
Returns: {"hex": "4D5A90000300000004000000FFFF0000", "bytes": [77, 90, ...]}
```

### Memory Scanning

**Scan for value:**
```
Tool: scan_all
Args: {
  "value_type": "4byte",
  "scan_type": "exact",
  "value": "15000",
  "writable": true,
  "executable": false
}
Returns: {"count": 47, "addresses": ["0x123000", "0x456000", ...]}
```

**Next scan (filter results):**
```
Tool: next_scan
Args: {"value": "15100"}
Returns: {"count": 3, "addresses": ["0x123000", ...]}
```

**AOB (Array of Bytes) scan:**
```
Tool: aob_scan
Args: {
  "pattern": "48 8B 05 ?? ?? ?? ?? 48 85 C0",
  "writable": false,
  "executable": true
}
Returns: {"addresses": ["0x401000", "0x402500"]}
```

**Pointer scan:**
```
Tool: pointer_scan
Args: {
  "address": "0x789000",
  "max_level": 5,
  "max_offset": 4096
}
Returns: {"count": 12, "results": [{"base": "game.exe+0x1000", "offsets": [0x10, 0x8]}]}
```

### Code Analysis

**Disassemble:**
```
Tool: disassemble
Args: {"address": "0x401000", "count": 10}
Returns: {
  "instructions": [
    {"address": "0x401000", "bytes": "55", "disassembly": "push rbp"},
    {"address": "0x401001", "bytes": "4889E5", "disassembly": "mov rbp,rsp"}
  ]
}
```

**Analyze function:**
```
Tool: analyze_function
Args: {"address": "0x401000"}
Returns: {
  "prologue": "push rbp; mov rbp,rsp",
  "calls_count": 3,
  "references": ["0x402000", "0x403000"]
}
```

**Get RTTI class name:**
```
Tool: get_rtti_classname
Args: {"address": "0x500000"}
Returns: {"classname": "CPlayerInventory"}
```

**Find references to address:**
```
Tool: find_references
Args: {"address": "0x600000"}
Returns: {"count": 5, "references": ["0x401234", "0x402567"]}
```

### Structure Analysis

**Dissect structure:**
```
Tool: dissect_structure
Args: {"address": "0x500000", "size": 256}
Returns: {
  "fields": [
    {"offset": "0x00", "type": "vtable", "value": "0x401000"},
    {"offset": "0x08", "type": "int32", "value": 15},
    {"offset": "0x10", "type": "pointer", "value": "0x600000"}
  ]
}
```

### Debugging

**Set breakpoint:**
```
Tool: set_breakpoint
Args: {
  "address": "0x401000",
  "type": "hardware",
  "condition": "rax==5"
}
```

**Set data breakpoint (watch memory writes):**
```
Tool: set_data_breakpoint
Args: {"address": "0x789000", "size": 4, "type": "write"}
```

**Start DBVM watch (invisible debugging):**
```
Tool: start_dbvm_watch
Args: {"address": "0x401000"}
```

**What writes to address:**
```
Tool: what_writes
Args: {"address": "0x789000"}
```

**What accesses address:**
```
Tool: what_accesses
Args: {"address": "0x789000", "type": "read"}
```

### Memory Writing

**Write integer:**
```
Tool: write_integer
Args: {"address": "0x123000", "value": 9999, "size": 4}
```

**Write bytes:**
```
Tool: write_memory
Args: {"address": "0x401000", "bytes": [0x90, 0x90, 0x90]}
```

**Freeze value:**
```
Tool: freeze_address
Args: {"address": "0x123000", "value": 1000, "description": "Player Health"}
```

### Code Injection

**Inject DLL:**
```
Tool: inject_dll
Args: {"dll_path": "C:\\mods\\trainer.dll"}
```

**Execute shellcode:**
```
Tool: execute_code
Args: {
  "code": "mov rax, 1; ret",
  "address": "0x500000"
}
```

**Auto-assembler:**
```
Tool: auto_assemble
Args: {
  "script": "[ENABLE]\nalloc(hook,128)\nhook:\n  mov [health],#999\n  ret"
}
```

### Symbol Management

**Register symbol:**
```
Tool: register_symbol
Args: {"name": "PlayerHealth", "address": "0x789000"}
```

**Get symbol info:**
```
Tool: get_symbol_info
Args: {"symbol": "PlayerHealth"}
Returns: {"address": "0x789000", "type": "int32"}
```

**Enable Windows symbols (PDB):**
```
Tool: enable_windows_symbols
Args: {"enable": true}
```

### Cheat Table Operations

**Load cheat table:**
```
Tool: load_table
Args: {"path": "C:\\cheats\\game.CT"}
```

**Save cheat table:**
```
Tool: save_table
Args: {"path": "C:\\cheats\\backup.CT"}
```

**Get address list:**
```
Tool: get_address_list
Returns: [
  {"description": "Health", "address": "0x789000", "value": 100},
  {"description": "Gold", "address": "0x789100", "value": 5000}
]
```

## Common Workflows

### Finding a Dynamic Value

```
User: "Find my gold amount, it's currently 15000"

Agent workflow:
1. scan_all(value_type="4byte", value="15000")
   → Returns 47 addresses

User: "I bought something, gold is now 14750"

2. next_scan(value="14750")
   → Filters to 3 addresses

User: "What writes to the first one?"

3. set_data_breakpoint(address=results[0], type="write")
4. [User triggers gold change in game]
5. get_debug_info()
   → Returns instruction that modified gold

User: "Disassemble that function"

6. disassemble(address=breakpoint_address, count=50)
   → Shows full AddGold/SubtractGold logic
```

### Reverse Engineering a Pointer Chain

```
User: "Find the player's coordinates"

Agent workflow:
1. scan_all(value_type="float", value="125.5")  # Current X position
2. next_scan(value="126.3")  # After moving
3. pointer_scan(address=result_address, max_level=5)
   → Finds: [[game.exe+0x1234]+0x18]+0x30

User: "Verify that pointer is stable"

4. read_pointer_chain(base="game.exe+0x1234", offsets=[0x18, 0x30])
5. register_symbol(name="PlayerX", address=final_address)
```

### Creating an AOB Signature (Update-Proof)

```
User: "I found the health function at 0x401000, make it update-resistant"

Agent workflow:
1. disassemble(address="0x401000", count=20)
2. analyze_function(address="0x401000")
3. Identify unique byte pattern with wildcards:
   "48 8B 05 ?? ?? ?? ?? 48 85 C0 74 ?? 8B 40 ??"
4. aob_scan(pattern=generated_pattern)
   → Verify only 1 result
5. Returns AOB for use in trainer scripts
```

### Understanding a C++ Object

```
User: "What's at address 0x500000?"

Agent workflow:
1. get_rtti_classname(address="0x500000")
   → "CPlayerInventory"
2. dissect_structure(address="0x500000", size=256)
   → 0x00: vtable
   → 0x08: itemCount (int32) = 15
   → 0x10: itemArray (pointer) = 0x600000
3. read_pointer_chain(base="0x500000", offsets=[0x10])
4. dissect_structure(address="0x600000", size=64)
   → Array of CItem objects
```

## Configuration

### Environment Variables

Set before starting the MCP client:

```bash
# Timeout for MCP tool calls (default: 30 seconds)
set CE_MCP_TIMEOUT=60

# Enable shell execution tools (DANGEROUS - arbitrary code execution)
set CE_MCP_ALLOW_SHELL=1
```

### Lua Bridge Customization

Edit `ce_mcp_bridge.lua` to customize:

```lua
-- Change named pipe (if multiple instances needed)
local PIPE_NAME = "\\\\.\\pipe\\CE_MCP_Bridge_v99"

-- Adjust worker thread wait time
local WORKER_WAIT_MS = 10

-- Enable verbose logging
DEBUG_MODE = true
```

## Troubleshooting

### "Too many local variables" error

**Problem:** Cheat Engine Lua compiler limit (200 locals per chunk).

**Solution:** Use `dofile()` to load from disk instead of pasting script into cheat table:

```lua
dofile([[C:\path\to\ce_mcp_bridge.lua]])
```

### MCP client cannot connect

**Checklist:**
1. Cheat Engine shows `MCP Server Listening on: CE_MCP_Bridge_v99`
2. MCP client was restarted after adding server config
3. Python path in config is correct and uses forward slashes
4. `pip install mcp pywin32` completed successfully
5. Test with `ping` tool - should return `{"success": true, "version": "12.0.0"}`

### BSOD (CLOCK_WATCHDOG_TIMEOUT)

**Problem:** Conflict between DBVM and "Query memory region routines" when scanning protected memory.

**Solution:** **MUST disable** in Cheat Engine:
- Settings → Extra → **UNCHECK** "Query memory region routines"

### Process attach fails

```
Tool: get_process_list
→ Find exact process name

Tool: open_process
Args: {"process_name": "exact_name.exe"}  # Case-sensitive, include .exe
```

### Pointer scan returns no results

**Increase scan parameters:**
```
Tool: pointer_scan
Args: {
  "address": "0x789000",
  "max_level": 7,        # Increase depth
  "max_offset": 8192     # Increase offset range
}
```

### AOB scan finds too many results

**Add context bytes:**
```
# Too generic:
"89 45 ??"

# More specific (add surrounding instructions):
"48 8B 05 ?? ?? ?? ?? 48 85 C0 74 ?? 89 45 ?? C3"
```

**Limit search scope:**
```
Tool: aob_scan
Args: {
  "pattern": "...",
  "writable": false,
  "executable": true,  # Only search code sections
  "start_address": "game.exe",
  "end_address": "game.exe+0x500000"
}
```

## Testing

Run the test suite to verify installation:

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

## Example: Complete Trainer Creation

```
User: "Create a health trainer for game.exe"

Agent executes:
1. open_process(process_name="game.exe")
2. scan_all(value_type="4byte", value="100")  # Current health
3. [User takes damage]
4. next_scan(value="85")
5. [Repeat until 1-3 addresses remain]
6. set_data_breakpoint(address=result[0], type="write")
7. [User takes damage again]
8. disassemble(address=breakpoint_hit, count=30)
9. Identify health variable and function
10. generate_api_hook_script(address=health_function)
11. auto_assemble(script=hook_script)
12. freeze_address(address=health_addr, value=999, description="Infinite Health")
13. save_table(path="C:\\trainers\\game_trainer.CT")

Result: Cheat table with working infinite health
```

## Key Documentation Files

- **AI_Context/MCP_Bridge_Command_Reference.md** - All 180+ MCP tools documented
- **AI_Context/CE_LUA_Documentation.md** - Full Cheat Engine 7.6 Lua API reference
- **AI_Context/AI_Guide_MCP_Server_Implementation.md** - Technical architecture details

## Best Practices

1. **Always verify process attachment** before memory operations
2. **Use symbols** instead of hardcoded addresses for maintainability
3. **Test AOB signatures** after game updates to ensure reliability
4. **Prefer hardware breakpoints** over software breakpoints (no code modification)
5. **Use DBVM for anti-cheat evasion** when targeting protected games
6. **Save cheat tables frequently** during development
7. **Document pointer chains** with register_symbol for reuse

## Security Notes

⚠️ **This tool enables:**
- Reading/writing arbitrary process memory
- Code injection and execution
- DLL injection
- System-level debugging (DBVM)

**Use only for:**
- Educational purposes
- Single-player games/mods
- Security research
- Authorized penetration testing

**Do NOT use for:**
- Multiplayer cheating (violates TOS, illegal in some jurisdictions)
- Malware development
- Unauthorized access to protected software
