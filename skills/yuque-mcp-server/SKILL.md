---
name: yuque-mcp-server
description: MCP server for Yuque (语雀) knowledge base - search, create, and manage documents through AI assistants
triggers:
  - how do I connect to Yuque with MCP
  - search my Yuque knowledge base
  - create a document in Yuque
  - manage Yuque books and docs
  - set up Yuque MCP server
  - integrate Yuque with AI assistant
  - work with Yuque API through MCP
  - troubleshoot Yuque MCP connection
---

# Yuque MCP Server Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The Yuque MCP Server is a Model Context Protocol server that exposes the Yuque (语雀) knowledge base API to AI assistants. It allows you to search documents, create and update content, manage knowledge bases (books), and access resources through 19 different tools. Yuque is a collaborative documentation platform popular in China.

**Key capabilities:**
- Search across your Yuque knowledge base
- Create, read, and update documents
- Manage books (knowledge base collections)
- Handle resources (attachments, images)
- Manage table of contents and notes
- Support for both personal and team tokens
- Compatible with private Yuque deployments

## Installation

### Quick Install (Recommended)

Use the built-in CLI to auto-configure your MCP client:

```bash
# Interactive setup wizard
npx yuque-mcp setup

# Direct install with token
npx yuque-mcp install --token=$YUQUE_TOKEN --client=cursor

# With custom host for team/private deployment
npx yuque-mcp install --token=$YUQUE_TOKEN --client=cursor --host=https://your-space.yuque.com
```

Supported clients: `claude-desktop`, `vscode`, `cursor`, `windsurf`, `cline`, `trae`, `qoder`, `opencode`

### Manual Configuration

#### Claude Code
```bash
export YUQUE_TOKEN=your_token_here
claude mcp add yuque-mcp -- npx -y yuque-mcp
```

#### Claude Desktop / Cursor / Windsurf

Add to config file (`claude_desktop_config.json` or `mcp.json`):

```json
{
  "mcpServers": {
    "yuque": {
      "command": "npx",
      "args": ["-y", "yuque-mcp"],
      "env": {
        "YUQUE_TOKEN": "your_token_here"
      }
    }
  }
}
```

#### VS Code (GitHub Copilot)

Create `.vscode/mcp.json` in your workspace:

```json
{
  "servers": {
    "yuque": {
      "command": "npx",
      "args": ["-y", "yuque-mcp"],
      "env": {
        "YUQUE_TOKEN": "your_token_here"
      }
    }
  }
}
```

## Authentication

### Getting Your Token

1. Visit [Yuque Developer Settings](https://www.yuque.com/settings/tokens)
2. Create a personal access token
3. Store it securely as `YUQUE_TOKEN` environment variable

### Configuration Options

| Variable | Description |
|----------|-------------|
| `YUQUE_TOKEN` | Your Yuque API token (required) |
| `YUQUE_HOST` | Custom host for team tokens or private deployments |

**Priority order:**
- Token: `YUQUE_TOKEN` > `YUQUE_PERSONAL_TOKEN` > `--token`
- Host: `YUQUE_HOST` > `--host` > `YUQUE_BASE_URL` > `--base-url`

### Team Tokens and Private Deployment

```bash
# Environment variables
export YUQUE_TOKEN=your_token_here
export YUQUE_HOST=https://your-space.yuque.com

# CLI arguments
npx yuque-mcp --token=$YUQUE_TOKEN --host=https://your-space.yuque.com
```

## Available Tools (19 Total)

### User Tools

**`yuque_get_user`** - Get current user information

### Search Tools

**`yuque_search`** - Search across all documents
- Parameters: `q` (query string), `type` (optional: "doc", "book", "user")
- Returns matching documents with snippets

### Book Management

**`yuque_list_books`** - List all knowledge bases
- Parameters: `user` (optional: username or ID), `type` (optional: "Book", "Design")

**`yuque_get_book`** - Get book details
- Parameters: `namespace` (book identifier)

**`yuque_create_book`** - Create new knowledge base
- Parameters: `name`, `slug`, `description` (optional), `public` (optional)

**`yuque_update_book`** - Update book properties
- Parameters: `namespace`, `name`, `slug`, `description`, `public`

### Document Management

**`yuque_list_docs`** - List documents in a book
- Parameters: `namespace` (book identifier)

**`yuque_get_doc`** - Get document content
- Parameters: `namespace` (book), `slug` (doc identifier)

**`yuque_create_doc`** - Create new document
- Parameters: `namespace`, `title`, `slug` (optional), `body` (markdown/html), `format` (optional: "markdown", "lake", "html")

**`yuque_update_doc`** - Update document
- Parameters: `namespace`, `id`, `title`, `slug`, `body`, `format`

### Resource Management

**`yuque_get_resource`** - Get resource (attachment) details
- Parameters: `namespace`, `id`

**`yuque_create_resource`** - Upload resource
- Parameters: `namespace`, `file` (base64 or URL)

**`yuque_update_resource`** - Update resource metadata
- Parameters: `namespace`, `id`, `title`, `description`

### Table of Contents

**`yuque_get_toc`** - Get book TOC structure
- Parameters: `namespace`

**`yuque_update_toc`** - Update TOC structure
- Parameters: `namespace`, `toc` (JSON structure)

### Notes Management

**`yuque_list_notes`** - List personal notes

**`yuque_get_note`** - Get note content
- Parameters: `id`

**`yuque_create_note`** - Create personal note
- Parameters: `title`, `body`, `format`

**`yuque_update_note`** - Update note
- Parameters: `id`, `title`, `body`, `format`

## Common Usage Patterns

### Search Documents

Ask your AI assistant:
```
Search my Yuque docs for "API documentation"
```

The assistant will use `yuque_search` with your query.

### Create a Document

```
Create a new document in my "engineering-notes" book titled "Deploy Process" with markdown content explaining our deployment workflow
```

The assistant will:
1. Find the book using `yuque_list_books`
2. Create the doc using `yuque_create_doc`

### Update Existing Document

```
Update the "API Guide" document in my "docs" book to add a new section about authentication
```

The assistant will:
1. Find the document using `yuque_get_doc`
2. Update it using `yuque_update_doc`

### Organize Knowledge Base

```
Show me all my Yuque books and their document counts
```

The assistant will use `yuque_list_books` and `yuque_list_docs` to gather information.

## Development

### Local Setup

```bash
git clone https://github.com/yuque/yuque-mcp-server.git
cd yuque-mcp-server
npm install
```

### Build and Test

```bash
# Run tests
npm test

# Build TypeScript
npm run build

# Development mode with hot reload
npm run dev
```

### Testing with MCP Inspector

```bash
# Set token
export YUQUE_TOKEN=your_token_here

# Run with inspector
npx @modelcontextprotocol/inspector npx yuque-mcp
```

## Troubleshooting

### Authentication Errors

**Error:** `YUQUE_TOKEN ... is required`
```bash
# Solution: Set environment variable
export YUQUE_TOKEN=your_token_here

# Or pass as argument
npx yuque-mcp --token=$YUQUE_TOKEN
```

**Error:** `401 Unauthorized`
- Token is invalid or expired
- Regenerate at https://www.yuque.com/settings/tokens
- Verify token has correct permissions

### Rate Limiting

**Error:** `429 Rate Limited`
- Too many requests in short time
- Wait a moment before retrying
- Consider batching requests

### Resource Not Found

**Error:** `410 Gone`
- Resource has been permanently deleted
- API endpoint may be deprecated
- Verify the target document/book still exists in Yuque web interface

### Connection Issues

**Problem:** Server not responding
```bash
# Check if server is running
ps aux | grep yuque-mcp

# Restart your MCP client
# For Claude Desktop: Quit and reopen
# For Cursor: Reload window (Cmd/Ctrl+R)

# Test connection manually
npx yuque-mcp
```

### Version Issues

**Problem:** Tool not found
```bash
# Update to latest version
npx -y yuque-mcp@latest

# Clear npx cache if needed
rm -rf ~/.npm/_npx
```

### Configuration Issues

**Problem:** Server not loading in client

1. Check config file location:
   - Claude Desktop (macOS): `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Cursor: `~/.cursor/mcp.json`
   - Windsurf: `~/.windsurf/mcp.json`

2. Verify JSON syntax:
```bash
# Use jq to validate
cat ~/.cursor/mcp.json | jq .
```

3. Check logs:
   - Claude Desktop: View in app menu → "View Logs"
   - Cursor: Open Developer Tools (Cmd+Shift+P → "Toggle Developer Tools")

### Node.js Not Found

**Error:** `npx command not found`
```bash
# Install Node.js (v18 or later)
# macOS
brew install node

# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Windows: Download from https://nodejs.org/
```

## Code Examples

### TypeScript Integration

If you're building a custom MCP client or extending the server:

```typescript
import { MCPClient } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const transport = new StdioClientTransport({
  command: 'npx',
  args: ['-y', 'yuque-mcp'],
  env: {
    YUQUE_TOKEN: process.env.YUQUE_TOKEN
  }
});

const client = new MCPClient({
  name: 'my-yuque-client',
  version: '1.0.0'
});

await client.connect(transport);

// Call a tool
const result = await client.callTool({
  name: 'yuque_search',
  arguments: {
    q: 'API documentation'
  }
});

console.log(result);
```

### Custom Tool Wrapper

```typescript
interface YuqueConfig {
  token: string;
  host?: string;
}

class YuqueHelper {
  private config: YuqueConfig;
  
  constructor(config: YuqueConfig) {
    this.config = config;
  }
  
  async searchDocs(query: string, type?: string) {
    // Use MCP client to call yuque_search
    return await client.callTool({
      name: 'yuque_search',
      arguments: { q: query, type }
    });
  }
  
  async createDoc(namespace: string, title: string, body: string, format = 'markdown') {
    return await client.callTool({
      name: 'yuque_create_doc',
      arguments: { namespace, title, body, format }
    });
  }
}
```

## Additional Resources

- [Yuque API Documentation](https://www.yuque.com/yuque/developer/api)
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [GitHub Repository](https://github.com/yuque/yuque-mcp-server)
- [Project Website](https://yuque.github.io/yuque-ecosystem/)
