---
name: mcp-server-bash-sdk
description: Build lightweight MCP servers in pure Bash with zero runtime overhead for AI tool integration
triggers:
  - create an MCP server in bash
  - implement MCP protocol with shell scripts
  - build a bash MCP server
  - add tools to bash MCP server
  - configure MCP server in shell
  - write MCP tool functions in bash
  - debug bash MCP server
  - integrate bash MCP with Claude
---

# MCP Server Bash SDK

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The MCP Server Bash SDK is a lightweight, zero-overhead implementation of the Model Context Protocol (MCP) server in pure Bash. It allows you to create MCP servers without Node.js, Python, or other heavy runtimes—just Bash and `jq` for JSON processing. The SDK handles JSON-RPC 2.0 protocol communication over stdio while you focus on implementing tool functions.

**Key Benefits:**
- Zero runtime overhead compared to Node.js/Python
- Simple function-based tool definition
- Automatic tool discovery via naming convention
- External JSON configuration for tools and server metadata
- Perfect for API wrappers and system utilities

## Installation

### Requirements

- Bash shell (4.0+)
- `jq` for JSON processing

Install `jq`:
```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# RHEL/CentOS
sudo yum install jq
```

### Clone and Setup

```bash
git clone https://github.com/muthuishere/mcp-server-bash-sdk
cd mcp-server-bash-sdk
chmod +x mcpserver_core.sh moviemcpserver.sh
```

### Test Installation

```bash
echo '{"jsonrpc": "2.0", "method": "tools/call", "params": {"name": "get_movies"}, "id": 1}' | ./moviemcpserver.sh
```

## Core Concepts

### Architecture

- **mcpserver_core.sh**: Protocol layer handling JSON-RPC and MCP communication
- **Your server script**: Business logic with `tool_*` functions
- **assets/**: JSON configuration files for tools and server metadata
- **Communication**: JSON-RPC 2.0 over stdio

### Tool Function Contract

All tool functions must follow these rules:

1. **Naming**: Prefix with `tool_` + exact name from `tools_list.json`
2. **Parameters**: Accept single parameter `$1` containing JSON arguments
3. **Success**: Echo result and `return 0`
4. **Failure**: Echo error message and `return 1`
5. **Discovery**: Automatically exposed based on `tools_list.json`

## Creating Your First MCP Server

### Step 1: Create Server Script

Create `weatherserver.sh`:

```bash
#!/bin/bash

# Override configuration paths BEFORE sourcing core
MCP_CONFIG_FILE="$(dirname "${BASH_SOURCE[0]}")/assets/weatherserver_config.json"
MCP_TOOLS_LIST_FILE="$(dirname "${BASH_SOURCE[0]}")/assets/weatherserver_tools.json"
MCP_LOG_FILE="$(dirname "${BASH_SOURCE[0]}")/logs/weatherserver.log"

# Source the MCP server core
source "$(dirname "${BASH_SOURCE[0]}")/mcpserver_core.sh"

# Access environment variables
API_KEY="${WEATHER_API_KEY:-}"
BASE_URL="${WEATHER_API_URL:-https://api.openweathermap.org/data/2.5}"

# Tool: Get current weather
tool_get_weather() {
  local args="$1"
  local location=$(echo "$args" | jq -r '.location')
  
  # Validate required parameters
  if [[ -z "$location" ]]; then
    echo "Missing required parameter: location"
    return 1
  fi
  
  if [[ -z "$API_KEY" ]]; then
    echo "WEATHER_API_KEY environment variable not set"
    return 1
  fi
  
  # Call external API
  local response=$(curl -s "${BASE_URL}/weather?q=${location}&appid=${API_KEY}&units=metric")
  
  # Check for API errors
  local cod=$(echo "$response" | jq -r '.cod')
  if [[ "$cod" != "200" ]]; then
    local message=$(echo "$response" | jq -r '.message // "API request failed"')
    echo "Weather API error: $message"
    return 1
  fi
  
  # Return formatted result
  echo "$response" | jq '{
    location: .name,
    temperature: .main.temp,
    description: .weather[0].description,
    humidity: .main.humidity
  }'
  return 0
}

# Tool: Get weather forecast
tool_get_forecast() {
  local args="$1"
  local location=$(echo "$args" | jq -r '.location')
  local days=$(echo "$args" | jq -r '.days // 3')
  
  if [[ -z "$location" ]]; then
    echo "Missing required parameter: location"
    return 1
  fi
  
  # Call forecast API
  local response=$(curl -s "${BASE_URL}/forecast?q=${location}&appid=${API_KEY}&units=metric&cnt=$((days * 8))")
  
  echo "$response" | jq '{
    location: .city.name,
    forecast: [.list[] | {
      date: .dt_txt,
      temp: .main.temp,
      description: .weather[0].description
    }]
  }'
  return 0
}

# Start the MCP server
run_mcp_server "$@"
```

### Step 2: Create Tools Definition

Create `assets/weatherserver_tools.json`:

```json
{
  "tools": [
    {
      "name": "get_weather",
      "description": "Get current weather conditions for a specified location",
      "inputSchema": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "City name (e.g., 'London', 'New York') or coordinates"
          }
        },
        "required": ["location"]
      }
    },
    {
      "name": "get_forecast",
      "description": "Get weather forecast for the next few days",
      "inputSchema": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "City name or coordinates"
          },
          "days": {
            "type": "integer",
            "description": "Number of days to forecast (1-5)",
            "default": 3
          }
        },
        "required": ["location"]
      }
    }
  ]
}
```

### Step 3: Create Server Configuration

Create `assets/weatherserver_config.json`:

```json
{
  "protocolVersion": "2025-03-26",
  "serverInfo": {
    "name": "WeatherServer",
    "version": "1.0.0"
  },
  "capabilities": {
    "tools": {
      "listChanged": true
    }
  },
  "instructions": "Provides weather information and forecasts using OpenWeatherMap API. Requires WEATHER_API_KEY environment variable."
}
```

### Step 4: Make Executable and Test

```bash
chmod +x weatherserver.sh

# Test tool call
echo '{"jsonrpc": "2.0", "method": "tools/call", "params": {"name": "get_weather", "arguments": {"location": "London"}}, "id": 1}' | WEATHER_API_KEY=your_key ./weatherserver.sh

# Test tools list
echo '{"jsonrpc": "2.0", "method": "tools/list", "id": 1}' | ./weatherserver.sh
```

## MCP Protocol Methods

### Initialize

```bash
echo '{"jsonrpc": "2.0", "method": "initialize", "params": {"protocolVersion": "2025-03-26", "capabilities": {}}, "id": 1}' | ./weatherserver.sh
```

### List Tools

```bash
echo '{"jsonrpc": "2.0", "method": "tools/list", "id": 1}' | ./weatherserver.sh
```

### Call Tool

```bash
echo '{"jsonrpc": "2.0", "method": "tools/call", "params": {"name": "get_weather", "arguments": {"location": "Paris"}}, "id": 1}' | ./weatherserver.sh
```

### List Prompts (if configured)

```bash
echo '{"jsonrpc": "2.0", "method": "prompts/list", "id": 1}' | ./weatherserver.sh
```

## Configuration

### Environment Variables

Access environment variables in your tool functions:

```bash
# In your server script
API_KEY="${MY_API_KEY:-default_value}"
BASE_URL="${MY_BASE_URL:-https://api.example.com}"
DEBUG="${MCP_DEBUG:-false}"

tool_example() {
  local args="$1"
  
  if [[ "$DEBUG" == "true" ]]; then
    echo "Debug: Processing with API key: ${API_KEY:0:5}..." >&2
  fi
  
  # Use environment variables in API calls
  curl -H "Authorization: Bearer $API_KEY" "$BASE_URL/endpoint"
}
```

### Custom Configuration Paths

Override default paths before sourcing core:

```bash
#!/bin/bash

# Custom paths
MCP_CONFIG_FILE="/custom/path/config.json"
MCP_TOOLS_LIST_FILE="/custom/path/tools.json"
MCP_LOG_FILE="/var/log/myserver.log"

source "$(dirname "${BASH_SOURCE[0]}")/mcpserver_core.sh"
```

### Logging

Enable debug logging:

```bash
# Set log file path
MCP_LOG_FILE="./logs/debug.log"

# Logs are written to stderr and optionally to file
# Check logs while running
tail -f ./logs/debug.log
```

## Integration with AI Assistants

### VS Code with GitHub Copilot

Add to `.vscode/settings.json`:

```json
{
  "mcp": {
    "servers": {
      "weather-server": {
        "type": "stdio",
        "command": "/absolute/path/to/weatherserver.sh",
        "args": [],
        "env": {
          "WEATHER_API_KEY": "your-api-key",
          "MCP_DEBUG": "false"
        }
      }
    }
  }
}
```

### Claude Desktop

Add to Claude config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "weather-server": {
      "command": "/absolute/path/to/weatherserver.sh",
      "args": [],
      "env": {
        "WEATHER_API_KEY": "your-api-key"
      }
    }
  }
}
```

### Testing with Copilot Chat

```
@workspace /mcp weather-server get weather for Tokyo
```

## Common Patterns

### Pattern: External API Wrapper

```bash
#!/bin/bash

source "$(dirname "${BASH_SOURCE[0]}")/mcpserver_core.sh"

API_TOKEN="${GITHUB_TOKEN:-}"
API_BASE="https://api.github.com"

tool_get_repo_info() {
  local args="$1"
  local owner=$(echo "$args" | jq -r '.owner')
  local repo=$(echo "$args" | jq -r '.repo')
  
  if [[ -z "$owner" ]] || [[ -z "$repo" ]]; then
    echo "Missing required parameters: owner and repo"
    return 1
  fi
  
  local response=$(curl -s -H "Authorization: token $API_TOKEN" \
    "${API_BASE}/repos/${owner}/${repo}")
  
  echo "$response" | jq '{
    name: .name,
    stars: .stargazers_count,
    forks: .forks_count,
    description: .description
  }'
  return 0
}

run_mcp_server "$@"
```

### Pattern: System Command Wrapper

```bash
#!/bin/bash

source "$(dirname "${BASH_SOURCE[0]}")/mcpserver_core.sh"

tool_disk_usage() {
  local args="$1"
  local path=$(echo "$args" | jq -r '.path // "/"')
  
  if [[ ! -d "$path" ]]; then
    echo "Path does not exist: $path"
    return 1
  fi
  
  df -h "$path" | awk 'NR==2 {
    print "{\"path\":\"'$path'\",\"size\":\""$2"\",\"used\":\""$3"\",\"available\":\""$4"\",\"use_percent\":\""$5"\"}"
  }'
  return 0
}

tool_process_list() {
  local args="$1"
  local filter=$(echo "$args" | jq -r '.filter // ""')
  
  ps aux | grep -i "$filter" | head -20 | jq -R -s -c 'split("\n") | map(select(length > 0))'
  return 0
}

run_mcp_server "$@"
```

### Pattern: Data Transformation

```bash
#!/bin/bash

source "$(dirname "${BASH_SOURCE[0]}")/mcpserver_core.sh"

tool_json_to_csv() {
  local args="$1"
  local json_data=$(echo "$args" | jq -r '.data')
  
  if [[ -z "$json_data" ]]; then
    echo "Missing required parameter: data"
    return 1
  fi
  
  echo "$json_data" | jq -r '
    (.[0] | keys_unsorted) as $keys |
    $keys, 
    (.[] | [.[$keys[]]] | @csv)
  ' | paste -sd ',' -
  return 0
}

run_mcp_server "$@"
```

### Pattern: File Operations

```bash
#!/bin/bash

source "$(dirname "${BASH_SOURCE[0]}")/mcpserver_core.sh"

WORKSPACE_DIR="${WORKSPACE_DIR:-./workspace}"

tool_read_file() {
  local args="$1"
  local filepath=$(echo "$args" | jq -r '.path')
  
  local fullpath="${WORKSPACE_DIR}/${filepath}"
  
  if [[ ! -f "$fullpath" ]]; then
    echo "File not found: $filepath"
    return 1
  fi
  
  jq -n --arg content "$(cat "$fullpath")" '{content: $content}'
  return 0
}

tool_write_file() {
  local args="$1"
  local filepath=$(echo "$args" | jq -r '.path')
  local content=$(echo "$args" | jq -r '.content')
  
  local fullpath="${WORKSPACE_DIR}/${filepath}"
  local dir=$(dirname "$fullpath")
  
  mkdir -p "$dir"
  echo "$content" > "$fullpath"
  
  echo "{\"success\": true, \"path\": \"$filepath\"}"
  return 0
}

run_mcp_server "$@"
```

## Troubleshooting

### Server Not Responding

**Symptom**: No output when sending JSON-RPC requests

**Solutions**:
```bash
# 1. Check if script is executable
chmod +x yourserver.sh

# 2. Verify shebang line
head -1 yourserver.sh  # Should show #!/bin/bash

# 3. Test jq installation
which jq
echo '{"test": "value"}' | jq .

# 4. Enable debug logging
MCP_LOG_FILE="./debug.log" ./yourserver.sh
```

### Tool Function Not Found

**Symptom**: Error "Tool not found" when calling a tool

**Solutions**:
```bash
# 1. Verify function name matches tools_list.json
# Function: tool_get_weather
# JSON: "name": "get_weather"

# 2. Check tools_list.json is valid
jq . assets/yourserver_tools.json

# 3. Verify function is defined before run_mcp_server
grep "tool_get_weather" yourserver.sh

# 4. Test tool listing
echo '{"jsonrpc": "2.0", "method": "tools/list", "id": 1}' | ./yourserver.sh
```

### JSON Parsing Errors

**Symptom**: `jq` errors or malformed JSON responses

**Solutions**:
```bash
# 1. Validate input JSON
echo "$args" | jq . 2>&1

# 2. Check for shell expansion in strings
# BAD: echo "{"key": "$value"}"
# GOOD: jq -n --arg val "$value" '{key: $val}'

# 3. Escape special characters properly
local result=$(curl ... | jq -R -s .)

# 4. Debug jq pipeline
echo "$data" | jq . | jq '.field' | jq -c .
```

### Environment Variables Not Set

**Symptom**: Tool fails due to missing API keys

**Solutions**:
```bash
# 1. Check environment in VS Code settings
# Verify "env" object in mcp server config

# 2. Use defaults for optional vars
API_KEY="${MY_API_KEY:-default_key}"

# 3. Validate required vars in tool function
if [[ -z "$API_KEY" ]]; then
  echo "MY_API_KEY environment variable required"
  return 1
fi

# 4. Test with explicit environment
MY_API_KEY=test ./yourserver.sh
```

### cURL or External Command Failures

**Symptom**: Tool returns errors from external commands

**Solutions**:
```bash
# 1. Check command exit status
local response=$(curl -s "https://api.example.com/endpoint")
if [[ $? -ne 0 ]]; then
  echo "API request failed"
  return 1
fi

# 2. Add timeout to curl
curl -s --max-time 30 "https://api.example.com"

# 3. Handle HTTP errors
local http_code=$(curl -s -w "%{http_code}" -o /tmp/response.json "url")
if [[ "$http_code" -ge 400 ]]; then
  echo "HTTP error: $http_code"
  return 1
fi

# 4. Validate response format
if ! echo "$response" | jq -e . >/dev/null 2>&1; then
  echo "Invalid JSON response"
  return 1
fi
```

### Configuration File Not Found

**Symptom**: Server fails to start or reads wrong config

**Solutions**:
```bash
# 1. Use absolute paths
MCP_CONFIG_FILE="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)/assets/config.json"

# 2. Verify paths before sourcing
ls -la "$MCP_CONFIG_FILE" "$MCP_TOOLS_LIST_FILE"

# 3. Check file permissions
chmod 644 assets/*.json

# 4. Create missing directories
mkdir -p assets logs
```

### Debugging Tips

```bash
# Enable verbose output
set -x  # Add at top of script for trace

# Log to stderr (visible in AI assistant logs)
echo "Debug: tool called with: $args" >&2

# Validate JSON at each step
validate_json() {
  if ! echo "$1" | jq . >/dev/null 2>&1; then
    echo "Invalid JSON: $1" >&2
    return 1
  fi
}

# Test individual tool functions
tool_get_weather '{"location":"London"}'
echo "Exit code: $?"
```

## Advanced Examples

### Multi-Step Tool with Error Handling

```bash
tool_github_pr_summary() {
  local args="$1"
  local owner=$(echo "$args" | jq -r '.owner')
  local repo=$(echo "$args" | jq -r '.repo')
  local pr_number=$(echo "$args" | jq -r '.pr_number')
  
  # Validate inputs
  if [[ -z "$owner" ]] || [[ -z "$repo" ]] || [[ -z "$pr_number" ]]; then
    echo "Missing required parameters"
    return 1
  fi
  
  # Step 1: Get PR details
  local pr_data=$(curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
    "https://api.github.com/repos/${owner}/${repo}/pulls/${pr_number}")
  
  if [[ $(echo "$pr_data" | jq -r '.message // empty') == "Not Found" ]]; then
    echo "Pull request not found"
    return 1
  fi
  
  # Step 2: Get PR commits
  local commits=$(curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
    "https://api.github.com/repos/${owner}/${repo}/pulls/${pr_number}/commits")
  
  # Step 3: Aggregate data
  jq -n \
    --argjson pr "$pr_data" \
    --argjson commits "$commits" \
    '{
      title: $pr.title,
      state: $pr.state,
      author: $pr.user.login,
      commits: ($commits | length),
      additions: $pr.additions,
      deletions: $pr.deletions,
      files_changed: $pr.changed_files
    }'
  
  return 0
}
```

This skill provides comprehensive guidance for building, configuring, and troubleshooting MCP servers using the Bash SDK.
