---
name: railway-mcp-server
description: Use Railway's MCP server to manage Railway projects, deployments, databases, and infrastructure through the Railway CLI
triggers:
  - deploy my app to railway
  - create a railway project
  - check railway deployment status
  - manage railway database
  - configure railway environment variables
  - view railway service logs
  - set up railway mcp server
  - interact with railway infrastructure
---

# Railway MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Railway MCP Server is an official Model Context Protocol server that enables AI assistants to interact with Railway infrastructure. It's now bundled directly into the Railway CLI, allowing you to manage Railway projects, deployments, databases, environment variables, and services programmatically.

## Installation

### Install Railway CLI

The Railway MCP server is bundled with the Railway CLI (v4.0.0+):

```bash
bash <(curl -fsSL https://railway.com/install.sh)
```

### Configure MCP Client

Automatically configure supported MCP clients (Claude Desktop, Cline, etc.):

```bash
railway mcp install
```

For remote (hosted) MCP server instead of local stdio:

```bash
railway mcp install --remote
```

### Manual Configuration

Add to your MCP client configuration (e.g., `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "railway": {
      "command": "railway",
      "args": ["mcp"]
    }
  }
}
```

## Authentication

Login to Railway before using MCP commands:

```bash
railway login
```

This opens a browser for authentication and stores credentials locally.

## Core Capabilities

### Project Management

**List all projects:**
```bash
railway projects
```

**Create a new project:**
```bash
railway init
```

**Link current directory to a project:**
```bash
railway link
```

**Switch projects:**
```bash
railway project
```

### Service Deployment

**Deploy current directory:**
```bash
railway up
```

**Deploy with specific Dockerfile:**
```bash
railway up --dockerfile ./Dockerfile.prod
```

**View deployment status:**
```bash
railway status
```

**List all services:**
```bash
railway service
```

### Environment Variables

**Set environment variable:**
```bash
railway variables set KEY=value
```

**Set multiple variables:**
```bash
railway variables set API_KEY=$API_KEY DB_HOST=postgres.railway.internal
```

**List all variables:**
```bash
railway variables
```

**Delete a variable:**
```bash
railway variables delete KEY
```

### Database Management

**Add a database (Postgres, MySQL, Redis, MongoDB):**
```bash
railway add
```

**Connect to database shell:**
```bash
railway connect postgres
```

**Get database connection string:**
```bash
railway variables | grep DATABASE_URL
```

### Logs and Monitoring

**View service logs:**
```bash
railway logs
```

**Follow logs in real-time:**
```bash
railway logs --follow
```

**View logs for specific deployment:**
```bash
railway logs --deployment <deployment-id>
```

### Domains

**Add custom domain:**
```bash
railway domain
```

**Generate Railway subdomain:**
```bash
railway domain --generate
```

## MCP Server Usage

When running through MCP, the Railway server exposes tools that AI assistants can invoke:

### Available MCP Tools

- `railway_projects_list` - List all Railway projects
- `railway_project_create` - Create a new project
- `railway_services_list` - List services in a project
- `railway_service_create` - Create a new service
- `railway_deploy` - Deploy a service
- `railway_deployments_list` - List deployments
- `railway_deployment_status` - Get deployment status
- `railway_variables_list` - List environment variables
- `railway_variables_set` - Set environment variables
- `railway_variables_delete` - Delete environment variables
- `railway_logs_get` - Retrieve service logs
- `railway_databases_add` - Add a database service
- `railway_domains_list` - List domains
- `railway_domains_add` - Add a custom domain

### Example MCP Interactions

**AI Assistant Usage:**

When an AI assistant has Railway MCP configured, users can make natural language requests:

*User: "Deploy my Node.js app to Railway"*

The AI will:
1. Check if project is linked (`railway_project_status`)
2. Create project if needed (`railway_project_create`)
3. Deploy the service (`railway_deploy`)
4. Monitor deployment status (`railway_deployment_status`)
5. Return the deployment URL

*User: "Add Postgres database and set DATABASE_URL"*

The AI will:
1. Add Postgres plugin (`railway_databases_add`)
2. Retrieve connection string (`railway_variables_list`)
3. Set DATABASE_URL if needed (`railway_variables_set`)

## Configuration Files

### railway.json (Project Config)

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS",
    "buildCommand": "npm run build"
  },
  "deploy": {
    "startCommand": "npm start",
    "healthcheckPath": "/health",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### railway.toml (Multi-Service Config)

```toml
[build]
builder = "NIXPACKS"
buildCommand = "npm run build"

[deploy]
startCommand = "npm start"
healthcheckPath = "/health"
healthcheckTimeout = 300
restartPolicyType = "ON_FAILURE"

[[services]]
name = "frontend"
source = "./frontend"

[[services]]
name = "backend"
source = "./backend"
```

## Common Patterns

### Full Stack Deployment

```bash
# Initialize project
railway init

# Add Postgres database
railway add --database postgres

# Deploy backend service
cd backend
railway up

# Deploy frontend service
cd ../frontend
railway up

# Set environment variables
railway variables set \
  BACKEND_URL=https://backend.railway.app \
  DATABASE_URL=$DATABASE_PRIVATE_URL
```

### CI/CD Integration

GitHub Actions example:

```yaml
name: Deploy to Railway
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Railway
        run: bash <(curl -fsSL https://railway.com/install.sh)
      
      - name: Deploy
        run: railway up --service backend
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
```

### Environment-Specific Variables

```bash
# Production environment
railway environment production
railway variables set NODE_ENV=production API_URL=https://api.prod.example.com

# Staging environment
railway environment staging
railway variables set NODE_ENV=staging API_URL=https://api.staging.example.com
```

## Troubleshooting

### MCP Server Not Found

If `railway mcp` command fails:

```bash
# Check Railway CLI version (must be 4.0.0+)
railway --version

# Upgrade CLI
bash <(curl -fsSL https://railway.com/install.sh)
```

### Authentication Issues

```bash
# Re-authenticate
railway logout
railway login

# Verify authentication
railway whoami
```

### Deployment Failures

```bash
# View detailed logs
railway logs --deployment <deployment-id>

# Check build logs
railway logs --build

# Verify service configuration
railway status
```

### MCP Client Configuration Issues

Remove legacy `@railway/mcp-server` npm package configurations:

```json
// OLD - Remove this
{
  "mcpServers": {
    "railway": {
      "command": "npx",
      "args": ["-y", "@railway/mcp-server"]
    }
  }
}

// NEW - Use this
{
  "mcpServers": {
    "railway": {
      "command": "railway",
      "args": ["mcp"]
    }
  }
}
```

### Connection String Access

```bash
# Private network URL (for internal services)
railway variables | grep PRIVATE_URL

# Public URL (for external access)
railway variables | grep DATABASE_URL
```

## Documentation

- Official Docs: https://docs.railway.com
- CLI MCP Docs: https://docs.railway.com/cli/mcp
- Railway CLI Reference: https://docs.railway.com/cli/quick-start
- Railway Templates: https://railway.app/templates
