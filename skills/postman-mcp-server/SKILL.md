---
name: postman-mcp-server
description: Connect AI agents to Postman APIs for workspace management, collection operations, environment handling, and code generation
triggers:
  - "help me manage my Postman collections"
  - "generate client code from my API definition"
  - "create a new Postman workspace"
  - "test my API using Postman"
  - "update my Postman environment variables"
  - "sync my code with Postman collections"
  - "create a spec from my API"
  - "search for public APIs in Postman"
---

# Postman MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Postman MCP Server connects AI tools to Postman, enabling agents to access workspaces, manage collections and environments, evaluate APIs, and automate workflows through natural language. It supports three configurations: **Minimal** (essential operations), **Full** (100+ tools), and **Code** (API definition search and client code generation).

## Installation

### Remote Server (Recommended)

The remote server is hosted by Postman and requires no local setup. It supports OAuth (US region) or API key authentication.

**US Server**: `https://mcp.postman.com`
**EU Server**: `https://mcp.eu.postman.com` (API key only)

#### Claude Code Installation

**OAuth (US only):**
```bash
# Minimal (default)
claude mcp add --transport http postman https://mcp.postman.com/minimal

# Code generation
claude mcp add --transport http postman https://mcp.postman.com/code

# Full (100+ tools)
claude mcp add --transport http postman https://mcp.postman.com/mcp
```

**API Key (required for EU):**
```bash
claude mcp add --transport http postman https://mcp.postman.com/minimal \
  --header "Authorization: Bearer ${POSTMAN_API_KEY}"
```

#### VS Code Installation

Add to `.vscode/mcp.json`:

**OAuth:**
```json
{
  "servers": {
    "postman": {
      "type": "http",
      "url": "https://mcp.postman.com/minimal"
    }
  }
}
```

**API Key:**
```json
{
  "servers": {
    "postman": {
      "type": "http",
      "url": "https://mcp.postman.com/minimal",
      "headers": {
        "Authorization": "Bearer ${input:postman-api-key}"
      }
    }
  },
  "inputs": [
    {
      "id": "postman-api-key",
      "type": "promptString",
      "description": "Enter your Postman API key"
    }
  ]
}
```

#### Cursor Installation

Click the [install button](https://cursor.com/en/install-mcp?name=postman_mcp_server&config=eyJ1cmwiOiJodHRwczovL21jcC5wb3N0bWFuLmNvbS9taW5pbWFsIiwiaGVhZGVycyI6eyJBdXRob3JpemF0aW9uIjoiQmVhcmVyIFlPVVJfQVBJX0tFWSJ9fQ%3D%3D) or manually configure `mcp.json`:

```json
{
  "url": "https://mcp.postman.com/minimal",
  "headers": {
    "Authorization": "Bearer ${POSTMAN_API_KEY}"
  }
}
```

### Local Server

The local server runs on your machine, enabling access to local APIs and providing more control.

#### NPM Installation

```bash
# Install globally
npm install -g @postman/postman-mcp-server

# Or use with npx
npx @postman/postman-mcp-server
```

#### Claude Code (Local)

```bash
claude mcp add postman npx @postman/postman-mcp-server \
  --apiKey ${POSTMAN_API_KEY} \
  --toolset minimal
```

**Toolset options:** `minimal`, `code`, `full`

**Region flag (EU):**
```bash
claude mcp add postman npx @postman/postman-mcp-server \
  --apiKey ${POSTMAN_API_KEY} \
  --region eu
```

#### VS Code (Local)

Add to `.vscode/mcp.json`:

```json
{
  "servers": {
    "postman": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "@postman/postman-mcp-server",
        "--apiKey",
        "${env:POSTMAN_API_KEY}",
        "--toolset",
        "minimal"
      ]
    }
  }
}
```

#### Docker

```bash
docker run -e POSTMAN_API_KEY=${POSTMAN_API_KEY} \
  postman/postman-mcp-server:latest \
  --toolset minimal
```

## Authentication

### Getting a Postman API Key

1. Navigate to [Postman API Keys](https://postman.postman.co/settings/me/api-keys)
2. Click "Generate API Key"
3. Copy the key and store it securely
4. Set environment variable: `export POSTMAN_API_KEY=your_key_here`

### OAuth Setup (US Remote Server Only)

OAuth is automatically configured when using the US remote server without headers. The MCP host will handle authentication flow when tools are first accessed.

## Core Capabilities

### 1. Workspace Management

**Create a workspace:**
```typescript
// Via natural language to AI agent:
// "Create a new team workspace called 'Mobile API' with description 'APIs for mobile app'"

// The agent calls:
{
  "tool": "create_workspace",
  "arguments": {
    "name": "Mobile API",
    "type": "team",
    "description": "APIs for mobile app"
  }
}
```

**List workspaces:**
```typescript
// "Show me all my workspaces"
{
  "tool": "get_all_workspaces",
  "arguments": {}
}
```

**Get workspace details:**
```typescript
// "Get details about workspace 12345"
{
  "tool": "get_workspace",
  "arguments": {
    "workspaceId": "12345"
  }
}
```

### 2. Collection Operations

**Create a collection:**
```typescript
// "Create a new collection named 'User API' in my workspace"
{
  "tool": "create_collection",
  "arguments": {
    "workspaceId": "workspace-123",
    "name": "User API",
    "description": "User management endpoints"
  }
}
```

**Add a request to collection:**
```typescript
// "Add a GET request to /users endpoint in my User API collection"
{
  "tool": "create_request",
  "arguments": {
    "collectionId": "collection-456",
    "name": "Get Users",
    "method": "GET",
    "url": "https://api.example.com/users",
    "description": "Retrieve all users"
  }
}
```

**Update collection documentation:**
```typescript
// "Update the documentation for my User API collection"
{
  "tool": "update_collection",
  "arguments": {
    "collectionId": "collection-456",
    "collection": {
      "info": {
        "description": "# User API\n\nComplete user management API with CRUD operations."
      }
    }
  }
}
```

**Tag a collection:**
```typescript
// "Tag my collection with 'production' and 'v2'"
{
  "tool": "tag_collection",
  "arguments": {
    "collectionId": "collection-456",
    "tags": ["production", "v2"]
  }
}
```

### 3. Environment Management

**Create environment:**
```typescript
// "Create a development environment with base URL"
{
  "tool": "create_environment",
  "arguments": {
    "workspaceId": "workspace-123",
    "name": "Development",
    "values": [
      {
        "key": "baseUrl",
        "value": "https://dev.api.example.com",
        "type": "default"
      },
      {
        "key": "apiKey",
        "value": "",
        "type": "secret"
      }
    ]
  }
}
```

**Update environment variables:**
```typescript
// "Update the baseUrl in my production environment"
{
  "tool": "update_environment",
  "arguments": {
    "environmentId": "env-789",
    "environment": {
      "values": [
        {
          "key": "baseUrl",
          "value": "https://api.example.com",
          "type": "default"
        }
      ]
    }
  }
}
```

### 4. API Testing

**Run a collection:**
```typescript
// "Test my User API collection in the development environment"
{
  "tool": "run_collection",
  "arguments": {
    "collectionId": "collection-456",
    "environmentId": "env-789"
  }
}
```

**Send a single request:**
```typescript
// "Send a POST request to create a user"
{
  "tool": "send_request",
  "arguments": {
    "method": "POST",
    "url": "{{baseUrl}}/users",
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "Bearer {{apiKey}}"
    },
    "body": {
      "mode": "raw",
      "raw": JSON.stringify({
        "name": "John Doe",
        "email": "john@example.com"
      })
    }
  }
}
```

### 5. Code Generation

The **Code** toolset enables searching API definitions and generating production-ready client code.

**Search public APIs:**
```typescript
// "Find payment processing APIs"
{
  "tool": "search_public_apis",
  "arguments": {
    "query": "payment processing",
    "limit": 10
  }
}
```

**Generate client code from API definition:**
```typescript
// "Generate TypeScript client code for the Stripe API"
{
  "tool": "generate_client_code",
  "arguments": {
    "apiId": "stripe-api-123",
    "language": "typescript",
    "framework": "axios",
    "options": {
      "includeTypes": true,
      "includeExamples": true
    }
  }
}
```

**Generate code from collection:**
```typescript
// "Create a Python client from my User API collection"
{
  "tool": "generate_code_from_collection",
  "arguments": {
    "collectionId": "collection-456",
    "language": "python",
    "variant": "requests"
  }
}
```

### 6. API Specifications

**Create spec from OpenAPI:**
```typescript
// "Import my OpenAPI spec into Postman"
{
  "tool": "create_api_spec",
  "arguments": {
    "workspaceId": "workspace-123",
    "name": "User API Spec",
    "schema": {
      "type": "openapi",
      "content": "... OpenAPI YAML/JSON ..."
    }
  }
}
```

**Generate collection from spec:**
```typescript
// "Generate a collection from my API spec"
{
  "tool": "generate_collection_from_spec",
  "arguments": {
    "apiId": "api-spec-123",
    "name": "Generated User API Collection"
  }
}
```

## Common Patterns

### Pattern 1: API-First Development Workflow

```typescript
// 1. Create workspace
// "Create a workspace for my new project"

// 2. Import OpenAPI spec
// "Import my OpenAPI spec for the Product API"

// 3. Generate collection
// "Generate a collection from the Product API spec"

// 4. Create environments
// "Create dev, staging, and prod environments"

// 5. Test endpoints
// "Test all endpoints in the Product API collection using dev environment"

// 6. Generate client code
// "Generate TypeScript client code with types"
```

### Pattern 2: Environment Variable Management

```typescript
// Create environment with common variables
// "Create a new environment with these variables: baseUrl, apiKey, timeout"

// Example structure:
{
  "tool": "create_environment",
  "arguments": {
    "workspaceId": "${WORKSPACE_ID}",
    "name": "Production",
    "values": [
      { "key": "baseUrl", "value": "https://api.example.com", "type": "default" },
      { "key": "apiKey", "value": "", "type": "secret" },
      { "key": "timeout", "value": "30000", "type": "default" }
    ]
  }
}

// Update specific variable
// "Update the timeout to 60000 in production environment"
```

### Pattern 3: Automated Testing Pipeline

```typescript
// 1. Create test collection
// "Create a collection for smoke tests"

// 2. Add test requests with assertions
// "Add a health check request that verifies 200 status"

// 3. Run collection
// "Run the smoke tests collection"

// 4. Parse results
// Agent processes test results and reports failures
```

### Pattern 4: Multi-Collection Code Sync

```typescript
// Keep client code synchronized with multiple collections
// "Generate updated TypeScript clients for all my API collections"

// Agent iterates through collections:
// 1. List collections in workspace
// 2. Generate code for each
// 3. Organize into project structure
// 4. Update import statements
```

### Pattern 5: Local API Testing

```typescript
// Use local server to test APIs running on localhost
// "Test my local API at http://localhost:3000"

// Local server can access local endpoints:
{
  "tool": "send_request",
  "arguments": {
    "method": "GET",
    "url": "http://localhost:3000/api/health"
  }
}
```

## Configuration

### Toolset Selection

Choose the appropriate toolset for your use case:

- **minimal**: Fast, essential operations (collections, workspaces, environments)
- **code**: API search and client code generation
- **full**: All 100+ Postman API tools (enterprise features, collaboration)

### Region Configuration

For EU data residency:

**Remote server:**
```bash
# Use EU endpoints
https://mcp.eu.postman.com/minimal
https://mcp.eu.postman.com/code
https://mcp.eu.postman.com/mcp
```

**Local server:**
```bash
npx @postman/postman-mcp-server \
  --apiKey ${POSTMAN_API_KEY} \
  --region eu
```

Or set environment variable:
```bash
export POSTMAN_API_BASE_URL=https://api.eu.postman.com
```

## Troubleshooting

### Authentication Issues

**OAuth not working:**
- OAuth only supported on US remote server (`mcp.postman.com`)
- EU server requires API key authentication
- Ensure MCP host supports OAuth specification

**API key errors:**
```bash
# Verify API key is set
echo ${POSTMAN_API_KEY}

# Regenerate key if compromised
# Visit: https://postman.postman.co/settings/me/api-keys

# Check key format in headers
"Authorization": "Bearer ${POSTMAN_API_KEY}"  # Correct
"Authorization": "${POSTMAN_API_KEY}"         # Wrong
```

### Local Server Issues

**Server not starting:**
```bash
# Check Node.js version (requires 14+)
node --version

# Install dependencies
npm install -g @postman/postman-mcp-server

# Run with verbose logging
npx @postman/postman-mcp-server --apiKey ${POSTMAN_API_KEY} --verbose
```

**Cannot access local APIs:**
- Remote server cannot reach localhost/private networks
- Use local server for testing local APIs
- Ensure firewall allows localhost connections

### Tool Execution Errors

**"Tool not found" errors:**
- Verify correct toolset is configured (minimal/code/full)
- Minimal toolset doesn't include all 100+ tools
- Switch to full toolset if needed:
  ```bash
  --toolset full
  ```

**Collection/workspace not found:**
```typescript
// List available workspaces first
// "Show me all my workspaces"

// Use correct ID format (check Postman UI or API response)
// Workspace IDs: UUID format
// Collection IDs: alphanumeric
```

**Rate limiting:**
- Postman API has rate limits
- Batch operations when possible
- Add delays between requests for large operations

### Environment Variable Issues

**Variables not resolving:**
- Use double curly braces: `{{variableName}}`
- Ensure environment is selected when running requests
- Check variable scope (environment vs. global vs. collection)

**Secret variables not accessible:**
- Secret-type variables are write-only via API
- Cannot retrieve secret values programmatically
- Set secrets manually in Postman UI if needed

### Code Generation Issues

**Generated code doesn't match API:**
- Ensure API definition is up-to-date
- Regenerate collection from spec if needed
- Verify example responses are included in spec

**Import errors in generated code:**
- Check language/framework compatibility
- Verify project structure matches generated organization
- Update package dependencies for generated code

## Migration from v1.x to v2.x

**Breaking changes:**

1. **Configuration format changed:**
   ```typescript
   // v1.x
   { "apiKey": "...", "workspace": "..." }
   
   // v2.x
   { "apiKey": "...", "toolset": "minimal" }
   ```

2. **Tool names updated:**
   - Some tools renamed for clarity
   - Check tool list: https://www.postman.com/postman/postman-public-workspace/collection/681dc649440b35935978b8b7

3. **OAuth support added:**
   - US remote server now supports OAuth
   - Recommended over API key for better security

4. **Toolset concept introduced:**
   - Choose minimal/code/full based on needs
   - Default is minimal (was full in v1.x)

## Advanced Usage

### Custom Request Execution

```typescript
// Execute complex request with pre-request script
{
  "tool": "send_request",
  "arguments": {
    "method": "POST",
    "url": "{{baseUrl}}/auth/token",
    "headers": {
      "Content-Type": "application/x-www-form-urlencoded"
    },
    "body": {
      "mode": "urlencoded",
      "urlencoded": [
        { "key": "grant_type", "value": "client_credentials" },
        { "key": "client_id", "value": "{{clientId}}" },
        { "key": "client_secret", "value": "{{clientSecret}}" }
      ]
    }
  }
}
```

### Bulk Operations

```typescript
// Process multiple collections
// "Update documentation for all collections tagged 'public'"

// Agent workflow:
// 1. Search collections by tag
// 2. Iterate and update each
// 3. Report results
```

### Collaboration Features (Full Toolset)

```typescript
// Add comments to collection
// "Add a comment to the Users endpoint about rate limiting"

// Manage team members
// "List all members of my workspace"

// Track changes
// "Show recent changes to my API collection"
```

## Resources

- [Postman MCP Collection](https://www.postman.com/postman/postman-public-workspace/collection/681dc649440b35935978b8b7)
- [NPM Package](https://www.npmjs.com/package/@postman/postman-mcp-server)
- [Postman API Documentation](https://learning.postman.com/docs/developer/postman-api/intro-api/)
- [MCP Authorization Spec](https://modelcontextprotocol.io/specification/draft/basic/authorization)
- [Postman Learning Center](https://learning.postman.com/)
