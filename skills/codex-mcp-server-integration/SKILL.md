---
name: codex-mcp-server-integration
description: Use OpenAI Codex CLI through MCP to get AI-powered code analysis, generation, review, and web search directly in your editor
triggers:
  - analyze this code with codex
  - use codex to refactor this function
  - review my uncommitted changes with codex
  - search for latest documentation on this library
  - explain this code using codex
  - create a codex session for this feature
  - review my branch changes before merging
  - use codex to optimize this algorithm
---

# Codex MCP Server Integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Codex MCP Server bridges Claude Code, Cursor, and other MCP-compatible editors with OpenAI's Codex CLI. It provides AI-powered code analysis, generation, review, and web search capabilities through the Model Context Protocol (MCP).

**Architecture:**
```
Claude Code/Cursor → Codex MCP Server → Codex CLI → OpenAI API
```

## Installation

### Step 1: Install Codex CLI

```bash
# Via npm (recommended)
npm install -g @openai/codex

# Via Homebrew
brew install codex

# Verify installation
codex --version  # Should be 0.75.0 or higher
```

### Step 2: Authenticate Codex CLI

```bash
# Set your OpenAI API key
codex login --api-key "$OPENAI_API_KEY"

# Verify authentication
codex ping
```

### Step 3: Install MCP Server

**For Claude Code:**
```bash
claude mcp add codex-cli -- npx -y codex-mcp-server
```

**Manual configuration** (add to MCP settings file):

```json
{
  "mcpServers": {
    "codex-cli": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "codex-mcp-server"]
    }
  }
}
```

**With static callback URI:**
```json
{
  "mcpServers": {
    "codex-cli": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "codex-mcp-server"],
      "env": {
        "CODEX_MCP_CALLBACK_URI": "http://localhost:8080/mcp-callback"
      }
    }
  }
}
```

## Available Tools

### 1. codex — AI Coding Assistant

The main tool for code analysis, generation, and assistance.

**Basic usage:**
```typescript
// Ask a simple question about code
codex({
  prompt: "Explain what this function does and suggest improvements",
  context: ["src/auth/login.ts"]
})
```

**Parameters:**
- `prompt` (required): Your question or instruction
- `context` (optional): Array of file paths to include
- `sessionId` (optional): Resume a previous conversation
- `model` (optional): Override model (e.g., "o3", "gpt-4")
- `reasoningEffort` (optional): "low", "medium", "high" for reasoning models
- `fullAuto` (optional): Enable autonomous mode
- `sandbox` (optional): Sandbox permissions ("read-only", "workspace-write")
- `callbackUri` (optional): MCP callback URI for this request
- `structuredContent` (optional): Return threadId and structured metadata

**Response includes:**
- `response`: The AI's text response
- `threadId`: Conversation thread ID (if available from Codex 0.87+)
- `metadata`: Additional context about the response

### 2. review — Code Review

AI-powered code review for uncommitted changes, branches, or commits.

**Review uncommitted changes:**
```typescript
review({
  uncommitted: true
})
```

**Review a branch:**
```typescript
review({
  base: "main",
  head: "feature/new-api"
})
```

**Review specific commits:**
```typescript
review({
  commit: "abc123..def456"
})
```

**Parameters:**
- `uncommitted` (optional): Review uncommitted changes
- `base` (optional): Base branch for comparison
- `head` (optional): Head branch/commit to review
- `commit` (optional): Specific commit or range
- `model` (optional): Override model
- `reasoningEffort` (optional): Reasoning level

### 3. websearch — Web Search

Search the web using Codex CLI's integrated search.

```typescript
websearch({
  query: "React Server Components best practices 2025",
  numResults: 10,
  searchDepth: "full"
})
```

**Parameters:**
- `query` (required): Search query
- `numResults` (optional): Number of results (default: 10)
- `searchDepth` (optional): "quick" or "full" (default: "quick")

### 4. listSessions — View Active Sessions

List all active conversation sessions for this server instance.

```typescript
listSessions()
```

Returns array of session objects with `id` and `messageCount`.

### 5. ping — Test Connection

Verify the server is responding.

```typescript
ping()
```

### 6. help — Get CLI Help

Get help information from Codex CLI.

```typescript
help({
  command: "review"  // Optional: specific command
})
```

## Common Patterns

### Multi-Turn Conversations

Use `sessionId` to maintain context across multiple interactions:

```typescript
// Start a refactoring session
codex({
  prompt: "Analyze this authentication module for security issues",
  context: ["src/auth/index.ts"],
  sessionId: "auth-refactor"
})

// Continue in the same session
codex({
  prompt: "Implement the security fixes you suggested",
  sessionId: "auth-refactor"
})

// Follow up with more questions
codex({
  prompt: "Add unit tests for the new security checks",
  sessionId: "auth-refactor"
})

// Check active sessions
listSessions()
// Returns: [{ id: "auth-refactor", messageCount: 3 }]
```

### Code Analysis Workflows

**Security audit:**
```typescript
codex({
  prompt: "Perform a security audit focusing on: 1) Input validation, 2) Authentication bypasses, 3) SQL injection risks, 4) XSS vulnerabilities",
  context: ["src/api/**/*.ts"],
  model: "o3",
  reasoningEffort: "high"
})
```

**Performance optimization:**
```typescript
codex({
  prompt: "Identify performance bottlenecks and suggest optimizations with Big O analysis",
  context: ["src/services/data-processor.ts"],
  sessionId: "perf-optimization"
})
```

**Refactoring guidance:**
```typescript
codex({
  prompt: "Suggest refactoring to improve maintainability: extract reusable patterns, reduce complexity, improve naming",
  context: ["src/legacy/user-manager.js"]
})
```

### Pre-Commit Code Review

```typescript
// Review all uncommitted changes
review({
  uncommitted: true,
  model: "gpt-4"
})

// Review specific branch before PR
review({
  base: "main",
  head: "feature/user-permissions",
  reasoningEffort: "high"
})
```

### Research and Documentation

```typescript
// Find latest best practices
websearch({
  query: "TypeScript 5.8 new features decorators",
  numResults: 15,
  searchDepth: "full"
})

// Learn about a library
websearch({
  query: "Prisma vs TypeORM 2025 comparison pros cons",
  numResults: 10
})

// Then ask codex to help implement
codex({
  prompt: "Based on current best practices, help me migrate this ORM code to Prisma",
  context: ["src/models/user.ts"]
})
```

### Autonomous Mode with Sandbox

For tasks that require file modifications:

```typescript
codex({
  prompt: "Implement a REST API endpoint for user registration with validation, error handling, and tests",
  fullAuto: true,
  sandbox: "workspace-write",
  context: ["src/api/routes/"]
})
```

**Sandbox modes:**
- `"read-only"`: Can read but not modify files
- `"workspace-write"`: Can create/modify files in workspace

### Using Thread IDs (Codex 0.87+)

When using Codex CLI 0.87+, capture thread IDs for conversation tracking:

```typescript
const result = codex({
  prompt: "Design a caching strategy for this API",
  context: ["src/api/handlers.ts"],
  structuredContent: true
})

// result.threadId available for tracking
// result.metadata contains additional context
```

## Configuration

### Environment Variables

**CODEX_MCP_CALLBACK_URI**: Set a static MCP callback URI for all requests.

```bash
export CODEX_MCP_CALLBACK_URI="http://localhost:8080/mcp-callback"
```

This can be overridden per-request using the `callbackUri` parameter in the `codex` tool.

### Model Selection

Override the default model for specific tasks:

```typescript
// Use reasoning model for complex logic
codex({
  prompt: "Design a distributed locking mechanism",
  model: "o3",
  reasoningEffort: "high"
})

// Use faster model for simple tasks
codex({
  prompt: "Add JSDoc comments to this function",
  model: "gpt-4-turbo"
})
```

### Codex CLI Configuration

Check your Codex CLI configuration:

```bash
# View current config
codex config list

# Set default model
codex config set model gpt-4

# View authentication status
codex whoami
```

## Troubleshooting

### "Codex CLI not found"

**Solution:**
```bash
# Verify installation
which codex

# Reinstall if needed
npm install -g @openai/codex@latest

# Ensure it's in PATH
echo $PATH
```

### "Authentication failed"

**Solution:**
```bash
# Re-authenticate
codex login --api-key "$OPENAI_API_KEY"

# Verify key is valid
codex ping
```

### "Session not found"

Sessions are scoped to the MCP server instance. They reset when the server restarts.

**Solution:**
- List active sessions: `listSessions()`
- Start a new session with the same ID to resume conceptually

### "Version compatibility error"

**Solution:**
```bash
# Check Codex CLI version
codex --version

# Update to latest
npm update -g @openai/codex

# Minimum required: 0.75.0
# Recommended: 0.87.0+ for thread ID support
```

### "Model not available"

Some models require specific API access.

**Solution:**
```typescript
// Fallback to widely available model
codex({
  prompt: "your prompt",
  model: "gpt-4"  // or omit to use default
})
```

### Server not responding

**Solution:**
```bash
# Test server connection
ping()

# Restart MCP server in editor
# For Claude Code: Reload window or restart
# For Cursor: Reload window

# Check MCP logs in editor's output panel
```

## Best Practices

1. **Use sessions for related work**: Group related questions in a session to maintain context.

2. **Provide context files**: Always include relevant files in the `context` array for better responses.

3. **Choose appropriate models**: Use reasoning models (`o3` + `reasoningEffort: "high"`) for complex logic, faster models for simple tasks.

4. **Review before merging**: Use `review` tool on branches before creating PRs.

5. **Leverage web search**: Combine `websearch` with `codex` to incorporate latest best practices.

6. **Scope sandbox permissions**: Use `"read-only"` by default; only use `"workspace-write"` when file modifications are needed.

7. **Track conversations**: Use `structuredContent: true` with Codex 0.87+ to capture thread IDs for audit trails.

## Example Integration Script

```typescript
// workflow-helper.ts - Automated code review workflow

async function preCommitWorkflow() {
  // 1. Review uncommitted changes
  const reviewResult = await review({
    uncommitted: true,
    reasoningEffort: "high"
  });
  
  console.log("Review:", reviewResult.review);
  
  // 2. If issues found, get fix suggestions
  if (reviewResult.issues?.length > 0) {
    const fixes = await codex({
      prompt: `These issues were found in code review:\n${reviewResult.issues.join('\n')}\n\nProvide specific fixes with code examples.`,
      sessionId: "pre-commit-fixes"
    });
    
    console.log("Suggested fixes:", fixes.response);
  }
  
  // 3. Search for related best practices
  const research = await websearch({
    query: "TypeScript error handling best practices 2025",
    numResults: 5
  });
  
  console.log("Best practices:", research.results);
}

// Run before committing
preCommitWorkflow();
```

## Related Resources

- **Codex CLI Documentation**: Run `codex help` or visit OpenAI documentation
- **MCP Protocol**: https://modelcontextprotocol.io
- **API Reference**: See project's `docs/api-reference.md`
- **Session Management**: See project's `docs/session-management.md`
