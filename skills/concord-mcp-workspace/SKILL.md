---
name: concord-mcp-workspace
description: Shared work-state and task coordination for AI coding agents using the Model Context Protocol
triggers:
  - set up concord workspace for agent coordination
  - claim work in concord and track task ownership
  - hand off work to another agent using concord
  - register my agent presence in the workspace
  - check what other agents are working on
  - create a review packet for this task
  - accept or decline a task handoff
  - manage multi-agent collaboration with concord
---

# Concord MCP Workspace Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Concord MCP provides a shared workspace for AI coding agents to coordinate work, preserve decisions, and hand off tasks cleanly. It's "Google Workspace for AI agents" — giving Claude Code, Codex, Cursor, and other MCP-capable agents one local-first place to track presence, ownership, task memory, and handoffs within a repository.

**Core capabilities:**
- Agent presence registration and roster tracking
- Task claiming with scoped ownership and version control
- Explicit handoffs with acceptance/decline workflows
- Task memory (decisions, assumptions, findings) attached to work
- Stale claim detection when agents disappear
- Review packet generation with scope, tests, risks, and provenance

## Installation

### Global CLI Installation

```bash
npm install -g @concord-ai/concord-mcp
```

### Per-Client Setup

**Cursor (one-click):**
```bash
# Use the deeplink or:
concord install
```

**Claude Code, Codex, other MCP clients:**
```bash
concord install
```

The `concord install` command:
- Registers the MCP server in `.mcp.json`, `.cursor/mcp.json`, or `~/.codex/config.toml`
- Writes agent instructions to `CLAUDE.md`, `AGENTS.md`, `.codex/`, `.cursor/rules/`
- Merges with existing config (safe to re-run)

**Manual MCP registration** (if using `--no-mcp`):
```json
{
  "mcpServers": {
    "concord": {
      "command": "npx",
      "args": ["-y", "@concord-ai/concord-mcp@latest"]
    }
  }
}
```

### Initialize Repository Workspace

```bash
cd /path/to/your/repo
concord init
```

Creates `.concord/` directory with SQLite workspace (auto-added to `.gitignore`).

## Environment Variables

```bash
# Optional: Set explicit repository root
export CONCORD_REPO_ROOT=/path/to/repo

# Optional: Restrict dynamic workspace joins (path-delimited)
export CONCORD_ALLOWED_ROOTS=/path/to/repo1:/path/to/repo2

# Optional: Disable CLI update checks
export CONCORD_NO_UPDATE_CHECK=1
```

## MCP Tools Reference

### Workspace & Presence

#### `join_workspace`
Join or select a repository workspace without restarting MCP.

```typescript
{
  "repository_path": "/absolute/path/to/repo"
}
// Returns: { workspace_id: "ws_...", repository_root: "..." }
```

#### `register_agent`
Register agent presence and keep it alive.

```typescript
{
  "agent_id": "cursor-agent-1",
  "agent_type": "cursor",
  "capabilities": ["code-generation", "review"]
  // Optional: workspace_id
}
// Returns: { agent_id, expires_at, ... }
```

#### `get_work_state`
Get roster, active tasks, overlaps, and stale claims.

```typescript
{
  "agent_id": "cursor-agent-1"
  // Optional: workspace_id
}
// Returns: { agents: [...], tasks: [...], overlaps: [...], stale_claims: [...] }
```

### Task Creation & Claiming

#### `claim_work`
Claim scoped work and create a task.

```typescript
{
  "agent_id": "cursor-agent-1",
  "task_id": "refactor-auth",
  "scope": {
    "files": ["src/auth.ts", "src/middleware/auth.ts"],
    "modules": ["authentication"],
    "description": "Refactor authentication to use JWT"
  }
  // Optional: workspace_id, metadata
}
// Returns: { task_id, version, status: "active", ... }
```

#### `update_task`
Record task memory (decisions, assumptions, findings).

```typescript
{
  "agent_id": "cursor-agent-1",
  "task_id": "refactor-auth",
  "context_update": {
    "decisions": ["Use jsonwebtoken library", "Store tokens in httpOnly cookies"],
    "assumptions": ["Token expiry set to 24 hours"],
    "findings": ["Existing auth uses sessions, needs migration"]
  }
  // Optional: workspace_id, expected_version
}
```

#### `get_task_context`
Read task memory and ownership history.

```typescript
{
  "task_id": "refactor-auth"
  // Optional: workspace_id
}
// Returns: { context: {...}, ownership_history: [...] }
```

### Task Assignment & Transfer

#### `assign_task`
Assign work to a named agent (requires acceptance).

```typescript
{
  "agent_id": "cursor-agent-1",
  "task_id": "refactor-auth",
  "assignee_id": "claude-code-agent",
  "expected_version": 1
  // Optional: workspace_id
}
// Task moves to "assigned" status, awaiting acceptance
```

#### `accept_task`
Accept an assigned task.

```typescript
{
  "agent_id": "claude-code-agent",
  "task_id": "refactor-auth",
  "expected_version": 2
  // Optional: workspace_id
}
// Task moves to "active" with new owner
```

#### `release_task`
Release ownership without assigning.

```typescript
{
  "agent_id": "cursor-agent-1",
  "task_id": "refactor-auth",
  "expected_version": 1
  // Optional: workspace_id
}
```

#### `reassign_task`
Transfer to another agent with versioning.

```typescript
{
  "agent_id": "cursor-agent-1",
  "task_id": "refactor-auth",
  "new_assignee_id": "codex-agent",
  "expected_version": 1
  // Optional: workspace_id
}
```

### Handoffs

#### `offer_handoff`
Deliver handoff to named recipient with acceptance workflow.

```typescript
{
  "agent_id": "cursor-agent-1",
  "task_id": "refactor-auth",
  "recipient_id": "claude-code-agent",
  "handoff_data": {
    "summary": "JWT auth refactor complete",
    "completed_work": ["Implemented JWT signing/verification", "Added middleware"],
    "remaining_work": ["Write integration tests", "Update documentation"],
    "files_changed": ["src/auth.ts", "src/middleware/auth.ts"],
    "tests_added": ["src/auth.test.ts"],
    "risks": ["Session migration not handled"]
  },
  "expected_version": 1
  // Optional: workspace_id, expires_in_seconds (default 300)
}
```

#### `accept_handoff`
Accept offered handoff.

```typescript
{
  "agent_id": "claude-code-agent",
  "task_id": "refactor-auth"
  // Optional: workspace_id
}
```

#### `decline_handoff`
Decline offered handoff.

```typescript
{
  "agent_id": "claude-code-agent",
  "task_id": "refactor-auth",
  "reason": "Need to finish current task first"
  // Optional: workspace_id
}
```

#### `handoff`
Record completion/review evidence without transferring ownership.

```typescript
{
  "agent_id": "cursor-agent-1",
  "task_id": "refactor-auth",
  "handoff_data": {
    "summary": "Ready for review",
    "completed_work": ["All implementation done"],
    "files_changed": ["src/auth.ts"],
    "tests_added": ["src/auth.test.ts"]
  }
  // Optional: workspace_id
}
```

### Task Closure

#### `close_task`
Mark task complete or closed.

```typescript
{
  "agent_id": "cursor-agent-1",
  "task_id": "refactor-auth",
  "outcome": "complete",
  "expected_version": 3
  // Optional: workspace_id
}
```

#### `reopen_task`
Reopen a closed task (audited).

```typescript
{
  "agent_id": "claude-code-agent",
  "task_id": "refactor-auth",
  "reason": "Found edge case in production"
  // Optional: workspace_id
}
```

## CLI Commands

### Workspace Management

```bash
# Initialize workspace in current repo
concord init

# Select workspace by repository path
concord --repo /path/to/project status

# Select workspace by ID
concord --workspace ws_abc123 status
```

### Status & Monitoring

```bash
# View roster, active work, overlaps, stale claims
concord status

# Live dashboard (TUI)
concord dashboard
# Tab: change panes | j/k: navigate | /: filter | ?: help | q: quit

# List active agents and their work
concord who

# List all tasks
concord tasks
```

### Handoff & Review

```bash
# Print latest handoff for task
concord handoff refactor-auth

# Print review packet
concord review-packet refactor-auth

# Export markdown artifacts to .concord/
concord export markdown
```

### Diagnostics

```bash
# Run workspace health checks
concord doctor
```

## Common Patterns

### Multi-Agent Workflow

```typescript
// Agent 1: Claim and start work
await use_mcp_tool("concord", "register_agent", {
  agent_id: "agent-1",
  agent_type: "cursor",
  capabilities: ["code-generation"]
});

await use_mcp_tool("concord", "claim_work", {
  agent_id: "agent-1",
  task_id: "add-logging",
  scope: {
    files: ["src/logger.ts"],
    description: "Add structured logging"
  }
});

// Record decisions as you work
await use_mcp_tool("concord", "update_task", {
  agent_id: "agent-1",
  task_id: "add-logging",
  context_update: {
    decisions: ["Use winston library", "Log to both console and file"],
    assumptions: ["Log rotation handled by external service"]
  }
});

// Agent 2: Register and check for overlaps
await use_mcp_tool("concord", "register_agent", {
  agent_id: "agent-2",
  agent_type: "claude-code"
});

const state = await use_mcp_tool("concord", "get_work_state", {
  agent_id: "agent-2"
});
// Check state.overlaps before claiming similar scope

// Agent 1: Offer handoff when ready
await use_mcp_tool("concord", "offer_handoff", {
  agent_id: "agent-1",
  task_id: "add-logging",
  recipient_id: "agent-2",
  handoff_data: {
    summary: "Logging implementation complete, needs tests",
    completed_work: ["Implemented winston logger", "Added config"],
    remaining_work: ["Write unit tests", "Add error scenarios"],
    files_changed: ["src/logger.ts", "src/config/logger.ts"]
  },
  expected_version: 1
});

// Agent 2: Accept handoff
await use_mcp_tool("concord", "accept_handoff", {
  agent_id: "agent-2",
  task_id: "add-logging"
});
```

### Task Context Preservation

```typescript
// Continuously update context as you learn
await use_mcp_tool("concord", "update_task", {
  agent_id: "my-agent",
  task_id: "optimize-queries",
  context_update: {
    findings: [
      "Query uses N+1 pattern in user.posts",
      "Database missing index on posts.user_id"
    ],
    decisions: [
      "Add eager loading with includes",
      "Create migration for index"
    ]
  }
});

// Later: Read context before continuing
const context = await use_mcp_tool("concord", "get_task_context", {
  task_id: "optimize-queries"
});
// Use context.decisions and context.findings to continue work
```

### Review-Ready Handoff

```typescript
// Complete work and prepare review packet
await use_mcp_tool("concord", "handoff", {
  agent_id: "my-agent",
  task_id: "api-versioning",
  handoff_data: {
    summary: "Added v2 API with backwards compatibility",
    completed_work: [
      "Implemented /api/v2 routes",
      "Added version middleware",
      "Maintained v1 compatibility"
    ],
    files_changed: [
      "src/routes/v2/*.ts",
      "src/middleware/version.ts"
    ],
    tests_added: [
      "tests/api/v2/users.test.ts",
      "tests/middleware/version.test.ts"
    ],
    risks: [
      "V1 deprecation timeline not finalized",
      "Migration guide needed for clients"
    ],
    notes: "Consider adding deprecation warnings to v1 responses"
  }
});

// Close when reviewed and merged
await use_mcp_tool("concord", "close_task", {
  agent_id: "my-agent",
  task_id: "api-versioning",
  outcome: "complete",
  expected_version: 2
});
```

### Handling Stale Claims

```bash
# Check for stale claims
concord status

# From CLI, manually release if needed:
# (MCP agents should monitor get_work_state for stale_claims)
```

```typescript
// In agent: Detect and handle stale claims
const state = await use_mcp_tool("concord", "get_work_state", {
  agent_id: "my-agent"
});

if (state.stale_claims.length > 0) {
  // Option 1: Claim abandoned work
  const stale = state.stale_claims[0];
  await use_mcp_tool("concord", "claim_work", {
    agent_id: "my-agent",
    task_id: stale.task_id,
    scope: stale.scope
  });
  
  // Option 2: Read context and continue
  const context = await use_mcp_tool("concord", "get_task_context", {
    task_id: stale.task_id
  });
}
```

## Workspace Structure

```
.concord/
├── concord.db          # SQLite source of truth
├── HANDOFF.md          # Human-readable latest handoff
├── REVIEW_PACKET.md    # Generated review packet
└── WORK_STATE.json     # Optional exported state
```

## Troubleshooting

### MCP Server Not Recognized

**Symptom:** Agent can't see Concord tools

**Solution:**
```bash
# Re-run install
concord install

# Verify registration
cat .cursor/mcp.json  # or .mcp.json

# Restart your AI agent/IDE
```

### Workspace Not Found

**Symptom:** "No workspace selected" errors

**Solution:**
```bash
# Ensure you're in a repo with .concord/
ls .concord/

# Or initialize
concord init

# Or set explicit root
export CONCORD_REPO_ROOT=$(pwd)

# Or use join_workspace tool
await use_mcp_tool("concord", "join_workspace", {
  repository_path: "/absolute/path/to/repo"
});
```

### Version Conflicts

**Symptom:** "Expected version X but found Y" errors

**Solution:**
```typescript
// Always read current version first
const context = await use_mcp_tool("concord", "get_task_context", {
  task_id: "my-task"
});

// Use the current version
await use_mcp_tool("concord", "close_task", {
  agent_id: "my-agent",
  task_id: "my-task",
  expected_version: context.version,
  outcome: "complete"
});
```

### Agent Presence Expired

**Symptom:** Agent shows as offline in roster

**Solution:**
```typescript
// Register more frequently (any write operation extends presence)
await use_mcp_tool("concord", "register_agent", {
  agent_id: "my-agent",
  agent_type: "cursor"
});

// Or make presence implicit by working
await use_mcp_tool("concord", "update_task", {
  agent_id: "my-agent",  // Keeps presence alive
  task_id: "my-task",
  context_update: { /* ... */ }
});
```

### Linked Worktree Issues

**Symptom:** Different `.concord/` in worktree vs main checkout

**Solution:** Concord automatically follows Git's `commondir` to share workspace. If not working:

```bash
# Verify Git worktree setup
git worktree list

# Ensure .git/commondir exists in worktree
cat .git/commondir

# Both should resolve to same .concord/ location
```

### Upgrade Issues

```bash
# Update global package
npm install -g @concord-ai/concord-mcp@latest

# Verify version
concord --version

# Migrations run automatically on next workspace access
```

## Best Practices

1. **Register early:** Call `register_agent` when starting work
2. **Check overlaps:** Use `get_work_state` before claiming new scope
3. **Update context frequently:** Record decisions/findings as you go
4. **Use versions:** Always pass `expected_version` for lifecycle changes
5. **Explicit handoffs:** Use `offer_handoff` + `accept_handoff` for clean transfers
6. **Review packets:** Call `handoff` with comprehensive data before marking complete
7. **Monitor stale claims:** Check `get_work_state.stale_claims` regularly
