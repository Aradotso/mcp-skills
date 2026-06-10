---
name: multi-llm-mcp-server
description: MCP server enabling Claude Code to orchestrate Codex CLI and query multiple LLMs (GPT, Kimi, DeepSeek, Qwen) in parallel
triggers:
  - ask multiple AI models the same question
  - use Codex CLI to modify my project files
  - compare responses from different LLMs
  - let Codex analyze my codebase
  - query GPT, Kimi, and DeepSeek simultaneously
  - check what LLM providers are configured
  - run Codex in read-only sandbox mode
  - maintain a conversation session with a specific model
---

# multi-llm-mcp-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

An MCP server that enables Claude Code to delegate tasks to Codex CLI and query multiple LLM providers (GPT, Kimi, DeepSeek, Qwen, Claude) in parallel or individually. Built on FastMCP, it solves two key problems: (1) letting Claude Code offload work to Codex CLI, and (2) querying multiple models simultaneously for comparison and cross-referencing.

## Installation

### Prerequisites

```bash
# Python 3.10+ required
pip install fastmcp openai

# Optional: Install Codex CLI for ask_codex functionality
# Ensure 'codex' is available in PATH and authenticated
```

### Environment Variables

Configure API keys for the models you want to use:

```bash
# Windows (requires terminal/IDE restart)
setx OPENAI_API_KEY "your-key"
setx DEEPSEEK_API_KEY "your-key"
setx MOONSHOT_API_KEY "your-key"
setx DASHSCOPE_API_KEY "your-key"
setx ANTHROPIC_API_KEY "your-key"  # Optional for Claude

# Linux/macOS
export OPENAI_API_KEY="your-key"
export DEEPSEEK_API_KEY="your-key"
export MOONSHOT_API_KEY="your-key"
export DASHSCOPE_API_KEY="your-key"
export ANTHROPIC_API_KEY="your-key"
```

### MCP Configuration

Add to Claude Code:

```bash
# Basic setup (adjust path to your LLM_MIX.py location)
claude mcp add --scope user llm-mix -- python /absolute/path/to/LLM_MIX.py

# Windows with full Python path
claude mcp add --scope user llm-mix -- "C:\Python310\python.exe" "C:\Projects\multi-llm-mcp\LLM_MIX.py"

# Verify installation
claude mcp list
```

## Available Tools

| Tool | Purpose |
|------|---------|
| `ask` | Query a single model with session support |
| `ask_many` | Query multiple models in parallel |
| `wait_many` | Poll for multi-model job completion |
| `review` | Have multiple models analyze the same content |
| `ask_codex` | Execute tasks via Codex CLI |
| `wait_codex` | Poll for Codex job completion |
| `clear_session` | Clear a specific conversation session |
| `clear_all_sessions` | Clear all sessions |
| `list_sessions` | View active sessions |
| `health_check` | Check server status and configured providers |

## Core Usage Patterns

### 1. Health Check

Always start by checking what's configured:

```python
# Tool call from Claude Code
health_check()

# Example response:
{
  "status": "healthy",
  "codex_available": true,
  "configured_models": ["gpt", "deepseek", "kimi", "qwen"],
  "missing_keys": ["ANTHROPIC_API_KEY"]
}
```

### 2. Single Model Query with Session

```python
# First message in a conversation
ask(
    model="deepseek",
    prompt="Explain dependency injection in Python",
    session_id="di-tutorial"
)

# Follow-up in same session
ask(
    model="deepseek",
    prompt="Show me a FastAPI example",
    session_id="di-tutorial"
)

# Clear when done
clear_session(session_id="di-tutorial")
```

### 3. Multi-Model Parallel Query

```python
# Query multiple models simultaneously
result = ask_many(
    models=["gpt", "deepseek", "kimi", "qwen"],
    prompt="What are the trade-offs between monorepo and polyrepo?"
)

# If job is still running:
if result["status"] == "running":
    wait_many(job_id=result["job_id"])

# Response format:
{
  "success": true,
  "status": "completed",
  "responses": {
    "gpt": "Monorepos provide...",
    "deepseek": "The main trade-offs...",
    "kimi": "Let me break this down...",
    "qwen": "From an architectural perspective..."
  }
}
```

### 4. Multi-Model Review

```python
# Have multiple models analyze code/content
review(
    models=["gpt", "deepseek"],
    content="""
def process_data(items):
    result = []
    for i in items:
        if i > 0:
            result.append(i * 2)
    return result
""",
    instruction="Review this code for performance and readability"
)
```

### 5. Codex CLI Integration

```python
# Read-only: safe for code analysis
ask_codex(
    sandbox_mode="read-only",
    prompt="Analyze the project structure and list all Python modules"
)

# Workspace write: for file modifications
ask_codex(
    sandbox_mode="workspace-write",
    prompt="Refactor the authentication module to use async/await"
)

# Danger mode: full system access (use cautiously)
ask_codex(
    sandbox_mode="danger-full-access",
    prompt="Install dependencies and run tests"
)

# Handle long-running tasks
result = ask_codex(
    sandbox_mode="workspace-write",
    prompt="Migrate all print statements to logging"
)

if result["status"] == "running":
    # Continue waiting
    wait_codex(job_id=result["job_id"])
```

## Configuration Details

### Model Providers

The server supports these providers out of the box:

```python
# Available models (case-insensitive)
"gpt"       # OpenAI GPT models (gpt-4, gpt-3.5-turbo, etc.)
"deepseek"  # DeepSeek models (deepseek-chat, deepseek-coder)
"kimi"      # Moonshot AI (moonshot-v1-8k, moonshot-v1-32k)
"qwen"      # Alibaba Qwen (qwen-turbo, qwen-plus, qwen-max)
"claude"    # Anthropic Claude (requires manual enable)
```

### Enabling Claude Provider

By default, Claude is commented out. To enable:

1. Edit `LLM_MIX.py` and uncomment the `claude` block in `PROVIDERS`
2. Add `"claude"` to the `ModelName` literal type
3. Set `ANTHROPIC_API_KEY` environment variable
4. Restart the MCP server

### Codex Sandbox Modes

```python
# read-only: Can read files, analyze code, no modifications
# Best for: code review, analysis, documentation generation

# workspace-write: Can modify files within workspace
# Best for: refactoring, bug fixes, feature implementation

# danger-full-access: Full system access
# Best for: system setup, package installation (use sparingly)
```

## Common Workflows

### Compare Framework Recommendations

```python
ask_many(
    models=["gpt", "deepseek", "kimi"],
    prompt="Should I use FastAPI or Flask for a microservices API with heavy async I/O?"
)
```

### Code Review by Multiple Models

```python
review(
    models=["gpt", "deepseek"],
    content=open("src/payment_processor.py").read(),
    instruction="Review for security vulnerabilities and suggest improvements"
)
```

### Iterative Development with Codex

```python
# Step 1: Analyze
ask_codex(
    sandbox_mode="read-only",
    prompt="Find all TODO comments in the codebase"
)

# Step 2: Implement
ask_codex(
    sandbox_mode="workspace-write",
    prompt="Implement the authentication TODO in auth.py using JWT"
)

# Step 3: Test
ask_codex(
    sandbox_mode="workspace-write",
    prompt="Write unit tests for the new JWT authentication"
)
```

### Session Management for Complex Queries

```python
# Start a focused conversation
ask(
    model="gpt",
    prompt="I'm designing a rate limiter. What algorithms should I consider?",
    session_id="ratelimit-design"
)

ask(
    model="gpt",
    prompt="How would token bucket work with Redis?",
    session_id="ratelimit-design"
)

ask(
    model="gpt",
    prompt="Show me Python implementation",
    session_id="ratelimit-design"
)

# Clean up
clear_session(session_id="ratelimit-design")
```

## Troubleshooting

### Tool Not Found in Claude Code

```bash
# Verify MCP is registered
claude mcp list

# Check if Python path is correct
which python  # Linux/macOS
where python  # Windows

# Re-add with explicit Python path
claude mcp add --scope user llm-mix -- /usr/bin/python3 /path/to/LLM_MIX.py
```

### Codex Authentication Failure

```bash
# Codex requires initial authentication
codex

# Follow the login prompts, then retry in Claude Code
```

### Model API Key Not Detected

```python
# Check configuration
health_check()

# Verify environment variable is set
echo $OPENAI_API_KEY  # Linux/macOS
echo %OPENAI_API_KEY%  # Windows

# Restart IDE/terminal after setting env vars
```

### Long-Running Task Timeout

```python
# All long tasks return job_id
result = ask_codex(sandbox_mode="workspace-write", prompt="Large refactor")

if result["status"] == "running":
    # Keep polling
    wait_codex(job_id=result["job_id"])
    
# Same pattern for multi-model
result = ask_many(models=["gpt", "deepseek", "kimi"], prompt="Complex question")
if result["status"] == "running":
    wait_many(job_id=result["job_id"])
```

### Session Accumulation

```python
# List all sessions
list_sessions()

# Clear specific session
clear_session(session_id="old-conversation")

# Nuclear option: clear everything
clear_all_sessions()
```

## Best Practices

1. **Start with `health_check`** to verify configuration before complex workflows
2. **Use read-only mode first** when exploring codebases with Codex
3. **Session IDs** should be descriptive: `"auth-refactor"` not `"session1"`
4. **Multi-model queries** are best for subjective or complex design questions
5. **Single model sessions** are best for iterative, context-dependent conversations
6. **Always clear sessions** when switching topics to avoid context pollution
7. **Poll long jobs** with `wait_*` tools instead of making the same request twice

## Examples in Practice

### Architectural Decision

```python
# Get diverse perspectives
ask_many(
    models=["gpt", "deepseek", "qwen"],
    prompt="""
    I'm building a real-time collaborative editor. 
    Should I use Operational Transform, CRDT, or a hybrid approach?
    Consider: 10k concurrent users, sub-100ms latency requirement, mobile clients.
    """
)
```

### Refactoring Workflow

```python
# 1. Understand current state
ask_codex(
    sandbox_mode="read-only",
    prompt="Explain the architecture of the user service module"
)

# 2. Plan changes
ask(
    model="deepseek",
    prompt="How should I refactor this to support OAuth2 and SAML?",
    session_id="auth-refactor"
)

# 3. Execute
ask_codex(
    sandbox_mode="workspace-write",
    prompt="Refactor user service to support OAuth2, following the plan from previous conversation"
)

# 4. Review
review(
    models=["gpt", "deepseek"],
    content="<paste new code>",
    instruction="Check for security issues and edge cases"
)
```
