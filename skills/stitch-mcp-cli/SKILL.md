---
name: stitch-mcp-cli
description: CLI for moving AI-generated UI designs from Google's Stitch platform into development workflows with local preview, site generation, and agent integration.
triggers:
  - "preview my Stitch designs locally"
  - "build an Astro site from Stitch screens"
  - "set up Stitch MCP integration"
  - "serve Stitch project screens"
  - "connect Stitch designs to my coding agent"
  - "export Stitch UI to my project"
  - "browse Stitch designs in terminal"
  - "authenticate with Stitch API"
---

# stitch-mcp CLI

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`stitch-mcp` is a CLI tool that bridges Google's Stitch AI-generated UI design platform with local development workflows. It enables you to:

- Preview all screens from a Stitch project on a local dev server
- Generate deployable Astro sites by mapping screens to routes
- Proxy Stitch MCP tools to coding agents with automatic token refresh
- Browse and explore design data interactively in the terminal
- Invoke MCP tools directly from the command line

The tool handles authentication, fetching HTML/CSS designs via the Stitch API, and structuring them for development or agent consumption.

## Installation

**Run directly with npx (no installation needed):**

```bash
npx @_davideast/stitch-mcp <command>
```

**Or install globally:**

```bash
npm install -g @_davideast/stitch-mcp
```

## Quick Start

### 1. Initialize Authentication

The `init` command sets up gcloud SDK, OAuth, and MCP client configuration:

```bash
npx @_davideast/stitch-mcp init
```

This wizard will:
- Install an isolated gcloud SDK (if needed)
- Guide you through OAuth authentication
- Enable the Stitch API on your project
- Generate MCP client configuration

### 2. Verify Setup

```bash
npx @_davideast/stitch-mcp doctor
```

Use `--verbose` flag for detailed diagnostics:

```bash
npx @_davideast/stitch-mcp doctor --verbose
```

### 3. Preview Designs Locally

Serve all screens from a project on a Vite dev server:

```bash
npx @_davideast/stitch-mcp serve -p <project-id>
```

This starts a local server (default: http://localhost:5173) with all project screens accessible.

### 4. Build an Astro Site

Generate a deployable Astro project from Stitch screens:

```bash
npx @_davideast/stitch-mcp site -p <project-id>
```

The command will:
- Prompt you to map screens to routes (e.g., "/" for home, "/about" for about page)
- Fetch HTML/CSS for each screen
- Generate a complete Astro project structure
- Output to `./stitch-site` directory by default

**Specify output directory:**

```bash
npx @_davideast/stitch-mcp site -p <project-id> -o ./my-site
```

## Authentication Methods

### Method 1: Automatic OAuth (Recommended)

```bash
npx @_davideast/stitch-mcp init
```

Follow the wizard. Your credentials are stored in `~/.stitch-mcp/`.

### Method 2: API Key

Set the API key as an environment variable:

```bash
export STITCH_API_KEY="your-api-key"
```

This skips OAuth entirely.

### Method 3: System gcloud

If you have gcloud CLI already configured:

```bash
gcloud auth application-default login
gcloud config set project <PROJECT_ID>
gcloud beta services mcp enable stitch.googleapis.com --project=<PROJECT_ID>
```

Then use with `STITCH_USE_SYSTEM_GCLOUD=1`:

```bash
STITCH_USE_SYSTEM_GCLOUD=1 npx @_davideast/stitch-mcp serve -p <project-id>
```

### Logout

```bash
npx @_davideast/stitch-mcp logout
```

Force logout and clear all config:

```bash
npx @_davideast/stitch-mcp logout --force --clear-config
```

## MCP Integration

### Configure Your Agent

Add this to your MCP client config (e.g., `claude_desktop_config.json`, `.cursor/config.json`):

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"]
    }
  }
}
```

**Using API key:**

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"],
      "env": {
        "STITCH_API_KEY": "${STITCH_API_KEY}"
      }
    }
  }
}
```

**Using system gcloud:**

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"],
      "env": {
        "STITCH_USE_SYSTEM_GCLOUD": "1"
      }
    }
  }
}
```

### Virtual Tools for Agents

The proxy exposes high-level tools that combine multiple API calls:

#### `build_site`

Builds a site from a project by mapping screens to routes. Returns HTML for each page.

**Input schema:**

```typescript
{
  projectId: string;      // Required
  routes: Array<{
    screenId: string;     // Required
    route: string;        // Required, e.g. "/" or "/about"
  }>;
}
```

**Example usage from CLI:**

```bash
npx @_davideast/stitch-mcp tool build_site -d '{
  "projectId": "123456",
  "routes": [
    { "screenId": "abc123", "route": "/" },
    { "screenId": "def456", "route": "/about" },
    { "screenId": "ghi789", "route": "/contact" }
  ]
}'
```

**Prompt for agent:**

```
Use the build_site tool to create a website from my Stitch project 123456.
Map screen abc123 to the home page, def456 to /about, and ghi789 to /contact.
```

#### `get_screen_code`

Retrieves a screen and downloads its HTML code content.

```bash
npx @_davideast/stitch-mcp tool get_screen_code -d '{
  "projectId": "123456",
  "screenId": "abc123"
}'
```

#### `get_screen_image`

Retrieves a screen and downloads its screenshot image as base64.

```bash
npx @_davideast/stitch-mcp tool get_screen_image -d '{
  "projectId": "123456",
  "screenId": "abc123"
}'
```

## Key Commands

### Exploration & Browsing

#### View Projects

```bash
npx @_davideast/stitch-mcp view --projects
```

Opens an interactive browser listing all your Stitch projects.

#### View Specific Project

```bash
npx @_davideast/stitch-mcp view --project <project-id>
```

Shows all screens in the project.

#### View Specific Screen

```bash
npx @_davideast/stitch-mcp view --project <project-id> --screen <screen-id>
```

**Interactive browser controls:**
- Arrow keys: navigate
- Enter: drill into nested data
- `c`: copy selected value to clipboard
- `s`: preview HTML in browser
- `o`: open project in Stitch
- `q`: quit

#### Browse Screens in Terminal

```bash
npx @_davideast/stitch-mcp screens -p <project-id>
```

Displays an interactive list of all screens with preview and copy actions.

### Tool Invocation

#### List All Available Tools

```bash
npx @_davideast/stitch-mcp tool
```

#### Show Tool Schema

```bash
npx @_davideast/stitch-mcp tool <tool-name> -s
```

Example:

```bash
npx @_davideast/stitch-mcp tool build_site -s
```

#### Invoke a Tool

```bash
npx @_davideast/stitch-mcp tool <tool-name> -d '<json-data>'
```

Example:

```bash
npx @_davideast/stitch-mcp tool get_screen_code -d '{
  "projectId": "123456",
  "screenId": "abc123"
}'
```

### Development Workflow

#### Serve Project Locally

```bash
npx @_davideast/stitch-mcp serve -p <project-id>
```

**Custom port:**

```bash
npx @_davideast/stitch-mcp serve -p <project-id> --port 3000
```

#### Generate Astro Site

```bash
npx @_davideast/stitch-mcp site -p <project-id>
```

**Non-interactive mode with config file:**

Create `routes.json`:

```json
{
  "routes": [
    { "screenId": "abc123", "route": "/" },
    { "screenId": "def456", "route": "/about" }
  ]
}
```

Then run:

```bash
npx @_davideast/stitch-mcp site -p <project-id> --config routes.json -o ./output
```

#### Save Screen Snapshot

```bash
npx @_davideast/stitch-mcp snapshot -p <project-id> -s <screen-id> -o ./snapshot.html
```

### MCP Proxy

Start the proxy server for agent integration:

```bash
npx @_davideast/stitch-mcp proxy
```

**Debug mode (logs to `/tmp/stitch-proxy-debug.log`):**

```bash
npx @_davideast/stitch-mcp proxy --debug
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `STITCH_API_KEY` | API key for direct authentication (skips OAuth) |
| `STITCH_ACCESS_TOKEN` | Pre-existing access token |
| `STITCH_USE_SYSTEM_GCLOUD` | Use system gcloud config instead of isolated config |
| `STITCH_PROJECT_ID` | Override project ID |
| `GOOGLE_CLOUD_PROJECT` | Alternative project ID variable |
| `STITCH_HOST` | Custom Stitch API endpoint |

## Common Patterns

### Pattern 1: Preview All Designs Before Building

```bash
# 1. Browse available screens
npx @_davideast/stitch-mcp screens -p <project-id>

# 2. Serve locally to review designs
npx @_davideast/stitch-mcp serve -p <project-id>

# 3. Build site after confirming screens
npx @_davideast/stitch-mcp site -p <project-id>
```

### Pattern 2: Agent-Driven Site Generation

**User prompt to agent:**

```
I have a Stitch project with ID 123456. It has three screens:
- Home page (screen ID: abc123)
- About page (screen ID: def456)
- Contact page (screen ID: ghi789)

Use the build_site tool to generate HTML for all three pages with routes /, /about, and /contact.
Then help me integrate them into my Next.js project.
```

The agent will use `build_site` and can help integrate the resulting HTML.

### Pattern 3: Iterative Design Review

```bash
# View specific screen details
npx @_davideast/stitch-mcp view --project <project-id> --screen <screen-id>

# Preview in browser (press 's' in interactive view)
# Or manually:
npx @_davideast/stitch-mcp snapshot -p <project-id> -s <screen-id> -o ./preview.html
open ./preview.html

# Get the HTML code for integration
npx @_davideast/stitch-mcp tool get_screen_code -d '{
  "projectId": "<project-id>",
  "screenId": "<screen-id>"
}'
```

### Pattern 4: Automated Site Builds in CI

```bash
#!/bin/bash
# ci-build-stitch.sh

export STITCH_API_KEY="${STITCH_API_KEY}"
PROJECT_ID="123456"

# Generate site with predefined routes
npx @_davideast/stitch-mcp site -p "$PROJECT_ID" \
  --config ./stitch-routes.json \
  -o ./dist/stitch-site \
  --non-interactive

# Deploy the generated site
cd ./dist/stitch-site
npm install
npm run build
```

`stitch-routes.json`:

```json
{
  "routes": [
    { "screenId": "home_abc", "route": "/" },
    { "screenId": "about_def", "route": "/about" },
    { "screenId": "contact_ghi", "route": "/contact" }
  ]
}
```

## Troubleshooting

### "Permission Denied" Errors

**Symptoms:** API calls fail with 403 or permission errors.

**Solutions:**

1. Verify your GCP role (need Owner or Editor):
   ```bash
   gcloud projects get-iam-policy <PROJECT_ID> \
     --flatten="bindings[].members" \
     --filter="bindings.members:user:your-email@example.com"
   ```

2. Ensure billing is enabled:
   ```bash
   gcloud billing projects describe <PROJECT_ID>
   ```

3. Enable Stitch API:
   ```bash
   gcloud beta services mcp enable stitch.googleapis.com --project=<PROJECT_ID>
   ```

4. Run diagnostics:
   ```bash
   npx @_davideast/stitch-mcp doctor --verbose
   ```

### Authentication URL Not Appearing

**Symptoms:** No browser opens or URL not printed during `init`.

**Solutions:**

1. Look for URL in terminal output (5-second timeout)
2. Check for URLs starting with `https://accounts.google.com`
3. In proxy debug mode, check `/tmp/stitch-proxy-debug.log`
4. In remote environments (WSL/SSH/Docker), copy the URL manually:
   ```bash
   npx @_davideast/stitch-mcp init
   # Copy the URL from terminal output and open in browser
   ```

### "Already Authenticated" Issues

**Symptoms:** Shows as logged in but commands fail.

**Cause:** The bundled gcloud SDK has separate auth from system gcloud.

**Solution:**

```bash
npx @_davideast/stitch-mcp logout --force --clear-config
npx @_davideast/stitch-mcp init
```

### API Connection Fails After Setup

**Solutions:**

1. Run full diagnostics:
   ```bash
   npx @_davideast/stitch-mcp doctor --verbose
   ```

2. Re-authenticate:
   ```bash
   npx @_davideast/stitch-mcp logout --force
   npx @_davideast/stitch-mcp init
   ```

3. Verify project and billing:
   ```bash
   gcloud config get-value project
   gcloud billing projects describe $(gcloud config get-value project)
   ```

### WSL / SSH / Docker Environments

**Symptoms:** Browser-based auth doesn't work automatically.

**Solution:** The CLI detects these environments. Copy the OAuth URL from terminal output and open it in a browser manually. The URL will be printed to stdout with clear instructions.

### MCP Proxy Not Responding

**Symptoms:** Agent can't connect to Stitch tools.

**Solutions:**

1. Run proxy with debug logging:
   ```bash
   npx @_davideast/stitch-mcp proxy --debug
   ```

2. Check logs:
   ```bash
   tail -f /tmp/stitch-proxy-debug.log
   ```

3. Verify MCP client config syntax:
   ```bash
   cat ~/.config/claude/claude_desktop_config.json | jq .
   ```

4. Restart your MCP client (VS Code, Cursor, etc.)

### Tool Invocation Failures

**Symptoms:** `tool` command returns errors or empty responses.

**Solutions:**

1. Check tool schema:
   ```bash
   npx @_davideast/stitch-mcp tool <tool-name> -s
   ```

2. Validate JSON data:
   ```bash
   echo '{"projectId":"123"}' | jq .
   ```

3. Test with simpler tool first:
   ```bash
   npx @_davideast/stitch-mcp tool list_projects
   ```

4. Ensure project and screen IDs are correct:
   ```bash
   npx @_davideast/stitch-mcp view --projects
   ```

## Development & Testing

```bash
# Clone and install
git clone https://github.com/davideast/stitch-mcp.git
cd stitch-mcp
bun install

# Run locally
bun run dev init

# Run specific command
bun run dev serve -p <project-id>

# Run tests
bun test

# Build
bun run build

# Verify package
bun run verify-pack
```

## Real-World Agent Prompts

### Example 1: Site Generation

```
I have a Stitch project (ID: 456789) with these screens:
- Landing page: screen_landing_001
- Features page: screen_features_002
- Pricing page: screen_pricing_003

Please use the build_site tool to generate a site with:
- Landing at /
- Features at /features
- Pricing at /pricing

Then show me how to integrate the HTML into my existing React project.
```

### Example 2: Design Review

```
Get the HTML code for screen abc123 from project 456789 using get_screen_code.
Then analyze the CSS and suggest improvements for mobile responsiveness.
```

### Example 3: Batch Export

```
For project 456789, use get_screen_image to export all screen screenshots as base64.
Save them to a design-assets/ directory for our design system documentation.
```

## License

Apache 2.0 © David East

**Disclaimer:** This is an experimental, independent project not affiliated with Google LLC or the Stitch team. Use at your own risk.
