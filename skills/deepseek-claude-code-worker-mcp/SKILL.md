---
name: deepseek-claude-code-worker-mcp
description: Delegate coding tasks to DeepSeek V4 through Claude Code MCP worker, saving Codex main-thread tokens with async background jobs
triggers:
  - "delegate this implementation to a DeepSeek worker"
  - "start a background coding job with DeepSeek"
  - "check the status of my DeepSeek worker"
  - "use the DeepSeek code worker MCP for this task"
  - "set up the DeepSeek Claude Code worker"
  - "run this coding work in a separate DeepSeek session"
  - "save tokens by using the DeepSeek worker"
  - "review the DeepSeek worker's changes"
---

# DeepSeek Claude Code Worker MCP

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

DeepSeek Claude Code Worker MCP is a coding-worker server for Codex Desktop that delegates expensive code reading, editing, and checking to DeepSeek V4 through Claude Code. The workflow is: Codex plans and reviews, DeepSeek V4 executes implementation work in isolated background jobs. This saves Codex main-thread tokens by 40-60% on suitable coding tasks.

**Key concept**: This is not a standalone DeepSeek client. It includes a `claude-deepseek` launcher that runs the local Claude Code CLI against DeepSeek's Anthropic-compatible endpoint.

**Current beta**: `v0.3.20-beta.38`

## Installation

### GitHub (no global install)

Add to your MCP config (`~/.codex/config.toml` or similar):

```json
{
  "mcpServers": {
    "deepseek-code-worker": {
      "command": "npx",
      "args": [
        "github:louchi1984-coder/deepseek-claude-code-worker-mcp#v0.3.20-beta.38"
      ]
    }
  }
}
```

### Source mode (recommended for development)

```bash
git clone https://github.com/louchi1984-coder/deepseek-claude-code-worker-mcp.git
cd deepseek-claude-code-worker-mcp
npm install
npm run mcp:setup
npm run mcp:doctor
```

Source-mode MCP config:

```json
{
  "mcpServers": {
    "deepseek-code-worker": {
      "command": "node",
      "args": ["/absolute/path/to/deepseek-claude-code-worker-mcp/src/deepseek-worker-mcp.mjs"]
    }
  }
}
```

### Quick check without installing

```bash
npx github:louchi1984-coder/deepseek-claude-code-worker-mcp#v0.3.20-beta.38 --doctor
```

Expected output:

```json
{
  "server_version": "0.3.20-beta.38",
  "ok": true
}
```

## Requirements

- Node.js 20+
- Claude Code CLI (`@anthropic-ai/claude-code`)
- DeepSeek API key
- macOS / Linux (Windows is best-effort)

### Environment variables

- `DEEPSEEK_API_KEY` or `~/.codex/secrets/deepseek_api_key`
- `ANTHROPIC_AUTH_TOKEN` (for Claude Code)
- `DEEPSEEK_API_KEY_FILE` (alternative key location)
- `CLAUDE_BIN` (custom Claude Code CLI path)

Setup can interactively install Claude Code and prompt for DeepSeek key if missing.

## Core Tools

### deepseek_start_implementation

Starts a background coding job. Returns `job_id` immediately.

```javascript
// Minimal start
{
  "name": "deepseek_start_implementation",
  "arguments": {
    "cwd": "/absolute/project/path",
    "task": "Add error handling to the authentication module"
  }
}

// With boundaries and validation
{
  "name": "deepseek_start_implementation",
  "arguments": {
    "cwd": "/home/user/myproject",
    "task": "Implement rate limiting middleware with Redis backend",
    "allowed_dirs": ["src/middleware", "src/lib/redis"],
    "forbidden_paths": ["src/config/secrets.json", ".env"],
    "generated_paths": ["docs/RATE_LIMIT_EVAL.md"],
    "validation_commands": [
      "npm test -- middleware.test.js",
      "npm run lint src/middleware"
    ],
    "use_case": "simple_agent_task",
    "worker_profile": "scoped_patch"
  }
}

// Complex reasoning task
{
  "name": "deepseek_start_implementation",
  "arguments": {
    "cwd": "/home/user/complex-app",
    "task": "Debug and fix the race condition in the WebSocket message queue",
    "use_case": "debug_loop",
    "allowed_dirs": ["src/websocket", "src/queue", "tests"],
    "validation_commands": ["npm run test:integration:ws"]
  }
}
```

**Returns**:

```json
{
  "job_id": "dsw_abc123",
  "status": "running",
  "cwd": "/home/user/myproject",
  "started_at": "2026-06-01T10:30:00Z"
}
```

### deepseek_get_job

Reads compact job status. Does not include logs/events/diffs by default.

```javascript
// Basic status check
{
  "name": "deepseek_get_job",
  "arguments": {
    "job_id": "dsw_abc123"
  }
}

// With full evidence (use sparingly)
{
  "name": "deepseek_get_job",
  "arguments": {
    "job_id": "dsw_abc123",
    "include_logs": true,
    "include_events": true,
    "include_diff": true
  }
}
```

**Returns** (compact):

```json
{
  "job_id": "dsw_abc123",
  "status": "completed",
  "exit_code": 0,
  "files_changed": ["src/middleware/rateLimiter.js", "tests/middleware.test.js"],
  "checks": {
    "npm test -- middleware.test.js": {"exit_code": 0, "stderr": ""},
    "npm run lint src/middleware": {"exit_code": 0, "stderr": ""}
  },
  "policy": {
    "out_of_scope": [],
    "forbidden_touched": [],
    "generated_changed": ["docs/RATE_LIMIT_EVAL.md"]
  },
  "tool_activity": {
    "total_actions": 8,
    "bash": 3,
    "edit": 2,
    "write": 1,
    "read": 2
  }
}
```

### deepseek_tail_job

Reads compact status with optional logs. Similar to `get_job` but designed for progress checks.

```javascript
{
  "name": "deepseek_tail_job",
  "arguments": {
    "job_id": "dsw_abc123",
    "include_logs": false  // default
  }
}
```

### deepseek_wait_for_job

Short observation window (default 30s). Does NOT kill the worker on timeout. Use for quick status updates, not as a main loop.

```javascript
{
  "name": "deepseek_wait_for_job",
  "arguments": {
    "job_id": "dsw_abc123",
    "timeout_sec": 30
  }
}
```

### deepseek_cancel_job

Requests job cancellation.

```javascript
{
  "name": "deepseek_cancel_job",
  "arguments": {
    "job_id": "dsw_abc123"
  }
}
```

### deepseek_implement_in_workspace

Synchronous mode for tiny edits. Blocks until complete.

```javascript
{
  "name": "deepseek_implement_in_workspace",
  "arguments": {
    "cwd": "/home/user/myproject",
    "task": "Fix typo in README.md line 42",
    "use_case": "fast_patch"
  }
}
```

## Use Cases and Model Selection

The MCP chooses DeepSeek model and reasoning effort based on `use_case`. **Goal**: save Codex main-thread tokens, not DeepSeek tokens.

| `use_case` | Default model | effort | Best for |
|---|---|---|---|
| `auto` | `deepseek-v4-flash` | `max` | general implementation |
| `fast_patch` | `deepseek-v4-flash` | `high` | small patches |
| `simple_agent_task` | `deepseek-v4-flash` | `high` | simple agentic coding |
| `scaffold_or_tests` | `deepseek-v4-flash` | `high` | scaffolding, glue, tests |
| `debug_loop` | `deepseek-v4-pro[1m]` | `max` | reproduce, locate, fix, validate |
| `agentic_coding` | `deepseek-v4-pro[1m]` | `max` | multi-step implementation |
| `complex_reasoning` | `deepseek-v4-pro[1m]` | `max` | architecture, hard logic |
| `long_context_codebase` | `deepseek-v4-pro[1m]` | `max` | broad codebase work |
| `docs_generation` | `deepseek-v4-pro[1m]` | `high` | documentation |

**Selection rules**:

- Use `auto` by default
- Use `fast_patch` for obviously tiny edits
- Use `scaffold_or_tests` for tests, scaffolding, glue code
- Use `debug_loop` for reproduce/locate/fix/validate workflows
- Use Pro[1m] presets for cross-file implementation, complex logic, or broad context
- **Don't** default to Pro[1m] just because it sounds stronger

Override with explicit `model`, `thinking`, or `reasoning_effort` if needed.

## Worker Profiles

Control permission scope with `worker_profile`:

- `default`: Standard Claude Code scoped permissions
- `scoped_patch`: Tightly scoped, narrow `allowed_dirs`
- `safe_readonly`: Restricted bash, read-only operations (use with `safety_mode: "safe"`)

## Safety and Permissions

**Not a sandbox**. Guardrails:

- Temporary Claude Code `dontAsk` settings per worker
- `PreToolUse` hook blocks clearly dangerous Bash and forbidden paths
- Hooks log compact action summaries
- Final snapshot policy checks report out-of-scope changes

**Default**: `safety_mode: "permissive"` (Bash allowed except dangerous commands)

**Strict**: `safety_mode: "safe"` (Bash restricted to read-only and explicit checks)

**`bypassPermissions`**: disabled by default. Keep it off unless you add external sandboxing.

### Path boundaries

- `allowed_dirs`: Target directories for implementation
- `forbidden_paths`: Hard failure if touched
- `generated_paths`: Validation/eval outputs; reported as `generated_changed`, not out-of-scope

**v0.3.20-beta.38 principle**: Report actions as facts, don't auto-fail. Out-of-scope changes are reported; `forbidden_paths` remains a hard failure.

## Token-Saving Discipline

To maximize Codex token savings:

1. **Narrow task scope**: One goal, clear boundaries, explicit validation
2. **Minimal context**: Don't read whole codebase before delegating
3. **Compact status**: Avoid logs/events/diffs while job is running
4. **Selective review**: After completion, review only `files_changed`, key ranges, checks, risks
5. **Project brief pattern**:

```text
Project brief:
- Project: <one-line goal>
- Current slice: <module/feature>
- Task: <single implementation goal>
- Boundaries: <allowed_dirs>
- Generated outputs: <eval reports>
- Do not touch: <forbidden_paths>
- Validate: <commands>
- Previous result: <job_id + status + diff/check summary>
```

6. **Follow-up workers**: Pass only necessary previous-result summary, not full history

## Configuration Examples

### Scoped patch with validation

```javascript
{
  "cwd": "/home/user/app",
  "task": "Add input sanitization to user registration endpoint",
  "use_case": "simple_agent_task",
  "worker_profile": "scoped_patch",
  "allowed_dirs": ["src/routes/auth", "src/lib/sanitize"],
  "forbidden_paths": ["src/config", ".env", "secrets"],
  "validation_commands": [
    "npm test -- auth.test.js",
    "npm run security-scan src/routes/auth"
  ],
  "safety_mode": "permissive"
}
```

### Debug loop with Pro model

```javascript
{
  "cwd": "/home/user/complex-service",
  "task": "Reproduce and fix the memory leak in the background job processor",
  "use_case": "debug_loop",  // Uses deepseek-v4-pro[1m]
  "allowed_dirs": ["src/jobs", "src/workers", "tests/integration"],
  "validation_commands": [
    "npm run test:memory-profile",
    "npm run test:jobs"
  ],
  "generated_paths": ["logs/memory_profile.txt"]
}
```

### Documentation generation

```javascript
{
  "cwd": "/home/user/library",
  "task": "Generate API documentation for the public interfaces",
  "use_case": "docs_generation",
  "allowed_dirs": ["docs/api", "src"],
  "generated_paths": ["docs/api/generated"],
  "validation_commands": ["npm run docs:validate"]
}
```

## Workflow Patterns

### Pattern 1: Start and check later

```javascript
// 1. Start job
const startResult = await deepseek_start_implementation({
  cwd: "/home/user/project",
  task: "Implement user profile caching with Redis"
});

// 2. Do other Codex work...

// 3. Check status when needed
const status = await deepseek_get_job({
  job_id: startResult.job_id
});

// 4. If completed, review changes
if (status.status === "completed") {
  // Review status.files_changed, status.checks, status.policy
}
```

### Pattern 2: Follow-up worker

```javascript
// First job failed validation
const firstJob = await deepseek_get_job({ job_id: "dsw_first123" });
// status.checks shows test failure

// Start follow-up with context
await deepseek_start_implementation({
  cwd: "/home/user/project",
  task: `Fix the test failure from previous job dsw_first123.
Previous result: Tests failed with 'TypeError: Cannot read property id of undefined'.
Fix the null check in src/handlers/profile.js and ensure all tests pass.`,
  use_case: "debug_loop",
  allowed_dirs: ["src/handlers", "tests"],
  validation_commands: ["npm test -- profile.test.js"]
});
```

### Pattern 3: Tiny synchronous edit

```javascript
// For obviously tiny changes, use synchronous mode
const result = await deepseek_implement_in_workspace({
  cwd: "/home/user/project",
  task: "Change the default port from 3000 to 8080 in config.js",
  use_case: "fast_patch"
});
// Blocks until complete
```

## Setup and Verification

### Initial setup

```bash
npm run mcp:setup
```

Interactively installs Claude Code if missing and prompts for DeepSeek key.

### Health check

```bash
npm run mcp:doctor
```

Expected output:

```json
{
  "server_version": "0.3.20-beta.38",
  "claude_code_installed": true,
  "claude_code_version": "1.2.3",
  "deepseek_key_configured": true,
  "ok": true
}
```

**Common issue**: `claude_code_version` failed but `claude` command exists.

**Cause**: Wrapper script or stale shim intercepts `claude --version`.

**Fix**: Fix Claude Code install or set `CLAUDE_BIN` to real executable, then rerun `doctor`.

### Smoke tests

```bash
npm run mcp:smoke:stream      # Stream processing
npm run mcp:smoke:permission  # Permission hooks
npm run mcp:smoke:restore     # Snapshot restore
npm run mcp:smoke             # Full real worker test (requires CLI + key)
```

## Troubleshooting

### Worker seems stuck

**DeepSeek V4 Pro can spend ~10 minutes in one continuous thinking segment** on complex tasks. This is not cumulative job runtime — it's a single reasoning phase.

- Don't assume quiet = stuck
- Don't poll logs while running
- Check compact status only when you need facts
- Use `deepseek_get_job` without `include_logs` for quick checks

### Job completed but validation failed

```javascript
const status = await deepseek_get_job({
  job_id: "dsw_abc123",
  include_logs: true  // Get validation output
});

// Check status.checks for failure details
// Example:
// status.checks["npm test"] = { exit_code: 1, stderr: "..." }
```

Start a follow-up worker with the failure context.

### Out-of-scope changes reported

Version `0.3.20-beta.38` reports out-of-scope changes as facts, not automatic failures (unless `forbidden_paths` touched).

```javascript
const status = await deepseek_get_job({ job_id: "dsw_abc123" });

if (status.policy.out_of_scope.length > 0) {
  // Review: Are these changes acceptable?
  // status.policy.out_of_scope = ["src/other-module/file.js"]
}

if (status.policy.forbidden_touched.length > 0) {
  // Hard failure: Forbidden paths were touched
  // This is a real error
}
```

### Claude Code version check fails

If `doctor` reports `claude_code_version` as failed:

1. Check `which claude` — is it a wrapper?
2. Run `claude --version` manually — does it output a version?
3. Set `CLAUDE_BIN` to the real Claude Code executable:

```bash
export CLAUDE_BIN="/usr/local/bin/claude-code"
npm run mcp:doctor
```

### Permission denied errors

- Ensure `allowed_dirs` covers the target files
- Check that `forbidden_paths` doesn't block necessary files
- Use `generated_paths` for validation outputs
- Consider `worker_profile: "scoped_patch"` for tight scope

### DeepSeek API key not found

Setup looks for key in:

1. `DEEPSEEK_API_KEY` env var
2. `~/.codex/secrets/deepseek_api_key` file
3. `DEEPSEEK_API_KEY_FILE` env var (custom path)

Set one of these or run `npm run mcp:setup` for interactive prompt.

## Best Practices

1. **Start with narrow scope**: Single module, clear boundaries
2. **Use appropriate use_case**: Don't over-provision (Pro for everything) or under-provision (flash for complex tasks)
3. **Validate explicitly**: Include `validation_commands` for automated checks
4. **Review after completion**: Check `files_changed`, `checks`, `policy` before accepting
5. **One worker per task**: Don't start multiple workers for the same implementation
6. **Compact status while running**: Avoid logs/events/diffs until terminal status
7. **Follow-up context**: Pass previous job id + terminal status + relevant summary, not full history
8. **Generated outputs**: Use `generated_paths` for eval reports, not `allowed_dirs` widening

## API Summary

| Tool | Purpose | Blocking |
|---|---|---|
| `deepseek_start_implementation` | Start background job | No |
| `deepseek_get_job` | Get status | No |
| `deepseek_tail_job` | Get status with optional logs | No |
| `deepseek_wait_for_job` | Short observation window | No (doesn't kill) |
| `deepseek_cancel_job` | Cancel job | No |
| `deepseek_implement_in_workspace` | Synchronous implementation | Yes |

## Current Status

Beta `v0.3.20-beta.38`. Suitable for internal projects and early adopters. Not yet published to npm registry.
