---
name: mcpspend-cost-tracking
description: Real-time cost observability for MCP tool calls - track spend per tool/project/customer across Claude Desktop, Cursor, Windsurf, and VS Code
triggers:
  - track MCP tool call costs
  - monitor AI agent spending
  - set up MCPSpend proxy
  - wrap MCP server with cost tracking
  - query MCPSpend usage data
  - configure cost observability for MCP
  - analyze MCP tool call expenses
  - integrate MCPSpend dashboard
---

# MCPSpend Cost Tracking Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

MCPSpend provides real-time cost observability for Model Context Protocol (MCP) tool calls. It wraps any MCP server as a transparent proxy, attributing spend per tool, project, and customer across Claude Desktop, Cursor, Windsurf, and VS Code.

## Installation

### One-command setup (recommended)

```bash
npx --yes @mcpspend/proxy@latest init --key mcps_live_YOUR_KEY
```

This auto-detects your MCP clients (Claude Desktop, Cursor, Windsurf, VS Code) and wraps all configured servers. It creates a `.mcpspend.bak` backup of your original config.

### Manual installation

```bash
npm install -g @mcpspend/proxy
# or
pnpm add -g @mcpspend/proxy
```

### Get API key

1. Visit [mcpspend.com](https://mcpspend.com)
2. Sign up (free tier: 25K calls/month)
3. Copy your API key from the dashboard

## Configuration

### Wrapping existing MCP servers

MCPSpend modifies your MCP client config to route through the proxy. Example transformation:

**Before (Claude Desktop config.json):**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/you/docs"]
    }
  }
}
```

**After (automatic wrapping):**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@mcpspend/proxy@latest",
        "wrap",
        "--key",
        "mcps_live_YOUR_KEY",
        "--server",
        "filesystem",
        "--",
        "npx",
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/you/docs"
      ]
    }
  }
}
```

### Environment variables

```bash
export MCPSPEND_API_KEY=mcps_live_YOUR_KEY
export MCPSPEND_API_URL=https://api.mcpspend.com  # optional, default
```

### Client-specific config locations

| Client | Config path |
|--------|-------------|
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Desktop (Windows) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Cursor | `~/.cursor/mcp.json` |
| Windsurf | `~/.windsurf/mcp_config.json` |
| VS Code (user) | `~/.vscode/mcp.json` |
| VS Code (workspace) | `.vscode/mcp.json` |

## CLI Commands

### Initialize proxy wrapper

```bash
# Auto-detect and wrap all MCP servers
mcpspend-proxy init --key mcps_live_YOUR_KEY

# Specify client explicitly
mcpspend-proxy init --key mcps_live_YOUR_KEY --client cursor

# Dry run (preview changes)
mcpspend-proxy init --key mcps_live_YOUR_KEY --dry-run
```

### Wrap individual server

```bash
mcpspend-proxy wrap \
  --key mcps_live_YOUR_KEY \
  --server my-server \
  -- \
  npx -y @some/mcp-server --arg value
```

### Wrap HTTP/SSE remote servers

```bash
mcpspend-proxy wrap-http \
  --key mcps_live_YOUR_KEY \
  --url https://remote-mcp-server.com \
  --server remote-server
```

### Unwrap (restore original config)

```bash
mcpspend-proxy unwrap --client claude
```

## MCP Server for Usage Queries

Add the MCPSpend MCP server to query your usage from within any MCP client:

```json
{
  "mcpServers": {
    "mcpspend": {
      "command": "npx",
      "args": ["-y", "@mcpspend/mcp-server"],
      "env": {
        "MCPSPEND_API_KEY": "mcps_live_YOUR_KEY"
      }
    }
  }
}
```

### Available tools

```typescript
// Get today's cost
{
  "name": "get_today_cost",
  "arguments": {}
}

// List top tools by cost
{
  "name": "list_top_tools",
  "arguments": {
    "limit": 10,
    "startDate": "2026-01-01",
    "endDate": "2026-01-31"
  }
}

// Get tool usage details
{
  "name": "get_tool_usage",
  "arguments": {
    "toolName": "filesystem_read",
    "days": 7
  }
}

// List recent sessions
{
  "name": "list_sessions",
  "arguments": {
    "limit": 20
  }
}
```

## HTTP MCP Server (Remote Access)

Query usage via HTTP from any MCP client that supports HTTP transport:

**Endpoint:** `https://api.mcpspend.com/api/mcp`

**Authentication:** Bearer token

```json
{
  "mcpServers": {
    "mcpspend-remote": {
      "url": "https://api.mcpspend.com/api/mcp",
      "transport": "http",
      "headers": {
        "Authorization": "Bearer mcps_live_YOUR_KEY"
      }
    }
  }
}
```

## Code Examples

### TypeScript: Manual proxy integration

```typescript
import { MCPSpendProxy } from '@mcpspend/proxy';

const proxy = new MCPSpendProxy({
  apiKey: process.env.MCPSPEND_API_KEY!,
  serverName: 'my-custom-server',
  organizationId: 'org_123', // optional, for multi-tenant
  metadata: {
    project: 'production',
    customer: 'acme-corp'
  }
});

// Wrap an MCP server process
await proxy.wrap({
  command: 'npx',
  args: ['-y', '@modelcontextprotocol/server-filesystem', '/data']
});
```

### TypeScript: Programmatic usage query

```typescript
import { MCPSpendClient } from '@mcpspend/proxy';

const client = new MCPSpendClient(process.env.MCPSPEND_API_KEY!);

// Get usage for date range
const usage = await client.getUsage({
  startDate: '2026-01-01',
  endDate: '2026-01-31',
  groupBy: 'tool'
});

console.log(usage.totalCost);
console.log(usage.topTools);

// Export to CSV
const csv = await client.exportCSV({
  startDate: '2026-01-01',
  endDate: '2026-01-31'
});
```

### JavaScript: Simple wrapper script

```javascript
#!/usr/bin/env node
const { spawn } = require('child_process');
const { MCPSpendProxy } = require('@mcpspend/proxy');

const proxy = new MCPSpendProxy({
  apiKey: process.env.MCPSPEND_API_KEY,
  serverName: 'filesystem',
  metadata: {
    environment: process.env.NODE_ENV || 'development'
  }
});

proxy.wrap({
  command: process.argv[2],
  args: process.argv.slice(3)
}).catch(console.error);
```

## Common Patterns

### Per-customer attribution

```json
{
  "mcpServers": {
    "customer-fs": {
      "command": "npx",
      "args": [
        "-y", "@mcpspend/proxy@latest", "wrap",
        "--key", "mcps_live_YOUR_KEY",
        "--server", "filesystem",
        "--metadata", "{\"customer\":\"acme-corp\",\"project\":\"website\"}",
        "--",
        "npx", "-y", "@modelcontextprotocol/server-filesystem", "/customer-data"
      ]
    }
  }
}
```

### Budget alerts

Set up in dashboard at [mcpspend.com/dashboard/alerts](https://mcpspend.com/dashboard/alerts):

- Daily/weekly/monthly thresholds
- Per-tool or per-project limits
- Slack/email notifications
- Auto-pause on budget exceeded

### Multi-environment tracking

```bash
# Development
MCPSPEND_API_KEY=mcps_live_DEV_KEY \
MCPSPEND_METADATA='{"env":"dev"}' \
mcpspend-proxy wrap --server dev-fs -- npx ...

# Production
MCPSPEND_API_KEY=mcps_live_PROD_KEY \
MCPSPEND_METADATA='{"env":"prod"}' \
mcpspend-proxy wrap --server prod-fs -- npx ...
```

### Filtering and querying

```typescript
// Query specific tool usage
const fsUsage = await client.getToolUsage('filesystem_read', {
  startDate: '2026-01-01',
  metadata: {
    customer: 'acme-corp'
  }
});

// Get aggregated stats
const stats = await client.getAggregatedStats({
  groupBy: ['tool', 'model'],
  startDate: '2026-01-01',
  endDate: '2026-01-31'
});
```

## Troubleshooting

### Proxy not reporting data

**Check API key:**

```bash
curl -H "Authorization: Bearer mcps_live_YOUR_KEY" \
  https://api.mcpspend.com/api/health
```

**Verify proxy is active:**

```bash
# Check if config was modified
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json | grep mcpspend

# Check for backup
ls -la ~/Library/Application\ Support/Claude/.mcpspend.bak
```

**Enable debug logging:**

```bash
export MCPSPEND_DEBUG=1
# Restart your MCP client
```

### Config not auto-detected

```bash
# Manually specify config path
mcpspend-proxy init \
  --key mcps_live_YOUR_KEY \
  --config /path/to/mcp.json
```

### Tool calls not attributed correctly

Ensure `--server` name matches your config:

```json
{
  "mcpServers": {
    "my-server-name": {  // ← This must match --server flag
      "command": "npx",
      "args": ["@mcpspend/proxy", "wrap", "--server", "my-server-name", ...]
    }
  }
}
```

### High latency

The proxy adds ~1-5ms overhead per tool call. If experiencing issues:

```bash
# Check proxy version
npx @mcpspend/proxy --version

# Update to latest
npm update -g @mcpspend/proxy
```

### Rate limits

Free tier: 25K calls/month. If exceeded:

- Upgrade at [mcpspend.com/pricing](https://mcpspend.com/pricing)
- Check current usage: `mcpspend-proxy usage`
- Set up budget alerts in dashboard

### Unwrap servers

```bash
# Restore from backup
mcpspend-proxy unwrap --client claude

# Or manually restore
cp ~/Library/Application\ Support/Claude/.mcpspend.bak \
   ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

## Privacy & Security

- **No tool arguments or responses** are sent to MCPSpend servers
- Only metadata tracked: tool name, server name, model, latency, success, approximate token count
- API keys stored as SHA-256 hashes
- EU-hosted (GDPR compliant)
- Self-serve data export/deletion at [mcpspend.com/dashboard/account/privacy](https://mcpspend.com/dashboard/account/privacy)

## Resources

- **Dashboard:** [mcpspend.com/dashboard](https://mcpspend.com/dashboard)
- **API docs:** [mcpspend.com/docs](https://mcpspend.com/docs)
- **npm package:** [@mcpspend/proxy](https://www.npmjs.com/package/@mcpspend/proxy)
- **GitHub:** [andreisirbu91-lab/MCPSpend](https://github.com/andreisirbu91-lab/MCPSpend)
- **Support:** support@mcpspend.com
