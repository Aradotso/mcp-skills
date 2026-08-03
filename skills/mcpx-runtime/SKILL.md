---
name: mcpx-runtime
description: Run MCPX MCP Runtime to connect AI clients to local development environments with workspace management, source control, changesets, and terminal execution
triggers:
  - set up MCPX gateway for AI development
  - connect Claude/ChatGPT to my local workspace
  - configure MCPX runtime with MCP tools
  - use MCPX for AI-assisted coding with remote sessions
  - manage workspace changesets through MCPX
  - run terminal commands via MCPX MCP server
  - inspect project structure with MCPX
  - bridge AI clients to local MCP servers
---

# MCPX Runtime

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

MCPX is an MCP Runtime (gateway) that runs in your development environment. It exposes a unified MCP interface over Streamable HTTP, allowing ChatGPT, Claude, Cursor, Grok, and other AI clients to understand projects, view unified diffs, modify source code, run tasks, collect environment information, and invoke local MCP servers and skills.

Development state is persisted in SQLite Remote Sessions, independent of any AI vendor or single `Mcp-Session-Id`. Different clients can query, authorize handoff, and continue the same development work.

## Installation

### From Release (Recommended)

Download the binary for your platform from [GitHub Releases](https://github.com/opentokenz/mcpx/releases):

```bash
# macOS/Linux
curl -L https://github.com/opentokenz/mcpx/releases/latest/download/mcpx-server-$(uname -s)-$(uname -m).tar.gz | tar xz
chmod +x mcpx-server
sudo mv mcpx-server /usr/local/bin/
```

### From Source

Requires **Go 1.26.1+**:

```bash
git clone https://github.com/opentokenz/mcpx.git
cd mcpx
go build -o bin/mcpx-server ./cmd/mcpx-server
sudo mv bin/mcpx-server /usr/local/bin/
```

## Starting the Server

```bash
# Basic start
mcpx-server

# Start with a workspace registered
mcpx-server --workspace /path/to/your/project

# Check version
mcpx-server -version
```

On first run, MCPX creates `~/.mcpx/` (override with `MCPX_HOME`) containing:

- `config.yaml` — global configuration (port, auth, security policies, workspaces)
- `.mcp.json` — upstream MCP server list (can be empty)
- `logs/` — audit logs
- `state/mcpx.db` — SQLite database for sessions, changesets, tasks, artifacts
- `tasks/` — persistent terminal task logs (mode 0600)
- `skills/` — optional skills directory
- `oauth-clients.json` — dynamic OAuth client registry (if using OAuth)
- `workspaces.example.yaml` — workspace configuration example

Default endpoint: `http://127.0.0.1:9090/mcp` (Streamable HTTP only)

## Configuration

### Basic `~/.mcpx/config.yaml`

```yaml
server:
  host: 127.0.0.1
  port: 9090

auth:
  mode: open  # or "bearer" or "oauth"
  # bearer:
  #   tokens:
  #     - env: MCPX_TOKEN
  
workspaces:
  - name: my-project
    root: /Users/you/projects/my-app
    description: Main application workspace
    
security:
  allow_commands:
    - npm
    - go
    - python
    - cargo
  deny_paths:
    - ~/.ssh
    - ~/.aws
    - /etc
    
limits:
  max_result_bytes: 262144  # 256KB inline result limit
  max_changeset_files: 100
```

### Workspace Configuration

Workspaces can be defined in `config.yaml` or separately in `~/.mcpx/workspaces.yaml`:

```yaml
workspaces:
  - name: frontend
    root: /Users/you/projects/app-ui
    description: React frontend
    
  - name: backend
    root: /Users/you/projects/app-api
    description: Go API server
    
  - name: docs
    root: /Users/you/projects/app-docs
    description: Documentation site
```

### Upstream MCP Configuration

Configure local MCP servers in `~/.mcpx/.mcp.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "uvx",
      "args": ["mcp-server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    }
  }
}
```

## Connecting AI Clients

### ChatGPT Desktop (macOS)

1. Open ChatGPT → Settings → Features → Model Context Protocol
2. Add server: `http://127.0.0.1:9090/mcp`
3. Name: `MCPX Local`

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcpx-local": {
      "transport": {
        "type": "streamable-http",
        "url": "http://127.0.0.1:9090/mcp"
      }
    }
  }
}
```

### Cursor

Add to Cursor settings:

```json
{
  "mcp.servers": {
    "mcpx": {
      "url": "http://127.0.0.1:9090/mcp"
    }
  }
}
```

## Key MCP Tools

MCPX exposes these tool categories through MCP:

### Workspace Management

```go
// List available workspaces
{
  "name": "workspace_list"
}

// Switch to a workspace
{
  "name": "workspace_switch",
  "arguments": {
    "name": "frontend"
  }
}
```

### Source Code Operations

```go
// Inspect project structure
{
  "name": "project_inspect",
  "arguments": {
    "action": "tree",
    "include_hidden": false
  }
}

// Search source code
{
  "name": "context_query",
  "arguments": {
    "query": "customer phone",
    "glob": "**/*.vue",
    "context_lines": 2
  }
}

// Read source files
{
  "name": "source_read",
  "arguments": {
    "paths": ["src/views/erp/order.vue"]
  }
}
```

### Changesets (Diff-First Workflow)

```go
// Prepare a changeset
{
  "name": "change_prepare",
  "arguments": {
    "draft_id": "fix-login-flow",
    "changes": [
      {
        "path": "internal/auth/login.go",
        "action": "update",
        "old_revision": "sha256:abc123...",
        "hunks": [
          {
            "old_start": 42,
            "old_count": 1,
            "new_start": 42,
            "new_count": 1,
            "lines": [
              " func Login(user string) error {",
              "-    return legacyLogin(user)",
              "+    return secureLogin(user)",
              " }"
            ]
          }
        ]
      }
    ]
  }
}

// Execute changeset (applies to workspace)
{
  "name": "change_execute",
  "arguments": {
    "draft_id": "fix-login-flow"
  }
}

// Rollback changeset
{
  "name": "change_rollback",
  "arguments": {
    "changeset_id": 42
  }
}
```

### Terminal Execution

```go
// Short command (inline result)
{
  "name": "command_execute",
  "arguments": {
    "command": "go test ./internal/auth",
    "description": "Run auth package tests"
  }
}

// Long-running task
{
  "name": "task_start",
  "arguments": {
    "command": "npm run dev",
    "description": "Start dev server",
    "persistent": true
  }
}

// List tasks
{
  "name": "task_list"
}

// Stop task
{
  "name": "task_stop",
  "arguments": {
    "task_id": "task-uuid"
  }
}
```

### Environment Information

```go
// Get environment snapshot
{
  "name": "environment_get",
  "arguments": {
    "sections": ["os", "toolchain", "network"]
  }
}

// Take screenshot
{
  "name": "screenshot_capture",
  "arguments": {
    "display": 0,
    "format": "png"
  }
}
```

### Upstream MCP Proxy

```go
// Call upstream MCP server
{
  "name": "mcp_call",
  "arguments": {
    "server_name": "github",
    "tool_name": "create_issue",
    "tool_args": {
      "owner": "opentokenz",
      "repo": "mcpx",
      "title": "Feature request",
      "body": "Add XYZ support"
    }
  }
}
```

## Remote Sessions

MCPX uses persistent SQLite-backed Remote Sessions that survive client reconnects:

```go
// Create a session bound to a workspace
{
  "name": "remote_session_create",
  "arguments": {
    "workspace_name": "frontend",
    "description": "Fix customer phone display bug"
  }
}

// List sessions
{
  "name": "remote_session_list",
  "arguments": {
    "workspace_name": "frontend"
  }
}

// Resume a session
{
  "name": "remote_session_attach",
  "arguments": {
    "session_id": "session-uuid"
  }
}
```

Sessions track:
- Changesets and their history
- Terminal tasks and logs
- Artifacts (test reports, build outputs)
- Confirmation requests
- ACL (who can access/modify)

## Workspace Observation

Monitor workspace activity from a separate terminal:

```bash
# Human-readable format
mcpx-server workspace frontend

# Machine-readable format
mcpx-server workspace --format json --history 200 frontend
```

Example text output:

```
╭─ #42 · 4f8c2e90 · command_execute
│ • Ran go test ./internal/auth
│   ↳ Modified login flow and ran tests
│ • Read stdout
│   ↳ 12 tests passed
╰────────────────────────

╭─ #43 · 4f8c2e90 · change_execute
│ • Edited internal/auth.go
│   ↳ internal/auth.go (update) +1 -1
│     -return legacyLogin()
│     +return secureLogin()
╰────────────────────────
```

## Security

### Authentication Modes

```yaml
auth:
  mode: open  # No auth (localhost only)
  
# OR bearer token
auth:
  mode: bearer
  bearer:
    tokens:
      - env: MCPX_TOKEN  # Read from env var
      - value: "static-token-here"  # Not recommended
      
# OR OAuth
auth:
  mode: oauth
  oauth:
    issuer: https://auth.example.com
    audience: mcpx-local
```

### Command & Path Policies

```yaml
security:
  allow_commands:
    - npm
    - go
    - python3
    - cargo
    - make
  deny_commands:
    - rm
    - dd
    - mkfs
    
  allow_paths:
    - ~/projects/**
  deny_paths:
    - ~/.ssh/**
    - ~/.aws/**
    - /etc/**
    
  semantic_confirmation:
    enabled: true
    threshold: critical  # or "high"
```

### Changeset Conflict Detection

MCPX validates file revisions before applying changes:

```go
// Read returns SHA-256
{
  "name": "source_read",
  "arguments": {
    "paths": ["src/app.go"]
  }
}
// Response includes: "revision": "sha256:abc123..."

// Prepare must match current revision
{
  "name": "change_prepare",
  "arguments": {
    "changes": [{
      "path": "src/app.go",
      "old_revision": "sha256:abc123...",  // Must match current
      "hunks": [...]
    }]
  }
}
```

If file changed externally, MCPX rejects the changeset.

## Real-World Workflow Example

```go
// 1. Agent lists workspaces
workspace_list() 
// → ["frontend", "backend", "docs"]

// 2. Create session for frontend work
remote_session_create({
  workspace_name: "frontend",
  description: "Add dark mode toggle"
})
// → session_id: "sess-abc123"

// 3. Search for theme-related code
context_query({
  query: "theme color",
  glob: "**/*.vue"
})
// → Finds src/components/ThemeToggle.vue

// 4. Read current implementation
source_read({
  paths: ["src/components/ThemeToggle.vue"]
})
// → Returns content + sha256:def456

// 5. Prepare changeset with diff
change_prepare({
  draft_id: "add-dark-mode",
  changes: [{
    path: "src/components/ThemeToggle.vue",
    action: "update",
    old_revision: "sha256:def456",
    hunks: [{
      old_start: 12,
      old_count: 2,
      new_start: 12,
      new_count: 5,
      lines: [
        " <template>",
        "-  <button>Toggle Theme</button>",
        "+  <button @click=\"toggleDarkMode\">",
        "+    {{ isDark ? '☀️' : '🌙' }} Toggle Theme",
        "+  </button>",
        " </template>"
      ]
    }]
  }]
})

// 6. Review unified diff
// (User confirms in UI or via progress_report)

// 7. Apply changeset
change_execute({
  draft_id: "add-dark-mode"
})
// → Writes to workspace, creates changeset #15

// 8. Run tests
command_execute({
  command: "npm test -- ThemeToggle",
  description: "Verify dark mode toggle"
})
// → Returns inline test results

// 9. If needed, rollback
change_rollback({
  changeset_id: 15
})
```

## Common Patterns

### Multi-File Refactoring

```go
// 1. Read all affected files
source_read({
  paths: [
    "src/auth/login.go",
    "src/auth/session.go",
    "src/middleware/auth.go"
  ]
})

// 2. Prepare atomic changeset
change_prepare({
  draft_id: "refactor-auth",
  changes: [
    { path: "src/auth/login.go", action: "update", hunks: [...] },
    { path: "src/auth/session.go", action: "update", hunks: [...] },
    { path: "src/middleware/auth.go", action: "update", hunks: [...] }
  ]
})

// 3. Execute atomically (all or nothing)
change_execute({ draft_id: "refactor-auth" })
```

### Long-Running Dev Server

```go
// Start persistent task
task_start({
  command: "npm run dev",
  persistent: true,
  description: "Vite dev server"
})
// → task_id: "task-123"

// Later: check if running
task_list()
// → Shows task-123 with PID, ports, uptime

// View logs
task_logs({
  task_id: "task-123",
  tail: 50
})

// Stop when done
task_stop({ task_id: "task-123" })
```

### Cross-Client Handoff

```go
// Developer A (Claude Desktop):
remote_session_create({
  workspace_name: "backend",
  description: "Add user export API"
})
// → session_id: "sess-xyz"

// Work, create changesets...

// Generate handoff token
remote_session_handoff({
  session_id: "sess-xyz",
  duration: 3600  // 1 hour
})
// → token: "handoff-token-abc"

// Developer B (ChatGPT):
remote_session_attach({
  session_id: "sess-xyz",
  handoff_token: "handoff-token-abc"
})
// → Continues same session, sees all history
```

## Troubleshooting

### Server won't start

```bash
# Check if port is in use
lsof -i :9090

# Try different port
mcpx-server --port 9091

# Check logs
tail -f ~/.mcpx/logs/audit.jsonl
```

### Client can't connect

```bash
# Verify server is running
curl http://127.0.0.1:9090/mcp

# Check auth mode
cat ~/.mcpx/config.yaml | grep auth -A 5

# If using bearer auth, verify token
curl -H "Authorization: Bearer $MCPX_TOKEN" http://127.0.0.1:9090/mcp
```

### Changeset conflicts

```go
// Re-read file to get current revision
source_read({ paths: ["src/app.go"] })

// Update old_revision in change_prepare
change_prepare({
  changes: [{
    old_revision: "sha256:NEW_HASH",  // Use fresh hash
    hunks: [...]
  }]
})
```

### Command policy blocked

```yaml
# Add to ~/.mcpx/config.yaml
security:
  allow_commands:
    - your-blocked-command
```

### Workspace not found

```bash
# List registered workspaces
mcpx-server workspace list

# Add workspace to config
cat >> ~/.mcpx/config.yaml <<EOF
workspaces:
  - name: myapp
    root: /full/path/to/myapp
EOF

# Restart server
```

## Advanced: Skills Integration

MCPX can discover and execute skills from `~/.mcpx/skills/`:

```markdown
<!-- ~/.mcpx/skills/deploy.md -->
---
name: deploy-staging
description: Deploy current branch to staging environment
executable: true
---

# Deploy to Staging

This skill deploys the current Git branch to staging.

**Usage:**

skill_call({
  name: "deploy-staging"
})
```

Skills can be:
- **Executable**: Run as shell commands
- **Informational**: Return documentation text

Define in `SKILL.md` or `skill.yaml` format.

## Resources

- **GitHub**: https://github.com/opentokenz/mcpx
- **Documentation**: See README.md in repository
- **MCP Specification**: https://modelcontextprotocol.io
- **License**: Apache-2.0

---

MCPX bridges the gap between AI clients and local development environments with persistent sessions, atomic changesets, secure command execution, and extensibility through upstream MCP servers and skills.
