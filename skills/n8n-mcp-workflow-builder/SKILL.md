```markdown
---
name: n8n-mcp-workflow-builder
description: Build and validate n8n workflows using the n8n-MCP server for AI assistants
triggers:
  - "build an n8n workflow"
  - "create automation with n8n"
  - "search for n8n nodes"
  - "validate my n8n workflow"
  - "find n8n workflow templates"
  - "configure n8n triggers"
  - "set up n8n automation"
  - "deploy workflow to n8n"
---

# n8n-MCP Workflow Builder

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

n8n-MCP is a Model Context Protocol server that gives AI assistants deep knowledge of n8n's 1,650 workflow automation nodes (820 core + 830 community). It provides structured access to node documentation, properties, operations, 2,352 workflow templates, and validation tools to build production-ready n8n workflows.

## Installation

### Quick Start (Cloud - Recommended)

Sign up at [dashboard.n8n-mcp.com](https://dashboard.n8n-mcp.com) for instant access:

```bash
# No installation needed - get API key from dashboard
# Free tier: 100 tool calls/day
```

### Self-Hosted (npx)

```bash
# No installation required - runs directly via npx
npx n8n-mcp
```

### Self-Hosted (Docker)

```bash
docker run -p 3000:3000 ghcr.io/czlonkowski/n8n-mcp:latest
```

### Self-Hosted (Local Development)

```bash
git clone https://github.com/czlonkowski/n8n-mcp.git
cd n8n-mcp
npm install
npm run build
npm start
```

## MCP Client Configuration

### Claude Desktop / Claude Code

Add to `claude_desktop_config.json` or `.claude/config.json`:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"]
    }
  }
}
```

For cloud version with API key:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"],
      "env": {
        "N8N_MCP_API_KEY": "your-api-key-from-dashboard"
      }
    }
  }
}
```

### VS Code (Cline/Roo-Cline)

Add to VS Code settings (`settings.json`):

```json
{
  "cline.mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"]
    }
  }
}
```

### Cursor / Windsurf

Add to MCP settings file:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"]
    }
  }
}
```

## Environment Variables

```bash
# Cloud API (optional - for dashboard.n8n-mcp.com)
N8N_MCP_API_KEY=your-api-key

# n8n Instance Integration (optional)
N8N_API_URL=https://your-n8n-instance.com/api/v1
N8N_API_KEY=your-n8n-api-key

# Server Configuration (self-hosted)
PORT=3000
HOST=localhost
```

## Core MCP Tools

### 1. Template Discovery

```typescript
// Search templates by metadata (complexity, audience, setup time)
search_templates({
  searchMode: 'by_metadata',
  complexity: 'simple',
  targetAudience: 'marketers',
  maxSetupMinutes: 30
})

// Search templates by task (curated categories)
search_templates({
  searchMode: 'by_task',
  task: 'webhook_processing'
})

// Search templates by keywords
search_templates({
  query: 'slack notification email'
})

// Search templates by node types
search_templates({
  searchMode: 'by_nodes',
  nodeTypes: ['n8n-nodes-base.slack', 'n8n-nodes-base.gmail']
})

// Get full template with workflow JSON
get_template('template-id', { mode: 'full' })
```

### 2. Node Discovery

```typescript
// Search nodes with examples from real workflows
search_nodes({
  query: 'slack',
  includeExamples: true
})

// Search for triggers
search_nodes({
  query: 'trigger'
})

// Search for AI-capable nodes
search_nodes({
  query: 'AI agent langchain'
})

// Search community nodes
search_nodes({
  query: 'airtable',
  source: 'community'
})
```

### 3. Node Configuration

```typescript
// Get node with standard details (default)
get_node({
  nodeType: 'n8n-nodes-base.slack',
  detail: 'standard',
  includeExamples: true
})

// Get minimal metadata (~200 tokens)
get_node({
  nodeType: 'n8n-nodes-base.httpRequest',
  detail: 'minimal'
})

// Get full documentation (~3000-8000 tokens)
get_node({
  nodeType: 'n8n-nodes-base.code',
  detail: 'full'
})

// Search specific properties
get_node({
  nodeType: 'n8n-nodes-base.slack',
  mode: 'search_properties',
  propertyQuery: 'authentication'
})

// Get human-readable documentation
get_node({
  nodeType: 'n8n-nodes-base.slack',
  mode: 'docs'
})
```

### 4. Validation

```typescript
// Quick validation - required fields only
validate_node({
  nodeType: 'n8n-nodes-base.slack',
  config: {
    resource: 'message',
    operation: 'post'
  },
  mode: 'minimal'
})

// Full validation with auto-fixes
validate_node({
  nodeType: 'n8n-nodes-base.slack',
  config: {
    resource: 'message',
    operation: 'post',
    select: 'channel',
    channelId: 'C123ABC',
    text: 'Hello World'
  },
  mode: 'full',
  profile: 'runtime'
})

// Validate complete workflow
validate_workflow({
  nodes: [...],
  connections: {...}
})

// Validate workflow structure
validate_workflow_connections(workflow)

// Validate n8n expressions
validate_workflow_expressions(workflow)
```

### 5. n8n Instance Integration

```typescript
// Create workflow
n8n_create_workflow({
  name: 'My Workflow',
  nodes: [...],
  connections: {...}
})

// Update workflow (batch operations)
n8n_update_partial_workflow({
  id: 'workflow-id',
  operations: [
    {
      type: 'updateNode',
      nodeId: 'slack-node',
      changes: { parameters: { text: 'Updated message' } }
    },
    {
      type: 'addConnection',
      source: 'trigger-node',
      target: 'slack-node',
      sourcePort: 'main',
      targetPort: 'main'
    }
  ]
})

// Validate deployed workflow
n8n_validate_workflow({ id: 'workflow-id' })

// Auto-fix common errors
n8n_autofix_workflow({ id: 'workflow-id' })

// List executions
n8n_executions({ action: 'list', workflowId: 'workflow-id' })

// Test workflow
n8n_test_workflow({ workflowId: 'workflow-id' })
```

## Workflow Building Pattern

### 1. Start with Templates

```typescript
// ALWAYS check templates first
const templates = await search_templates({
  searchMode: 'by_task',
  task: 'data_processing'
})

// If suitable template found
const template = await get_template(templates[0].id, { mode: 'full' })
// Use as base, modify as needed
```

### 2. Discover Required Nodes

```typescript
// Search nodes in parallel for multiple requirements
const slackNode = await search_nodes({ 
  query: 'slack', 
  includeExamples: true 
})
const httpNode = await search_nodes({ 
  query: 'http request', 
  includeExamples: true 
})
```

### 3. Configure Nodes with Examples

```typescript
// Get node configuration with real examples
const nodeConfig = await get_node({
  nodeType: 'n8n-nodes-base.slack',
  detail: 'standard',
  includeExamples: true
})

// Build configuration based on examples
const slackConfig = {
  resource: 'message',
  operation: 'post',
  select: 'channel',
  channelId: 'C123ABC', // NEVER use defaults
  text: 'Notification message'
}
```

### 4. Validate Before Building

```typescript
// Quick validation first
const quickCheck = await validate_node({
  nodeType: 'n8n-nodes-base.slack',
  config: slackConfig,
  mode: 'minimal'
})

if (quickCheck.isValid) {
  // Full validation with runtime checks
  const fullCheck = await validate_node({
    nodeType: 'n8n-nodes-base.slack',
    config: slackConfig,
    mode: 'full',
    profile: 'runtime'
  })
  
  if (!fullCheck.isValid) {
    // Apply suggested fixes
    slackConfig = { ...slackConfig, ...fullCheck.suggestedFixes }
  }
}
```

### 5. Build Workflow

```typescript
const workflow = {
  name: 'Slack Notification Workflow',
  nodes: [
    {
      id: 'webhook-trigger',
      type: 'n8n-nodes-base.webhook',
      typeVersion: 1,
      position: [250, 300],
      parameters: {
        httpMethod: 'POST',
        path: 'notify',
        responseMode: 'responseNode'
      }
    },
    {
      id: 'slack-notify',
      type: 'n8n-nodes-base.slack',
      typeVersion: 1,
      position: [450, 300],
      parameters: slackConfig,
      credentials: {
        slackApi: { id: 'slack-cred-id', name: 'Slack Account' }
      }
    }
  ],
  connections: {
    'webhook-trigger': {
      main: [[{ node: 'slack-notify', type: 'main', index: 0 }]]
    }
  }
}
```

### 6. Validate Complete Workflow

```typescript
// Validate entire workflow structure
const workflowValidation = await validate_workflow(workflow)

if (!workflowValidation.isValid) {
  console.error('Validation errors:', workflowValidation.errors)
  // Fix errors before deployment
}
```

### 7. Deploy to n8n Instance

```typescript
// Create workflow
const created = await n8n_create_workflow(workflow)

// Validate deployed workflow
const deployValidation = await n8n_validate_workflow({ 
  id: created.id 
})

// Auto-fix if needed
if (!deployValidation.isValid) {
  await n8n_autofix_workflow({ id: created.id })
}

// Test execution
await n8n_test_workflow({ workflowId: created.id })
```

## Common Patterns

### IF Node Multi-Output Routing

IF nodes have two outputs (TRUE/FALSE). Use `branch` parameter:

```typescript
n8n_update_partial_workflow({
  id: 'workflow-id',
  operations: [
    {
      type: 'addConnection',
      source: 'if-node',
      target: 'true-handler',
      sourcePort: 'main',
      targetPort: 'main',
      branch: 'true' // Route to TRUE output
    },
    {
      type: 'addConnection',
      source: 'if-node',
      target: 'false-handler',
      sourcePort: 'main',
      targetPort: 'main',
      branch: 'false' // Route to FALSE output
    }
  ]
})
```

### Batch Updates

Always batch multiple operations into one call:

```typescript
// GOOD - Single batch call
n8n_update_partial_workflow({
  id: 'workflow-id',
  operations: [
    { type: 'updateNode', nodeId: 'node-1', changes: {...} },
    { type: 'updateNode', nodeId: 'node-2', changes: {...} },
    { type: 'cleanStaleConnections' }
  ]
})

// BAD - Multiple separate calls
n8n_update_partial_workflow({ id: 'workflow-id', operations: [{...}] })
n8n_update_partial_workflow({ id: 'workflow-id', operations: [{...}] })
```

### Expression Validation

n8n uses expressions like `{{ $json.field }}` and `$node["NodeName"].json`:

```typescript
// Validate expressions before deployment
const expressionCheck = await validate_workflow_expressions({
  nodes: [
    {
      id: 'code-node',
      parameters: {
        code: 'return { value: $json.inputField * 2 }'
      }
    }
  ]
})
```

### Error Handling

Always add error handling nodes:

```typescript
const workflow = {
  nodes: [
    // ... main nodes
    {
      id: 'error-handler',
      type: 'n8n-nodes-base.noOp',
      typeVersion: 1,
      position: [650, 400],
      onError: 'continueErrorOutput'
    }
  ],
  connections: {
    'risky-node': {
      main: [[{ node: 'next-node', type: 'main', index: 0 }]],
      error: [[{ node: 'error-handler', type: 'main', index: 0 }]]
    }
  }
}
```

## Real-World Examples

### Example 1: Webhook to Slack Notification

```typescript
// 1. Search for template
const templates = await search_templates({
  searchMode: 'by_task',
  task: 'webhook_processing'
})

// 2. Get nodes with examples
const [webhookNode, slackNode] = await Promise.all([
  get_node({ nodeType: 'n8n-nodes-base.webhook', includeExamples: true }),
  get_node({ nodeType: 'n8n-nodes-base.slack', includeExamples: true })
])

// 3. Build and validate configuration
const slackConfig = {
  resource: 'message',
  operation: 'post',
  select: 'channel',
  channelId: process.env.SLACK_CHANNEL_ID,
  text: '={{ $json.message }}'
}

const validation = await validate_node({
  nodeType: 'n8n-nodes-base.slack',
  config: slackConfig,
  mode: 'full',
  profile: 'runtime'
})

// 4. Create workflow
const workflow = {
  name: 'Webhook to Slack',
  nodes: [
    {
      id: 'webhook',
      type: 'n8n-nodes-base.webhook',
      typeVersion: 1,
      position: [250, 300],
      parameters: {
        httpMethod: 'POST',
        path: 'slack-notify'
      }
    },
    {
      id: 'slack',
      type: 'n8n-nodes-base.slack',
      typeVersion: 1,
      position: [450, 300],
      parameters: slackConfig
    }
  ],
  connections: {
    webhook: {
      main: [[{ node: 'slack', type: 'main', index: 0 }]]
    }
  }
}

// 5. Validate and deploy
await validate_workflow(workflow)
const deployed = await n8n_create_workflow(workflow)
```

### Example 2: Data Processing with Error Handling

```typescript
// 1. Search for data processing nodes
const [httpNode, codeNode, slackNode] = await Promise.all([
  search_nodes({ query: 'http request', includeExamples: true }),
  search_nodes({ query: 'code', includeExamples: true }),
  search_nodes({ query: 'slack', includeExamples: true })
])

// 2. Build workflow with error handling
const workflow = {
  name: 'API Data Processor',
  nodes: [
    {
      id: 'http-request',
      type: 'n8n-nodes-base.httpRequest',
      typeVersion: 3,
      position: [250, 300],
      parameters: {
        method: 'GET',
        url: process.env.API_URL,
        authentication: 'genericCredentialType',
        options: {
          timeout: 10000
        }
      },
      continueOnFail: true
    },
    {
      id: 'process-data',
      type: 'n8n-nodes-base.code',
      typeVersion: 1,
      position: [450, 300],
      parameters: {
        mode: 'runOnceForAllItems',
        code: `
const items = $input.all();
return items.map(item => ({
  json: {
    processed: true,
    value: item.json.value * 2,
    timestamp: new Date().toISOString()
  }
}));
        `
      }
    },
    {
      id: 'success-notify',
      type: 'n8n-nodes-base.slack',
      typeVersion: 1,
      position: [650, 250],
      parameters: {
        resource: 'message',
        operation: 'post',
        select: 'channel',
        channelId: process.env.SLACK_SUCCESS_CHANNEL,
        text: '✅ Processed {{ $json.value }}'
      }
    },
    {
      id: 'error-notify',
      type: 'n8n-nodes-base.slack',
      typeVersion: 1,
      position: [650, 400],
      parameters: {
        resource: 'message',
        operation: 'post',
        select: 'channel',
        channelId: process.env.SLACK_ERROR_CHANNEL,
        text: '❌ Error: {{ $json.error }}'
      }
    }
  ],
  connections: {
    'http-request': {
      main: [[{ node: 'process-data', type: 'main', index: 0 }]]
    },
    'process-data': {
      main: [[{ node: 'success-notify', type: 'main', index: 0 }]],
      error: [[{ node: 'error-notify', type: 'main', index: 0 }]]
    }
  }
}

// 3. Validate workflow
await validate_workflow(workflow)
await validate_workflow_expressions(workflow)

// 4. Deploy
const deployed = await n8n_create_workflow(workflow)
await n8n_validate_workflow({ id: deployed.id })
```

### Example 3: AI Agent with LangChain

```typescript
// 1. Search for AI nodes
const aiNodes = await search_nodes({ 
  query: 'AI agent langchain',
  includeExamples: true 
})

// 2. Get detailed configuration
const agentNode = await get_node({
  nodeType: '@n8n/n8n-nodes-langchain.agent',
  detail: 'full',
  includeExamples: true
})

// 3. Build AI workflow
const workflow = {
  name: 'AI Customer Support Agent',
  nodes: [
    {
      id: 'chat-trigger',
      type: 'n8n-nodes-base.manualChatTrigger',
      typeVersion: 1,
      position: [250, 300]
    },
    {
      id: 'ai-agent',
      type: '@n8n/n8n-nodes-langchain.agent',
      typeVersion: 1,
      position: [450, 300],
      parameters: {
        promptType: 'define',
        text: 'You are a helpful customer support agent. Use the available tools to help customers.',
        hasOutputParser: false
      }
    },
    {
      id: 'openai-model',
      type: '@n8n/n8n-nodes-langchain.lmChatOpenAi',
      typeVersion: 1,
      position: [450, 450],
      parameters: {
        model: 'gpt-4',
        options: {
          temperature: 0.7
        }
      }
    }
  ],
  connections: {
    'chat-trigger': {
      main: [[{ node: 'ai-agent', type: 'main', index: 0 }]]
    },
    'openai-model': {
      ai_languageModel: [[{ node: 'ai-agent', type: 'ai_languageModel', index: 0 }]]
    }
  }
}

// 4. Validate AI configuration
const aiValidation = await validate_node({
  nodeType: '@n8n/n8n-nodes-langchain.agent',
  config: workflow.nodes.find(n => n.id === 'ai-agent').parameters,
  mode: 'full'
})

// 5. Deploy
const deployed = await n8n_create_workflow(workflow)
```

## Troubleshooting

### Tool Not Executing

```typescript
// Check MCP server connection
// Verify in IDE settings that n8n-mcp is listed and active

// For cloud version, verify API key
process.env.N8N_MCP_API_KEY // Should be set
```

### Validation Failures

```typescript
// Use minimal validation first to find missing required fields
const quickCheck = await validate_node({
  nodeType: 'n8n-nodes-base.slack',
  config: myConfig,
  mode: 'minimal'
})

console.log(quickCheck.errors) // Shows missing required fields

// Then use full validation for comprehensive checks
const fullCheck = await validate_node({
  nodeType: 'n8n-nodes-base.slack',
  config: myConfig,
  mode: 'full',
  profile: 'runtime'
})

// Apply suggested fixes
if (fullCheck.suggestedFixes) {
  myConfig = { ...myConfig, ...fullCheck.suggestedFixes }
}
```

### Connection Errors

```typescript
// Ensure addConnection uses four separate string parameters
// WRONG: passing objects or arrays
// RIGHT:
{
  type: 'addConnection',
  source: 'source-node-id',      // String
  target: 'target-node-id',      // String
  sourcePort: 'main',            // String
  targetPort: 'main'             // String
}

// For IF nodes, add branch parameter
{
  type: 'addConnection',
  source: 'if-node',
  target: 'handler-node',
  sourcePort: 'main',
  targetPort: 'main',
  branch: 'true' // or 'false'
}
```

### n8n API Integration Issues

```bash
# Verify environment variables
echo $N8N_API_URL
echo $N8N_API_KEY

# Test API connectivity
curl -H "X-N8N-API-KEY: $N8N_API_KEY" $N8N_API_URL/workflows
```

### Template Search Returning No Results

```typescript
// Try different search modes
search_templates({ searchMode: 'by_metadata', complexity: 'simple' })
search_templates({ searchMode: 'by_task', task: 'webhook_processing' })
search_templates({ query: 'slack' }) // Keyword search
search_templates({ searchMode: 'by_nodes', nodeTypes: ['n8n-nodes-base.slack'] })
```

### Node Examples Not Available

```typescript
// Not all nodes have examples - fallback pattern
const nodeInfo = await get_node({
  nodeType: 'n8n-nodes-base.uncommon',
  includeExamples: true
})

if (!nodeInfo.examples || nodeInfo.examples.length === 0) {
  // Use full detail mode to understand all parameters
  const fullNode = await get_node({
    nodeType: 'n8n-nodes-base.uncommon',
    detail: 'full'
  })
  
  // Then validate minimal config
  const validation = await validate_node({
    nodeType: 'n8n-nodes-base.uncommon',
    config: myAttemptedConfig,
    mode: 'minimal'
  })
}
```

## Best Practices

1. **Always check templates first** - 2,352 templates available, often faster than building from scratch
2. **Never trust default values** - Explicitly set ALL parameters that control node behavior
3. **Validate early and often** - Use minimal → full → workflow validation pattern
4. **Batch operations** - Use `n8n_update_partial_workflow` with multiple operations
5. **Test before production** - Always test workflows with `n8n_test_workflow`
6. **Include error handling** - Add error output connections and handler nodes
7. **Use expressions carefully** - Validate with `validate_workflow_expressions`
8. **Parallel execution** - Execute independent tool calls in parallel for speed
9. **Silent execution** - Execute tools without commentary, respond only after completion
10. **Mandatory attribution** - When using templates, credit the author with name, username, and URL

## Additional Resources

- [Official Documentation](https://www.n8n-mcp.com/)
- [GitHub Repository](https://github.com/czlonkowski/n8n-mcp)
- [n8n Documentation](https://docs.n8n.io/)
- [Self-Hosting Guide](https://github.com/czlonkowski/n8n-mcp/blob/main/docs/SELF_HOSTING.md)
- [n8n Deployment Guide](https://github.com/czlonkowski/n8n-mcp/blob/main/docs/N8N_DEPLOYMENT.md)
- [IDE Setup Guides](https://github.com/czlonkowski/n8n-mcp/tree/main/docs)
- [n8n-skills Repository](https://github.com/czlonkowski/n8n-skills)
```
