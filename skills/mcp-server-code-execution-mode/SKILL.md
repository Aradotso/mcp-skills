---
name: mcp-server-code-execution-mode
description: Execute Python code in isolated rootless containers with MCP server proxying to reduce context bloat from 30K to 200 tokens
triggers:
  - run python code in a secure container
  - execute code with MCP server access
  - reduce MCP tool context overhead
  - proxy MCP servers through code execution
  - discover and use MCP tools dynamically
  - run isolated python with tool discovery
  - setup code execution mode for Claude
  - search for MCP tools at runtime
---

# MCP Server Code Execution Mode

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This MCP server implements Anthropic's "Code Execution with MCP" pattern, exposing a single `run_python` tool instead of hundreds of individual tools. The agent writes Python code to discover, call, and compose MCP tools dynamically, reducing context overhead from ~30,000 tokens to ~200 tokens.

**Key capabilities:**
- Execute Python in rootless Podman/Docker containers
- Proxy any stdio MCP server into the sandbox
- Discover tools at runtime (no context preloading)
- Fuzzy search across all connected servers
- Persistent sessions (variables/state retained)
- Security: no network, read-only filesystem, dropped capabilities

## Installation

### Prerequisites

Install a container runtime (rootless mode):

```bash
# macOS (Podman Desktop recommended)
brew install podman
podman machine init
podman machine start

# Linux
sudo apt install podman  # or dnf/yum/pacman
podman system migrate    # enable rootless

# Windows (WSL2 + Podman Desktop)
# Download from https://podman-desktop.io/
```

### Install the Bridge

```bash
# Via pip
pip install mcp-code-execution

# Via uv (recommended)
uv pip install mcp-code-execution

# From source
git clone https://github.com/elusznik/mcp-server-code-execution-mode.git
cd mcp-server-code-execution-mode
uv pip install -e .
```

### Pull the Container Image

```bash
# Pre-built image (recommended)
podman pull ghcr.io/elusznik/mcp-code-execution:latest

# Or build custom image
podman build -t mcp-code-execution:custom -f Dockerfile .
```

## Configuration

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "code-execution": {
      "command": "uv",
      "args": [
        "--directory",
        "/path/to/mcp-server-code-execution-mode",
        "run",
        "mcp-code-execution"
      ],
      "env": {
        "MCP_BRIDGE_RUNTIME": "podman",
        "MCP_BRIDGE_IMAGE": "ghcr.io/elusznik/mcp-code-execution:latest",
        "MCP_BRIDGE_OUTPUT_MODE": "compact"
      }
    }
  }
}
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_BRIDGE_RUNTIME` | `podman` | Container runtime (`podman` or `docker`) |
| `MCP_BRIDGE_IMAGE` | `ghcr.io/elusznik/mcp-code-execution:latest` | Container image to use |
| `MCP_BRIDGE_TIMEOUT` | `300` | Execution timeout (seconds) |
| `MCP_BRIDGE_OUTPUT_MODE` | `compact` | Output format (`compact`, `toon`, or `json`) |
| `MCP_BRIDGE_MEMORY_LIMIT` | `512m` | Container memory limit |
| `MCP_BRIDGE_PIDS_LIMIT` | `128` | Max processes in container |
| `MCP_BRIDGE_STARTUP_TIMEOUT` | `60` | Server startup timeout (seconds) |

### Proxying Other MCP Servers

Create a `mcp_bridge_config.json` in your working directory:

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
      "env": {}
    },
    "github": {
      "command": "uvx",
      "args": ["mcp-server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-e", "POSTGRES_CONNECTION_STRING",
        "mcp/postgres"
      ],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    }
  }
}
```

The bridge auto-discovers configs from:
- `./mcp_bridge_config.json`
- `~/.config/mcp-bridge/config.json`
- `MCP_BRIDGE_CONFIG` env var (JSON string)
- Claude Desktop config (proxies sibling servers)

## Usage Patterns

### Basic Code Execution

```python
# Simple calculation
result = 42 * 1.5
print(f"Answer: {result}")
```

```python
# Data analysis
import pandas as pd
import numpy as np

data = pd.DataFrame({
    'x': np.random.randn(100),
    'y': np.random.randn(100)
})

correlation = data['x'].corr(data['y'])
print(f"Correlation: {correlation:.3f}")
```

### Discovering MCP Servers

```python
from mcp import runtime

# List all available servers
servers = await runtime.discovered_servers()
for server in servers:
    print(f"- {server}")
```

### Querying Tool Schemas

```python
from mcp import runtime

# Get all tools from a specific server
github_tools = await runtime.query_tool_docs("github")

for tool_name, schema in github_tools.items():
    print(f"{tool_name}: {schema.get('description', 'No description')}")
```

### Fuzzy Tool Search

```python
from mcp import runtime

# Search across all servers
matches = await runtime.search_tool_docs("create issue", limit=5)

for hit in matches:
    print(f"{hit['server']}.{hit['tool']}: {hit.get('description', '')}")
    print(f"  Score: {hit.get('score', 0):.2f}")
```

### Calling MCP Tools Directly

```python
# Dynamic lookup
result = await mcp_servers["github"].call_tool(
    "create_issue",
    {
        "owner": "elusznik",
        "repo": "mcp-server-code-execution-mode",
        "title": "Add feature X",
        "body": "Description here"
    }
)
print(result)
```

```python
# Attribute access
result = await mcp_github.create_issue(
    owner="elusznik",
    repo="mcp-server-code-execution-mode",
    title="Bug report",
    body="Steps to reproduce..."
)
```

```python
# Module import pattern
from mcp.servers.github import create_issue

issue = await create_issue(
    owner="myorg",
    repo="myrepo",
    title="Task",
    body="Details"
)
```

### Composing Multiple Tools

```python
from mcp import runtime

# 1. Search for calendar tools
cal_tools = await runtime.search_tool_docs("calendar events", limit=3)
calendar_server = cal_tools[0]["server"] if cal_tools else None

if calendar_server:
    # 2. Get today's events
    events = await mcp_servers[calendar_server].call_tool(
        "list_events",
        {"date": "2025-01-15"}
    )
    
    # 3. Create GitHub issues for each event
    for event in events:
        await mcp_github.create_issue(
            owner="myorg",
            repo="tasks",
            title=f"Follow-up: {event['title']}",
            body=f"From calendar: {event['description']}"
        )
    
    print(f"Created {len(events)} issues")
```

### Error Handling

```python
from mcp import runtime

try:
    result = await mcp_servers["github"].call_tool(
        "get_issue",
        {"owner": "invalid", "repo": "repo", "issue_number": 999}
    )
except Exception as e:
    print(f"Tool call failed: {e}")
    
    # Fallback: search for alternative tools
    alternatives = await runtime.search_tool_docs("get issue")
    print(f"Found {len(alternatives)} alternative tools")
```

### Persistent State

Variables and imports persist across calls in the same session:

```python
# First call
import pandas as pd
df = pd.DataFrame({'a': [1, 2, 3], 'b': [4, 5, 6]})
```

```python
# Second call (same session)
# df and pandas are still available
print(df.describe())
```

## Advanced Patterns

### Building a Tool Catalog

```python
from mcp import runtime
import json

catalog = {}

# Discover all servers
servers = await runtime.discovered_servers()

for server_name in servers:
    # Get all tools for this server
    tools = await runtime.query_tool_docs(server_name)
    catalog[server_name] = {
        tool_name: {
            "description": schema.get("description", ""),
            "parameters": schema.get("inputSchema", {}).get("properties", {})
        }
        for tool_name, schema in tools.items()
    }

# Save catalog
with open("/tmp/tool_catalog.json", "w") as f:
    json.dump(catalog, f, indent=2)

print(f"Cataloged {len(catalog)} servers")
```

### Conditional Tool Execution

```python
from mcp import runtime

# Search for weather tools
weather_tools = await runtime.search_tool_docs("weather forecast", limit=1)

if weather_tools:
    server = weather_tools[0]["server"]
    tool = weather_tools[0]["tool"]
    
    forecast = await mcp_servers[server].call_tool(
        tool,
        {"location": "San Francisco"}
    )
    
    # If rain predicted, create calendar reminder
    if "rain" in forecast.lower():
        cal_tools = await runtime.search_tool_docs("create event")
        if cal_tools:
            await mcp_servers[cal_tools[0]["server"]].call_tool(
                cal_tools[0]["tool"],
                {
                    "title": "Bring umbrella",
                    "date": "2025-01-16",
                    "time": "08:00"
                }
            )
```

### Batch Processing

```python
import asyncio

repos = ["repo1", "repo2", "repo3"]
results = []

for repo in repos:
    try:
        issues = await mcp_github.list_issues(
            owner="myorg",
            repo=repo,
            state="open"
        )
        results.append({"repo": repo, "count": len(issues)})
    except Exception as e:
        results.append({"repo": repo, "error": str(e)})

for r in results:
    if "error" in r:
        print(f"{r['repo']}: ERROR - {r['error']}")
    else:
        print(f"{r['repo']}: {r['count']} open issues")
```

### Data Science Workflow

```python
import pandas as pd
import matplotlib.pyplot as plt
from io import StringIO

# Fetch data from MCP tool
csv_data = await mcp_filesystem.read_file(path="/tmp/sales.csv")

# Process
df = pd.read_csv(StringIO(csv_data))
summary = df.groupby("region")["sales"].sum().sort_values(ascending=False)

# Generate chart
fig, ax = plt.subplots()
summary.plot(kind="bar", ax=ax)
plt.title("Sales by Region")
plt.tight_layout()
plt.savefig("/tmp/sales_chart.png")

# Save results
report = f"""
Sales Analysis
==============
Total Sales: ${df['sales'].sum():,.2f}
Top Region: {summary.index[0]} (${summary.iloc[0]:,.2f})

Chart saved to /tmp/sales_chart.png
"""

await mcp_filesystem.write_file(
    path="/tmp/sales_report.txt",
    content=report
)

print(report)
```

## Troubleshooting

### Container Runtime Not Found

**Error:** `RuntimeNotFoundError: Neither podman nor docker found`

**Solution:**
```bash
# Install Podman
brew install podman  # macOS
sudo apt install podman  # Linux

# Or install Docker
brew install docker  # macOS (with Docker Desktop)
```

### Permission Denied (Rootless)

**Error:** `Error: creating container: mkdir /run/user/1000: permission denied`

**Solution:**
```bash
# Enable rootless mode
podman system migrate

# Or use Docker rootless
dockerd-rootless-setuptool.sh install
```

### Image Pull Timeout

**Error:** `TimeoutError: Container image pull exceeded 60s`

**Solution:**
```bash
# Pre-pull image manually
podman pull ghcr.io/elusznik/mcp-code-execution:latest

# Or increase timeout
export MCP_BRIDGE_STARTUP_TIMEOUT=180
```

### Server Discovery Fails

**Error:** No servers found via `discovered_servers()`

**Solution:**

1. Check config file exists:
   ```bash
   cat ~/.config/mcp-bridge/config.json
   ```

2. Validate JSON syntax:
   ```python
   import json
   with open("mcp_bridge_config.json") as f:
       json.load(f)  # Should not raise
   ```

3. Test server manually:
   ```bash
   npx -y @modelcontextprotocol/server-filesystem /tmp
   # Should output JSON-RPC messages
   ```

### Tool Call Hangs

**Error:** Tool execution never completes

**Solution:**

1. Check timeout settings:
   ```bash
   export MCP_BRIDGE_TIMEOUT=600  # Increase to 10 minutes
   ```

2. Debug in container:
   ```bash
   podman run -it --rm ghcr.io/elusznik/mcp-code-execution:latest /bin/bash
   # Test commands manually
   ```

### Memory Limit Exceeded

**Error:** `OOMKilled` or memory allocation errors

**Solution:**
```bash
export MCP_BRIDGE_MEMORY_LIMIT=2g  # Increase to 2GB
```

### Volume Mount Issues (macOS)

**Error:** Files not accessible in container

**Solution:**
```bash
# Ensure Podman machine has volume sharing enabled
podman machine stop
podman machine set --rootful=false --volume /Users:/Users
podman machine start
```

### Output Format Issues

**Problem:** Responses too verbose or unstructured

**Solution:**
```bash
# Use compact mode (default)
export MCP_BRIDGE_OUTPUT_MODE=compact

# Or TOON for deterministic tokens
export MCP_BRIDGE_OUTPUT_MODE=toon
```

## Security Best Practices

1. **Never expose the bridge directly to untrusted users** — the agent can execute arbitrary Python.

2. **Use environment variables for secrets:**
   ```json
   {
     "servers": {
       "github": {
         "env": {
           "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
         }
       }
     }
   }
   ```

3. **Restrict filesystem access** for proxied servers:
   ```json
   {
     "servers": {
       "filesystem": {
         "args": ["-y", "@modelcontextprotocol/server-filesystem", "/safe/path"]
       }
     }
   }
   ```

4. **Monitor resource usage:**
   ```bash
   podman stats $(podman ps -q --filter ancestor=ghcr.io/elusznik/mcp-code-execution)
   ```

5. **Audit logs:** Check container logs for suspicious activity:
   ```bash
   podman logs <container-id>
   ```

## Performance Tips

1. **Warm up servers** by calling `discovered_servers()` early in the session.

2. **Cache tool schemas** instead of querying repeatedly:
   ```python
   from mcp import runtime
   
   # Cache at session start
   all_tools = {}
   for server in await runtime.discovered_servers():
       all_tools[server] = await runtime.query_tool_docs(server)
   ```

3. **Use attribute access** (slightly faster than dynamic lookup):
   ```python
   # Preferred
   await mcp_github.create_issue(...)
   
   # Slower
   await mcp_servers["github"].call_tool("create_issue", ...)
   ```

4. **Pre-pull images** to avoid startup delays:
   ```bash
   podman pull ghcr.io/elusznik/mcp-code-execution:latest
   ```

## References

- [GitHub Repository](https://github.com/elusznik/mcp-server-code-execution-mode)
- [Anthropic Blog: Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)
- [Cloudflare Blog: Code Mode](https://blog.cloudflare.com/code-mode/)
- [MCP Protocol Documentation](https://modelcontextprotocol.io/)
- [Apple Research: CodeAct](https://machinelearning.apple.com/research/codeact)
