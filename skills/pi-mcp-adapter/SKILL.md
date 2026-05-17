---
name: pi-mcp-adapter
description: Token-efficient MCP adapter for Pi coding agent that enables MCP server integration without burning context window
triggers:
  - how do I use MCP servers with Pi
  - configure pi-mcp-adapter for my project
  - add MCP server to Pi agent
  - set up direct tools in MCP
  - connect chrome devtools MCP to Pi
  - troubleshoot MCP server connection
  - configure lazy loading for MCP servers
  - use OAuth with MCP servers in Pi
---

# pi-mcp-adapter

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

pi-mcp-adapter is a token-efficient proxy for Model Context Protocol (MCP) servers in the Pi coding agent. Instead of exposing hundreds of tool definitions (10k+ tokens per server), it provides a single ~200 token proxy tool that discovers MCP capabilities on-demand. Servers lazy-load when needed, keeping your context window clean.

**Key benefits:**
- **One proxy tool** instead of hundreds cluttering context
- **Lazy server loading** — only connect when tools are called
- **Direct tool registration** for high-priority tools
- **Standard MCP file support** — reads `.mcp.json` and `~/.config/mcp/mcp.json`
- **Interactive configuration** via `/mcp` overlay

## Installation

```bash
pi install npm:pi-mcp-adapter
```

Restart Pi after installation. The adapter auto-detects standard MCP config files on first run.

## Configuration Files

### File Precedence

Pi reads MCP configs in this order (later overrides earlier):

1. `~/.config/mcp/mcp.json` — User-global shared config
2. `~/.pi/agent/mcp.json` — Pi global override (or `$PI_CODING_AGENT_DIR/mcp.json`)
3. `.mcp.json` — Project-local shared config
4. `.pi/mcp.json` — Pi project override

### Basic Server Configuration

Create `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
    }
  }
}
```

### HTTP/SSE Servers

For servers with HTTP endpoints:

```json
{
  "mcpServers": {
    "figma": {
      "url": "http://localhost:3845/mcp",
      "headers": {
        "Authorization": "Bearer ${FIGMA_TOKEN}"
      }
    }
  }
}
```

### OAuth Configuration

For servers requiring OAuth:

```json
{
  "mcpServers": {
    "google-drive": {
      "command": "npx",
      "args": ["-y", "gdrive-mcp"],
      "auth": "oauth",
      "oauth": {
        "grantType": "authorization_code"
      }
    },
    "api-service": {
      "url": "https://api.example.com/mcp",
      "auth": "oauth",
      "oauth": {
        "grantType": "client_credentials"
      },
      "env": {
        "CLIENT_ID": "${API_CLIENT_ID}",
        "CLIENT_SECRET": "${API_CLIENT_SECRET}"
      }
    }
  },
  "settings": {
    "autoAuth": true
  }
}
```

**OAuth flow:**
- With `autoAuth: true`, adapter runs OAuth automatically on first tool call
- Without `autoAuth`, run `/mcp` and press Enter on the server or use `ctrl+a`
- `client_credentials` grant skips interactive prompts for machine-to-machine auth

## Using MCP Tools

### Proxy Mode (Default)

All MCP tools are accessed through the `mcp` proxy tool:

```typescript
// Search for available tools
mcp({ search: "screenshot" })
// Returns: chrome_devtools_take_screenshot - Take a screenshot of the page...

// Describe a specific tool
mcp({ describe: "chrome_devtools_take_screenshot" })
// Returns: Full parameter schema

// Call a tool (args must be JSON string)
mcp({ 
  tool: "chrome_devtools_take_screenshot", 
  args: '{"format": "png", "fullPage": true}' 
})
```

**Available proxy actions:**
- `{ search: "query" }` — Find tools by keyword
- `{ list: true }` — List all available tools
- `{ describe: "tool_name" }` — Get tool details
- `{ tool: "name", args: "json" }` — Execute a tool
- `{ action: "resources", server: "name" }` — List server resources
- `{ action: "read-resource", server: "name", uri: "file://..." }` — Read resource content
- `{ action: "ui-messages" }` — Get pending UI messages

### Direct Tools

Register specific tools individually for immediate LLM visibility:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"],
      "directTools": true
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "directTools": ["search_repositories", "get_file_contents"]
    }
  }
}
```

**Direct tool options:**
- `true` — Register all tools from this server
- `["tool_a", "tool_b"]` — Register only specified tools
- `false` or omitted — Proxy only (default)

**Token cost:** Each direct tool adds ~150-300 tokens to system prompt. Good for 5-20 high-priority tools.

### Global Direct Tool Default

```json
{
  "settings": {
    "directTools": true
  },
  "mcpServers": {
    "huge-server": {
      "command": "npx",
      "args": ["-y", "mega-mcp@latest"],
      "directTools": false
    }
  }
}
```

### Excluding Tools

Hide specific tools from proxy and direct registration:

```json
{
  "mcpServers": {
    "figma": {
      "url": "http://localhost:3845/mcp",
      "directTools": true,
      "excludeTools": ["get_figjam", "figma_get_code_connect_map"]
    }
  }
}
```

## Lifecycle Management

### Lifecycle Modes

```json
{
  "mcpServers": {
    "lazy-server": {
      "command": "npx",
      "args": ["-y", "some-mcp"],
      "lifecycle": "lazy",
      "idleTimeout": 10
    },
    "eager-server": {
      "command": "npx",
      "args": ["-y", "another-mcp"],
      "lifecycle": "eager"
    },
    "critical-server": {
      "command": "npx",
      "args": ["-y", "important-mcp"],
      "lifecycle": "keep-alive"
    }
  }
}
```

**Lifecycle options:**
- **`lazy`** (default) — Connect on first use, disconnect after idle timeout. Cached metadata keeps search working without connections.
- **`eager`** — Connect at startup, no auto-reconnect. No idle timeout unless explicitly set.
- **`keep-alive`** — Connect at startup, auto-reconnect, never timeout. For always-available servers.

### Idle Timeout

```json
{
  "settings": {
    "idleTimeout": 10
  },
  "mcpServers": {
    "keep-warm": {
      "command": "npx",
      "args": ["-y", "warm-mcp"],
      "idleTimeout": 0
    }
  }
}
```

Global `idleTimeout` (default 10 minutes) applies to all servers. Set to `0` to disable. Per-server overrides global setting.

## Tool Prefixing

Control how tool names are prefixed:

```json
{
  "settings": {
    "toolPrefix": "server"
  }
}
```

**Prefix modes:**
- `"server"` (default) — `chrome_devtools_take_screenshot`
- `"short"` — Strips `-mcp` suffix: `chrome_devtools_take_screenshot`
- `"none"` — No prefix: `take_screenshot`

## Interactive Configuration

### `/mcp` Overlay

Run `/mcp` in Pi to open interactive panel:

- View all configured servers with connection status
- See tool counts and lifecycle modes
- Toggle tools between direct and proxy registration
- Reconnect servers manually
- Trigger OAuth authentication (Enter on server or `ctrl+a`)

### `/mcp setup` Guided Setup

Run `/mcp setup` for first-time configuration:

1. **Detect existing configs** — Scans for `.mcp.json`, `~/.config/mcp/mcp.json`, and host-specific configs (Cursor, Claude Code, etc.)
2. **Import compatibility** — Adopt host configs into Pi with preview before writing
3. **Scaffold minimal config** — Create basic `.mcp.json` with prompts
4. **Quick-add RepoPrompt** — One-shot add popular MCP servers
5. **Preview file changes** — Shows exact before/after diffs before any writes

### CLI Tool

After installation, `pi-mcp-adapter` CLI is available:

```bash
# Scan for host configs and create compatibility imports
pi-mcp-adapter init

# Show detected config files
pi-mcp-adapter info
```

## Environment Variables

### Variable Interpolation

Supports `${VAR}` and `$env:VAR` syntax:

```json
{
  "mcpServers": {
    "db": {
      "command": "npx",
      "args": ["-y", "db-mcp"],
      "env": {
        "DATABASE_URL": "${DB_URL}",
        "API_KEY": "$env:SECRET_KEY"
      },
      "cwd": "${PROJECT_ROOT}/data"
    }
  }
}
```

**Home directory expansion:**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "~/Documents"]
    }
  }
}
```

## Advanced Settings

### Complete Settings Reference

```json
{
  "settings": {
    "toolPrefix": "server",
    "idleTimeout": 10,
    "directTools": false,
    "disableProxyTool": false,
    "autoAuth": false,
    "sampling": true,
    "samplingAutoApprove": false
  }
}
```

**Settings:**
- `toolPrefix` — `"server"`, `"short"`, or `"none"`
- `idleTimeout` — Minutes before disconnect (0 to disable)
- `directTools` — Global default for all servers
- `disableProxyTool` — Hide `mcp` proxy once direct tools are cached
- `autoAuth` — Auto-run OAuth on tool calls
- `sampling` — Allow MCP servers to sample through Pi models
- `samplingAutoApprove` — Skip sampling confirmation prompts (required for non-UI sessions)

### Debugging

Enable server stderr output:

```json
{
  "mcpServers": {
    "problematic": {
      "command": "npx",
      "args": ["-y", "debug-mcp"],
      "debug": true
    }
  }
}
```

## MCP UI Integration

Servers can ship interactive UIs via [MCP UI standard](https://github.com/MCP-UI-Org/mcp-ui). When a tool returns UI metadata, pi-mcp-adapter opens it in a native window (macOS with Glimpse) or browser.

### Installing Glimpse (macOS)

```bash
pi install npm:glimpseui
```

**Force browser rendering:**

```bash
export MCP_UI_VIEWER=browser
```

**Require native rendering:**

```bash
export MCP_UI_VIEWER=glimpse
```

### UI Communication Flow

1. Agent calls tool with UI resource: `mcp({ tool: "launch_dashboard", args: "{}" })`
2. Adapter fetches UI HTML from `_meta.ui.resourceUri`
3. UI opens in native window or browser
4. UI can call MCP tools and send messages back
5. Agent retrieves messages: `mcp({ action: "ui-messages" })`
6. Agent responds, enabling bidirectional conversation

### Session Reuse

When the agent calls the same tool while its UI is open, the adapter pushes new results to the existing window instead of replacing it. Enables live updates without losing current view.

## Common Patterns

### Multi-Server Project Setup

```json
{
  "settings": {
    "toolPrefix": "server",
    "idleTimeout": 15,
    "directTools": false
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${PROJECT_ROOT}"],
      "lifecycle": "keep-alive",
      "directTools": ["read_file", "write_file", "list_directory"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      },
      "directTools": ["search_repositories", "get_file_contents", "create_issue"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

### Working with Resources

MCP servers can expose resources (files, data) that aren't tools:

```typescript
// List resources from a server
mcp({ action: "resources", server: "filesystem" })
// Returns: List of available resources with URIs

// Read a specific resource
mcp({ 
  action: "read-resource", 
  server: "filesystem", 
  uri: "file:///Users/me/project/README.md" 
})
```

**Enable/disable resource tools:**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "~/projects"],
      "exposeResources": true
    }
  }
}
```

Default is `true`. Set to `false` to hide resource-access tools.

### Subagent Integration

Subagents can request direct MCP tools in frontmatter:

```markdown
---
name: github-reviewer
tools:
  - read
  - edit
  - mcp:github
---

Review pull request and create detailed comments using GitHub MCP tools.
```

`mcp:server-name` syntax requests all direct tools from that server.

## Troubleshooting

### Server Won't Connect

**Check config syntax:**

```bash
cat .mcp.json | jq .
```

**Enable debug output:**

```json
{
  "mcpServers": {
    "failing-server": {
      "command": "npx",
      "args": ["-y", "problem-mcp"],
      "debug": true
    }
  }
}
```

**Force reconnect:**

Run `/mcp` and press Enter on the server, or:

```typescript
mcp({ action: "reconnect", server: "failing-server" })
```

### Tools Not Appearing

**Check cache:**

Direct tools load from cache at `~/.pi/agent/mcp-cache.json`. On first run after adding `directTools`, cache won't exist yet.

**Force cache refresh:**

```bash
# In Pi
/mcp reconnect <server-name>
```

**Verify tool registration:**

```typescript
// List all available tools
mcp({ list: true })

// Search for specific tool
mcp({ search: "expected_tool_name" })
```

### OAuth Not Working

**Manual OAuth trigger:**

1. Run `/mcp`
2. Navigate to OAuth server
3. Press Enter or `ctrl+a`

**Enable auto-auth:**

```json
{
  "settings": {
    "autoAuth": true
  }
}
```

**Check client credentials:**

For `client_credentials` grant, ensure env vars are set:

```bash
export API_CLIENT_ID="your-client-id"
export API_CLIENT_SECRET="your-secret"
```

### High Token Usage

**Problem:** Too many direct tools burning context.

**Solution 1 — Use proxy only:**

```json
{
  "mcpServers": {
    "huge-server": {
      "command": "npx",
      "args": ["-y", "mega-mcp"],
      "directTools": false
    }
  }
}
```

**Solution 2 — Select specific tools:**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "directTools": ["search_repositories", "get_file_contents"]
    }
  }
}
```

**Solution 3 — Hide proxy tool:**

```json
{
  "settings": {
    "disableProxyTool": true,
    "directTools": true
  }
}
```

Only works when all needed tools are registered direct and cached.

### Environment Variables Not Expanding

**Ensure proper syntax:**

```json
{
  "env": {
    "CORRECT": "${MY_VAR}",
    "ALSO_CORRECT": "$env:MY_VAR",
    "WRONG": "$MY_VAR"
  }
}
```

**Check variable is exported:**

```bash
echo $MY_VAR
# Should print value

export MY_VAR="value"
```

### Server Keeps Disconnecting

**Adjust idle timeout:**

```json
{
  "mcpServers": {
    "chatty-server": {
      "command": "npx",
      "args": ["-y", "active-mcp"],
      "idleTimeout": 0
    }
  }
}
```

**Or use keep-alive:**

```json
{
  "mcpServers": {
    "critical": {
      "command": "npx",
      "args": ["-y", "important-mcp"],
      "lifecycle": "keep-alive"
    }
  }
}
```

## File Locations

- **Global shared config:** `~/.config/mcp/mcp.json`
- **Pi global override:** `~/.pi/agent/mcp.json` (or `$PI_CODING_AGENT_DIR/mcp.json`)
- **Project shared config:** `.mcp.json`
- **Pi project override:** `.pi/mcp.json`
- **Tool cache:** `~/.pi/agent/mcp-cache.json` (or `$PI_CODING_AGENT_DIR/mcp-cache.json`)

## Key Commands Summary

| Command | Purpose |
|---------|---------|
| `/mcp` | Open interactive config overlay |
| `/mcp setup` | Guided first-run configuration |
| `/mcp reconnect <server>` | Force server reconnection |
| `mcp({ search: "query" })` | Find tools by keyword |
| `mcp({ list: true })` | List all available tools |
| `mcp({ describe: "tool" })` | Get tool schema |
| `mcp({ tool: "name", args: "json" })` | Execute MCP tool |
| `mcp({ action: "resources", server: "name" })` | List server resources |
| `mcp({ action: "ui-messages" })` | Get pending UI messages |

## Example: Complete Chrome DevTools Setup

```json
{
  "settings": {
    "toolPrefix": "server",
    "idleTimeout": 5
  },
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"],
      "lifecycle": "lazy",
      "directTools": [
        "take_screenshot",
        "get_console_logs",
        "execute_javascript"
      ],
      "excludeTools": ["deprecated_tool"]
    }
  }
}
```

**Usage:**

```typescript
// Direct tool (appears in agent's tool list)
chrome_devtools_take_screenshot({ format: "png", fullPage: true })

// Or via proxy
mcp({ 
  tool: "chrome_devtools_get_console_logs", 
  args: '{}' 
})

// Search for other tools
mcp({ search: "navigate" })
```
