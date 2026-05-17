---
name: mcp-cli-tool
description: Lightweight CLI for interacting with MCP (Model Context Protocol) servers - discover, inspect, and execute MCP tools from the command line
triggers:
  - how do I use mcp-cli to call MCP servers
  - list available MCP tools with mcp-cli
  - configure mcp-cli for MCP servers
  - call an MCP tool from the command line
  - discover MCP server capabilities
  - chain mcp-cli commands with jq
  - set up mcp servers config file
  - troubleshoot mcp-cli connection issues
---

# mcp-cli Tool

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`mcp-cli` is a lightweight, Bun-based CLI for interacting with [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) servers. It provides:

- **On-demand schema loading** - Only fetch tool schemas when needed, saving AI context tokens
- **Shell composability** - JSON output for piping with `jq`, chaining, and scripting
- **Connection pooling** - Lazy-spawn daemon keeps connections warm (60s idle timeout)
- **Universal support** - Works with both stdio and HTTP MCP servers
- **Tool filtering** - Allow/disable specific tools per server via config

**Use cases:**
- AI agents accessing MCP tools without loading full schemas into context
- Shell scripts automating MCP server interactions
- Discovering and exploring available MCP capabilities

## Installation

### Quick Install (Recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/philschmid/mcp-cli/main/install.sh | bash
```

### Manual Install (Requires Bun)

```bash
bun install -g https://github.com/philschmid/mcp-cli
```

### Verify Installation

```bash
mcp-cli --version
```

## Configuration

### Config File Location

Create `mcp_servers.json` in one of these locations (searched in order):

1. Path from `MCP_CONFIG_PATH` environment variable
2. Path from `-c/--config` CLI argument
3. `./mcp_servers.json` (current directory)
4. `~/.mcp_servers.json`
5. `~/.config/mcp/mcp_servers.json`

### Basic Config Format

Compatible with Claude Desktop, Gemini, and VS Code:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "."
      ]
    },
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "deepwiki": {
      "url": "https://mcp.deepwiki.com/mcp"
    }
  }
}
```

### Advanced Config Options

```json
{
  "mcpServers": {
    "local-server": {
      "command": "node",
      "args": ["./server.js"],
      "env": {
        "API_KEY": "${API_KEY}"
      },
      "cwd": "/path/to/directory",
      "allowedTools": ["read_*", "list_*"],
      "disabledTools": ["delete_*"]
    },
    "remote-server": {
      "url": "https://mcp.example.com",
      "headers": {
        "Authorization": "Bearer ${TOKEN}"
      }
    }
  }
}
```

### Environment Variable Substitution

Use `${VAR_NAME}` syntax anywhere in config:

```json
{
  "mcpServers": {
    "api-server": {
      "command": "node",
      "args": ["server.js"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}",
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

**Control behavior:**
- `MCP_STRICT_ENV=true` (default): Error on missing variables
- `MCP_STRICT_ENV=false`: Use empty values with warning

### Tool Filtering

Restrict available tools using glob patterns:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."],
      "allowedTools": ["read_file", "list_directory"],
      "disabledTools": ["delete_file"]
    }
  }
}
```

**Rules:**
- `allowedTools`: Whitelist (supports `*`, `?` glob patterns)
- `disabledTools`: Blacklist (takes precedence over allowedTools)
- Applies to all operations (info, grep, call)

**Examples:**

```json
// Only read operations
"allowedTools": ["read_*", "list_*", "search_*"]

// Disable destructive operations
"disabledTools": ["delete_*", "write_*", "create_*"]

// Combine filters
"allowedTools": ["*file*"],
"disabledTools": ["delete_file"]
```

## Core Commands

### List All Servers and Tools

```bash
# Basic listing
mcp-cli

# With descriptions
mcp-cli -d
```

**Output:**
```
github
  • search_repositories
  • get_file_contents
  • create_or_update_file
filesystem
  • read_file
  • write_file
  • list_directory
```

### Search Tools by Pattern

```bash
# Find file-related tools
mcp-cli grep "*file*"

# Search with descriptions
mcp-cli grep "*search*" -d
```

**Output:**
```
github/get_file_contents
github/create_or_update_file
filesystem/read_file
filesystem/write_file
```

### View Server Details

```bash
mcp-cli info <server>
```

**Example:**
```bash
mcp-cli info github
```

**Output:**
```
Server: github
Transport: stdio
Command: npx -y @modelcontextprotocol/server-github

Tools (12):
  search_repositories
    Search for GitHub repositories
    Parameters:
      • query (string, required) - Search query
      • page (number, optional) - Page number
  ...
```

### View Tool Schema

Both formats work:
```bash
mcp-cli info <server> <tool>
mcp-cli info <server>/<tool>
```

**Example:**
```bash
mcp-cli info github search_repositories
```

**Output:**
```
Tool: search_repositories
Server: github

Description:
  Search for GitHub repositories

Input Schema:
  {
    "type": "object",
    "properties": {
      "query": { "type": "string", "description": "Search query" },
      "page": { "type": "number" }
    },
    "required": ["query"]
  }
```

### Call a Tool

```bash
# Inline JSON
mcp-cli call <server> <tool> '{"key": "value"}'

# From stdin (auto-detected, no '-' needed)
echo '{"path": "./file"}' | mcp-cli call <server> <tool>

# Heredoc for complex JSON
mcp-cli call <server> <tool> <<EOF
{"content": "Text with 'quotes' and \"escapes\""}
EOF
```

**Example:**
```bash
mcp-cli call github search_repositories '{"query": "mcp server", "per_page": 5}'
```

**Output (JSON):**
```json
{
  "content": [
    {
      "type": "text",
      "text": "{\"items\": [{\"name\": \"mcp-cli\", \"url\": \"...\"}]}"
    }
  ]
}
```

## Practical Usage Patterns

### 1. Discover → Inspect → Execute Workflow

```bash
# Step 1: List available servers
mcp-cli

# Step 2: View server tools
mcp-cli info filesystem

# Step 3: Get tool schema
mcp-cli info filesystem read_file

# Step 4: Call the tool
mcp-cli call filesystem read_file '{"path": "./README.md"}'
```

### 2. Pipe JSON with jq

```bash
# Extract specific field
mcp-cli call github search_repositories '{"query": "mcp"}' | jq '.content[0].text'

# Parse nested JSON
mcp-cli call github search_repositories '{"query": "mcp"}' \
  | jq -r '.content[0].text | fromjson | .items[].html_url'

# Filter results
mcp-cli call filesystem list_directory '{"path": "."}' \
  | jq -r '.content[0].text | split("\n")[] | select(endswith(".md"))'
```

### 3. Chain Multiple MCP Calls

```bash
# Search and read first result
mcp-cli call filesystem search_files '{"path": "src/", "pattern": "*.ts"}' \
  | jq -r '.content[0].text | split("\n")[0]' \
  | xargs -I {} mcp-cli call filesystem read_file '{"path": "{}"}'

# Process all matching files
mcp-cli call filesystem search_files '{"path": ".", "pattern": "*.md"}' \
  | jq -r '.content[0].text | split("\n")[]' \
  | while read file; do
      echo "=== $file ==="
      mcp-cli call filesystem read_file "{\"path\": \"$file\"}" | jq -r '.content[0].text'
    done
```

### 4. Error Handling in Scripts

```bash
# Conditional execution
mcp-cli call filesystem list_directory '{"path": "."}' \
  | jq -e '.content[0].text | contains("README.md")' \
  && mcp-cli call filesystem read_file '{"path": "./README.md"}'

# Fallback on error
if result=$(mcp-cli call filesystem read_file '{"path": "./config.json"}' 2>/dev/null); then
  echo "$result" | jq '.content[0].text | fromjson'
else
  echo "File not found, using defaults"
fi
```

### 5. Save Output to File

```bash
mcp-cli call github get_file_contents '{"owner": "user", "repo": "project", "path": "src/main.ts"}' \
  | jq -r '.content[0].text' > main.ts
```

### 6. Aggregate Results from Multiple Servers

```bash
{
  mcp-cli call github search_repositories '{"query": "mcp", "per_page": 3}'
  mcp-cli call filesystem list_directory '{"path": "./src"}'
} | jq -s '.'
```

### 7. Complex JSON Arguments

For JSON with special characters, use stdin to avoid shell escaping:

```bash
# Heredoc (recommended)
mcp-cli call server tool <<EOF
{
  "content": "Text with 'single quotes' and \"double quotes\"",
  "metadata": {
    "nested": "value"
  }
}
EOF

# From file
cat args.json | mcp-cli call server tool

# Using jq to build complex JSON
jq -n '{query: "mcp", filters: ["active", "starred"]}' | mcp-cli call github search
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `MCP_CONFIG_PATH` | Path to config file | (none) |
| `MCP_DEBUG` | Enable debug output | `false` |
| `MCP_TIMEOUT` | Request timeout (seconds) | `1800` |
| `MCP_CONCURRENCY` | Servers processed in parallel | `5` |
| `MCP_MAX_RETRIES` | Retry attempts for transient errors | `3` |
| `MCP_RETRY_DELAY` | Base retry delay (milliseconds) | `1000` |
| `MCP_STRICT_ENV` | Error on missing `${VAR}` in config | `true` |
| `MCP_NO_DAEMON` | Disable connection caching | `false` |
| `MCP_DAEMON_TIMEOUT` | Idle timeout for cached connections (seconds) | `60` |

**Example:**
```bash
# Enable debug mode
export MCP_DEBUG=true
mcp-cli info github

# Use custom config
export MCP_CONFIG_PATH=/path/to/config.json
mcp-cli

# Disable connection pooling
export MCP_NO_DAEMON=true
mcp-cli call server tool '{}'
```

## CLI Options

| Option | Description |
|--------|-------------|
| `-h, --help` | Show help message |
| `-v, --version` | Show version number |
| `-d, --with-descriptions` | Include tool descriptions |
| `-c, --config <path>` | Path to config file |

**Example:**
```bash
# List with descriptions
mcp-cli -d

# Use custom config
mcp-cli -c ./my-config.json info github

# Get help
mcp-cli --help
```

## Common MCP Server Setups

### Filesystem Server

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/directory"
      ],
      "allowedTools": ["read_file", "list_directory", "search_files"],
      "disabledTools": ["delete_file"]
    }
  }
}
```

**Usage:**
```bash
mcp-cli call filesystem read_file '{"path": "./README.md"}'
mcp-cli call filesystem list_directory '{"path": "."}'
```

### GitHub Server

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Usage:**
```bash
mcp-cli call github search_repositories '{"query": "mcp server"}'
mcp-cli call github get_file_contents '{"owner": "user", "repo": "repo", "path": "README.md"}'
```

### HTTP Server

```json
{
  "mcpServers": {
    "api": {
      "url": "https://mcp.example.com",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

**Usage:**
```bash
mcp-cli info api
mcp-cli call api tool_name '{"param": "value"}'
```

### Custom Local Server

```json
{
  "mcpServers": {
    "custom": {
      "command": "node",
      "args": ["./my-server.js"],
      "cwd": "/path/to/project",
      "env": {
        "PORT": "3000",
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

## Troubleshooting

### Config File Not Found

**Error:**
```
Error: No MCP configuration found
```

**Solution:**
1. Create `mcp_servers.json` in current directory or `~/.config/mcp/`
2. Or set `MCP_CONFIG_PATH` environment variable
3. Or use `-c` flag: `mcp-cli -c /path/to/config.json`

### Server Not Starting

**Error:**
```
Error: Failed to connect to server 'github'
```

**Solution:**
1. Check command is installed: `npx @modelcontextprotocol/server-github --version`
2. Verify environment variables are set
3. Enable debug mode: `MCP_DEBUG=true mcp-cli info github`
4. Check server logs in stderr

### Missing Environment Variable

**Error:**
```
Error: Environment variable GITHUB_TOKEN not found
```

**Solution:**
1. Set the variable: `export GITHUB_TOKEN=your_token`
2. Or use `MCP_STRICT_ENV=false` to allow empty values (not recommended)

### Tool Not Found

**Error:**
```
Error: Tool 'invalid_tool' not found on server 'github'
```

**Solution:**
1. List available tools: `mcp-cli info github`
2. Check for typos in tool name
3. Verify tool isn't disabled in config (`disabledTools`)

### Tool Filtered Out

**Error:**
```
Error: Tool 'write_file' is disabled by configuration
```

**Solution:**
1. Check config `allowedTools` and `disabledTools`
2. Update config to allow the tool
3. Remove or modify filtering rules

### JSON Parsing Error

**Error:**
```
Error: Invalid JSON arguments
```

**Solution:**
1. Use stdin for complex JSON: `echo '{}' | mcp-cli call server tool`
2. Escape quotes properly: `'{"key": "value"}'`
3. Use heredoc for multi-line JSON

### Connection Timeout

**Error:**
```
Error: Request timeout after 1800 seconds
```

**Solution:**
1. Increase timeout: `export MCP_TIMEOUT=3600`
2. Check server responsiveness
3. Disable daemon for fresh connection: `export MCP_NO_DAEMON=true`

### Permission Denied

**Error:**
```
Error: EACCES: permission denied
```

**Solution:**
1. Check file permissions in `cwd` directory
2. Verify user has permission to execute command
3. For filesystem server, ensure paths are accessible

## AI Agent Integration

### System Prompt Template

Add this to your AI agent's system prompt:

```markdown
## MCP Servers

You have access to MCP servers via the `mcp-cli` CLI.

Commands:
- `mcp-cli info` - List all servers
- `mcp-cli info <server>` - Show server tools  
- `mcp-cli info <server> <tool>` - Get tool schema
- `mcp-cli grep "<pattern>"` - Search tools
- `mcp-cli call <server> <tool> '{}'` - Call with JSON args
- `echo '{}' | mcp-cli call <server> <tool>` - Call from stdin

Workflow:
1. Discover: `mcp-cli info` to see available servers
2. Inspect: `mcp-cli info <server> <tool>` to get schema
3. Execute: `mcp-cli call <server> <tool> '{}'` with arguments

Use stdin for complex JSON to avoid shell escaping issues.
```

### Token-Efficient Workflow

Instead of loading all tool schemas:

```bash
# ❌ Avoid: Loading everything into context
mcp-cli -d  # Thousands of tokens

# ✅ Efficient: On-demand loading
mcp-cli                               # List servers (minimal tokens)
mcp-cli info github                   # Show tools when needed
mcp-cli info github search_repositories  # Get schema only when calling
mcp-cli call github search_repositories '{"query": "mcp"}'
```

### Script Generation Example

AI can generate shell scripts combining multiple MCP calls:

```bash
#!/bin/bash
# Generated by AI agent to analyze repository

# Search for repos
repos=$(mcp-cli call github search_repositories '{"query": "mcp server", "per_page": 5}')

# Extract URLs
urls=$(echo "$repos" | jq -r '.content[0].text | fromjson | .items[].html_url')

# For each repo, get README
for url in $urls; do
  owner=$(echo "$url" | cut -d'/' -f4)
  repo=$(echo "$url" | cut -d'/' -f5)
  
  echo "=== $owner/$repo ==="
  mcp-cli call github get_file_contents "{\"owner\": \"$owner\", \"repo\": \"$repo\", \"path\": \"README.md\"}" \
    | jq -r '.content[0].text'
done
```

## Best Practices

1. **Use stdin for complex JSON** - Avoid shell escaping issues
2. **Enable descriptions sparingly** - Use `-d` only when needed to save tokens
3. **Filter tools in config** - Reduce attack surface and simplify discovery
4. **Set environment variables** - Use `${VAR}` substitution for secrets
5. **Pipe to jq** - Process JSON output efficiently
6. **Check schemas first** - Use `info` before `call` to validate arguments
7. **Handle errors** - Check exit codes and stderr in scripts
8. **Use connection pooling** - Default daemon keeps connections warm
9. **Search before listing** - Use `grep` for specific tools instead of listing all
