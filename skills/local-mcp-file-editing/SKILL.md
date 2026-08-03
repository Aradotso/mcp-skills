---
name: local-mcp-file-editing
description: Use local-mcp to enable AI agents to safely read, write, and execute commands on local files with sandboxed permissions
triggers:
  - edit local files with MCP
  - run sandboxed commands locally
  - set up local file editing for AI
  - use local-mcp with my project
  - configure local-mcp sessions
  - manage file permissions with local-mcp
  - execute commands in sandbox
  - read and write files through MCP
---

# local-mcp File Editing Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`local-mcp` is a Rust-based MCP (Model Context Protocol) server that exposes local filesystem and command execution capabilities to AI agents. It provides:

- **File operations**: Read, write, and list files
- **Image reading**: Native MCP image content for PNG, JPEG, GIF, WebP, BMP, TIFF, AVIF
- **Sandboxed command execution**: Network-isolated commands using Landlock (Linux) or Seatbelt (macOS)
- **Session-based permissions**: Each project gets its own permission context
- **Live activity monitoring**: Real-time diffs and command output in the start UI
- **Background job management**: Long-running commands with polling support

Commands run sandboxed by default (no network access). Unsandboxed commands require explicit approval through the interactive UI.

## Installation

### From Source (Cargo)

```sh
# Clone and build
git clone https://github.com/nakasyou/local-mcp.git
cd local-mcp
cargo build --release

# Binary will be at target/release/local-mcp
# On Linux, also outputs target/release/codex-linux-sandbox
```

### With Nix

```sh
# Run directly
nix run github:nakasyou/local-mcp

# Or build locally
nix build
./result/bin/local-mcp

# Development shell
nix develop
```

### System Requirements

- **Linux**: Requires `bwrap` (bubblewrap) in PATH, plus the `codex-linux-sandbox` helper binary
- **macOS**: Uses system `/usr/bin/sandbox-exec`, no extra dependencies
- **Windows**: Not yet supported

## Starting the Server

### 1. Start the MCP Server

```sh
# Run the persistent MCP server
local-mcp mcp

# Server listens on Unix domain sockets for each session
```

### 2. Start a Session

```sh
# Start session in your project directory
cd /path/to/your/project
local-mcp start

# Or use a stable session ID
local-mcp start my-project-name

# Session ID is printed - give this to your AI agent
```

The session working directory is where you ran `local-mcp start`. All relative paths resolve from there.

## Permission Management

In the `local-mcp start` UI, manage permissions with these commands:

```sh
# Allow all unsandboxed commands (for session lifetime only)
/permissions yolo

# Require approval for each unsandboxed command
/permissions ask

# Allow access to specific directory outside working dir
/permissions allow ../another-project

# Revoke directory permission
/permissions revoke ../another-project

# List all permissions
/permissions list

# Show current permission status
/permissions status
```

**Note**: `/permission` (singular) is also accepted.

## MCP Tools API

All tools require a `session_id` parameter (provided by the agent).

### session_info

Confirm session configuration and working directory.

```rust
// Tool parameters
{
    "session_id": "my-project-name"
}

// Returns
{
    "working_directory": "/path/to/your/project",
    "sandbox_roots": ["/path/to/your/project"],
    "permission_mode": "yolo" // or "ask"
}
```

### read_file

Read text file contents.

```rust
{
    "session_id": "my-project-name",
    "path": "src/main.rs"  // Relative to working directory
}

// Returns file content as string
```

### get_image

Read image files as native MCP image content. Supports PNG, JPEG, GIF, WebP, BMP, TIFF, AVIF.

```rust
{
    "session_id": "my-project-name",
    "path": "assets/logo.png"
}

// Returns MCP image content
```

### list_directory

List directory contents with file metadata.

```rust
{
    "session_id": "my-project-name",
    "path": "src"  // Optional, defaults to working directory
}

// Returns array of file entries with size, modified time, type
```

### write_file

Write or modify a file (sandboxed within working directory).

```rust
{
    "session_id": "my-project-name",
    "path": "config.toml",
    "content": "[server]\nport = 8080\n"
}

// Shows unified diff in start UI
```

### execute

Run a command, sandboxed (no network access). Returns immediately for commands under 30 seconds, otherwise returns a job_id.

```rust
{
    "session_id": "my-project-name",
    "command": "cargo build",
    "args": ["--release"],
    "cwd": "."  // Optional, relative to working dir
}

// Quick commands return:
{
    "stdout": "   Compiling local-mcp v0.1.0\n...",
    "stderr": "",
    "exit_code": 0
}

// Long commands return:
{
    "job_id": "abc123",
    "message": "Command started in background"
}
```

### without_sandbox

Execute command with full network and host permissions. **Requires approval** unless `/permissions yolo` is set.

```rust
{
    "session_id": "my-project-name",
    "command": "curl",
    "args": ["https://api.example.com/data"],
    "cwd": "."
}

// Same return format as execute
```

### start_command

Start a background command immediately (sandboxed). Always returns a job_id without waiting.

```rust
{
    "session_id": "my-project-name",
    "command": "cargo",
    "args": ["watch", "-x", "test"]
}

// Returns:
{
    "job_id": "def456"
}
```

### poll_job

Check status of a background job.

```rust
{
    "session_id": "my-project-name",
    "job_id": "abc123"
}

// Still running:
{
    "status": "running"
}

// Completed:
{
    "status": "completed",
    "stdout": "...",
    "stderr": "...",
    "exit_code": 0
}
```

### stop_job

Terminate a background job.

```rust
{
    "session_id": "my-project-name",
    "job_id": "abc123"
}

// Returns confirmation
```

## Common Patterns

### Multi-File Refactoring

```rust
// 1. List files to refactor
list_directory({ session_id: "proj", path: "src" })

// 2. Read each file
read_file({ session_id: "proj", path: "src/main.rs" })
read_file({ session_id: "proj", path: "src/lib.rs" })

// 3. Write updated content
write_file({
    session_id: "proj",
    path: "src/main.rs",
    content: "// Updated code..."
})

// 4. Run tests (sandboxed)
execute({
    session_id: "proj",
    command: "cargo",
    args: ["test"]
})
```

### Building and Deploying

```rust
// 1. Build (sandboxed, no network)
execute({
    session_id: "proj",
    command: "cargo",
    args: ["build", "--release"]
})

// 2. Deploy (requires network, needs approval)
without_sandbox({
    session_id: "proj",
    command: "scp",
    args: ["target/release/app", "user@server:/opt/"]
})
```

### Long-Running Dev Server

```rust
// Start dev server in background
start_command({
    session_id: "proj",
    command: "cargo",
    args: ["run", "--", "serve"]
})
// Returns: { job_id: "server-xyz" }

// Check if server started
poll_job({
    session_id: "proj",
    job_id: "server-xyz"
})

// Later, stop it
stop_job({
    session_id: "proj",
    job_id: "server-xyz"
})
```

### Working with Images

```rust
// List images
list_directory({ session_id: "proj", path: "assets/images" })

// Load image for analysis
get_image({
    session_id: "proj",
    path: "assets/screenshot.png"
})
// Returns native MCP image content for vision analysis
```

## Configuration

### Session Configuration

Sessions are configured through the `local-mcp start` command and the interactive UI. Configuration is **session-lifetime only** and includes:

- Working directory (set by where `start` was run)
- Permission mode (`ask` or `yolo`)
- Approved directories outside working dir

### Environment Variables

The project uses standard Rust environment variables during build:

- `CARGO_BUILD_TARGET`: Target triple for cross-compilation
- `RUSTFLAGS`: Compiler flags

No runtime environment variables are required.

### MCP Server Configuration

The server runs on Unix domain sockets (one per session). Socket paths are managed internally and communicated to the `start` UI automatically.

## Troubleshooting

### "bwrap not found" (Linux)

Install bubblewrap:

```sh
# Debian/Ubuntu
sudo apt install bubblewrap

# Fedora
sudo dnf install bubblewrap

# Arch
sudo pacman -S bubblewrap
```

### "codex-linux-sandbox not found" (Linux)

The helper binary must be in the same directory as `local-mcp`:

```sh
# After building
ls target/release/
# Should show: local-mcp  codex-linux-sandbox

# When installing, copy both
sudo cp target/release/local-mcp /usr/local/bin/
sudo cp target/release/codex-linux-sandbox /usr/local/bin/
```

### Commands Failing with Permission Denied

Check your permission mode in the start UI:

```sh
/permissions status

# If in "ask" mode and you want to auto-approve:
/permissions yolo
```

For paths outside the working directory, explicitly allow them:

```sh
/permissions allow /path/to/other/directory
```

### Session ID Not Working

Ensure the session ID only contains:
- Letters (a-z, A-Z)
- Numbers (0-9)
- Hyphens (`-`)
- Underscores (`_`)
- Periods (`.`)

Invalid characters will be rejected.

### Background Jobs Not Returning Output

Use `poll_job` to check status:

```rust
poll_job({ session_id: "proj", job_id: "job-id" })
```

Jobs may still be running. Check the `status` field in the response.

### macOS Sandbox Restrictions

macOS `sandbox-exec` profiles are more restrictive. If sandboxed commands fail unexpectedly, try `without_sandbox` (requires approval).

## Security Considerations

- **Sandboxed commands** have no network access and can't escape the working directory tree
- **Unsandboxed commands** run with full host permissions and require explicit approval (unless `yolo` mode)
- **YOLO mode** only lasts for the session lifetime; restart `local-mcp start` to reset to `ask` mode
- **Session sockets** are permission-restricted Unix domain sockets
- Give session IDs only to trusted agents; anyone with the ID can make tool calls

## Integration Example

When integrating with an AI agent:

1. Start the MCP server: `local-mcp mcp`
2. Start your project session: `cd /your/project && local-mcp start my-proj`
3. In your agent prompt, include:

```
I'm working in a local-mcp session with ID "my-proj". 
Use session_info to confirm the working directory, then use 
the MCP tools to read, edit, and test files.
```

4. Set permissions as needed in the UI
5. The agent can now safely interact with your local filesystem
