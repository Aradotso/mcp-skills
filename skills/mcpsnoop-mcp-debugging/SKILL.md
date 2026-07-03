---
name: mcpsnoop-mcp-debugging
description: Debug MCP client-server traffic in real-time using mcpsnoop, a transparent proxy for Model Context Protocol communications
triggers:
  - debug MCP traffic between client and server
  - inspect MCP tool calls and responses
  - troubleshoot MCP server communication issues
  - set up mcpsnoop to monitor MCP messages
  - replay MCP tool calls for testing
  - view MCP capabilities negotiation
  - monitor JSON-RPC frames in MCP
  - diagnose why MCP tool isn't being called
---

# mcpsnoop MCP Debugging Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

mcpsnoop is a transparent proxy for Model Context Protocol (MCP) that sits between your AI client (Claude Desktop, Cursor, Claude Code) and your MCP servers. Unlike the official MCP Inspector which connects as a separate client, mcpsnoop intercepts the actual traffic between your real client and server, showing every JSON-RPC frame live in your terminal.

## Installation

### Using Go

```bash
go install github.com/kerlenton/mcpsnoop/cmd/mcpsnoop@latest
```

### Using Homebrew

```bash
brew tap kerlenton/mcpsnoop
brew install mcpsnoop
```

### Pre-built Binaries

Download from the [Releases page](https://github.com/kerlenton/mcpsnoop/releases) for your platform.

## Core Concepts

mcpsnoop operates in two modes:

1. **Shim mode** (`mcpsnoop -- <server-command>`): Acts as a transparent proxy between client and server
2. **UI mode** (`mcpsnoop`): Displays the captured traffic in an interactive TUI

The shim and UI communicate via a well-known socket and on-disk logs, so they can start in any order.

## Configuration

### Stdio Transport (Most Common)

Wrap your server command in your MCP client's configuration file. For Claude Desktop, edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "my-server": {
      "command": "mcpsnoop",
      "args": ["--", "node", "build/index.js"]
    }
  }
}
```

Everything after `--` is your normal server launch command.

**Examples:**

Python server:
```json
{
  "mcpServers": {
    "weather": {
      "command": "mcpsnoop",
      "args": ["--", "python", "-m", "weather_server"]
    }
  }
}
```

TypeScript server with npx:
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcpsnoop",
      "args": ["--", "npx", "-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"]
    }
  }
}
```

Compiled binary:
```json
{
  "mcpServers": {
    "custom": {
      "command": "mcpsnoop",
      "args": ["--", "/usr/local/bin/my-mcp-server", "--port", "8080"]
    }
  }
}
```

### HTTP/SSE Transport

For servers using streamable HTTP, run mcpsnoop as a reverse proxy:

```bash
mcpsnoop http --target http://localhost:3000/mcp --listen :7000
```

Then point your client to `http://localhost:7000`.

## Key Commands

### CLI Commands

| Command | Description |
|---------|-------------|
| `mcpsnoop` | Launch the interactive TUI to view traffic |
| `mcpsnoop -- <cmd>` | Wrap a server command (shim mode) |
| `mcpsnoop http` | Start HTTP reverse proxy |
| `mcpsnoop demo` | Run a scripted demo session |
| `mcpsnoop --help` | Show all options |

### TUI Keybindings

| Key | Action |
|-----|--------|
| `enter` | Drill into frame details |
| `esc` | Go back |
| `r` | Replay selected tool call |
| `c` | View capability negotiation |
| `y` | Copy frame to clipboard |
| `/` | Enter filter mode |
| `:` | Command mode |
| `p` | Pause/unpause stream |
| `f` | Follow mode (scroll to latest) |
| `ctrl-d` | Delete session |
| `j`/`k` | Move up/down |
| `ctrl-f`/`ctrl-b` | Page forward/back |
| `g`/`G` | Jump to top/bottom |
| `?` | Show help |

## Filtering Traffic

Press `/` in the TUI to filter frames. Combine tokens with spaces (ANDed):

### Filter Syntax

```
tool:search status:slow           # Slow calls to search tool
dir:s2c kind:req                   # Server-to-client requests
method:tools/call id:123           # Specific method and ID
status:error tool:database         # Failed database tool calls
kind:notif                         # All notifications
dir:c2s status:pending             # Pending client requests
```

### Available Filters

- **tool:** Filter by tool name (e.g., `tool:search`, `tool:write_file`)
- **method:** JSON-RPC method (e.g., `method:tools/call`, `method:initialize`)
- **id:** Request/response ID
- **kind:** `req` (request), `resp` (response), `notif` (notification)
- **dir:** `c2s` (client-to-server), `s2c` (server-to-client)
- **status:** `ok`, `error`, `slow`, `pending`

Plain text searches match method, tool name, ID, and payload content.

## Common Patterns

### Debugging a Tool That Isn't Being Called

1. Configure mcpsnoop wrapper in your client
2. Restart the client
3. Open mcpsnoop TUI: `mcpsnoop`
4. Press `c` to view capabilities
5. Verify your tool is listed in server capabilities
6. Check if client accepted the capability
7. Use filter: `/tool:your_tool_name`
8. Attempt to use the tool in your client
9. Look for `PENDING` status (indicates hung call) or absence of call

### Inspecting Capability Negotiation

```bash
# Start your client with mcpsnoop wrapper
# Open TUI
mcpsnoop

# Press 'c' to view capabilities screen
# Navigate between:
# - Client capabilities (what the AI client supports)
# - Server capabilities (what your server advertises)
# - Handshake details
```

### Replaying a Tool Call

```bash
# In TUI, navigate to a tools/call request
# Press 'r' to replay
# mcpsnoop spawns a fresh isolated server instance
# Executes the exact same call
# Shows the new response
```

This is useful for:
- Testing fixes without restarting your client
- Comparing responses after code changes
- Debugging non-deterministic behavior

### Monitoring Slow Tools

```bash
# In TUI
/status:slow

# Shows only slow calls (threshold configurable)
# Each frame shows elapsed time
# PENDING calls show live timer
```

### Debugging Server Stderr

Server stderr appears in the stream with special highlighting:

```
[STDERR] Warning: database connection slow
[STDERR] Error: timeout connecting to external API
```

Filter to see only stderr: `/kind:notif`

## Real-World Examples

### Example 1: Debugging a File System Server

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcpsnoop",
      "args": [
        "--",
        "npx",
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/dev/projects"
      ]
    }
  }
}
```

In TUI:
```
# Filter to file operations
/tool:read_file

# Or see all filesystem tools
/method:tools/call

# Check which directory was authorized
# Press 'c' for capabilities, look for "roots"
```

### Example 2: Monitoring a Database Tool

```json
{
  "mcpServers": {
    "database": {
      "command": "mcpsnoop",
      "args": ["--", "python", "db_server.py"],
      "env": {
        "DATABASE_URL": "postgresql://localhost/mydb"
      }
    }
  }
}
```

In TUI:
```
# See only database queries
/tool:query

# Find slow queries
/status:slow tool:query

# Inspect a specific query
# Navigate with j/k, press enter to see full JSON
# Press 'y' to copy query details
```

### Example 3: HTTP Server Debugging

```bash
# Terminal 1: Start your HTTP MCP server
python http_server.py --port 3000

# Terminal 2: Start mcpsnoop proxy
mcpsnoop http --target http://localhost:3000/mcp --listen :7000

# Terminal 3: View traffic
mcpsnoop

# Configure client to connect to http://localhost:7000
```

### Example 4: Multi-Server Setup

```json
{
  "mcpServers": {
    "weather": {
      "command": "mcpsnoop",
      "args": ["--", "python", "weather_server.py"]
    },
    "database": {
      "command": "mcpsnoop",
      "args": ["--", "node", "db-server.js"]
    },
    "filesystem": {
      "command": "mcpsnoop",
      "args": ["--", "npx", "-y", "@modelcontextprotocol/server-filesystem", "/home/user"]
    }
  }
}
```

The TUI shows all servers in separate sessions. Use `:sessions` command to switch between them.

## Troubleshooting

### UI Shows "No sessions"

**Cause:** Shim hasn't started or client hasn't launched the server yet.

**Solution:**
1. Verify mcpsnoop is in your client config
2. Restart your AI client completely
3. Trigger an MCP action to start the server
4. Wait a few seconds and refresh UI

### Tool Call Hangs (Shows PENDING)

**Symptoms:** Request shows `PENDING` with increasing timer, no response.

**Debug:**
1. Check server stderr in the frame list
2. Press `enter` on the PENDING frame to see full request
3. Look for resource locks or external API timeouts
4. Press `r` to replay with debug logging

### Server Not Starting

**Check:**
```bash
# Test server command directly
node build/index.js  # Or your actual command

# Check mcpsnoop can find the command
which node
which mcpsnoop

# Look at client logs
# Claude Desktop: ~/Library/Logs/Claude/
```

### Capabilities Mismatch

**Symptoms:** Client doesn't see tool server advertises.

**Debug:**
1. Press `c` in TUI
2. Compare server capabilities vs client capabilities
3. Check server's `initialize` response
4. Verify tool schema matches MCP spec

### Frames Not Appearing in TUI

**Check:**
1. Socket connection: `ls -la ~/.config/mcpsnoop/` (Linux/macOS)
2. Log directory: Check for `.jsonl` files
3. Permissions: Ensure shim can write logs
4. Try `mcpsnoop demo` to verify TUI works

### Replay Fails

**Possible causes:**
- Server has state dependencies
- External resources unavailable
- Non-idempotent operations

**Solution:**
Check stderr in replay output and compare request parameters.

## Advanced Usage

### Custom Log Directory

```bash
mcpsnoop --log-dir /custom/path
```

### Filtering in Shim Mode

```bash
# Only log specific tools (reduces disk usage)
mcpsnoop --filter tool:search -- python server.py
```

### Exporting Sessions

```bash
# Sessions are stored as JSONL
cat ~/.config/mcpsnoop/sessions/*.jsonl | jq .
```

### Performance Considerations

- mcpsnoop adds minimal latency (microseconds for copying frames)
- Disk I/O is async and buffered
- Large payloads (>1MB) may slow down TUI rendering
- Use filters to reduce noise in busy systems

## Integration with Development Workflow

### During Development

```bash
# Terminal 1: Your editor/IDE
code .

# Terminal 2: mcpsnoop TUI
mcpsnoop

# Terminal 3: Test client or manual testing
# Make changes to server code
# Restart client or use replay feature to test immediately
```

### Testing Tools

```bash
# Capture a successful tool call
# Press 'y' to copy JSON
# Paste into your test suite as expected output

# Replay with modified server code
# Press 'r' repeatedly while iterating
```

### CI/CD Integration

mcpsnoop logs are structured JSONL, suitable for automated testing:

```bash
# Run test, capture logs
mcpsnoop --log-dir ./test-logs -- python server.py &
# Run your MCP client test
# Parse logs
jq 'select(.kind == "response" and .status == "error")' test-logs/*.jsonl
```

## Resources

- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [mcpsnoop GitHub](https://github.com/kerlenton/mcpsnoop)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector) (alternative tool)
- [Demo Walkthrough](https://github.com/kerlenton/mcpsnoop/blob/main/docs/DEMO.md)
