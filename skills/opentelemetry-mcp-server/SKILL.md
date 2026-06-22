---
name: opentelemetry-mcp-server
description: Query and analyze OpenTelemetry traces (Jaeger, Tempo, Traceloop) with AI assistance for debugging and observability
triggers:
  - search opentelemetry traces
  - analyze llm token usage
  - debug distributed traces
  - find slow traces
  - query jaeger backend
  - analyze opentelemetry spans
  - check trace errors
  - review llm model performance
---

# OpenTelemetry MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that enables AI assistants to query and analyze OpenTelemetry traces across multiple backends (Jaeger, Grafana Tempo, Traceloop). Specialized for LLM observability with support for token tracking, error debugging, and performance analysis.

## What This Project Does

The OpenTelemetry MCP Server provides tools for:

- **Trace Search**: Query distributed traces with advanced filtering
- **Span Analysis**: Deep-dive into individual operations
- **LLM Observability**: Track token usage, model performance, and costs
- **Error Detection**: Find and analyze failed requests
- **Performance Monitoring**: Identify slow operations and bottlenecks

Supports multiple backends: Jaeger, Grafana Tempo, and Traceloop cloud.

## Installation

### Quick Start (No Installation)

Configure your MCP client to run directly from PyPI using `pipx` or `uvx`:

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, `%APPDATA%\Claude\claude_desktop_config.json` on Windows):

```json
{
  "mcpServers": {
    "opentelemetry-mcp": {
      "command": "pipx",
      "args": ["run", "opentelemetry-mcp"],
      "env": {
        "BACKEND_TYPE": "jaeger",
        "BACKEND_URL": "http://localhost:16686"
      }
    }
  }
}
```

**Cursor/Windsurf** (Settings → MCP):

```json
{
  "opentelemetry-mcp": {
    "command": "uvx",
    "args": ["opentelemetry-mcp"],
    "env": {
      "BACKEND_TYPE": "jaeger",
      "BACKEND_URL": "http://localhost:16686"
    }
  }
}
```

**Gemini CLI** (`~/.gemini/config.json`):

```json
{
  "mcpServers": {
    "opentelemetry-mcp": {
      "command": "pipx",
      "args": ["run", "opentelemetry-mcp"],
      "env": {
        "BACKEND_TYPE": "tempo",
        "BACKEND_URL": "http://localhost:3200"
      }
    }
  }
}
```

### Global Installation

```bash
# Install with pipx (recommended)
pipx install opentelemetry-mcp

# Or with pip
pip install opentelemetry-mcp

# Verify installation
opentelemetry-mcp --help
```

### Development Setup

```bash
# Clone repository
git clone https://github.com/traceloop/opentelemetry-mcp-server.git
cd opentelemetry-mcp-server

# Install with uv
uv sync

# Or pip with dev dependencies
pip install -e ".[dev]"
```

## Configuration

### Backend Types

| Backend   | Type        | URL Example                 | Auth Required |
|-----------|-------------|----------------------------|---------------|
| Jaeger    | Local       | http://localhost:16686     | No            |
| Tempo     | Local/Cloud | http://localhost:3200      | Optional      |
| Traceloop | Cloud       | https://api.traceloop.com  | Yes (API key) |

### Environment Variables

Create `.env` file:

```bash
# Required
BACKEND_TYPE=jaeger  # or tempo, traceloop
BACKEND_URL=http://localhost:16686

# Optional (for Traceloop or authenticated backends)
BACKEND_API_KEY=${TRACELOOP_API_KEY}

# Optional: Custom headers
BACKEND_HEADERS={"Authorization": "Bearer ${YOUR_TOKEN}"}
```

### CLI Configuration

Override environment variables with CLI arguments:

```bash
# Jaeger backend
opentelemetry-mcp --backend jaeger --url http://localhost:16686

# Traceloop with API key
opentelemetry-mcp --backend traceloop --url https://api.traceloop.com --api-key ${TRACELOOP_API_KEY}

# Tempo with custom headers
opentelemetry-mcp --backend tempo --url http://localhost:3200 --headers '{"X-Scope-OrgID": "my-org"}'
```

### Local Development Configuration

For Claude Desktop with local repository:

```json
{
  "mcpServers": {
    "opentelemetry-mcp": {
      "command": "uv",
      "args": [
        "--directory",
        "/absolute/path/to/opentelemetry-mcp-server",
        "run",
        "opentelemetry-mcp"
      ],
      "env": {
        "BACKEND_TYPE": "jaeger",
        "BACKEND_URL": "http://localhost:16686"
      }
    }
  }
}
```

## Available Tools

### 1. search_traces

Search traces with advanced filtering.

**Parameters:**
- `service_name` (optional): Filter by service
- `operation_name` (optional): Filter by operation
- `tags` (optional): Key-value tag filters
- `min_duration` (optional): Minimum duration in microseconds
- `max_duration` (optional): Maximum duration in microseconds
- `start_time` (optional): Start time (ISO 8601 or relative like "1h")
- `end_time` (optional): End time (ISO 8601)
- `limit` (optional): Max results (default: 20)
- `filters` (optional): Advanced filter expressions

**Example Usage:**

```python
# Natural language queries that trigger this tool:
"Show me traces from the api-gateway service in the last hour"
"Find traces with errors from my-service"
"Search for slow requests taking more than 5 seconds"
```

**Filter Examples:**

```python
# Find errors
filters = [
    {
        "attribute": "status.code",
        "operator": "eq",
        "value": "ERROR"
    }
]

# Duration range
filters = [
    {
        "attribute": "duration",
        "operator": "gte",
        "value": 1000000  # 1 second in microseconds
    }
]

# Tag matching
filters = [
    {
        "attribute": "http.status_code",
        "operator": "gte",
        "value": 500
    }
]
```

### 2. search_spans

Search individual spans within traces.

**Parameters:**
- `service_name` (required for Jaeger): Service to search
- `operation_name` (optional): Operation filter
- `tags` (optional): Tag filters
- `min_duration` (optional): Minimum duration
- `max_duration` (optional): Maximum duration
- `start_time` (optional): Start time
- `end_time` (optional): End time
- `limit` (optional): Max results

**Example:**

```python
# "Find database query spans slower than 100ms"
{
    "service_name": "api-gateway",
    "operation_name": "SELECT",
    "min_duration": 100000,  # 100ms in microseconds
    "start_time": "2h"
}
```

### 3. get_trace

Retrieve complete trace details by ID.

**Parameters:**
- `trace_id` (required): Trace ID to fetch

**Example:**

```python
# "Get details for trace abc123def456"
{
    "trace_id": "abc123def456"
}
```

### 4. get_llm_usage

Aggregate LLM token usage metrics.

**Parameters:**
- `service_name` (optional): Filter by service
- `model` (optional): Filter by model (e.g., "gpt-4")
- `start_time` (optional): Start time
- `end_time` (optional): End time

**Example:**

```python
# "Show me token usage for gpt-4 today"
{
    "model": "gpt-4",
    "start_time": "24h"
}

# "What's the total token cost for my-service this week"
{
    "service_name": "my-service",
    "start_time": "7d"
}
```

### 5. list_services

List all instrumented services.

**Example:**

```python
# "What services are being traced?"
# No parameters required
```

### 6. find_errors

Find traces containing errors.

**Parameters:**
- `service_name` (optional): Filter by service
- `start_time` (optional): Start time (default: 1h)
- `end_time` (optional): End time
- `limit` (optional): Max results

**Example:**

```python
# "Show me errors from the last 30 minutes"
{
    "start_time": "30m",
    "limit": 50
}

# "Find errors in the checkout-service"
{
    "service_name": "checkout-service",
    "start_time": "1h"
}
```

### 7. list_llm_models

Discover which LLM models are in use.

**Parameters:**
- `start_time` (optional): Start time
- `end_time` (optional): End time

**Example:**

```python
# "What LLM models are we using?"
{
    "start_time": "24h"
}
```

### 8. get_llm_model_stats

Get performance statistics per model.

**Parameters:**
- `model` (optional): Specific model to analyze
- `start_time` (optional): Start time
- `end_time` (optional): End time

**Example:**

```python
# "Compare performance of gpt-4 vs gpt-3.5-turbo"
# Call twice, once per model:
{
    "model": "gpt-4",
    "start_time": "24h"
}
```

### 9. get_llm_expensive_traces

Find traces with highest token usage.

**Parameters:**
- `service_name` (optional): Filter by service
- `model` (optional): Filter by model
- `start_time` (optional): Start time
- `end_time` (optional): End time
- `limit` (optional): Number of traces (default: 10)

**Example:**

```python
# "Show me the 5 most expensive LLM calls today"
{
    "limit": 5,
    "start_time": "24h"
}
```

### 10. get_llm_slow_traces

Find slowest LLM operations.

**Parameters:**
- `service_name` (optional): Filter by service
- `model` (optional): Filter by model
- `start_time` (optional): Start time
- `end_time` (optional): End time
- `limit` (optional): Number of traces (default: 10)

**Example:**

```python
# "What are the slowest gpt-4 requests?"
{
    "model": "gpt-4",
    "limit": 10,
    "start_time": "1h"
}
```

## Common Patterns

### Time Ranges

Specify time ranges using relative or absolute formats:

```python
# Relative (most common)
"start_time": "1h"   # Last hour
"start_time": "30m"  # Last 30 minutes
"start_time": "24h"  # Last 24 hours
"start_time": "7d"   # Last 7 days

# Absolute (ISO 8601)
"start_time": "2024-01-15T10:00:00Z"
"end_time": "2024-01-15T11:00:00Z"
```

### Filter Operators

Available operators for advanced filtering:

```python
# Equality
{"attribute": "http.method", "operator": "eq", "value": "POST"}

# Comparison
{"attribute": "http.status_code", "operator": "gte", "value": 500}
{"attribute": "duration", "operator": "lt", "value": 1000000}

# Pattern matching (where supported)
{"attribute": "http.url", "operator": "contains", "value": "/api/"}

# Negation
{"attribute": "status.code", "operator": "ne", "value": "OK"}
```

### LLM Cost Analysis

Track token usage and costs:

```python
# Daily token usage summary
{
    "tool": "get_llm_usage",
    "params": {
        "start_time": "24h",
        "end_time": "now"
    }
}

# Most expensive calls
{
    "tool": "get_llm_expensive_traces",
    "params": {
        "model": "gpt-4",
        "limit": 10,
        "start_time": "7d"
    }
}

# Model comparison
# First get list of models
{
    "tool": "list_llm_models",
    "params": {"start_time": "24h"}
}

# Then get stats for each
{
    "tool": "get_llm_model_stats",
    "params": {
        "model": "gpt-4",
        "start_time": "24h"
    }
}
```

### Error Investigation

Debug failures efficiently:

```python
# Find recent errors
{
    "tool": "find_errors",
    "params": {
        "service_name": "api-gateway",
        "start_time": "1h",
        "limit": 20
    }
}

# Get full trace for an error
{
    "tool": "get_trace",
    "params": {
        "trace_id": "error-trace-id-from-above"
    }
}

# Search for specific error patterns
{
    "tool": "search_traces",
    "params": {
        "filters": [
            {
                "attribute": "error.type",
                "operator": "eq",
                "value": "TimeoutError"
            }
        ],
        "start_time": "24h"
    }
}
```

### Performance Optimization

Identify bottlenecks:

```python
# Find slow requests
{
    "tool": "search_traces",
    "params": {
        "min_duration": 5000000,  # 5 seconds
        "start_time": "1h",
        "limit": 20
    }
}

# Slow LLM calls
{
    "tool": "get_llm_slow_traces",
    "params": {
        "limit": 10,
        "start_time": "24h"
    }
}

# Slow database operations
{
    "tool": "search_spans",
    "params": {
        "service_name": "database",
        "min_duration": 100000,  # 100ms
        "start_time": "1h"
    }
}
```

## Real-World Examples

### Example 1: Debug Production Error

User says: "We're seeing 500 errors in production, help me debug"

```python
# Step 1: Find error traces
{
    "tool": "find_errors",
    "params": {
        "start_time": "30m",
        "limit": 50
    }
}

# Step 2: Look for patterns in error responses
{
    "tool": "search_traces",
    "params": {
        "filters": [
            {
                "attribute": "http.status_code",
                "operator": "gte",
                "value": 500
            }
        ],
        "start_time": "30m"
    }
}

# Step 3: Get detailed trace for analysis
{
    "tool": "get_trace",
    "params": {
        "trace_id": "<trace-id-from-step-1>"
    }
}
```

### Example 2: Optimize LLM Costs

User says: "Our OpenAI bill is too high, find what's expensive"

```python
# Step 1: Overall usage
{
    "tool": "get_llm_usage",
    "params": {
        "start_time": "7d"
    }
}

# Step 2: Most expensive traces
{
    "tool": "get_llm_expensive_traces",
    "params": {
        "limit": 20,
        "start_time": "7d"
    }
}

# Step 3: Model comparison
{
    "tool": "get_llm_model_stats",
    "params": {
        "model": "gpt-4",
        "start_time": "7d"
    }
}

# Step 4: Examine expensive trace details
{
    "tool": "get_trace",
    "params": {
        "trace_id": "<expensive-trace-id>"
    }
}
```

### Example 3: Service Discovery

User says: "What services are instrumented and how are they performing?"

```python
# Step 1: List all services
{
    "tool": "list_services"
}

# Step 2: Check each service for errors
{
    "tool": "find_errors",
    "params": {
        "service_name": "api-gateway",
        "start_time": "1h"
    }
}

# Step 3: Performance check
{
    "tool": "search_traces",
    "params": {
        "service_name": "api-gateway",
        "min_duration": 1000000,  # Slow requests > 1s
        "start_time": "1h"
    }
}
```

### Example 4: Monitor Model Performance

User says: "How is gpt-4 performing compared to yesterday?"

```python
# Today's stats
{
    "tool": "get_llm_model_stats",
    "params": {
        "model": "gpt-4",
        "start_time": "24h"
    }
}

# Yesterday's stats for comparison
{
    "tool": "get_llm_model_stats",
    "params": {
        "model": "gpt-4",
        "start_time": "48h",
        "end_time": "24h"
    }
}

# Check for slow calls today
{
    "tool": "get_llm_slow_traces",
    "params": {
        "model": "gpt-4",
        "limit": 10,
        "start_time": "24h"
    }
}
```

## Troubleshooting

### Connection Issues

**Problem**: "Failed to connect to backend"

**Solution**:
```bash
# Verify backend URL is accessible
curl http://localhost:16686/api/services

# Check environment variables
echo $BACKEND_TYPE
echo $BACKEND_URL

# For Traceloop, verify API key
echo $BACKEND_API_KEY
```

### No Results Returned

**Problem**: Search returns empty results

**Solutions**:
```python
# Check if services exist
{
    "tool": "list_services"
}

# Expand time range
{
    "tool": "search_traces",
    "params": {
        "start_time": "24h"  # Instead of "1h"
    }
}

# Remove restrictive filters
{
    "tool": "search_traces",
    "params": {
        "service_name": "my-service"
        # Remove min_duration, tags, etc.
    }
}
```

### Permission Errors (Traceloop/Tempo)

**Problem**: "Unauthorized" or "Forbidden"

**Solution**:
```bash
# Verify API key is set
printenv | grep BACKEND_API_KEY

# For Tempo with multi-tenancy
export BACKEND_HEADERS='{"X-Scope-OrgID": "your-org-id"}'

# Update configuration
opentelemetry-mcp --backend traceloop \
  --url https://api.traceloop.com \
  --api-key ${TRACELOOP_API_KEY}
```

### Jaeger Span Search Requires Service Name

**Problem**: "service_name is required for Jaeger span search"

**Solution**:
```python
# Always include service_name for Jaeger
{
    "tool": "search_spans",
    "params": {
        "service_name": "my-service",  # Required
        "operation_name": "database-query"
    }
}
```

### Invalid Time Format

**Problem**: "Invalid time format"

**Solution**:
```python
# Use relative time
"start_time": "1h"   # ✓ Correct
"start_time": "1 hour"  # ✗ Wrong

# Or ISO 8601
"start_time": "2024-01-15T10:00:00Z"  # ✓ Correct
"start_time": "2024-01-15 10:00:00"   # ✗ Wrong
```

### LLM Tools Return No Data

**Problem**: LLM-specific tools return empty results

**Solution**:
```python
# Ensure traces use OpenLLMetry conventions
# Check if models are being tracked
{
    "tool": "list_llm_models",
    "params": {
        "start_time": "7d"  # Wider range
    }
}

# If empty, traces may not have LLM semantic conventions
# Verify instrumentation includes:
# - gen_ai.request.model
# - gen_ai.usage.input_tokens
# - gen_ai.usage.output_tokens
```

### Duration Filtering Issues

**Problem**: Duration filters not working as expected

**Solution**:
```python
# Durations are in MICROSECONDS, not milliseconds
1_000_000 microseconds = 1 second
100_000 microseconds = 100 milliseconds
1_000 microseconds = 1 millisecond

# Example: Find requests > 5 seconds
{
    "min_duration": 5000000  # 5 seconds
}

# NOT
{
    "min_duration": 5000  # This is 5ms
}
```

### MCP Server Not Starting

**Problem**: Server fails to start in Claude/Cursor

**Solution**:
```bash
# Test server manually
opentelemetry-mcp --backend jaeger --url http://localhost:16686

# Check logs location
# macOS: ~/Library/Logs/Claude/
# Windows: %APPDATA%\Claude\logs\

# Verify Python version
python --version  # Must be 3.11+

# Reinstall
pipx uninstall opentelemetry-mcp
pipx install opentelemetry-mcp
```

## Advanced Configuration

### Multiple Backends

Configure different backends for different environments:

```json
{
  "mcpServers": {
    "otel-jaeger-local": {
      "command": "pipx",
      "args": ["run", "opentelemetry-mcp"],
      "env": {
        "BACKEND_TYPE": "jaeger",
        "BACKEND_URL": "http://localhost:16686"
      }
    },
    "otel-traceloop-prod": {
      "command": "pipx",
      "args": ["run", "opentelemetry-mcp"],
      "env": {
        "BACKEND_TYPE": "traceloop",
        "BACKEND_URL": "https://api.traceloop.com",
        "BACKEND_API_KEY": "${TRACELOOP_API_KEY}"
      }
    }
  }
}
```

### Custom Headers for Authentication

```bash
# Tempo with authentication
export BACKEND_HEADERS='{"Authorization": "Bearer ${YOUR_TOKEN}", "X-Scope-OrgID": "my-org"}'

# Or in configuration
{
  "env": {
    "BACKEND_TYPE": "tempo",
    "BACKEND_URL": "http://tempo.example.com:3200",
    "BACKEND_HEADERS": "{\"X-Scope-OrgID\": \"production\"}"
  }
}
```

## Resources

- **GitHub**: https://github.com/traceloop/opentelemetry-mcp-server
- **PyPI**: https://pypi.org/project/opentelemetry-mcp/
- **OpenLLMetry**: https://github.com/traceloop/openllmetry (LLM instrumentation)
- **MCP Specification**: https://modelcontextprotocol.io/
- **OpenTelemetry**: https://opentelemetry.io/
