---
name: codex-control-plane-mcp
description: Durable MCP control plane for long-running Codex Desktop tasks with retry-safe operations, Plan Mode workflows, and approval handling
triggers:
  - set up codex control plane for long running tasks
  - create a durable codex desktop workflow
  - handle codex plan mode approvals
  - submit retry safe codex operations
  - manage codex desktop automation with mcp
  - poll codex task status and diagnostics
  - configure codex control plane mcp server
  - troubleshoot codex desktop operations
---

# codex-control-plane-mcp

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

`codex-control-plane-mcp` is a durable MCP server that turns Codex Desktop into a reliable worker for long-running tasks. It provides retry-safe operations, Plan Mode workflows, approval handling, and comprehensive diagnostics through a simple poll-based API.

## Overview

Unlike thin Codex wrappers that block on multi-hour calls or lose state on retry, this control plane provides:

- **Durable async operations**: Submit a task, get an `operationId` immediately, poll until complete
- **Retry safety**: Same `client_request_id` returns existing operation instead of creating duplicates
- **Plan Mode workflows**: Start plan → poll → approve → execute → read final report
- **Approval handling**: Pending interactions exposed as pollable MCP state
- **Diagnostics**: Health checks, issue analysis, and dry-run repairs
- **SQLite persistence**: Local history of operations, workflows, turns, hooks, and diagnostics

## Installation

### Using pipx (recommended)

```powershell
pipx install codex-control-plane-mcp
```

### Using uvx (run directly)

```powershell
uvx codex-control-plane-mcp
```

### From GitHub

```powershell
python -m pip install "codex-control-plane-mcp @ git+https://github.com/aresyn/codex-control-plane-mcp.git"
```

### Local development

```powershell
git clone https://github.com/aresyn/codex-control-plane-mcp.git
cd codex-control-plane-mcp
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -e ".[dev]"
python -m pytest -q
```

## Initial Setup

### Generate configuration

```powershell
codex-control-plane-mcp-admin init --state-db .\state\codex-mcp-state.sqlite3 --projects-root C:\Users\you\Projects
```

This generates a JSON config block you can add to your MCP client configuration.

### Install Codex hooks

```powershell
codex-control-plane-mcp-hooks install --state-db .\state\codex-mcp-state.sqlite3
codex-control-plane-mcp-hooks status
codex-control-plane-mcp-hooks doctor
```

**Important**: Restart Codex Desktop after installing or changing hooks.

### MCP client configuration

Minimal stdio entry for MCP client config (e.g., `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "codex-control-plane": {
      "command": "codex-control-plane-mcp",
      "args": [],
      "env": {
        "CODEX_MCP_STATE_DB": "C:\\Users\\you\\state\\codex-mcp-state.sqlite3",
        "CODEX_PROJECTS_ROOT": "C:\\Users\\you\\Projects",
        "CODEX_CONTROL_PLANE_MCP_LOG": "C:\\Users\\you\\logs\\codex-mcp.log"
      }
    }
  }
}
```

## Configuration

Configuration via environment variables or `CODEX_CONTROL_PLANE_MCP_CONFIG` JSON file:

| Variable | Description | Default |
|----------|-------------|---------|
| `CODEX_HOME` | Codex home directory | `%USERPROFILE%\.codex` |
| `CODEX_PROJECTS_ROOT` | Project root for catalog/read tools | - |
| `CODEX_ALLOWED_ROOTS` | Semicolon-separated path allowlist | - |
| `CODEX_PROJECTS_REGISTRY` | Optional JSON project registry | - |
| `CODEX_MCP_STATE_DB` | Local MCP state database | - |
| `CODEX_CONTROL_PLANE_MCP_LOG` | Log file path | - |
| `CODEX_MCP_HOOK_HISTORY_ENABLED` | Enable SQLite hook history | `true` |

## Core Workflows

### 1. Submit a Durable Task

```python
# MCP tool call from your agent/orchestrator
result = mcp_client.call_tool("codex_submit_task", {
    "operation_type": "send_message",
    "project_id": "my-project",
    "prompt": "Refactor the authentication module to use OAuth2",
    "client_request_id": "unique-request-id-1",  # Retry-safe
    "wait_for_completion": False  # Return immediately
})

operation_id = result["operationId"]

# Poll for status
status = mcp_client.call_tool("codex_get_operation_status", {
    "operation_id": operation_id
})

# status["state"] can be: queued, running, waiting_for_approval, completed, failed
```

### 2. Steer an Active Turn

Add context to an active turn without creating a new one:

```python
# Start initial task
result = mcp_client.call_tool("codex_submit_task", {
    "operation_type": "send_message",
    "project_id": "my-project",
    "prompt": "Add logging to the API handlers",
    "client_request_id": "req-1"
})

thread_id = result["threadId"]
turn_id = result["turnId"]

# Later, steer the active turn
steer_result = mcp_client.call_tool("codex_submit_task", {
    "operation_type": "steer_turn",
    "thread_id": thread_id,
    "expected_turn_id": turn_id,
    "message": "Also add error handling for network timeouts",
    "client_request_id": "req-2"
})

# Poll the steering operation
steer_status = mcp_client.call_tool("codex_get_operation_status", {
    "operation_id": steer_result["operationId"]
})
```

### 3. Plan Mode Workflow

```python
# Start a plan workflow
workflow_result = mcp_client.call_tool("codex_start_plan_workflow", {
    "project_id": "my-project",
    "prompt": "Migrate database from MySQL to PostgreSQL",
    "client_request_id": "plan-req-1"
})

workflow_id = workflow_result["workflowId"]

# Poll workflow status
status = mcp_client.call_tool("codex_get_workflow_status", {
    "workflow_id": workflow_id
})

# status["phase"] can be: wait_plan, review_plan, execute_plan, completed, failed

# When phase is "review_plan", approve it
if status["phase"] == "review_plan":
    approve_result = mcp_client.call_tool("codex_approve_plan", {
        "workflow_id": workflow_id,
        "approved": True,
        "feedback": None  # Optional feedback before approval
    })
    
    execution_op_id = approve_result["executionOperationId"]

# Continue polling until completed
final_status = mcp_client.call_tool("codex_get_workflow_status", {
    "workflow_id": workflow_id
})

if final_status["phase"] == "completed":
    print(final_status["finalReport"])
```

### 4. Handle Pending Approvals

```python
# List all pending interactions
pending = mcp_client.call_tool("codex_list_pending_interactions", {})

for interaction in pending["interactions"]:
    if interaction["type"] == "approval_required":
        # Answer the approval
        mcp_client.call_tool("codex_answer_pending_interaction", {
            "interaction_id": interaction["id"],
            "approved": True,
            "answer": None  # Optional answer for questions
        })
```

### 5. Runtime Capabilities Check

Always check capabilities on startup or reconnect:

```python
capabilities = mcp_client.call_tool("codex_get_runtime_capabilities", {
    "refresh": False  # Use cached snapshot (valid 5 min)
})

print(f"Models: {capabilities['runtimeCapabilities']['modelCount']}")
print(f"Default model: {capabilities['runtimeCapabilities']['defaultModel']}")
print(f"Sandbox ready: {capabilities['runtimeCapabilities']['sandboxReady']}")
print(f"Hooks: {capabilities['runtimeCapabilities']['hookCount']}")

# Check supported app-server methods
methods = capabilities['runtimeCapabilities']['supportedAppServerMethods']
for method in methods:
    print(f"{method['method']} - {method['source']}")
```

### 6. Health Summary

Get a quick health check without starting app-server:

```python
health = mcp_client.call_tool("codex_health_summary", {})

print(f"Server: {health['version']['serverName']} v{health['version']['serverVersion']}")
print(f"Contract: {health['version']['contractVersion']}")
print(f"App server: {health['appServer']['status']}")
print(f"State DB: {health['stateDb']['ok']}")
print(f"Hooks: {health['hooks']['installedCount']}")
```

## Diagnostics and Troubleshooting

### Collect diagnostics

```python
diagnostics = mcp_client.call_tool("codex_collect_diagnostics", {
    "include_runtime_capabilities": True,
    "include_recent_operations": True,
    "include_app_server_logs": True
})

# Returns comprehensive diagnostics including:
# - Runtime capabilities
# - Recent operations
# - App server status and logs
# - Pending interactions
# - Hook status
```

### Analyze issues

```python
analysis = mcp_client.call_tool("codex_analyze_issue", {
    "symptom": "operation_timeout",
    "context": {
        "operation_id": "op-123",
        "thread_id": "thread-456"
    }
})

print(f"Severity: {analysis['severity']}")
for check in analysis['checks']:
    print(f"{check['check']}: {check['status']} - {check['message']}")
```

### Repair issues

```python
# Dry run first (default)
repair = mcp_client.call_tool("codex_repair_issue", {
    "issue_code": "stale_app_server",
    "dry_run": True
})

if repair["ok"]:
    # Apply the repair
    actual_repair = mcp_client.call_tool("codex_repair_issue", {
        "issue_code": "stale_app_server",
        "dry_run": False
    })
```

## Progress Tracking

Get detailed progress events from operations:

```python
status = mcp_client.call_tool("codex_get_operation_status", {
    "operation_id": "op-123",
    "progress_events": 50,  # Max events to return
    "progress_max_chars": 10000  # Max chars per event
})

for event in status.get("progressEvents", []):
    print(f"[{event['timestamp']}] {event['type']}: {event.get('text', '')}")
    
    if event["type"] == "assistant_text_delta":
        print(f"  Delta: {event['delta']}")
    elif event["type"] == "token_usage":
        print(f"  Tokens: {event['totalTokens']}")
    elif event["type"] == "model_reroute":
        print(f"  From: {event['fromModel']} → To: {event['toModel']}")
```

## Interrupt Operations

```python
# Interrupt by operation ID
interrupt = mcp_client.call_tool("codex_interrupt_turn", {
    "operation_id": "op-123"
})

# Or by workflow ID
interrupt = mcp_client.call_tool("codex_interrupt_turn", {
    "workflow_id": "wf-456"
})

# Or by thread and turn
interrupt = mcp_client.call_tool("codex_interrupt_turn", {
    "thread_id": "thread-789",
    "turn_id": "turn-012"
})
```

## Reading Chat History

```python
# List projects
projects = mcp_client.call_tool("codex_list_projects", {})

# List chats in a project
chats = mcp_client.call_tool("codex_list_project_chats", {
    "project_id": "my-project"
})

# Get full chat transcript
chat = mcp_client.call_tool("codex_get_chat", {
    "project_id": "my-project",
    "thread_id": "thread-123"
})

# Search chats
results = mcp_client.call_tool("codex_search_chats", {
    "query": "authentication refactor",
    "project_id": "my-project",  # Optional
    "limit": 10
})
```

## Error Handling

All tools return structured errors:

```python
result = mcp_client.call_tool("codex_submit_task", {
    "operation_type": "send_message",
    "project_id": "unknown-project",
    "prompt": "Test"
})

if not result["ok"]:
    error = result["error"]
    print(f"Error: {error['code']}")
    print(f"Message: {error['message']}")
    print(f"Retryable: {error['retryable']}")
    print(f"Details: {error.get('details', {})}")
```

Common error codes:
- `OPERATION_NOT_FOUND`: Operation ID doesn't exist
- `WORKFLOW_NOT_FOUND`: Workflow ID doesn't exist
- `INVALID_WORKFLOW_PHASE`: Can't perform action in current workflow phase
- `APP_SERVER_ERROR`: App server call failed
- `INVALID_OPERATION_TYPE`: Unknown operation type
- `DUPLICATE_PROMPT`: Active turn already exists with this prompt

## Common Patterns

### Retry-safe task submission

```python
import uuid

client_request_id = str(uuid.uuid4())

try:
    result = mcp_client.call_tool("codex_submit_task", {
        "operation_type": "send_message",
        "project_id": "my-project",
        "prompt": "Implement feature X",
        "client_request_id": client_request_id
    })
except TimeoutError:
    # Retry with same client_request_id
    result = mcp_client.call_tool("codex_submit_task", {
        "operation_type": "send_message",
        "project_id": "my-project",
        "prompt": "Implement feature X",
        "client_request_id": client_request_id  # Same ID = same operation
    })
```

### Long-running task polling

```python
import time

operation_id = "op-123"
max_wait = 3600  # 1 hour
poll_interval = 5  # 5 seconds
elapsed = 0

while elapsed < max_wait:
    status = mcp_client.call_tool("codex_get_operation_status", {
        "operation_id": operation_id
    })
    
    state = status["state"]
    
    if state == "completed":
        print(f"Success: {status['result']}")
        break
    elif state == "failed":
        print(f"Failed: {status['error']}")
        break
    elif state == "waiting_for_approval":
        # Handle approvals
        pending = mcp_client.call_tool("codex_list_pending_interactions", {})
        # ... answer approvals ...
    
    time.sleep(poll_interval)
    elapsed += poll_interval
```

### Plan Mode with feedback

```python
workflow_id = "wf-123"

# Wait for plan
while True:
    status = mcp_client.call_tool("codex_get_workflow_status", {
        "workflow_id": workflow_id
    })
    
    if status["phase"] == "review_plan":
        plan = status["planSummary"]
        
        # Provide feedback instead of immediate approval
        feedback_result = mcp_client.call_tool("codex_approve_plan", {
            "workflow_id": workflow_id,
            "approved": False,
            "feedback": "Please add database migration rollback steps"
        })
        
        # Wait for revised plan
        continue
    
    if status["phase"] == "review_plan":
        # Approve revised plan
        mcp_client.call_tool("codex_approve_plan", {
            "workflow_id": workflow_id,
            "approved": True
        })
        break
    
    time.sleep(5)
```

## Troubleshooting

### App server won't start

```python
# Check app server status
status = mcp_client.call_tool("codex_get_app_server_status", {})

if status["status"] != "running":
    # Try restart
    restart = mcp_client.call_tool("codex_restart_app_server", {})
    
    if not restart["ok"]:
        # Check diagnostics
        diag = mcp_client.call_tool("codex_collect_diagnostics", {
            "include_app_server_logs": True
        })
        print(diag["appServer"])
```

### Operations stuck in "running"

```python
# Interrupt the operation
mcp_client.call_tool("codex_interrupt_turn", {
    "operation_id": "stuck-op-id"
})

# Or analyze the issue
analysis = mcp_client.call_tool("codex_analyze_issue", {
    "symptom": "operation_timeout",
    "context": {"operation_id": "stuck-op-id"}
})
```

### Hooks not recording history

```powershell
# Check hook status
codex-control-plane-mcp-hooks status

# Verify state DB configuration
codex-control-plane-mcp-hooks doctor

# Reinstall hooks if needed
codex-control-plane-mcp-hooks install --state-db .\state\codex-mcp-state.sqlite3
```

### Duplicate turn detection

The control plane prevents creating duplicate turns with the same prompt. If you get a `DUPLICATE_PROMPT` error:

```python
result = mcp_client.call_tool("codex_submit_task", {
    "operation_type": "send_message",
    "project_id": "my-project",
    "prompt": "Same prompt as before"
})

if not result["ok"] and result["error"]["code"] == "DUPLICATE_PROMPT":
    details = result["error"]["details"]
    existing_op_id = details["existingOperationId"]
    
    # Use the existing operation instead
    status = mcp_client.call_tool("codex_get_operation_status", {
        "operation_id": existing_op_id
    })
```

## Best Practices

1. **Always use `client_request_id`** for retry safety
2. **Check capabilities on startup** with `codex_get_runtime_capabilities`
3. **Poll, don't block** on long operations
4. **Handle approvals** through `codex_list_pending_interactions`
5. **Use Plan Mode** for complex, multi-step tasks
6. **Enable hook history** for better diagnostics
7. **Run in `dry_run` mode** before applying repairs
8. **Keep state DB and logs private** — they may contain sensitive data
9. **Restart Codex Desktop** after hook installation or changes
10. **Use `read-only` permission** for untrusted repositories
