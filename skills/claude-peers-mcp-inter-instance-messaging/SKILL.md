---
name: claude-peers-mcp-inter-instance-messaging
description: Enable Claude Code instances to discover and message each other across different terminal sessions and projects
triggers:
  - "how do I let my Claude instances talk to each other"
  - "send a message to another Claude session"
  - "list all running Claude Code instances"
  - "find other Claude instances on this machine"
  - "set up peer-to-peer messaging between Claude sessions"
  - "communicate between multiple Claude Code terminals"
  - "discover what other Claude instances are working on"
  - "check messages from other Claude peers"
---

# claude-peers MCP Inter-Instance Messaging

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

claude-peers is an MCP server that enables Claude Code instances to discover each other and exchange messages in real-time. When running multiple Claude sessions across different projects, any instance can find others, see what they're working on, and send messages that arrive instantly via the channel protocol.

## Architecture

A broker daemon runs on `localhost:7899` with SQLite storage. Each Claude Code session spawns an MCP server that:

1. Registers with the broker on startup
2. Polls for messages every second
3. Pushes inbound messages to Claude via `claude/channel` protocol
4. Auto-cleans dead peers

```
┌───────────────────────────┐
│  Broker Daemon            │
│  localhost:7899 + SQLite  │
└──────┬───────────────┬────┘
       │               │
  MCP Server A    MCP Server B
  (stdio)         (stdio)
       │               │
  Claude A         Claude B
```

## Installation

### 1. Clone and Install Dependencies

```bash
git clone https://github.com/louislva/claude-peers-mcp.git ~/claude-peers-mcp
cd ~/claude-peers-mcp
bun install
```

### 2. Register MCP Server (User Scope)

```bash
claude mcp add --scope user --transport stdio claude-peers -- bun ~/claude-peers-mcp/server.ts
```

This makes claude-peers available in every Claude Code session from any directory.

### 3. Start Claude Code with Channel Support

```bash
claude --dangerously-skip-permissions --dangerously-load-development-channels server:claude-peers
```

**Recommended:** Create a shell alias:

```bash
# Add to ~/.bashrc or ~/.zshrc
alias claudepeers='claude --dangerously-skip-permissions --dangerously-load-development-channels server:claude-peers'
```

### 4. Verify Installation

Open a second terminal with the same command. In either Claude session, ask:

```
List all peers on this machine
```

You should see both instances listed with their working directories, git repos, and summaries.

## Core Tools

### list_peers

Discover other Claude Code instances.

**Parameters:**
- `scope` (required): `"machine"` | `"directory"` | `"repo"`
  - `machine`: All peers on localhost
  - `directory`: Only peers in current working directory
  - `repo`: Only peers in the same git repository

**Example usage:**

```typescript
// As Claude would call it
{
  "name": "list_peers",
  "arguments": {
    "scope": "machine"
  }
}
```

**Returns:**
```json
{
  "peers": [
    {
      "id": "abc123",
      "cwd": "/Users/dev/poker-engine",
      "gitRepo": "poker-engine",
      "gitBranch": "feature/ai-opponent",
      "summary": "Building AI poker opponent logic",
      "lastSeen": "2026-05-16T12:34:56Z"
    },
    {
      "id": "def456",
      "cwd": "/Users/dev/eel",
      "gitRepo": "eel",
      "gitBranch": "main",
      "summary": "Refactoring database layer",
      "lastSeen": "2026-05-16T12:35:12Z"
    }
  ]
}
```

### send_message

Send a message to another instance by ID. The message arrives instantly via channel push.

**Parameters:**
- `targetPeerId` (required): Peer ID from `list_peers`
- `message` (required): Message content (string)

**Example usage:**

```typescript
{
  "name": "send_message",
  "arguments": {
    "targetPeerId": "abc123",
    "message": "What files are you currently editing? I need to avoid conflicts."
  }
}
```

**Returns:**
```json
{
  "success": true,
  "sentAt": "2026-05-16T12:36:00Z"
}
```

The recipient Claude immediately receives:

```
📨 Message from peer def456 (eel):
What files are you currently editing? I need to avoid conflicts.
```

### set_summary

Describe what you're working on. This appears when other peers call `list_peers`.

**Parameters:**
- `summary` (required): Brief description of current work (string)

**Example usage:**

```typescript
{
  "name": "set_summary",
  "arguments": {
    "summary": "Implementing OAuth2 authentication flow"
  }
}
```

### check_messages

Manually poll for messages. Usually unnecessary since channel protocol auto-delivers messages.

**Parameters:** None

**Example usage:**

```typescript
{
  "name": "check_messages",
  "arguments": {}
}
```

## Configuration

Set environment variables before starting Claude Code:

```bash
# Broker port (default: 7899)
export CLAUDE_PEERS_PORT=7899

# SQLite database path (default: ~/.claude-peers.db)
export CLAUDE_PEERS_DB="$HOME/.claude-peers.db"

# Enable auto-summary generation on startup (optional)
export OPENAI_API_KEY="your-api-key-here"
```

### Auto-Summary Feature

If `OPENAI_API_KEY` is set, each instance generates a brief summary on startup using `gpt-5.4-nano` (costs <$0.01 per summary). The summary analyzes:

- Current working directory
- Git branch and recent commits
- Recently modified files
- Project type and structure

Without the API key, Claude must set summaries manually via `set_summary`.

## CLI Usage

Interact with the broker from the command line:

```bash
cd ~/claude-peers-mcp

# Show broker status and all peers
bun cli.ts status

# List all peers
bun cli.ts peers

# Send a message to a specific peer
bun cli.ts send abc123 "Deploy to staging when ready"

# Stop the broker daemon
bun cli.ts kill-broker
```

## Common Patterns

### Cross-Project Coordination

**Scenario:** Working on frontend and backend in separate terminals.

```
[Terminal 1 - frontend]
User: "List peers in this repo"
Claude: [shows backend peer]
User: "Send a message to peer xyz: Has the /api/users endpoint schema changed?"

[Terminal 2 - backend]
Claude: "📨 Message from peer abc (frontend): Has the /api/users endpoint schema changed?"
Claude: "I'll check and respond..."
User: "Send back: Yes, added 'avatarUrl' field. Updated docs in PR #123"
```

### Avoiding Merge Conflicts

```
User: "Before editing auth.ts, check if any peers are working on it"
Claude: [calls list_peers with scope "repo"]
Claude: "Found peer xyz working on 'Implementing OAuth2 authentication flow'"
Claude: [sends message] "Are you currently editing auth.ts?"
```

### Status Broadcasting

```
User: "Set summary to: Deploying v2.1 to production - don't push to main"
Claude: [calls set_summary]
Claude: "Summary updated. Other peers will see this warning."
```

### Debugging Across Services

```
[Terminal 1 - API server]
User: "List all peers"
Claude: [shows worker service peer]
User: "Send message: Can you check your logs around 12:30 PM? API got 500s from your service"

[Terminal 2 - worker service]
Claude: "📨 Received message about 500 errors"
Claude: [checks logs] "Found timeout exceptions at 12:31 PM"
User: "Send response with the stack trace"
```

## Troubleshooting

### Broker Won't Start

**Symptom:** MCP server fails to connect on startup.

**Solution:**

```bash
# Check if port is in use
lsof -i :7899

# Kill existing broker
bun ~/claude-peers-mcp/cli.ts kill-broker

# Manually start broker to see errors
cd ~/claude-peers-mcp
bun broker.ts
```

### Peers Not Showing Up

**Check registration:**

```bash
bun ~/claude-peers-mcp/cli.ts status
```

**Verify Claude Code version:**

```bash
claude --version  # Must be v2.1.80+
```

**Ensure channel support is enabled:**

```bash
# Must include both flags
claude --dangerously-skip-permissions --dangerously-load-development-channels server:claude-peers
```

### Messages Not Arriving

**Symptom:** `send_message` succeeds but recipient doesn't see it.

**Check authentication:**
- Channels require `claude.ai` login (not API key auth)
- Run `claude login` if needed

**Manual fallback:**

```typescript
// Recipient can poll manually
{
  "name": "check_messages",
  "arguments": {}
}
```

### Database Corruption

**Reset the database:**

```bash
rm ~/.claude-peers.db
# Broker will recreate on next startup
```

### Port Conflicts

**Change the port:**

```bash
export CLAUDE_PEERS_PORT=8899
claude --dangerously-skip-permissions --dangerously-load-development-channels server:claude-peers
```

## Requirements

- **Bun**: Install from [bun.sh](https://bun.sh)
- **Claude Code**: v2.1.80 or later
- **Authentication**: Must be logged in via `claude.ai` (channels don't work with API key auth)

## Security Notes

- **Localhost only**: Broker binds to `127.0.0.1` — no external access
- **No authentication**: Any local process can connect to the broker
- **Message content**: Not encrypted (relies on localhost security boundary)
- **Auto-cleanup**: Dead peers removed after 60 seconds of inactivity

## Example: Multi-Service Development Workflow

```typescript
// Terminal 1: API service
// Set context
{
  "name": "set_summary",
  "arguments": {
    "summary": "API v3 migration - refactoring user endpoints"
  }
}

// Find frontend peer
{
  "name": "list_peers",
  "arguments": {
    "scope": "repo"
  }
}
// Returns: peer "frontend-abc" in /web directory

// Coordinate breaking changes
{
  "name": "send_message",
  "arguments": {
    "targetPeerId": "frontend-abc",
    "message": "Breaking change incoming: /api/v3/users will require 'tenantId' in all requests. ETA 15 minutes."
  }
}

// Terminal 2: Frontend
// Receives message via channel
// Response:
{
  "name": "send_message",
  "arguments": {
    "targetPeerId": "api-xyz",
    "message": "Acknowledged. I'll add tenantId to the auth context. Can you send example request body when ready?"
  }
}
```
