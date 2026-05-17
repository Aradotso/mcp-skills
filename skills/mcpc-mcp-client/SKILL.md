---
name: mcpc-mcp-client
description: Universal CLI client for Model Context Protocol (MCP) with persistent sessions, OAuth, tasks, and JSON output for shell scripting
triggers:
  - how do I use mcpc to connect to an MCP server
  - set up an MCP client session with mcpc
  - call MCP tools from the command line
  - authenticate with OAuth for MCP servers
  - create a persistent MCP session
  - search for MCP tools across sessions
  - use mcpc in code mode for scripting
  - connect to local MCP server with mcpc
---

# mcpc MCP Client Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`mcpc` is a universal command-line client for the Model Context Protocol (MCP) that maps MCP operations to intuitive shell commands. It enables interactive debugging, scripting workflows, and AI agent integration with MCP servers through persistent sessions, OAuth 2.1 authentication, and JSON output mode.

Key capabilities:
- **Persistent sessions**: Keep multiple stateful MCP connections alive simultaneously
- **Full MCP support**: Tools, resources, prompts, async tasks, instructions
- **OAuth 2.1**: Secure authentication with OS keychain credential storage
- **Code mode**: JSON output for shell pipelines (`jq`, `xargs`, scripting)
- **Progressive discovery**: Search tools across sessions to save tokens
- **MCP proxy**: Share sessions across agents while protecting credentials

## Installation

```bash
# Install globally via npm
npm install -g @apify/mcpc

# Or with Bun
bun install -g @apify/mcpc

# Verify installation
mcpc --version
```

**Linux headless setup** (if keychain is needed):
```bash
# Install dependencies
sudo apt-get install libsecret-1-0 gnome-keyring

# Run with keychain
dbus-run-session -- bash -c "echo -n 'password' | gnome-keyring-daemon --unlock && mcpc ..."
```

## Core Commands

### Session Management

```bash
# List all active sessions and OAuth profiles
mcpc
mcpc --json  # JSON output for scripting

# Connect to remote MCP server (HTTPS)
mcpc connect mcp.apify.com @apify
mcpc connect https://mcp.example.com @example

# Connect to local MCP server via config file
mcpc connect ~/.vscode/mcp.json:filesystem @fs
mcpc connect ./mcp-config.json:my-server @local

# Show session info (capabilities, tools overview)
mcpc @apify

# Close a session
mcpc close @apify

# Restart session (loses state)
mcpc restart @apify

# Interactive shell
mcpc @apify shell
```

### Authentication

```bash
# Login with OAuth and save profile
mcpc login mcp.apify.com
mcpc login mcp.apify.com --profile production

# Use saved profile when connecting
mcpc connect mcp.apify.com @apify --profile production

# Logout (delete OAuth profile)
mcpc logout mcp.apify.com
mcpc logout mcp.apify.com --profile production
```

### Tool Discovery and Search

```bash
# Search tools and instructions across all sessions
mcpc grep "search"
mcpc grep "actor" --json

# Search within a single session
mcpc @apify grep "web scraping"

# Search with regex
mcpc grep "search|find" -E

# Case-sensitive search
mcpc grep "Search" --case-sensitive

# Limit results
mcpc grep "data" -m 10

# Search resources and prompts
mcpc grep "config" --resources --prompts
```

## MCP Operations

### Tools

```typescript
// List all tools
mcpc @apify tools-list
mcpc --json @apify tools-list  // JSON output

// Get tool details and schema
mcpc @apify tools-get search-actors

// Call tool with key:=value arguments
mcpc @apify tools-call search-actors keywords:="web scraper"
mcpc @apify tools-call search-actors keywords:="web scraper" limit:=5

// Call with JSON object
mcpc @apify tools-call search-actors '{"keywords":"web scraper","limit":5}'

// Call with stdin
echo '{"keywords":"web scraper","limit":5}' | mcpc @apify tools-call search-actors
cat args.json | mcpc @apify tools-call search-actors

// Force string type with JSON quotes
mcpc @apify tools-call get-item id:='"123"' flag:='"true"'

// Complex nested arguments
mcpc @apify tools-call create-actor 'config:={"timeout":300,"memory":512}'
```

**Argument parsing rules**:
- `key:=value` auto-parses: numbers, booleans, objects stay typed
- Invalid JSON becomes a string: `name:=hello` → `"hello"`
- Force string with quotes: `id:='"123"'` → `"123"` (string)
- No spaces around `:=`
- Quote shell expansions: `"query:=${VAR}"`

### Resources

```typescript
// List resources
mcpc @apify resources-list

// Read resource
mcpc @apify resources-read file:///path/to/resource

// Subscribe to resource updates
mcpc @apify resources-subscribe file:///config.json

// Unsubscribe
mcpc @apify resources-unsubscribe file:///config.json

// List resource templates
mcpc @apify resources-templates-list
```

### Prompts

```typescript
// List prompts
mcpc @apify prompts-list

// Get prompt with arguments
mcpc @apify prompts-get analyze-data dataset:="sales-2024"
mcpc @apify prompts-get analyze-data '{"dataset":"sales-2024","format":"csv"}'

// Pipe arguments
echo '{"dataset":"sales-2024"}' | mcpc @apify prompts-get analyze-data
```

### Async Tasks

```typescript
// List tasks
mcpc @apify tasks-list

// Get task status
mcpc @apify tasks-get task-12345

// Get task result (waits for completion)
mcpc @apify tasks-result task-12345

// Cancel task
mcpc @apify tasks-cancel task-12345
```

### Server Operations

```typescript
// Ping server
mcpc @apify ping

// Set logging level
mcpc @apify logging-set-level debug
mcpc @apify logging-set-level info
```

## Code Mode (JSON Output)

Use `--json` flag for shell scripting and pipelines:

```bash
# Get session list as JSON
mcpc --json | jq '.sessions[] | select(.status == "connected")'

# Extract tool names
mcpc --json @apify tools-list | jq -r '.tools[].name'

# Filter tools by category
mcpc --json @apify tools-list | jq '.tools[] | select(.description | contains("search"))'

# Call tool and parse result
RESULT=$(mcpc --json @apify tools-call search-actors keywords:="crawler")
echo "$RESULT" | jq '.content[0].text'

# Chain multiple operations
mcpc --json @apify tools-list | \
  jq -r '.tools[].name' | \
  xargs -I {} mcpc --json @apify tools-get {}

# Batch process with jq + xargs
echo '["actor1","actor2","actor3"]' | \
  jq -r '.[]' | \
  xargs -I {} mcpc --json @apify tools-call get-actor actorId:="{}"
```

## Configuration

### MCP Config File Format

Reference local MCP servers via config files (Claude Desktop format):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    },
    "postgres": {
      "command": "node",
      "args": ["/path/to/postgres-server/dist/index.js"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

Connect to entries:
```bash
mcpc connect ~/.vscode/mcp.json:filesystem @fs
mcpc connect ./config.json:postgres @db
```

### Environment Variables

```bash
# Use in MCP config files
export DATABASE_URL="postgresql://user:pass@localhost/db"
export API_KEY="your-api-key"

# mcpc options via env
export MCPC_TIMEOUT=600
export MCPC_MAX_CHARS=10000
```

### Global Options

```bash
# Request timeout
mcpc @apify tools-call long-task --timeout 600

# Truncate output
mcpc @apify tools-call verbose-tool --max-chars 5000

# Skip TLS verification (self-signed certs)
mcpc connect https://localhost:3000 @local --insecure

# Debug logging
mcpc --verbose @apify tools-call search-actors keywords:="test"

# Use specific OAuth profile
mcpc connect mcp.apify.com @apify --profile staging
```

## AI Agent Integration

### Bash Tool Pattern

Give agents a single `Bash()` tool with `mcpc` in scope:

```typescript
// Agent tool definition
const bashTool = {
  name: "bash",
  description: "Execute bash commands. mcpc is available for MCP operations.",
  parameters: {
    command: { type: "string", description: "Bash command to execute" }
  }
};

// Agent can now use MCP naturally
await bash("mcpc connect mcp.apify.com @apify");
await bash("mcpc @apify grep 'web scraping'");
await bash("mcpc --json @apify tools-call search-actors keywords:='crawler' | jq -r '.content[0].text'");
```

### Shared Sessions

Multiple agents can share the same `mcpc` sessions:

```bash
# Agent 1: Create session
mcpc connect mcp.apify.com @shared

# Agent 2: Use same session
mcpc @shared tools-list

# Agent 3: Call tools
mcpc @shared tools-call search-actors keywords:="data"
```

### MCP Proxy Mode

Protect credentials from AI-generated code:

```bash
# Start proxy in trusted environment
mcpc connect mcp.apify.com @apify
# Share session name with agent, not credentials

# Agent uses proxy without OAuth access
mcpc @apify tools-call search-actors keywords:="test"
```

## Common Patterns

### Discovery Workflow

```bash
# 1. Connect to server
mcpc connect mcp.apify.com @apify

# 2. Search for relevant tools
mcpc @apify grep "search"

# 3. Get tool details
mcpc @apify tools-get search-actors

# 4. Call tool
mcpc @apify tools-call search-actors keywords:="web crawler"
```

### Scripting Workflow

```bash
#!/bin/bash
set -e

# Connect if not already connected
mcpc connect mcp.apify.com @apify 2>/dev/null || true

# Search for actors
ACTORS=$(mcpc --json @apify tools-call search-actors keywords:="crawler")

# Extract actor IDs
echo "$ACTORS" | jq -r '.content[0].text | fromjson | .items[].id' | while read -r ID; do
  # Get details for each actor
  mcpc --json @apify tools-call get-actor actorId:="$ID"
done
```

### Multi-Server Orchestration

```bash
# Connect to multiple servers
mcpc connect mcp.apify.com @apify
mcpc connect ~/.vscode/mcp.json:filesystem @fs
mcpc connect ./config.json:database @db

# Search across all
mcpc grep "search" --json | jq -r '.results[] | "\(.session): \(.item.name)"'

# Coordinate operations
DATA=$(mcpc --json @fs tools-call read-file path:="/data/input.json")
RESULT=$(echo "$DATA" | mcpc @apify tools-call process-data)
echo "$RESULT" | mcpc @db tools-call store-result
```

### Progressive Tool Discovery

```bash
# Start broad, refine as needed
mcpc grep "actor" --json | jq -r '.results[].item.name' | head -3

# Get details only for relevant tools
mcpc @apify tools-get search-actors

# Call with minimal context
mcpc @apify tools-call search-actors keywords:="web scraping"
```

## Troubleshooting

### Session Issues

```bash
# Check session status
mcpc
mcpc --json | jq '.sessions[] | {name, status}'

# Restart crashed session
mcpc restart @apify

# Clean stale sessions
mcpc clean sessions

# Debug connection
mcpc --verbose connect mcp.apify.com @apify
```

### Authentication Issues

```bash
# Re-authenticate
mcpc logout mcp.apify.com
mcpc login mcp.apify.com

# Use different profile
mcpc login mcp.apify.com --profile staging
mcpc connect mcp.apify.com @apify --profile staging

# Check saved profiles
mcpc --json | jq '.profiles'

# Linux: verify keychain
secret-tool search service mcpc

# Clean credentials
mcpc clean profiles
```

### Tool Call Issues

```bash
# Get tool schema first
mcpc @apify tools-get search-actors

# Validate arguments
echo '{"keywords":"test"}' | jq .  # Validate JSON

# Debug with verbose
mcpc --verbose @apify tools-call search-actors keywords:="test"

# Check argument parsing
mcpc @apify tools-call test-tool \
  string:="hello" \
  number:=42 \
  bool:=true \
  obj:='{"key":"value"}' \
  arr:='[1,2,3]'
```

### Timeout Issues

```bash
# Increase timeout
mcpc @apify tools-call long-task --timeout 600

# For async tasks, use tasks-result
TASK_ID=$(mcpc --json @apify tools-call start-task | jq -r '.taskId')
mcpc @apify tasks-result "$TASK_ID"  # Waits for completion
```

### Credentials on Linux

```bash
# If keychain fails, mcpc falls back to ~/.mcpc/credentials

# Force keychain with gnome-keyring
dbus-run-session -- bash -c "
  echo -n 'password' | gnome-keyring-daemon --unlock
  mcpc login mcp.apify.com
"

# Check file-based credentials (mode 0600)
ls -la ~/.mcpc/credentials
```

### Clean Up

```bash
# Clean specific resources
mcpc clean sessions    # Close all sessions
mcpc clean profiles    # Delete OAuth profiles
mcpc clean logs        # Remove old log files
mcpc clean all         # Everything

# Manual cleanup
rm -rf ~/.mcpc/sessions/*
rm -rf ~/.mcpc/logs/*
```

## Advanced Usage

### Custom Timeout and Limits

```bash
# Per-command timeout
mcpc @apify tools-call slow-operation --timeout 900

# Truncate verbose output
mcpc @apify tools-call get-logs --max-chars 10000

# Combine options
mcpc --json --verbose --timeout 600 @apify tools-call complex-task
```

### Resource Subscriptions

```bash
# Subscribe to config changes
mcpc @apify resources-subscribe config://app-settings

# Monitor in background, process updates
mcpc @apify resources-subscribe config://app-settings &
# Updates arrive asynchronously

# Unsubscribe when done
mcpc @apify resources-unsubscribe config://app-settings
```

### Interactive Shell Commands

```bash
mcpc @apify shell

# Inside shell:
# > help
# > tools-list
# > tools-call search-actors keywords:="crawler"
# > exit
```

Use arrow keys for history, Ctrl+C to cancel, Ctrl+D or `exit` to quit.

## Security Notes

- OAuth tokens stored in OS keychain (macOS Keychain, Windows Credential Manager, Linux Secret Service)
- Fallback to `~/.mcpc/credentials` (mode `0600`) on headless Linux
- Use `--insecure` only for self-signed certs in dev environments
- Environment variables in config files: `"${VAR_NAME}"`
- Never commit credentials or `~/.mcpc/credentials` to version control
