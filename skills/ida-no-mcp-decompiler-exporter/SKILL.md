---
name: ida-no-mcp-decompiler-exporter
description: Export IDA Pro decompiled code and memory for AI-assisted reverse engineering
triggers:
  - export IDA decompiled code for AI analysis
  - how do I use IDA NO MCP plugin
  - export binary functions from IDA Pro
  - setup IDA decompiler export plugin
  - extract all functions from IDA database
  - export IDA memory dumps and strings
  - configure IDA NO MCP exporter
  - analyze binary with AI using IDA export
---

# IDA NO MCP Decompiler Exporter

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

IDA-NO-MCP is a plugin for IDA Pro that exports decompiled code, disassembly, strings, imports, exports, and memory dumps into AI-friendly formats. Instead of complex MCP integrations, it generates organized source files that can be directly analyzed by AI IDEs like Cursor or Claude Code.

## What It Does

- **Exports decompiled C code**: Each function as a separate `.c` file with metadata (address, callers, callees)
- **Automatic fallback**: Falls back to disassembly (`.asm`) when decompilation fails
- **Memory dumps**: Exports all memory segments as hexdump files (1MB chunks)
- **Metadata extraction**: Strings, imports, exports tables
- **Smart filtering**: Skips library functions and invalid functions automatically
- **Detailed logging**: Tracks successes, fallbacks, failures, and skipped functions

## Installation

### Plugin Mode (Recommended)

1. Copy `INP.py` to your IDA plugins directory:
   - **Windows**: `%APPDATA%\Hex-Rays\IDA Pro\plugins\`
   - **Linux/macOS**: `~/.idapro/plugins/`

2. Restart IDA Pro

3. Use the plugin:
   - **Shortcut**: `Ctrl-Shift-E` (quick export)
   - **Menu**: `Edit` → `Plugins` → `Export for AI`

### Script Mode

Run `INP.py` directly from IDA's script window (`Alt-F7`) or via command line:

```python
# From IDA script window
execfile('/path/to/INP.py')
```

## Output Structure

After export, the IDB directory contains:

```
your_binary.idb/
├── decompile/              # Decompiled C code (.c files)
├── disassembly/            # Fallback assembly (.asm files)
├── memory/                 # Memory dumps (hexdump format)
├── strings.txt             # All strings with addresses
├── imports.txt             # Import table
├── exports.txt             # Export table
├── disassembly_fallback.txt  # List of fallback functions
├── decompile_failed.txt    # Complete failures
└── decompile_skipped.txt   # Skipped library/invalid functions
```

## Function Export Format

Each exported function includes metadata headers:

```c
/*
 * func-name: sub_401000
 * func-address: 0x401000
 * export-type: decompile
 * callers: 0x402000, 0x403000
 * callees: 0x404000, 0x405000
 */

__int64 __fastcall sub_401000(__int64 a1, int a2)
{
  // Decompiled code here
  return result;
}
```

For disassembly fallback (`.asm` files):

```asm
/*
 * func-name: sub_401000
 * func-address: 0x401000
 * export-type: disassembly
 * callers: 0x402000, 0x403000
 * callees: 0x404000, 0x405000
 */

sub_401000 proc near
    push    rbp
    mov     rbp, rsp
    ; ... assembly code
    ret
sub_401000 endp
```

## Key Plugin Components

### Core Export Logic

```python
import idaapi
import idc
import idautils
import ida_hexrays
import ida_funcs
import os

def export_decompiled_code(func_ea):
    """Export decompiled code for a function"""
    try:
        # Get function name
        func_name = idc.get_func_name(func_ea)
        
        # Get callers and callees
        callers = [hex(xref.frm) for xref in idautils.XrefsTo(func_ea, 0)]
        callees = []
        for item_ea in idautils.FuncItems(func_ea):
            for xref in idautils.XrefsFrom(item_ea, 0):
                if xref.type in [ida_xref.fl_CN, ida_xref.fl_CF]:
                    callees.append(hex(xref.to))
        
        # Try decompilation
        cfunc = idaapi.decompile(func_ea)
        if cfunc:
            decompiled = str(cfunc)
            
            # Build metadata header
            header = f"""/*
 * func-name: {func_name}
 * func-address: {hex(func_ea)}
 * export-type: decompile
 * callers: {', '.join(callers) if callers else 'none'}
 * callees: {', '.join(callees) if callees else 'none'}
 */

"""
            return header + decompiled
        
    except Exception as e:
        print(f"Decompilation failed for {hex(func_ea)}: {e}")
        return None
```

### Memory Export

```python
def export_memory_segment(seg_ea, output_dir):
    """Export memory segment as hexdump"""
    seg = idaapi.getseg(seg_ea)
    if not seg:
        return
    
    seg_start = seg.start_ea
    seg_end = seg.end_ea
    seg_size = seg_end - seg_start
    
    max_size = 1024 * 1024  # 1MB chunks
    chunk_num = 0
    
    while seg_start < seg_end:
        chunk_end = min(seg_start + max_size, seg_end)
        filename = f"{hex(seg_start)}--{hex(chunk_end)}.txt"
        
        with open(os.path.join(output_dir, filename), 'w') as f:
            addr = seg_start
            while addr < chunk_end:
                # Read 16 bytes per line
                line_bytes = []
                ascii_chars = []
                
                for i in range(16):
                    if addr + i >= chunk_end:
                        break
                    byte = idc.get_wide_byte(addr + i)
                    line_bytes.append(f"{byte:02X}")
                    ascii_chars.append(chr(byte) if 32 <= byte <= 126 else '.')
                
                # Format: ADDRESS | HEX BYTES | ASCII
                hex_part = ' '.join(line_bytes).ljust(48)
                ascii_part = ''.join(ascii_chars)
                f.write(f"{hex(addr)} | {hex_part} | {ascii_part}\n")
                
                addr += 16
        
        seg_start = chunk_end
        chunk_num += 1
```

### String Extraction

```python
def export_strings(output_file):
    """Export all strings with metadata"""
    with open(output_file, 'w', encoding='utf-8') as f:
        strings = idautils.Strings()
        for s in strings:
            # Format: address, length, type, content
            str_type = {
                0: "ASCII",
                1: "UTF-16LE",
                2: "UTF-32LE"
            }.get(s.strtype, "UNKNOWN")
            
            f.write(f"{hex(s.ea)} | len={s.length} | {str_type} | {str(s)}\n")
```

## Common Usage Patterns

### Analyzing Exported Code with AI

After exporting, open the IDB directory in your AI IDE:

1. **Context-aware analysis**: AI can read all `.c` files and understand function relationships via caller/callee metadata
2. **Vulnerability hunting**: Ask AI to find buffer overflows, use-after-free, etc.
3. **Crypto detection**: Identify cryptographic functions and constants
4. **Protocol analysis**: Understand network protocol parsing logic

### Adding Extra Context

Create additional directories alongside exports:

```
your_binary.idb/
├── decompile/        # Auto-generated
├── docs/             # Your reverse engineering notes
├── codes/            # Frida scripts, exploits, tools
└── apk/              # APK decompilation (for Android)
```

AI tools will index all content for comprehensive analysis.

### Programmatic Integration

```python
# Run export from IDA Python script
import INP

# Trigger export programmatically
INP.main()  # Runs the full export process

# Or customize export paths
output_dir = "/custom/path/output"
INP.export_all(output_dir)
```

## Configuration

The plugin works out-of-the-box with defaults but can be customized by editing `INP.py`:

```python
# Skip library functions (default: True)
SKIP_LIB_FUNCS = True

# Maximum memory chunk size in bytes
MAX_CHUNK_SIZE = 1024 * 1024  # 1MB

# Progress reporting interval
PROGRESS_INTERVAL = 100  # Report every 100 functions

# Export types to include
EXPORT_DECOMPILE = True
EXPORT_DISASM_FALLBACK = True
EXPORT_MEMORY = True
EXPORT_STRINGS = True
EXPORT_IMPORTS = True
EXPORT_EXPORTS = True
```

## Troubleshooting

### Plugin doesn't appear in menu

- Verify `INP.py` is in the correct plugins directory
- Check IDA output window for Python errors
- Ensure IDA has Hex-Rays Decompiler installed (for decompilation feature)

### Decompilation fails for all functions

- Check if Hex-Rays Decompiler is licensed and active
- Some architectures may not support decompilation (will auto-fallback to disassembly)
- Check `decompile_failed.txt` for specific error messages

### Out of memory during export

- Large binaries may need chunked processing
- Reduce `MAX_CHUNK_SIZE` in the script
- Export specific function ranges instead of entire binary

### Special characters in function names

The plugin automatically sanitizes filenames:
- Replaces `/\:*?"<>|` with underscores
- Appends address suffix for duplicate names (e.g., `main_401000.c`)

### Missing callers/callees data

- Ensure IDA has completed auto-analysis (Wait for "AU: idle" in status bar)
- Run "Reanalyze program" from Edit menu if needed
- Check if functions are properly recognized (Edit → Functions → Reanalyze program)

## Integration with AI Workflows

### Example: Finding vulnerabilities

```bash
# After export, ask AI in your IDE:
"Analyze all functions in decompile/ for buffer overflow vulnerabilities"
"Find all memcpy/strcpy calls and check bounds validation"
```

### Example: Understanding malware behavior

```bash
"Trace the execution flow starting from entry point at 0x401000"
"Identify anti-debugging checks and obfuscation techniques"
"Extract C2 communication URLs from strings.txt and related functions"
```

### Example: Protocol reverse engineering

```bash
"Find packet parsing functions using imports.txt and decompiled code"
"Document the binary protocol structure based on recv/send call patterns"
```

## Advanced Tips

- **Incremental analysis**: Export once, iterate with AI on specific function subsets
- **Version control**: Commit exports to track understanding evolution
- **Cross-reference**: Combine with dynamic analysis (Frida logs, traces)
- **Custom scripts**: Write Python scripts that parse the exported metadata for automated analysis
