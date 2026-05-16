---
name: iida-mcp-ida-integration
description: IDA Pro plugin that exposes static analysis capabilities via MCP HTTP server for reverse engineering workflows
triggers:
  - analyze binary with IDA through MCP
  - connect to IDA Pro MCP server
  - use IDA decompiler via MCP
  - reverse engineer executable with iida-mcp
  - query IDA database through MCP tools
  - setup IDA Pro MCP integration
  - analyze functions and disassembly with IDA MCP
  - use iida-mcp for binary analysis
---

# iida-mcp IDA Integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

iida-mcp is an IDA Pro plugin that exposes the current IDB's static analysis capabilities through a local HTTP MCP service. It provides 77 MCP tools for binary analysis, supports multiple IDA instances with automatic routing, and offers optional Windows kernel driver capabilities for kernel-mode analysis.

## What iida-mcp Does

- **Static Analysis via MCP**: Exposes IDA Pro's reverse engineering capabilities through Model Context Protocol
- **Multi-Instance Support**: Automatically routes requests to the correct IDA instance when multiple IDBs are open
- **Comprehensive Tools**: 77 MCP tools covering disassembly, decompilation, CFG analysis, cross-references, and more
- **Kernel Analysis**: Optional Windows kernel driver for reading kernel memory and enumerating modules
- **x86/x86-64 Focus**: Primarily designed for x86/x86-64 architecture executables

## Installation

### Plugin Installation

1. Copy plugin files to IDA's `plugins/` directory:

```
IDA_DIR/plugins/
  iida.py
  iida_core/
    __init__.py
    cache.py
    kdriver.py
    protocol.py
    registry.py
    router.py
    server.py
    thread_safe.py
    tools.py
    worker.py
```

2. Restart IDA Pro (compatible with IDA 8+ and IDA 9.x)

### Starting the MCP Server

1. Open a target file in IDA Pro
2. Activate via `Edit > Plugins > iida-mcp` or press `Alt+Shift+I`
3. First IDA instance starts server on `0.0.0.0:13897`
4. Additional IDA instances automatically connect as workers
5. Toggle server/connection by pressing `Alt+Shift+I` again

### MCP Client Configuration

Configure your MCP client to connect to the HTTP endpoint:

```json
{
  "mcpServers": {
    "iida": {
      "url": "http://127.0.0.1:13897/mcp"
    }
  }
}
```

For remote connections (from another machine):

```json
{
  "mcpServers": {
    "iida-remote": {
      "url": "http://192.168.1.100:13897/mcp"
    }
  }
}
```

## Key MCP Tools

### File and Database Information

**list_files** - List all connected IDA instances and their files:
```python
# Returns list of active IDB files with their IDs
# Use file IDs for the 'f' parameter in other tools
```

**get_file_info** - Get metadata about the analyzed file:
```python
# Arguments: f (optional file ID)
# Returns: filename, path, MD5, SHA256, architecture, etc.
```

**read_bytes** - Read raw bytes from the binary:
```python
# Arguments: 
#   ea (effective address, hex string)
#   size (number of bytes)
#   f (optional file ID)
```

### Functions and Disassembly

**list_functions** - Enumerate all functions:
```python
# Arguments: f (optional file ID)
# Returns: array of {ea, name, size, flags}
```

**get_function_info** - Get detailed function information:
```python
# Arguments:
#   ea (function address, hex string)
#   f (optional file ID)
# Returns: start_ea, end_ea, size, name, frame size, flags
```

**disassemble** - Get disassembly listing:
```python
# Arguments:
#   ea (start address, hex string)
#   count (number of instructions, default 10)
#   f (optional file ID)
# Returns: array of disassembled instructions with addresses
```

**disasm_bytes** - Disassemble raw bytes using Capstone:
```python
# Arguments:
#   bytes_hex (hex-encoded bytes)
#   arch (optional: "x86", "x64", default auto-detect)
#   f (optional file ID)
# Requires: capstone installed in IDA's Python environment
```

### Decompilation (Requires Hex-Rays)

**decompile** - Get decompiled pseudocode:
```python
# Arguments:
#   ea (function address, hex string)
#   f (optional file ID)
# Returns: C-like pseudocode
```

**get_function_args** - Get function parameter information:
```python
# Arguments:
#   ea (function address, hex string)
#   f (optional file ID)
# Returns: array of {name, type, location}
```

**get_local_vars** - Get local variables:
```python
# Arguments:
#   ea (function address, hex string)
#   f (optional file ID)
```

### Control Flow and Cross-References

**get_function_cfg** - Get control flow graph:
```python
# Arguments:
#   ea (function address, hex string)
#   f (optional file ID)
# Returns: nodes and edges representing CFG
```

**get_xrefs_to** - Get cross-references to an address:
```python
# Arguments:
#   ea (target address, hex string)
#   f (optional file ID)
# Returns: array of {from, to, type}
```

**get_xrefs_from** - Get cross-references from an address:
```python
# Arguments:
#   ea (source address, hex string)
#   f (optional file ID)
```

**get_call_tree** - Build call tree (callers/callees):
```python
# Arguments:
#   ea (function address, hex string)
#   direction ("up" for callers, "down" for callees)
#   depth (recursion depth, default 3)
#   f (optional file ID)
```

### Searching

**search_text** - Search for text strings:
```python
# Arguments:
#   pattern (search string)
#   case_sensitive (boolean, default false)
#   f (optional file ID)
```

**search_bytes** - Search for byte patterns:
```python
# Arguments:
#   pattern (hex pattern, e.g. "48 8B ? ? 90")
#   f (optional file ID)
# Use ? for wildcard bytes
```

**search_immediate** - Search for immediate values:
```python
# Arguments:
#   value (decimal or hex string)
#   f (optional file ID)
```

### Modification Tools

**rename** - Rename address:
```python
# Arguments:
#   ea (address, hex string)
#   new_name (new symbol name)
#   f (optional file ID)
```

**set_comment** - Add/modify comment:
```python
# Arguments:
#   ea (address, hex string)
#   text (comment text)
#   repeatable (boolean, default false)
#   f (optional file ID)
```

**set_type** - Set type information:
```python
# Arguments:
#   ea (address, hex string)
#   type_str (C-style type declaration)
#   f (optional file ID)
# Example type_str: "int __fastcall(void *ptr, size_t len)"
```

**patch_bytes** - Modify bytes in database:
```python
# Arguments:
#   ea (address, hex string)
#   bytes_hex (hex-encoded replacement bytes)
#   f (optional file ID)
```

### Structures and Types

**list_structs** - List all structures:
```python
# Arguments: f (optional file ID)
# Returns: array of structure names and IDs
```

**get_struct_info** - Get structure definition:
```python
# Arguments:
#   name (structure name)
#   f (optional file ID)
# Returns: members with offsets, types, sizes
```

**list_enums** - List enumerations:
```python
# Arguments: f (optional file ID)
```

**typed_read** - Read memory with type interpretation:
```python
# Arguments:
#   ea (address, hex string)
#   type_str (C type, e.g. "unsigned int")
#   f (optional file ID)
# Returns: interpreted value
```

### Kernel Analysis (Windows Only)

**kernel_read_memory** - Read kernel memory:
```python
# Arguments:
#   address (kernel virtual address, hex string)
#   size (bytes to read)
# Requires: iida-mcp-ioctl.sys driver loaded
```

**kernel_list_modules** - Enumerate kernel modules:
```python
# Returns: array of {name, base, size}
# Requires: iida-mcp-ioctl.sys driver loaded
```

**kernel_get_module_base** - Get module base address:
```python
# Arguments:
#   name (module name, e.g. "ntoskrnl.exe")
# Requires: iida-mcp-ioctl.sys driver loaded
```

**map_ida_to_runtime** - Map IDA address to runtime address:
```python
# Arguments:
#   ea (IDA address, hex string)
#   module_name (target module name)
#   f (optional file ID)
# Useful for live debugging correlation
```

## Common Usage Patterns

### Single IDB Analysis

When working with one IDA database, omit the `f` parameter:

```python
# Get function info at specific address
get_function_info(ea="0x401000")

# Decompile function
decompile(ea="0x401000")

# Get cross-references
get_xrefs_to(ea="0x401000")
```

### Multi-IDB Workflow

When multiple IDA instances are connected:

```python
# 1. List available files
files = list_files()
# Returns: [{id: "file1", path: "C:\\samples\\malware.exe"}, ...]

# 2. Use file ID in subsequent calls
get_function_info(ea="0x401000", f="file1")
decompile(ea="0x401000", f="file1")
```

### Reverse Engineering Workflow

Typical analysis sequence:

```python
# 1. Get file overview
file_info = get_file_info()

# 2. List all functions
functions = list_functions()

# 3. Analyze interesting function
func = get_function_info(ea="0x401000")
code = decompile(ea="0x401000")
args = get_function_args(ea="0x401000")
xrefs = get_xrefs_to(ea="0x401000")

# 4. Search for patterns
strings = search_text(pattern="password")
crypto_calls = search_bytes(pattern="48 8B 05 ? ? ? ?")

# 5. Annotate findings
rename(ea="0x401000", new_name="decrypt_config")
set_comment(ea="0x401000", text="RC4 decryption routine")
set_type(ea="0x401000", type_str="void __fastcall(uint8_t *data, size_t len)")
```

### Kernel Driver Analysis

For kernel-mode binaries:

```python
# 1. Analyze driver in IDA
driver_info = get_file_info()

# 2. Map IDA addresses to runtime
runtime_addr = map_ida_to_runtime(
    ea="0x140001000",
    module_name="mydriver.sys"
)

# 3. Read live kernel memory
kernel_data = kernel_read_memory(
    address=runtime_addr,
    size=256
)

# 4. List loaded kernel modules
modules = kernel_list_modules()
```

## Configuration

### Ports

- **13897**: MCP HTTP service (listens on all interfaces)
- **13898**: Internal worker communication (localhost only)

### Network Access

By default, the MCP server listens on `0.0.0.0:13897`, allowing connections from:
- Localhost: `http://127.0.0.1:13897/mcp`
- LAN: `http://<host-ip>:13897/mcp`

For security, consider firewall rules if exposing to network.

### Dependencies

**Core Plugin**: No additional dependencies (uses IDA's built-in Python)

**Optional Dependencies**:
- **Hex-Rays Decompiler**: Required for `decompile`, `get_function_args`, `get_local_vars`
- **Capstone**: Required for `disasm_bytes` (install: `pip install capstone` in IDA's Python)
- **Kernel Driver**: Required for `kernel_*` and `map_ida_to_runtime` tools

### Kernel Driver Setup

The `iida-mcp-ioctl.sys` driver provides kernel memory access:

1. Driver is located in `driver/` directory
2. Requires proper code signing or test signing enabled
3. Load with `sc create` or driver loader tool
4. Without driver, kernel tools return clear error messages

**Test Signing** (development only):
```cmd
bcdedit /set testsigning on
```

**Load Driver**:
```cmd
sc create iida-mcp-ioctl binPath="C:\path\to\iida-mcp-ioctl.sys" type=kernel
sc start iida-mcp-ioctl
```

## Troubleshooting

### Server Won't Start

**Issue**: Plugin activated but server doesn't respond
- Check IDA Output window for error messages
- Verify port 13897 is not in use: `netstat -an | findstr 13897`
- Ensure IDA has network permissions (firewall)

### Tool Returns "capstone not installed"

**Issue**: `disasm_bytes` fails
```python
# Install Capstone in IDA's Python environment
# From IDA's Python console:
import subprocess
import sys
subprocess.check_call([sys.executable, "-m", "pip", "install", "capstone"])
```

### Kernel Tools Fail

**Issue**: `kernel_read_memory` returns error
- Verify `iida-mcp-ioctl.sys` is loaded: `sc query iida-mcp-ioctl`
- Check driver loaded correctly in DebugView or DbgView
- Ensure administrator privileges
- Verify test signing or proper code signature

### Multiple IDA Instances Not Routing

**Issue**: Tools access wrong IDB
- Always call `list_files()` first to get current file IDs
- Include `f` parameter with correct file ID
- Verify worker connection in IDA Output window

### Decompilation Tools Fail

**Issue**: `decompile` returns error
- Ensure Hex-Rays Decompiler is installed and licensed
- Verify address points to valid function: `get_function_info(ea="0x...")`
- Some functions may not decompile due to complexity or obfuscation

### Remote Connection Fails

**Issue**: Cannot connect from another machine
- Verify server listens on `0.0.0.0`: check IDA Output window on startup
- Check firewall allows inbound TCP 13897
- Use host's actual IP, not 127.0.0.1
- Ping host to verify network connectivity

### Tool Returns Empty Results

**Issue**: Search or query returns no data
- Verify address is valid: check IDA's disassembly view
- Ensure IDA has finished auto-analysis (check status bar)
- For searches, check pattern syntax (hex bytes use spaces: `"48 8B 05"`)
- Some tools require specific IDA analysis (e.g., functions must be recognized)

## Example Agent Workflow

When helping a user analyze a binary with iida-mcp:

1. **Verify Setup**: Confirm IDA is running with plugin active
2. **Check Connections**: Use `list_files()` to see available IDBs
3. **Gather Context**: Use `get_file_info()` for binary metadata
4. **Explore Functions**: Use `list_functions()` to enumerate code
5. **Deep Dive**: Combine `disassemble()`, `decompile()`, `get_xrefs_to()` for analysis
6. **Search & Pattern Match**: Use `search_text()`, `search_bytes()` for specific artifacts
7. **Annotate**: Apply findings with `rename()`, `set_comment()`, `set_type()`
8. **Export**: Document findings based on tool outputs

All addresses should be provided as hex strings (e.g., `"0x401000"`) and the `f` parameter should be included when multiple IDBs are active.
