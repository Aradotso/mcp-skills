---
name: atlassian-mcp-server
description: Connect Jira, Confluence, and Compass to AI agents and IDEs using Atlassian's remote MCP server with OAuth 2.1 or API token authentication.
triggers:
  - "connect to Atlassian MCP server"
  - "set up Jira integration with MCP"
  - "configure Confluence MCP access"
  - "create Jira issues from my IDE"
  - "search Confluence pages via MCP"
  - "authenticate with Atlassian Rovo"
  - "set up Compass component integration"
  - "configure atlassian-mcp-server"
---

# Atlassian MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Atlassian Rovo MCP Server is a cloud-based remote MCP server that bridges AI agents, IDEs, and other MCP clients with Atlassian Cloud (Jira, Confluence, Compass). It enables natural language interaction with Atlassian data using secure OAuth 2.1 or API token authentication, respecting user permissions.

## What It Does

- **Search & Summarize**: Query Jira issues, Confluence pages, and Compass components
- **Create & Update**: Generate issues, pages, and components from natural language
- **Automate Workflows**: Link content, bulk create items, extract data across Atlassian products
- **Respect Permissions**: All actions honor existing Atlassian Cloud access controls

## Server Endpoint

```
https://mcp.atlassian.com/v1/mcp
```

The server is hosted by Atlassian (remote MCP) — no local installation required. For desktop clients (Claude, Cursor, VS Code), you'll use the `mcp-remote` proxy package.

## Installation & Configuration

### For Desktop Clients (Claude, Cursor, VS Code)

#### 1. Install mcp-remote Proxy

```bash
# Using npm
npm install -g @modelcontextprotocol/mcp-remote

# Using npx (no install)
npx @modelcontextprotocol/mcp-remote
```

#### 2. Configure Client

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "atlassian-rovo": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/mcp-remote",
        "https://mcp.atlassian.com/v1/mcp"
      ],
      "env": {
        "MCP_REMOTE_AUTH_TYPE": "oauth"
      }
    }
  }
}
```

**Cursor** (`.cursor/config.json` in project root):

```json
{
  "mcpServers": {
    "atlassian-rovo": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/mcp-remote",
        "https://mcp.atlassian.com/v1/mcp"
      ],
      "env": {
        "MCP_REMOTE_AUTH_TYPE": "oauth"
      }
    }
  }
}
```

**VS Code** (settings.json):

```json
{
  "mcp.servers": {
    "atlassian-rovo": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/mcp-remote",
        "https://mcp.atlassian.com/v1/mcp"
      ],
      "env": {
        "MCP_REMOTE_AUTH_TYPE": "oauth"
      }
    }
  }
}
```

#### 3. Authentication Flow

On first use, the client will:
1. Open a browser to Atlassian OAuth consent screen
2. Prompt you to authorize access to Jira/Confluence/Compass
3. Store credentials securely for future sessions

### For Headless/API Token Authentication

Admins must first enable API token auth in Atlassian Administration.

```json
{
  "mcpServers": {
    "atlassian-rovo": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/mcp-remote",
        "https://mcp.atlassian.com/v1/mcp"
      ],
      "env": {
        "MCP_REMOTE_AUTH_TYPE": "apitoken",
        "ATLASSIAN_EMAIL": "${ATLASSIAN_EMAIL}",
        "ATLASSIAN_API_TOKEN": "${ATLASSIAN_API_TOKEN}",
        "ATLASSIAN_CLOUD_ID": "${ATLASSIAN_CLOUD_ID}"
      }
    }
  }
}
```

Required environment variables:
- `ATLASSIAN_EMAIL`: Your Atlassian account email
- `ATLASSIAN_API_TOKEN`: Rovo MCP scoped API token (from Atlassian account settings)
- `ATLASSIAN_CLOUD_ID`: Your site URL (e.g., `https://yoursite.atlassian.net`)

### For Web Clients (ChatGPT, Gemini CLI, GitHub Copilot)

Follow client-specific documentation to add the remote MCP server:
- **Server URL**: `https://mcp.atlassian.com/v1/mcp`
- **Auth Type**: OAuth 2.1 (browser flow)

Example for OpenAI ChatGPT:
1. Go to Settings → Integrations → MCP
2. Add server URL: `https://mcp.atlassian.com/v1/mcp`
3. Complete OAuth flow when prompted

## Optimizing Performance with AGENTS.md

Reduce token usage and tool calls by setting defaults in your project's `AGENTS.md`:

```markdown
## Atlassian Rovo MCP

When connected to atlassian-rovo-mcp:
- **MUST** use Jira project key = MYPROJ
- **MUST** use Confluence spaceId = "123456"
- **MUST** use cloudId = "https://mysite.atlassian.net" (do NOT call getAccessibleAtlassianResources)
- **MUST** use `maxResults: 10` or `limit: 10` for ALL Jira JQL and Confluence CQL search operations.
```

Replace with your actual values:
- `MYPROJ`: Your default Jira project key
- `123456`: Your Confluence space ID (get from space settings)
- `https://mysite.atlassian.net`: Your Atlassian Cloud site URL

## Common Usage Patterns

### Jira Operations

#### Search Issues

```
"Find all open bugs in project MYPROJ assigned to me"
```

The agent will use the `jira_searchForIssuesUsingJql` tool with JQL:
```jql
project = MYPROJ AND status = Open AND assignee = currentUser() AND type = Bug
```

#### Create Issue

```
"Create a story in MYPROJ titled 'Add dark mode support' with description 'Users want dark theme option'"
```

The agent calls `jira_createIssue` with payload:
```javascript
{
  "cloudId": "https://mysite.atlassian.net",
  "fields": {
    "project": { "key": "MYPROJ" },
    "summary": "Add dark mode support",
    "description": "Users want dark theme option",
    "issuetype": { "name": "Story" }
  }
}
```

#### Bulk Create from Notes

```
"Create Jira tickets from these requirements:
1. User authentication via OAuth
2. Password reset flow
3. Email verification"
```

Agent creates 3 issues sequentially using `jira_createIssue`.

#### Update Issue

```
"Update MYPROJ-123 to set status to In Progress and add comment 'Working on this now'"
```

Uses `jira_editIssue` and `jira_addComment`.

### Confluence Operations

#### Search Pages

```
"Find all Confluence pages about API documentation in the DEV space"
```

Uses `confluence_searchByCQL` with CQL:
```cql
space = DEV AND text ~ "API documentation"
```

#### Create Page

```
"Create a Confluence page in space DEV titled 'API Integration Guide' with content:
# Authentication
Use OAuth 2.0 for all API requests."
```

Calls `confluence_createPage`:
```javascript
{
  "cloudId": "https://mysite.atlassian.net",
  "spaceId": "123456",
  "status": "current",
  "title": "API Integration Guide",
  "body": {
    "representation": "storage",
    "value": "<h1>Authentication</h1><p>Use OAuth 2.0 for all API requests.</p>"
  }
}
```

#### Summarize Page

```
"Summarize the Q2 Planning page in DEV space"
```

Agent fetches page content and provides summary.

### Compass Operations

#### Create Component

```
"Create a Compass component called 'api-gateway' of type SERVICE with description 'Main API gateway service'"
```

Uses `compass_createComponent`:
```javascript
{
  "cloudId": "https://mysite.atlassian.net",
  "name": "api-gateway",
  "typeId": "SERVICE",
  "description": "Main API gateway service"
}
```

#### Query Dependencies

```
"What services depend on the api-gateway component?"
```

Uses `compass_searchComponents` to find dependencies.

#### Bulk Import

```
"Import these components from JSON:
[
  {\"name\": \"auth-service\", \"typeId\": \"SERVICE\"},
  {\"name\": \"user-db\", \"typeId\": \"DATABASE\"}
]"
```

### Cross-Product Workflows

#### Link Jira to Confluence

```
"Link Jira issues MYPROJ-100, MYPROJ-101, MYPROJ-102 to the 'Sprint Planning' Confluence page"
```

Agent retrieves page, updates content with Jira macros or links.

#### Create Issues from Confluence

```
"Read the 'Feature Requests' Confluence page and create a Jira ticket for each item"
```

Agent:
1. Fetches Confluence page content
2. Parses items
3. Creates Jira issues using `jira_createIssue`

## Advanced Configuration

### Custom Scopes

If you need specific Atlassian API scopes, configure during OAuth:
- `read:jira-work`: Read Jira data
- `write:jira-work`: Create/update Jira issues
- `read:confluence-content.all`: Read Confluence pages
- `write:confluence-content`: Create/update Confluence pages
- `read:compass:*`: Read Compass data
- `write:compass:*`: Write Compass data

The MCP server requests appropriate scopes automatically based on available tools.

### IP Allowlisting

If your organization uses IP allowlisting:
1. Ensure your current IP is allowed in Atlassian Administration
2. Requests through MCP server must originate from allowed IPs
3. Configure VPN if working remotely

### Multiple Sites

To work with multiple Atlassian sites, configure separate MCP server entries:

```json
{
  "mcpServers": {
    "atlassian-prod": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/mcp-remote", "https://mcp.atlassian.com/v1/mcp"],
      "env": {
        "MCP_REMOTE_AUTH_TYPE": "oauth",
        "ATLASSIAN_CLOUD_ID": "https://prod.atlassian.net"
      }
    },
    "atlassian-staging": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/mcp-remote", "https://mcp.atlassian.com/v1/mcp"],
      "env": {
        "MCP_REMOTE_AUTH_TYPE": "oauth",
        "ATLASSIAN_CLOUD_ID": "https://staging.atlassian.net"
      }
    }
  }
}
```

## Available Tools (Partial List)

The server exposes 50+ tools. Key examples:

### Jira
- `jira_searchForIssuesUsingJql`
- `jira_createIssue`
- `jira_editIssue`
- `jira_addComment`
- `jira_getIssue`
- `jira_deleteIssue`
- `jira_assignIssue`

### Confluence
- `confluence_searchByCQL`
- `confluence_createPage`
- `confluence_updatePage`
- `confluence_getPage`
- `confluence_deletePage`
- `confluence_getSpaces`

### Compass
- `compass_createComponent`
- `compass_searchComponents`
- `compass_getComponent`
- `compass_updateComponent`
- `compass_deleteComponent`

### Utility
- `getAccessibleAtlassianResources` (list available sites - avoid if using AGENTS.md defaults)

## Troubleshooting

### "Your site admin must authorize this app"

**Cause**: First user to connect must be a site admin.

**Solution**: 
1. Have a site admin complete OAuth flow first
2. Once installed, regular users can connect

### "You don't have permission to connect from this IP address"

**Cause**: IP allowlisting is enabled and your IP isn't allowed.

**Solution**:
1. Check Atlassian Administration → Security → IP allowlist
2. Add your IP range or VPN IP
3. Contact your admin if you can't modify settings

### OAuth flow doesn't open browser

**Cause**: Headless environment or browser not configured.

**Solution**:
1. Switch to API token authentication (see headless setup above)
2. Or ensure `BROWSER` environment variable points to valid browser

### "App not appearing in Connected apps"

**Cause**: Wrong account, wrong site, or permissions issue.

**Solution**:
1. Verify you're logged into correct Atlassian account
2. Check site URL matches `cloudId` in config
3. Ensure you have access to Jira/Confluence/Compass on that site
4. Try revoking and re-authorizing from Atlassian account settings

### High token usage / slow responses

**Cause**: Agent calling `getAccessibleAtlassianResources` repeatedly or searching without limits.

**Solution**:
1. Add AGENTS.md configuration with `cloudId`, project, and space defaults
2. Always specify `maxResults` or `limit` in search queries
3. Cache site/project/space IDs in conversation context

### Rate limiting errors

**Cause**: Too many API requests in short time.

**Solution**:
1. Batch operations where possible
2. Add delays between bulk creates
3. Use search with pagination instead of fetching all results

## Skills for Claude Desktop

Pre-built skills available in `skills/` directory:

- **create-jira-issue.md**: Create Jira issues from natural language
- **search-confluence.md**: Search and summarize Confluence pages
- **link-content.md**: Link Jira issues to Confluence pages

To use:
1. Copy skill file to `~/Library/Application Support/Claude/skills/`
2. Restart Claude Desktop
3. Reference skill: "Use the create-jira-issue skill to make a new bug"

## Admin Considerations

### First-Time Setup (Admin)

1. First user must have access to all Atlassian products being integrated (Jira, Confluence, Compass)
2. Complete OAuth flow to install app (lazy/JIT installation)
3. App appears in Atlassian Administration → Apps → Connected apps

### Enabling API Token Auth (Admin)

1. Go to Atlassian Administration → Security → Rovo MCP Server
2. Enable "API token authentication"
3. Users can then create scoped API tokens from account settings

### Monitoring Usage

- View audit logs in Atlassian Administration → Audit log
- Filter by "Rovo MCP Server" to see all actions
- Logs include user, timestamp, action, IP address

### Revoking Access

**Organization-wide**:
1. Atlassian Administration → Apps → Connected apps
2. Find "Atlassian Rovo MCP Server"
3. Click "Revoke" or "Uninstall"

**Per-user**:
1. User profile → Account settings → Security
2. Connected apps → Revoke "Atlassian Rovo MCP Server"

## Security Best Practices

1. **Never commit API tokens**: Use environment variables (`${ATLASSIAN_API_TOKEN}`)
2. **Use scoped tokens**: Request only needed permissions
3. **Rotate tokens regularly**: Refresh API tokens every 90 days
4. **Monitor audit logs**: Review MCP actions weekly
5. **Enable IP allowlisting**: Restrict access to known networks
6. **Educate users**: Train on what data agents can access
7. **Revoke unused access**: Remove old OAuth authorizations

## Resources

- [Official Documentation](https://support.atlassian.com/security-and-access-policies/docs/understand-atlassian-rovo-mcp-server/)
- [Admin Controls](https://support.atlassian.com/security-and-access-policies/docs/control-atlassian-rovo-mcp-server-settings/)
- [Monitoring Guide](https://support.atlassian.com/security-and-access-policies/docs/monitor-atlassian-rovo-mcp-server-activity/)
- [API Token Setup](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/configuring-authentication-via-api-token/)
- [Supported Domains](https://support.atlassian.com/security-and-access-policies/docs/available-atlassian-rovo-mcp-server-domains/)
