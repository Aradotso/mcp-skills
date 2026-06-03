---
name: ida-mcp-headless-reverse-engineering
description: Headless IDA Pro MCP server for AI-powered binary analysis and reverse engineering
triggers:
  - analyze this binary with IDA Pro
  - disassemble and decompile this executable
  - find cross-references in this binary
  - open this file in headless IDA
  - run IDA analysis on this malware sample
  - decompile this function with Hex-Rays
  - extract strings from this binary
  - analyze dyld_shared_cache module
---

# ida-mcp Headless Reverse Engineering

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`ida-mcp-rs` is a headless IDA Pro MCP server that enables AI agents to perform binary analysis, disassembly, decompilation, and reverse engineering tasks programmatically. It exposes IDA Pro's powerful analysis engine through 71 MCP tools without requiring the GUI.

**Key capabilities:**
- Open and analyze binaries (ELF, Mach-O, PE, etc.)
- Disassemble functions and code regions
- Decompile with Hex-Rays (if licensed)
- Cross-reference analysis (xrefs)
- String extraction and searching
- IDAPython script execution
- Apple dyld_shared_cache analysis
- Background task management for long-running operations

**Requirements:**
- IDA Pro 9.2+ with valid license (9.3sp1 recommended)
- Hex-Rays decompiler license (for decompilation features)

## Installation

### macOS

```bash
# Latest (IDA 9.3/9.3sp1)
brew install blacktop/tap/ida-mcp

# For IDA 9.2
brew install blacktop/tap/ida-mcp@9.2

# Configure with Claude Code
claude mcp add ida -- ida-mcp

# If you get library loading errors, set DYLD_LIBRARY_PATH
claude mcp add ida -e DYLD_LIBRARY_PATH='/Applications/IDA Professional 9.3.app/Contents/MacOS' -- ida-mcp
```

### Linux

```bash
# Via Homebrew
brew install blacktop/tap/ida-mcp

# Via Snap
sudo snap install ida-mcp
sudo snap connect ida-mcp:dot-idapro

# Configure
claude mcp add ida -- ida-mcp

# For non-standard install paths
claude mcp add ida -e IDADIR='/opt/ida-pro-9.3' -- ida-mcp
```

### Windows

```powershell
# Via Scoop (recommended)
scoop bucket add blacktop https://github.com/blacktop/scoop-bucket
scoop install blacktop/ida-mcp
claude mcp add ida -- ida-mcp

# OR copy binary to IDA directory
copy ida-mcp.exe "C:\Program Files\IDA Professional 9.3\"
claude mcp add ida -- "C:\Program Files\IDA Professional 9.3\ida-mcp.exe"

# OR set IDADIR manually
setx IDADIR "C:\Program Files\IDA Professional 9.3"
claude mcp add ida -- ida-mcp
```

### Version Matching

`ida-mcp` versions mirror IDA Pro versions:
- `v9.3.x` → IDA Pro 9.3/9.3sp1
- `v9.2.x` → IDA Pro 9.2

Version mismatches are detected at startup with clear error messages.

## Configuration

### Context Optimization

By default, `ida-mcp` exposes all 71 tools (~10k tokens). Filter the surface for smaller models or Gemini's 512-function limit:

```bash
# Via command-line flags
ida-mcp --toolsets=core,functions,disassembly,decompile,xrefs

# Via environment variables (in mcpServers.json)
{
  "mcpServers": {
    "ida-mcp": {
      "command": "ida-mcp",
      "env": {
        "IDA_MCP_TOOLSETS": "core,functions,disassembly,decompile,xrefs",
        "IDA_MCP_READ_ONLY": "true"
      }
    }
  }
}
```

**Available toolsets:** `core`, `functions`, `disassembly`, `decompile`, `xrefs`, `control_flow`, `memory`, `search`, `metadata`, `types`, `editing`, `scripting`

**Flags:**
- `--toolsets=cat1,cat2` — Enable specific categories
- `--tools=t1,t2` — Add individual tools
- `--exclude-tools=t1,t2` — Remove specific tools
- `--read-only` — Strip all mutating tools (`run_script`, `patch*`, `rename`, etc.)

### HTTP/SSE Worker Pool

For multi-client HTTP/SSE usage:

```bash
ida-mcp serve-http --bind 127.0.0.1:8765 --max-workers 4 --min-workers 1
```

Each HTTP session leases one child worker until `close_idb` or timeout. Configure:
- `--worker-idle-timeout-secs` — How long idle workers stay alive
- `--worker-disconnect-grace-secs` — Grace period for SSE reconnects
- `--session-keep-alive-secs` — POST-only session timeout (default 1800s)

## Core Workflows

### Opening and Analyzing a Binary

```python
# Open a binary (returns immediately)
result = open_idb(path="/path/to/malware.exe")
# Returns: {"task_id": "open-1", "message": "Opening /path/to/malware.exe"}

# Check if analysis is needed
status = analysis_status()
# Returns: {"status": "pending", "auto_analysis_enabled": true}

# For large binaries, run analysis in background
analyze = analyze_funcs(background=true)
# Returns: {"task_id": "analyze-1"}

# Poll progress
progress = task_status(task_id="analyze-1")
# Returns: {"status": "running", "progress": 45.2}

# Wait until complete
while task_status(task_id="analyze-1")["status"] != "complete":
    time.sleep(5)
```

### Function Discovery and Disassembly

```python
# List all functions (works immediately, no analysis needed)
funcs = list_functions(limit=20)
# Returns: [{"address": "0x100000f00", "name": "main", "size": 256}, ...]

# Disassemble by function name
asm = disasm_by_name(name="main", count=20)
# Returns assembly instructions with addresses

# Disassemble by address
asm = disasm(address="0x100000f00", count=50)

# Get function information
info = func_info(address="0x100000f00")
# Returns: {"name": "main", "start": "0x100000f00", "end": "0x100001000", ...}
```

### Decompilation (Hex-Rays Required)

```python
# Decompile a function (requires completed analysis)
code = decompile(address="0x100000f00")
# Returns C pseudocode

# Decompile by function name
code = decompile_by_name(name="main")

# For large binaries, ensure analysis is complete first
status = analysis_status()
if status["status"] == "pending":
    analyze_funcs(background=true)
    # Poll task_status until complete
```

### Cross-Reference Analysis

```python
# Find what calls this function
callers = callers(address="0x100000f00")
# Returns: [{"from": "0x100001234", "to": "0x100000f00", "type": "call"}, ...]

# Find what this function calls
callees = callees(address="0x100000f00")

# Data cross-references (what reads/writes this address)
data_xrefs = data_xrefs(address="0x100002000")

# Code cross-references to an address
code_xrefs = code_xrefs_to(address="0x100000f00")
```

### String Extraction

```python
# Extract strings from binary
strings = strings(limit=100)
# Returns: [{"address": "0x100002000", "value": "Hello World", "type": "ascii"}, ...]

# Search for specific strings
results = search_text(pattern="password")
# Returns: [{"address": "0x100002100", "match": "password123"}, ...]

# Binary search
results = search_binary(pattern="4883EC20")  # hex bytes
```

### Symbol and Import/Export Analysis

```python
# List imports
imports = list_imports()
# Returns: [{"address": "0x100000000", "name": "printf", "module": "libc.so.6"}, ...]

# List exports
exports = list_exports()

# List all symbols
symbols = list_symbols(limit=100)
```

### Memory and Segments

```python
# List memory segments
segments = list_segments()
# Returns: [{"start": "0x100000000", "end": "0x100001000", "name": ".text", "perm": "r-x"}, ...]

# Read bytes from memory
bytes_data = read_bytes(address="0x100000f00", size=64)
# Returns hex-encoded bytes

# Get database info
info = idb_info()
# Returns: {"path": "/path/to/binary.i64", "architecture": "x86_64", ...}
```

### IDAPython Scripting

```python
# Run inline Python script
result = run_script(code="""
import idautils
import idc

for func_ea in idautils.Functions():
    func_name = idc.get_func_name(func_ea)
    if 'crypto' in func_name.lower():
        print(f"{hex(func_ea)}: {func_name}")
""")
# Returns: {"stdout": "0x100001234: crypto_init\n...", "stderr": ""}

# Run script from file
result = run_script(file="/path/to/analysis_script.py")

# With custom timeout (default 120s, max 600s)
result = run_script(
    code="import ida_bytes; print(ida_bytes.get_bytes(0x1000, 16).hex())",
    timeout_secs=30
)
```

### Apple dyld_shared_cache Analysis

```python
# Open a module from dyld_shared_cache (first use creates .i64, takes minutes)
result = open_dsc(
    path="/System/Library/dyld/dyld_shared_cache_arm64e",
    arch="arm64e",
    module="/usr/lib/libobjc.A.dylib"
)
# Returns: {"task_id": "dsc-1"} if background processing started

# Poll until ready
while task_status(task_id="dsc-1")["status"] != "complete":
    time.sleep(10)

# Open with additional frameworks for cross-module references
result = open_dsc(
    path="/System/Library/dyld/dyld_shared_cache_arm64e",
    arch="arm64e",
    module="/usr/lib/libobjc.A.dylib",
    frameworks=["/System/Library/Frameworks/Foundation.framework/Foundation"]
)

# Add another dylib to already-open database
dsc_add_dylib(module="/usr/lib/libSystem.B.dylib")

# Add a data/GOT/stub region by address
dsc_add_region(address="0x180116000")

# Check analysis status after adding
status = analysis_status()
```

### Task Management

```python
# List all background tasks
tasks = list_tasks()
# Returns: [{"task_id": "analyze-1", "status": "running", "progress": 67.5}, ...]

# Get specific task status
status = task_status(task_id="analyze-1")

# Cancel a running task
cancel_task(task_id="analyze-1")
```

### Database Lifecycle

```python
# Open binary
open_idb(path="/path/to/binary")

# Check current database
info = idb_info()

# Close database
close_idb()

# Reanalyze all functions (useful after patching)
analyze_funcs(background=true)
```

## Common Patterns

### Complete Binary Analysis Pipeline

```python
# 1. Open binary
open_idb(path="/path/to/malware.exe")

# 2. Start background analysis
task = analyze_funcs(background=true)
task_id = task["task_id"]

# 3. Poll until complete
import time
while task_status(task_id=task_id)["status"] != "complete":
    progress = task_status(task_id=task_id)
    print(f"Analysis progress: {progress.get('progress', 0)}%")
    time.sleep(5)

# 4. Extract key information
funcs = list_functions(limit=50)
strings = strings(limit=100)
imports = list_imports()

# 5. Decompile main function
main_code = decompile_by_name(name="main")

# 6. Find crypto-related functions
crypto_funcs = []
for func in funcs:
    if 'crypt' in func['name'].lower() or 'aes' in func['name'].lower():
        crypto_funcs.append(func)
        code = decompile(address=func['address'])
        print(f"Found crypto function: {func['name']}\n{code}\n")

# 7. Close when done
close_idb()
```

### Finding and Analyzing Suspicious Functions

```python
# Open binary
open_idb(path="/path/to/suspicious.exe")
analyze_funcs(background=true)

# Wait for analysis...

# Search for suspicious strings
suspicious = search_text(pattern="cmd.exe|powershell|/bin/sh")

# For each hit, find what references it
for hit in suspicious:
    print(f"Found suspicious string at {hit['address']}: {hit['match']}")
    xrefs = code_xrefs_to(address=hit['address'])
    
    for xref in xrefs:
        func = func_info(address=xref['from'])
        print(f"  Referenced by {func['name']} at {xref['from']}")
        
        # Decompile the referencing function
        code = decompile(address=func['start'])
        print(f"  Decompiled:\n{code}\n")
```

### Control Flow Analysis

```python
# Get function CFG
cfg = func_cfg(address="0x100000f00")
# Returns: {"blocks": [...], "edges": [...]}

# Analyze basic blocks
for block in cfg['blocks']:
    print(f"Block {block['start']} -> {block['end']}")
    asm = disasm(address=block['start'], count=10)
    print(asm)
```

### Custom IDAPython Analysis

```python
# Find all functions that call malloc/free
result = run_script(code="""
import idautils
import idc

malloc_ea = idc.get_name_ea_simple('malloc')
free_ea = idc.get_name_ea_simple('free')

memory_funcs = set()

for xref in idautils.XrefsTo(malloc_ea):
    func = idaapi.get_func(xref.frm)
    if func:
        memory_funcs.add(func.start_ea)

for xref in idautils.XrefsTo(free_ea):
    func = idaapi.get_func(xref.frm)
    if func:
        memory_funcs.add(func.start_ea)

for func_ea in memory_funcs:
    print(f"{hex(func_ea)}: {idc.get_func_name(func_ea)}")
""")

print(result['stdout'])
```

## Tool Discovery

```python
# Search available tools
tools = tool_catalog(query="find callers")
# Returns tools matching the query with descriptions

# List all tools in a category
tools = tool_catalog(category="xrefs")
```

## Troubleshooting

### Library Loading Errors

**macOS:** `Library not loaded: @rpath/libida.dylib`
```bash
# Set DYLD_LIBRARY_PATH to IDA installation
claude mcp add ida -e DYLD_LIBRARY_PATH='/Applications/IDA Professional 9.3.app/Contents/MacOS' -- ida-mcp
```

**Linux:** `error while loading shared libraries: libida.so`
```bash
# Set IDADIR
claude mcp add ida -e IDADIR='/opt/ida-pro-9.3' -- ida-mcp
```

**Windows:** `The program can't start because ida.dll is missing`
```powershell
# Copy exe to IDA directory or set IDADIR
copy ida-mcp.exe "C:\Program Files\IDA Professional 9.3\"
# OR
setx IDADIR "C:\Program Files\IDA Professional 9.3"
```

### Version Mismatch

Error: `IDA version mismatch: expected 9.3, found 9.2`

Solution: Install matching version:
```bash
brew install blacktop/tap/ida-mcp@9.2
```

### Analysis Not Complete

If decompilation or xrefs fail with "analysis pending":

```python
# Check status
status = analysis_status()

# If pending, run analysis
if status["status"] == "pending":
    task = analyze_funcs(background=true)
    # Wait for completion
    while task_status(task_id=task["task_id"])["status"] != "complete":
        time.sleep(5)
```

### Decompilation Fails

Error: `Decompilation failed: Hex-Rays decompiler not available`

Solution: Requires Hex-Rays license. Check:
```python
info = idb_info()
print(info.get("decompiler_available"))
```

### Worker Pool Exhausted

Error: `Worker pool exhausted`

Solution: Close unused databases or increase worker count:
```bash
ida-mcp serve-http --max-workers 8
```

### dyld_shared_cache First-Time Delay

First `open_dsc` call can take 5-15 minutes to create `.i64`:

```python
# Start in background
result = open_dsc(path="/path/to/dyld_shared_cache_arm64e", 
                  arch="arm64e",
                  module="/usr/lib/libobjc.A.dylib")

# Poll with patience
task_id = result["task_id"]
while task_status(task_id=task_id)["status"] != "complete":
    time.sleep(30)  # Check every 30s
```

Subsequent opens of the same module are instant (reuses `.i64`).

### Performance Tips

1. **Use background analysis** for large binaries (>10MB)
2. **Filter tool surface** if using small models or hitting Gemini's 512-function limit
3. **Close databases** when done to free resources
4. **Poll task_status** instead of blocking on long operations
5. **Use worker pool** (`--max-workers`) for concurrent HTTP sessions

## Reference

**Lifecycle:** `open_idb`, `close_idb`, `idb_info`, `analysis_status`, `analyze_funcs`

**Functions:** `list_functions`, `func_info`, `func_cfg`, `disasm_by_name`, `disasm`, `decompile`, `decompile_by_name`

**XRefs:** `callers`, `callees`, `code_xrefs_to`, `data_xrefs`, `xrefs_from`

**Search:** `search_text`, `search_binary`, `strings`, `find_pattern`

**Symbols:** `list_imports`, `list_exports`, `list_symbols`, `resolve_name`

**Memory:** `read_bytes`, `list_segments`, `segment_info`

**Tasks:** `task_status`, `list_tasks`, `cancel_task`

**DSC:** `open_dsc`, `dsc_add_dylib`, `dsc_add_region`

**Scripting:** `run_script`

**Discovery:** `tool_catalog`

For complete tool list and parameters, use `tool_catalog()` or check the [GitHub repository](https://github.com/blacktop/ida-mcp-rs).
