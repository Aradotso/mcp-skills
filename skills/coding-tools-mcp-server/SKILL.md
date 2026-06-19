---
name: coding-tools-mcp-server
description: MCP server that gives AI agents local coding primitives for file operations, git commands, and safe command execution
triggers:
  - how do I set up the coding-tools MCP server
  - configure coding tools for file operations and git
  - run commands safely in a workspace with MCP
  - set up local coding agent with file access
  - install coding-tools-mcp for Claude or Cursor
  - use MCP server for repository inspection and patching
  - execute shell commands through MCP with safety controls
  - configure remote MCP server with authentication
---

# Coding Tools MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Coding Tools MCP is a model-neutral coding-agent runtime MCP server that exposes local coding primitives to any MCP client. It provides safe, workspace-bounded operations for:

- **File operations**: read, list, search files with automatic exclusion of build artifacts
- **Structured patching**: apply unified diff patches to files
- **Command execution**: run shell commands with safety controls, timeouts, and permission gates
- **Git operations**: status, diff, log, show, blame (read-only git inspection)
- **Interactive sessions**: manage stdin for long-running processes
- **Image viewing**: inspect image files with optional auto-resize

The server enforces workspace boundaries, rejects path traversal, blocks sensitive environment variables, and provides configurable permission modes.

## Installation

### Quick Install (Standalone)

```bash
# Install from PyPI
curl -fsSL https://raw.githubusercontent.com/xyTom/coding-tools-mcp/main/scripts/install.sh | bash

# Or run without persistent install
uvx coding-tools-mcp --workspace /path/to/repo
```

### Install with HTTP Server

```bash
# Start local HTTP server
curl -fsSL https://raw.githubusercontent.com/xyTom/coding-tools-mcp/main/scripts/install.sh \
  | bash -s -- --start --workspace /path/to/repo

# Exposed at http://127.0.0.1:8765/mcp
```

### Install with Development Dependencies

```bash
# From source checkout
git clone https://github.com/xyTom/coding-tools-mcp.git
cd coding-tools-mcp
python -m pip install -e ".[dev]"

# With image support for view_image auto-resize
python -m pip install -e ".[image]"
```

## MCP Client Configuration

### Claude Code

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "coding-tools": {
      "command": "uvx",
      "args": ["coding-tools-mcp", "--stdio", "--workspace", "/path/to/repo"]
    }
  }
}
```

### Cursor

Add to Cursor's MCP settings:

```json
{
  "mcpServers": {
    "coding-tools": {
      "command": "uvx",
      "args": ["coding-tools-mcp", "--stdio", "--workspace", "/path/to/repo"]
    }
  }
}
```

### Generic TOML Configuration

```toml
[mcp_servers.coding_tools]
command = "uvx"
args = ["coding-tools-mcp", "--stdio", "--workspace", "/path/to/repo"]
```

### HTTP Client Configuration

For Streamable HTTP clients using protocol version `2025-06-18`:

```bash
# Start server
coding-tools-mcp --workspace /path/to/repo

# Client connects to
# URL: http://127.0.0.1:8765/mcp
```

## Permission Modes

The server has three permission modes:

### Safe Mode (Default)

```bash
coding-tools-mcp --permission-mode safe --workspace /path/to/repo
```

- Blocks shell expansion (`$VAR`, `$(command)`, glob patterns)
- Blocks network-looking commands
- Checks for destructive operations
- Filters sensitive environment variables
- Requires explicit permission for risky operations

### Trusted Mode (Local Development)

```bash
coding-tools-mcp --permission-mode trusted --workspace /path/to/repo

# Or with inherited environment
CODING_TOOLS_MCP_SHELL_ENV_INHERIT=all \
  coding-tools-mcp --permission-mode trusted --workspace /path/to/repo
```

- Allows shell expansion for local toolchains
- Allows dependency downloads
- Allows inline interpreter snippets
- Still filters secrets and destructive commands

### Dangerous Mode (Isolated Environments Only)

```bash
coding-tools-mcp --permission-mode dangerous --workspace /path/to/repo
```

- Disables all `exec_command` permission gates
- **Only use inside isolated containers or VMs**
- Workspace path boundaries still apply for file operations

## Core Tools

### File Operations

#### read_file

```python
# Request
{
  "path": "src/main.py",
  "start_line": 10,  # optional
  "end_line": 50     # optional
}

# Response includes file content, encoding detection, and metadata
```

#### list_dir

```python
# Request
{
  "path": "src",
  "recursive": False
}

# Returns directory listing with file types
```

#### list_files

```python
# Request
{
  "path": ".",
  "pattern": "*.py",  # optional glob
  "recursive": True
}

# Returns filtered file list excluding .git, node_modules, etc.
```

#### search_text

```python
# Request
{
  "query": "def authenticate",
  "path": "src",
  "case_sensitive": False,
  "regex": False,
  "max_results": 100
}

# Returns matching files with line numbers and context
```

### Patching

#### apply_patch

```python
# Request
{
  "patch": """--- a/src/config.py
+++ b/src/config.py
@@ -10,3 +10,4 @@
 DEBUG = False
+FEATURE_FLAG = True
""",
  "dry_run": False  # optional: preview without applying
}

# Applies unified diff format patches
# Validates patch format and target file existence
```

### Command Execution

#### exec_command

```python
# Request
{
  "command": "pytest tests/test_auth.py -v",
  "cwd": ".",  # optional, workspace-relative
  "timeout": 30,  # optional, seconds
  "capture_output": True,  # optional
  "env": {  # optional additional env vars
    "PYTEST_ARGS": "--maxfail=1"
  }
}

# Response includes stdout, stderr, exit_code, and warnings
# Session ID returned for interactive processes
```

**Safety features:**
- Workspace-bounded working directory
- Timeout enforcement (default 30s for one-shot, 3600s for sessions)
- Output caps (1MB stdout, 256KB stderr per command)
- Sensitive value filtering in environment
- Destructive command checks
- Network command permission gates (in safe mode)
- Shell expansion gates (in safe mode)
- Landlock filesystem confinement on supported Linux hosts

#### write_stdin

```python
# Request
{
  "session_id": "exec_abc123",
  "data": "yes\n"  # send input to running process
}

# For interactive command sessions
```

#### kill_session

```python
# Request
{
  "session_id": "exec_abc123",
  "signal": "SIGTERM"  # optional, defaults to SIGTERM
}

# Terminates long-running command session
```

### Git Operations (Read-Only)

#### git_status

```python
# Request
{
  "cwd": "."  # optional, workspace-relative
}

# Returns working tree status
```

#### git_diff

```python
# Request
{
  "cwd": ".",
  "staged": False,  # optional: show staged changes
  "paths": ["src/"]  # optional: limit to paths
}

# Returns unified diff of changes
```

#### git_log

```python
# Request
{
  "cwd": ".",
  "max_count": 20,  # optional
  "path": "src/main.py"  # optional: file history
}

# Returns commit history
```

#### git_show

```python
# Request
{
  "cwd": ".",
  "revision": "HEAD~1",  # commit hash or ref
  "path": "src/config.py"  # optional: specific file
}

# Shows commit or file content at revision
```

#### git_blame

```python
# Request
{
  "path": "src/main.py",
  "start_line": 10,  # optional
  "end_line": 50  # optional
}

# Returns line-by-line commit attribution
```

### Workspace Management

#### get_default_cwd

```python
# Request: {}
# Returns current default working directory (workspace-relative)
```

#### set_default_cwd

```python
# Request
{
  "cwd": "services/api"  # workspace-relative path
}

# Sets default working directory for subsequent commands
```

#### server_info

```python
# Request: {}
# Returns workspace root, permission mode, profile, platform info
```

#### view_image

```python
# Request
{
  "path": "assets/logo.png",
  "max_dimension": 1024  # optional: auto-resize
}

# Returns base64-encoded image data with MIME type
# Requires [image] extra for resize support
```

### Permission Management

#### request_permissions

```python
# Request
{
  "permissions": ["network", "shell_expansion"],
  "reason": "Need to download dependencies with npm install"
}

# Request explicit permission for gated operations
# Client may prompt user or auto-approve based on policy
```

## Tool Profiles

Control which tools are exposed:

```bash
# Full profile (default): all tools with truthful annotations
coding-tools-mcp --workspace /path/to/repo

# Read-only profile: safe for remote/untrusted clients
# Only inspection, git read, image view, and cwd helpers
CODING_TOOLS_MCP_TOOL_PROFILE=read-only coding-tools-mcp --workspace /path/to/repo

# Compat readonly: exposes all tools but marks all as read-only
# NOT a safety mode - mutation tools still work
CODING_TOOLS_MCP_TOOL_PROFILE=compat-readonly-all coding-tools-mcp --workspace /path/to/repo
```

## Remote MCP Setup

### Bearer Token Auth (Recommended for Testing)

```bash
# Anonymous read-only tunnel (testing only)
CODING_TOOLS_MCP_AUTH_MODE=noauth \
CODING_TOOLS_MCP_TOOL_PROFILE=read-only \
./scripts/tunnel.sh cloudflared /path/to/repo

# Bearer token auth with custom token
CODING_TOOLS_MCP_AUTH_MODE=bearer \
CODING_TOOLS_MCP_BEARER_TOKEN="your-secret-token-here" \
./scripts/tunnel.sh cloudflared /path/to/repo

# Client configuration:
# URL: https://<tunnel-host>/mcp
# Header: Authorization: Bearer your-secret-token-here
```

### OAuth 2.1 Auth (For Production Clients)

```bash
# OAuth with Authorization Code + PKCE
CODING_TOOLS_MCP_AUTH_MODE=oauth \
./scripts/tunnel.sh cloudflared /path/to/repo

# Script prints generated OAuth password
# Server infers issuer from tunnel URL
# Accepts any non-empty client_id by default

# Optional: pin issuer and client credentials
CODING_TOOLS_MCP_AUTH_MODE=oauth \
CODING_TOOLS_MCP_SERVER_URL=https://your-domain.com \
CODING_TOOLS_MCP_OAUTH_CLIENT_ID=client-id \
CODING_TOOLS_MCP_OAUTH_CLIENT_SECRET=client-secret \
./scripts/tunnel.sh cloudflared /path/to/repo
```

### Supported Tunnels

```bash
# Cloudflared
scripts/tunnel.sh cloudflared /path/to/repo

# ngrok
scripts/tunnel.sh ngrok /path/to/repo

# Microsoft Dev Tunnel
scripts/tunnel.sh devtunnel /path/to/repo

# Auto-install tunnel CLI if missing
scripts/install.sh --tunnel cloudflared --auto-install-tunnel --workspace /path/to/repo
```

## Common Patterns

### Python Project Test Run

```python
# 1. Check repository structure
list_files({"path": ".", "recursive": True})

# 2. Read test configuration
read_file({"path": "pytest.ini"})

# 3. Check git status before running tests
git_status({"cwd": "."})

# 4. Run specific test file
exec_command({
  "command": "pytest tests/test_api.py -v --tb=short",
  "timeout": 60
})

# 5. Check for changes after test (e.g., coverage reports)
git_status({"cwd": "."})
```

### JavaScript Dependency Install

```python
# 1. Request permission for network operations (in safe mode)
request_permissions({
  "permissions": ["network"],
  "reason": "Install npm dependencies"
})

# 2. Read package.json
read_file({"path": "package.json"})

# 3. Install dependencies
exec_command({
  "command": "npm install",
  "timeout": 300
})

# 4. Verify installation
list_dir({"path": "node_modules"})
```

### Patch Application Workflow

```python
# 1. Search for target function
search_text({
  "query": "def process_payment",
  "path": "src",
  "regex": False
})

# 2. Read current implementation
read_file({
  "path": "src/payments/processor.py",
  "start_line": 45,
  "end_line": 75
})

# 3. Preview patch (dry run)
apply_patch({
  "patch": """--- a/src/payments/processor.py
+++ b/src/payments/processor.py
@@ -50,6 +50,7 @@
     def process_payment(self, amount, method):
+        self.validate_amount(amount)
         return self.gateway.charge(amount, method)
""",
  "dry_run": True
})

# 4. Apply patch
apply_patch({
  "patch": """--- a/src/payments/processor.py
+++ b/src/payments/processor.py
@@ -50,6 +50,7 @@
     def process_payment(self, amount, method):
+        self.validate_amount(amount)
         return self.gateway.charge(amount, method)
"""
})

# 5. Verify changes
git_diff({"cwd": ".", "staged": False})
```

### Interactive Command Session

```python
# 1. Start interactive Python REPL
response = exec_command({
  "command": "python -i",
  "capture_output": True
})
session_id = response["session_id"]

# 2. Send commands to stdin
write_stdin({
  "session_id": session_id,
  "data": "import sys\n"
})

write_stdin({
  "session_id": session_id,
  "data": "print(sys.version)\n"
})

# 3. Kill session when done
kill_session({
  "session_id": session_id,
  "signal": "SIGTERM"
})
```

### Git History Investigation

```python
# 1. Check recent commits
git_log({
  "cwd": ".",
  "max_count": 10
})

# 2. View specific commit
git_show({
  "cwd": ".",
  "revision": "a1b2c3d",
  "path": "src/config.py"
})

# 3. Check blame for suspicious line
git_blame({
  "path": "src/config.py",
  "start_line": 25,
  "end_line": 30
})

# 4. Compare with previous version
git_diff({
  "cwd": ".",
  "paths": ["src/config.py"]
})
```

## Environment Variables

```bash
# Trace mode: emit redacted JSON tool calls to stderr
CODING_TOOLS_MCP_TRACE=1

# Shell environment inheritance
CODING_TOOLS_MCP_SHELL_ENV_INHERIT=all  # inherit all non-secret vars
CODING_TOOLS_MCP_SHELL_ENV_INHERIT=none  # core only (default)

# Tool profile
CODING_TOOLS_MCP_TOOL_PROFILE=read-only

# Authentication (for remote MCP)
CODING_TOOLS_MCP_AUTH_MODE=bearer
CODING_TOOLS_MCP_BEARER_TOKEN=your-token

CODING_TOOLS_MCP_AUTH_MODE=oauth
CODING_TOOLS_MCP_SERVER_URL=https://your-domain.com
CODING_TOOLS_MCP_OAUTH_CLIENT_ID=client-id
CODING_TOOLS_MCP_OAUTH_CLIENT_SECRET=client-secret
```

## Troubleshooting

### Command Permission Denied

```bash
# If command blocked by network check in safe mode:
# Either switch to trusted mode
coding-tools-mcp --permission-mode trusted --workspace /path/to/repo

# Or request permission explicitly via request_permissions tool
# Client may prompt user for approval
```

### Path Outside Workspace Error

```bash
# All paths must be workspace-relative or within workspace
# Absolute paths, .. traversal, and symlink escapes are rejected

# ✗ Wrong
read_file({"path": "/etc/passwd"})
read_file({"path": "../../../secrets"})

# ✓ Correct
read_file({"path": "src/config.py"})
read_file({"path": "./data/input.json"})
```

### Command Timeout

```python
# Increase timeout for long-running commands
exec_command({
  "command": "npm run build",
  "timeout": 600  # 10 minutes
})

# Default timeout: 30s for one-shot, 3600s for sessions
```

### Environment Variables Missing

```bash
# For toolchains that need inherited environment (MSVC, conda, etc.)
CODING_TOOLS_MCP_SHELL_ENV_INHERIT=all \
  coding-tools-mcp --permission-mode trusted --workspace /path/to/repo

# Still filters secrets and loader vars unless dangerous mode
```

### File Not Found in Search/List

The server excludes common build artifacts and caches by default:
- `.git`, `.reference`
- `node_modules`, `target`, `dist`
- `venv`, `virtualenv`, `.venv`
- `__pycache__`, `.pytest_cache`
- `.mypy_cache`, `.ruff_cache`

If you need to inspect excluded directories, use `read_file` directly with the known path.

### Image Resize Not Working

```bash
# Install image extra for PIL/Pillow support
python -m pip install coding-tools-mcp[image]

# Or without resize, view_image returns original image
# up to size limits
```

### Landlock Warning on Windows/macOS

The exec_command Landlock confinement only works on Linux with kernel 5.13+. On Windows, macOS, or older Linux, you'll see a warning. For untrusted workspaces or commands, use external sandboxing:

- Docker containers
- VMs
- Windows Sandbox
- macOS sandboxd

## Security Notes

- **Workspace boundary**: All file operations are confined to the workspace root
- **Path validation**: Rejects absolute paths, `..` traversal, symlinks outside workspace
- **Command safety**: Timeout, output caps, environment filtering, destructive checks
- **Not a full sandbox**: Use Docker/VM for untrusted code; Landlock only works on modern Linux
- **Permission modes**: Default safe mode blocks shell expansion and network commands
- **Remote MCP**: Use bearer token or OAuth for remote access; `noauth` is testing-only

See `SECURITY.md` and `docs/security-boundary.md` for full security policy.
