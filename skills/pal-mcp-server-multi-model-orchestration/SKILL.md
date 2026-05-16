---
name: pal-mcp-server-multi-model-orchestration
description: Use PAL MCP to orchestrate multiple AI models (Gemini, OpenAI, Grok, Ollama) for code reviews, debugging, planning, and CLI bridging
triggers:
  - "use multiple AI models to review this code"
  - "get a second opinion from gemini on this implementation"
  - "debug this with o3 and then optimize with flash"
  - "perform a code review using gemini pro and gpt-5"
  - "spawn a gemini subagent to analyze this module"
  - "use clink to create an isolated code reviewer"
  - "get consensus from different models on this approach"
  - "continue with gemini pro for deep analysis"
---

# PAL MCP Server: Multi-Model AI Orchestration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

PAL MCP (Provider Abstraction Layer) is an MCP server that enables AI coding assistants like Claude Code, Cursor, and Codex CLI to orchestrate multiple AI models within a single workflow. It provides conversation continuity across models, CLI-to-CLI bridging, and specialized workflows for code review, debugging, planning, and security auditing.

## What PAL MCP Does

- **Multi-Model Orchestration**: Coordinate Gemini, OpenAI, Anthropic, Grok, Azure, Ollama, and OpenRouter models in one conversation
- **Conversation Threading**: Full context flows across tools and models - later models remember what earlier ones said
- **CLI Bridging (`clink`)**: Spawn isolated CLI instances (Gemini CLI, Codex CLI, Claude Code) as subagents with specialized roles
- **Context Isolation**: Run separate investigations without polluting your primary workspace
- **Extended Context Windows**: Delegate to models with larger context (Gemini's 1M tokens, O3's 200K tokens)
- **Specialized Workflows**: Built-in tools for code review, debugging, planning, security audits, and pre-commit checks

## Installation

### Prerequisites

- Python 3.10+
- Git
- [uv package manager](https://docs.astral.sh/uv/getting-started/installation/)

### Option A: Clone and Automatic Setup (Recommended)

```bash
git clone https://github.com/BeehiveInnovations/pal-mcp-server.git
cd pal-mcp-server

# Handles setup, config, API keys from environment
# Auto-configures Claude Desktop, Claude Code, Gemini CLI, Codex CLI
./run-server.sh
```

### Option B: Instant Setup with uvx

Add to `~/.claude/settings.json` or `.mcp.json`:

```json
{
  "mcpServers": {
    "pal": {
      "command": "bash",
      "args": ["-c", "for p in $(which uvx 2>/dev/null) $HOME/.local/bin/uvx /opt/homebrew/bin/uvx /usr/local/bin/uvx uvx; do [ -x \"$p\" ] && exec \"$p\" --from git+https://github.com/BeehiveInnovations/pal-mcp-server.git pal-mcp-server; done; echo 'uvx not found' >&2; exit 1"],
      "env": {
        "PATH": "/usr/local/bin:/usr/bin:/bin:/opt/homebrew/bin:~/.local/bin",
        "GEMINI_API_KEY": "",
        "OPENAI_API_KEY": "",
        "XAI_API_KEY": "",
        "OPENROUTER_API_KEY": "",
        "DISABLED_TOOLS": "analyze,refactor,testgen,secaudit,docgen,tracer",
        "DEFAULT_MODEL": "auto"
      }
    }
  }
}
```

### Required API Keys

Set at least one provider's API key as environment variables or in `.env`:

```bash
# Choose one or more providers
export GEMINI_API_KEY=your_key_here
export OPENAI_API_KEY=your_key_here
export OPENROUTER_API_KEY=your_key_here
export XAI_API_KEY=your_key_here
export AZURE_OPENAI_KEY=your_key_here
# For local models - no key needed
# Just run: ollama serve
```

## Core Tools

### `pal` - Multi-Model Orchestration

The main tool for querying multiple AI models with conversation continuity.

```python
# Basic usage - auto model selection
"Use pal to analyze this authentication code for security issues"

# Specific model
"Use pal with gemini-pro to review the API design"

# Multiple models for consensus
"Get consensus using pal with gpt-5 and gemini-pro: should we use REST or GraphQL?"

# Vision analysis
"Use pal with gemini-pro to analyze this architecture diagram"
```

**Key Parameters:**
- `prompt` (required): The instruction for the AI model
- `model` (optional): Specific model to use (e.g., "gemini-pro", "gpt-5", "o3-mini")
- `conversation_id` (optional): Continue a previous conversation thread
- `system_prompt` (optional): Custom system instructions
- `max_tokens` (optional): Response length limit
- `temperature` (optional): 0.0-2.0, controls randomness

**Common Models:**
- `gemini-pro` - Deep thinking, 1M token context
- `gemini-flash` - Fast responses
- `gpt-5` - Strong reasoning
- `o3-mini` - Fast OpenAI reasoning
- `grok-beta` - X.AI's model
- `llama3` - Local via Ollama

### `clink` - CLI-to-CLI Bridge

Spawn external AI CLIs as subagents with isolated contexts and specialized roles.

```python
# Spawn Gemini CLI subagent with code reviewer role
"Use clink with gemini codereviewer to audit the auth module"

# Codex subagent for isolated investigation
"Use clink with codex to investigate the memory leak in isolation"

# Custom role with system prompt
"Use clink with gemini role='security-expert' system_prompt='Focus on OWASP Top 10' to scan for vulnerabilities"

# Multi-step workflow
"""
1. Use consensus with gpt-5 and gemini-pro to decide: add dark mode or offline support next
2. Continue with clink gemini - implement the recommended feature
"""
```

**Key Parameters:**
- `cli_name` (required): "gemini", "codex", or "claude"
- `prompt` (required): Task for the subagent
- `role` (optional): "planner", "codereviewer", "debugger", or custom
- `system_prompt` (optional): Custom instructions for the subagent
- `timeout` (optional): Max execution time in seconds

**Benefits:**
- Fresh context window for heavy tasks
- Parallel investigations without context pollution
- Role specialization (planner vs. implementer)
- Full CLI capabilities (web search, file access, MCP tools)

### `codereview` - Multi-Pass Code Analysis

Systematic code review workflow with confidence tracking and multi-model consensus.

```python
# Basic code review
"Perform a codereview of the payment processing module"

# Multi-model review
"Use codereview with gemini-pro and o3 to review the authentication system"

# With planning and implementation
"""
Perform a codereview using gemini-pro and o3, then:
1. Use planner to generate a detailed fix plan
2. Implement the critical and high-priority fixes
3. Do a final precommit check
"""
```

**Workflow:**
1. Initial code exploration (confidence: exploring)
2. Pattern identification (confidence: low → medium)
3. Issue collection with severity levels (critical, high, medium, low)
4. Multi-model validation if specified
5. Consolidated report with actionable recommendations

### `debughelp` - Systematic Debugging Assistant

Root cause analysis with hypothesis tracking and confidence scoring.

```python
# Debug with error context
"Use debughelp to investigate why the API returns 500 on user creation"

# Multi-model debugging
"Debug this memory leak with o3, then verify the fix with gemini-pro"
```

**Workflow:**
1. Error symptom collection
2. Hypothesis generation with confidence levels
3. Systematic investigation
4. Root cause identification
5. Fix recommendations with validation steps

### `planner` - Strategic Planning Workflow

Break down complex tasks into phased, actionable plans.

```python
# Migration planning
"Use planner to create a strategy for migrating from REST to GraphQL"

# Multi-model consensus planning
"Use planner with gpt-5 and gemini-pro to design the microservices architecture"
```

**Workflow:**
1. Goal clarification
2. Constraint identification
3. Phase breakdown
4. Risk assessment
5. Deliverable definition with success criteria

### `precommit` - Pre-Commit Validation

Final validation before committing changes, aware of previous code review context.

```python
# After implementing fixes
"Perform a precommit check on the authentication changes"

# With specific model
"Use precommit with gemini-pro to validate the security fixes"
```

**Checks:**
- Regression risk assessment
- Security implications
- Performance impact
- Test coverage validation
- Documentation updates

## Configuration

### Environment Variables

```bash
# Provider API Keys
GEMINI_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here
OPENROUTER_API_KEY=your_key_here
XAI_API_KEY=your_key_here

# Azure OpenAI
AZURE_OPENAI_KEY=your_key_here
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o

# Model Settings
DEFAULT_MODEL=auto  # or specific model like "gemini-pro"
DEFAULT_TEMPERATURE=0.7
DEFAULT_MAX_TOKENS=8000

# Tool Configuration
DISABLED_TOOLS=analyze,refactor,testgen,secaudit,docgen,tracer

# Logging
LOG_LEVEL=INFO  # DEBUG, INFO, WARNING, ERROR
LOG_FILE=pal-mcp.log
```

### Model Aliases

PAL supports shorthand aliases for common models:

```python
# These are equivalent
"use pal with gemini-pro"
"use pal with gemini-2.0-flash-exp"

# Aliases
"gemini-flash" → "gemini-2.0-flash-exp"
"gemini-pro" → "gemini-2.0-pro-exp"
"gpt-5" → "gpt-5-turbo"
"o3" → "o3-mini"
```

### Cursor & VS Code Integration

Create `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "pal": {
      "command": "uv",
      "args": [
        "--directory",
        "/path/to/pal-mcp-server",
        "run",
        "pal-mcp-server"
      ],
      "env": {
        "GEMINI_API_KEY": "",
        "OPENAI_API_KEY": ""
      }
    }
  }
}
```

## Real-World Workflows

### Multi-Model Code Review Pipeline

```python
# Full code review → planning → implementation → validation
"""
1. Perform a codereview of the payment module using gemini-pro and o3
2. Use planner to break down the critical and high-priority fixes
3. Implement the fixes for issues marked 'critical'
4. Use precommit with gemini-pro to validate before committing
"""
```

**What happens:**
1. Claude walks the code, identifying issues
2. Gemini Pro performs deep analysis (1M token context)
3. O3 validates findings with strong reasoning
4. Consolidated report with severity levels
5. Planner creates phased fix strategy
6. Claude implements critical fixes
7. Gemini Pro validates changes with full context of original review

### Context Revival After Reset

```python
# When Claude's context resets mid-workflow
"""
Continue with gemini-pro from conversation_id abc123 to remind me of 
the security issues we discussed and the fixes we planned
"""
```

**Benefit:** Other models maintain conversation history and can "refresh" Claude's understanding without re-reading all files.

### Isolated Sub-Investigation

```python
# Main task in Claude, detailed analysis in subagent
"""
Use clink with gemini role='security-auditor' to:
1. Scan all API endpoints for OWASP Top 10 vulnerabilities
2. Check for SQL injection, XSS, and auth bypass risks
3. Return summary of critical findings only

Then I'll continue the main implementation here.
"""
```

**Benefit:** Heavy security scan runs in isolated context, returns only summary, main context stays clean.

### Consensus-Driven Architecture Decision

```python
# Multiple models debate approach
"""
Use pal to get consensus between gpt-5, gemini-pro, and o3:
Should we use event sourcing or traditional CRUD for the order management system?
Consider: scalability, complexity, team expertise, audit requirements.

Each model should provide reasoning, then synthesize a recommendation.
"""
```

### Vision-Assisted Code Review

```python
# Analyze architectural diagrams
"""
Use pal with gemini-pro to:
1. Analyze this system architecture diagram (screenshot attached)
2. Compare it to the actual codebase structure in /src
3. Identify discrepancies and outdated documentation
"""
```

## Common Patterns

### Pattern: Model-Specific Strengths

```python
# Extended thinking for complex problems
"Use pal with gemini-pro for deep architectural analysis"

# Fast iteration
"Use pal with gemini-flash for quick syntax checks"

# Strong reasoning
"Use pal with o3 for logical bug analysis"

# Privacy-sensitive code
"Use pal with llama3 via Ollama for local analysis"
```

### Pattern: Conversation Continuity

```python
# Step 1: Initial analysis
response1 = "Use pal with gemini-pro to analyze the auth flow"
# Returns conversation_id: conv_abc123

# Step 2: Follow-up with different model
"Continue conversation conv_abc123 with o3 to verify the CSRF protection"

# Step 3: Back to original model
"Continue conversation conv_abc123 with gemini-pro to implement the fixes"
```

### Pattern: Progressive Validation

```python
# Implementation workflow with validation gates
"""
1. Use planner to design the feature
2. Implement the core logic
3. Use pal with gemini-flash for quick sanity check
4. Use pal with o3 for security validation
5. Use precommit with gemini-pro for final review
"""
```

### Pattern: Parallel Sub-Tasks

```python
# Offload heavy tasks to subagents
"""
Spawn three parallel clink subagents:
1. clink gemini role='tester' - Generate unit tests for auth module
2. clink codex role='documenter' - Write API documentation
3. clink gemini role='security' - Perform security audit

I'll continue the main feature implementation while they work.
"""
```

## Troubleshooting

### MCP Connection Issues

```bash
# Check MCP server logs
tail -f ~/Library/Logs/Claude/mcp*.log

# Verify uvx installation
which uvx
uvx --version

# Test PAL directly
cd pal-mcp-server
uv run pal-mcp-server
```

### API Key Not Working

```bash
# Verify key is set
echo $GEMINI_API_KEY

# Check .env file
cat .env | grep GEMINI_API_KEY

# Test API directly
curl -H "Authorization: Bearer $GEMINI_API_KEY" \
  https://generativelanguage.googleapis.com/v1/models
```

### Model Not Available

```python
# List available models
"Use pal with model='list' to show all available models"

# Check provider status
"Use pal to test connection to gemini-pro"
```

### High Token Usage

```bash
# Disable unused tools to reduce context
export DISABLED_TOOLS="analyze,refactor,testgen,secaudit,docgen,tracer"

# Use flash models for quick tasks
"Use pal with gemini-flash instead of gemini-pro"

# Limit response length
"Use pal with max_tokens=2000 for concise analysis"
```

### Conversation Context Lost

```python
# Always capture conversation_id from responses
# Store it for later reference
"Continue conversation conv_abc123 with gemini-pro"

# If lost, start new thread with summary
"Use pal with gemini-pro, here's what we discussed before: [summary]"
```

### clink Subagent Timeout

```bash
# Increase timeout for heavy tasks
"Use clink with gemini timeout=600 to perform comprehensive security audit"

# Split into smaller tasks
"Use clink with gemini to audit only the authentication module first"
```

### Ollama Local Models

```bash
# Start Ollama server
ollama serve

# Pull models
ollama pull llama3
ollama pull codellama

# Verify availability
ollama list

# Use in PAL
"Use pal with llama3 to analyze this code locally"
```

## Best Practices

1. **Start with Specific Models**: Use `gemini-pro` for deep thinking, `gemini-flash` for speed, `o3` for reasoning
2. **Leverage Conversation IDs**: Track multi-step workflows across models
3. **Use clink for Isolation**: Keep main context clean by offloading heavy analysis to subagents
4. **Disable Unused Tools**: Reduce context window consumption with `DISABLED_TOOLS`
5. **Cache Conversation Context**: When context resets, use another model to "remind" your primary CLI
6. **Model Selection Strategy**: Match model strengths to task requirements (vision, speed, reasoning, context size)
7. **Progressive Validation**: Use multiple models as validation gates (fast check → deep analysis → security audit)

PAL MCP transforms your AI coding assistant into an orchestrator that can leverage the best model for each subtask while maintaining full conversation continuity across the entire workflow.
