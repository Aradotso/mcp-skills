---
name: mcp-code-execution-mode
description: Execute Python code in isolated rootless containers with MCP server proxying for token-efficient agent workflows
triggers:
  - run python code in a sandbox
  - execute code with mcp tools
  - use mcp servers without token bloat
  - proxy mcp servers in containers
  - discover mcp tools dynamically
  - run isolated python with docker
  - reduce mcp context overhead
  - execute python with tool discovery
---

# MCP Code Execution Mode

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What This Does

This MCP server solves the "token bloat" problem when connecting LLMs to multiple MCP servers. Instead of loading 30,000+ tokens of tool schemas into every prompt, it exposes a single `run_python` tool that executes Python code in rootless containers. The LLM discovers and calls other MCP tools programmatically, reducing context overhead by 95%+.

**Key benefits:**
- **Constant ~200 token overhead** regardless of server count
- **Discovery-first**: Query schemas only when needed
- **Universal proxying**: Works with any stdio MCP server
- **Production security**: Rootless containers, no network, read-only filesystem
- **Persistent sessions**: Variables and MCP clients survive across calls

## Installation

### Prerequisites

1. **Container runtime** (choose one):
   ```bash
   # Podman (recommended)
   brew install podman
   podman machine init
   podman machine start
   
   # Docker Desktop (alternative)
   # Download from docker.com
   ```

2. **Python 3.11+**:
   ```bash
   python3 --version  # Must be 3.11+
   ```

### Install via pip

```bash
pip install mcp-code-execution-mode
```

### Install from source

```bash
git clone https://github.com/elusznik/mcp-server-code-execution-mode.git
cd mcp-server-code-execution-mode
pip install -e .
```

## Configuration

### Claude Desktop Setup

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "code-execution": {
      "command": "python",
      "args": ["-m", "mcp_code_execution_mode"],
      "env": {
        "MCP_BRIDGE_RUNTIME": "podman",
        "MCP_BRIDGE_IMAGE": "ghcr.io/elusznik/mcp-code-execution-mode:latest",
        "MCP_BRIDGE_OUTPUT_MODE": "compact"
      }
    }
  }
}
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_BRIDGE_RUNTIME` | Auto-detect | `podman` or `docker` |
| `MCP_BRIDGE_IMAGE` | `ghcr.io/elusznik/mcp-code-execution-mode:latest` | Container image |
| `MCP_BRIDGE_OUTPUT_MODE` | `compact` | `compact` or `toon` |
| `MCP_BRIDGE_TIMEOUT` | `120` | Execution timeout (seconds) |
| `MCP_BRIDGE_MEMORY_LIMIT` | `512m` | Container memory limit |
| `MCP_BRIDGE_SESSION_PERSIST` | `true` | Keep variables between calls |

### Proxying Other MCP Servers

To give the agent access to other MCP servers (e.g., filesystem, GitHub), configure them in the same `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "code-execution": {
      "command": "python",
      "args": ["-m", "mcp_code_execution_mode"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/username/Documents"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

The bridge **auto-discovers** these servers at runtime. No manual catalog needed.

## Core API

### Discovery Functions

```python
from mcp import runtime

# List available MCP servers
servers = await runtime.discovered_servers()
# Returns: ["filesystem", "github", "slack"]

# Get tool schemas for a specific server
docs = await runtime.query_tool_docs("github")
# Returns: {"tools": [{"name": "create_issue", "description": "...", ...}]}

# Fuzzy search across all servers
matches = await runtime.search_tool_docs("list files", limit=5)
# Returns: [{"server": "filesystem", "tool": "list_directory", "description": "..."}]
```

### Calling MCP Tools

**Dynamic lookup:**
```python
from mcp import mcp_servers

# Call a tool
result = await mcp_servers["github"].create_issue(
    repo="owner/repo",
    title="Bug report",
    body="Description here"
)
```

**Attribute access:**
```python
from mcp import mcp_github

result = await mcp_github.create_issue(
    repo="owner/repo",
    title="Feature request",
    body="Add dark mode"
)
```

**Module import (explicit):**
```python
from mcp.servers.github import create_issue

result = await create_issue(
    repo="owner/repo",
    title="Enhancement",
    body="Improve performance"
)
```

### Session Persistence

Variables persist across calls in the same session:

```python
# First call
import pandas as pd
df = pd.DataFrame({"col": [1, 2, 3]})
df.to_csv("/tmp/data.csv", index=False)
```

```python
# Second call (same session)
import pandas as pd
df = pd.read_csv("/tmp/data.csv")
print(df.sum())  # Works! File still exists
```

## Common Patterns

### Pattern 1: Discovery → Execution

```python
from mcp import runtime, mcp_servers

# Step 1: Find tools related to "calendar"
matches = await runtime.search_tool_docs("calendar events", limit=3)

# Step 2: Load full schema for the first match
server_name = matches[0]["server"]
docs = await runtime.query_tool_docs(server_name)

# Step 3: Call the tool
result = await mcp_servers[server_name].list_events(
    start_date="2025-01-01",
    end_date="2025-01-31"
)

print(result)
```

### Pattern 2: Data Analysis Workflow

```python
import pandas as pd
import matplotlib.pyplot as plt
from mcp import mcp_filesystem

# Read data from filesystem server
csv_content = await mcp_filesystem.read_file(path="/data/sales.csv")

# Parse and analyze
df = pd.read_csv(pd.io.common.StringIO(csv_content))
monthly = df.groupby("month")["revenue"].sum()

# Generate chart
plt.bar(monthly.index, monthly.values)
plt.title("Monthly Revenue")
plt.savefig("/tmp/revenue.png")

# Write back
with open("/tmp/revenue.png", "rb") as f:
    await mcp_filesystem.write_file(
        path="/reports/revenue.png",
        content=f.read()
    )

print(f"Analyzed {len(df)} records, saved chart")
```

### Pattern 3: Multi-Server Orchestration

```python
from mcp import mcp_github, mcp_slack

# Get open issues
issues = await mcp_github.list_issues(
    repo="myorg/myrepo",
    state="open",
    labels=["bug"]
)

# Post summary to Slack
await mcp_slack.post_message(
    channel="#engineering",
    text=f"📊 {len(issues)} open bugs:\n" + 
         "\n".join(f"• {i['title']}" for i in issues[:5])
)

print(f"Posted {len(issues)} issues to Slack")
```

### Pattern 4: Error Handling & Retries

```python
from mcp import mcp_servers
import asyncio

async def safe_call(server, tool, **kwargs):
    for attempt in range(3):
        try:
            return await mcp_servers[server].__getattr__(tool)(**kwargs)
        except Exception as e:
            if attempt == 2:
                raise
            await asyncio.sleep(2 ** attempt)

result = await safe_call(
    "github",
    "create_issue",
    repo="owner/repo",
    title="Test",
    body="Retry logic"
)
```

### Pattern 5: Bash Commands

The sandbox includes common CLI tools:

```python
import subprocess

# List files
result = subprocess.run(["ls", "-lh", "/tmp"], capture_output=True, text=True)
print(result.stdout)

# Parse JSON with jq
json_data = '{"name": "test", "count": 42}'
result = subprocess.run(
    ["jq", ".count"],
    input=json_data,
    capture_output=True,
    text=True
)
print(f"Count: {result.stdout.strip()}")
```

## Troubleshooting

### Error: "No container runtime available"

**Cause:** Podman/Docker not installed or not running.

**Fix:**
```bash
# Check status
podman machine list
podman machine start

# Or switch to Docker
export MCP_BRIDGE_RUNTIME=docker
```

### Error: "Image pull failed"

**Cause:** Network issues or image not found.

**Fix:**
```bash
# Pre-pull the image
podman pull ghcr.io/elusznik/mcp-code-execution-mode:latest

# Or build locally
git clone https://github.com/elusznik/mcp-server-code-execution-mode.git
cd mcp-server-code-execution-mode
podman build -t mcp-code-execution:local -f Containerfile .

# Update config to use local image
export MCP_BRIDGE_IMAGE=mcp-code-execution:local
```

### Error: "Tool not found in server X"

**Cause:** Tool name mismatch or server not configured.

**Fix:**
```python
from mcp import runtime

# Check what's actually available
docs = await runtime.query_tool_docs("github")
print([t["name"] for t in docs["tools"]])

# Use exact name from output
await mcp_github.create_or_update_file(...)  # Not create_file
```

### Variables Not Persisting

**Cause:** Session restarted (happens on bridge reload).

**Fix:** Store critical data in files:
```python
import pickle

# Save state
state = {"counter": 42, "data": [1, 2, 3]}
with open("/tmp/state.pkl", "wb") as f:
    pickle.dump(state, f)

# Restore in next call
with open("/tmp/state.pkl", "rb") as f:
    state = pickle.load(f)
```

### Timeout Errors

**Cause:** Long-running computation exceeds 120s default.

**Fix:**
```bash
# Increase timeout
export MCP_BRIDGE_TIMEOUT=300
```

Or break work into chunks:
```python
# Bad: process 1M rows in one call
df = pd.read_csv("huge.csv")  # Times out

# Good: process in batches
for chunk in pd.read_csv("huge.csv", chunksize=10000):
    process(chunk)
```

### Permission Denied in Container

**Cause:** Trying to write to read-only filesystem.

**Fix:** Use `/tmp` for temporary files:
```python
# Bad
with open("/data/output.txt", "w") as f:  # Read-only
    f.write("data")

# Good
with open("/tmp/output.txt", "w") as f:  # Writable
    f.write("data")
```

## Advanced Configuration

### Custom Container Image

Build an image with extra dependencies:

```dockerfile
FROM ghcr.io/elusznik/mcp-code-execution-mode:latest

RUN pip install --no-cache-dir \
    scikit-learn \
    seaborn \
    sqlalchemy
```

```bash
podman build -t mcp-custom:latest .
export MCP_BRIDGE_IMAGE=mcp-custom:latest
```

### Resource Limits

```bash
# Increase memory for ML workloads
export MCP_BRIDGE_MEMORY_LIMIT=2g

# CPU quota (% of one core)
export MCP_BRIDGE_CPU_QUOTA=50000  # 50%

# Max processes
export MCP_BRIDGE_PIDS_LIMIT=200
```

### Security Hardening

The bridge already runs rootless with:
- `--cap-drop=ALL` (no capabilities)
- `--read-only` (immutable root)
- `--security-opt=no-new-privileges`
- `--network=none` (no internet)

For even stricter isolation:
```bash
# Use SELinux labels (Fedora/RHEL)
export MCP_BRIDGE_SECURITY_OPT="label=type:container_runtime_t"

# Disable session persistence
export MCP_BRIDGE_SESSION_PERSIST=false
```

## Best Practices

1. **Use discovery before calling**: Always `search_tool_docs()` or `query_tool_docs()` first to avoid guessing tool names.

2. **Handle errors gracefully**: MCP servers can fail. Wrap calls in try/except and provide fallback logic.

3. **Minimize round-trips**: Write loops and conditionals in Python instead of asking the LLM to orchestrate multiple calls.

4. **Persist critical state**: Save important data to `/tmp/` files. Variables persist within a session but not across bridge restarts.

5. **Test locally first**: Run `python -m mcp_code_execution_mode` standalone to verify configuration before integrating with Claude.

## Example: End-to-End Workflow

```python
from mcp import runtime, mcp_github, mcp_slack
import pandas as pd

# 1. Discover GitHub tools
servers = await runtime.discovered_servers()
if "github" not in servers:
    raise ValueError("GitHub MCP server not configured")

# 2. Fetch issues
issues = await mcp_github.list_issues(
    repo="myorg/myrepo",
    state="all",
    since="2025-01-01"
)

# 3. Analyze with pandas
df = pd.DataFrame(issues)
df["created"] = pd.to_datetime(df["created_at"])
monthly_counts = df.groupby(df["created"].dt.to_period("M")).size()

# 4. Format report
report = "📈 Issue Report\n\n"
for month, count in monthly_counts.items():
    report += f"{month}: {count} issues\n"

# 5. Post to Slack
if "slack" in servers:
    await mcp_slack.post_message(
        channel="#engineering",
        text=report
    )
    print("✅ Report posted to Slack")
else:
    print(report)
```

This workflow demonstrates discovery, data fetching, analysis, and multi-server orchestration—all in a single, token-efficient Python execution.
