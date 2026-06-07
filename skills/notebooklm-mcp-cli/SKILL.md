```markdown
---
name: notebooklm-mcp-cli
description: Programmatic access to Google NotebookLM via CLI and MCP server for AI-powered research workflows
triggers:
  - create a notebooklm notebook
  - add sources to notebooklm
  - generate a podcast from my research
  - query my notebooklm notebook
  - download notebooklm audio
  - share my notebooklm notebook
  - setup notebooklm mcp server
  - authenticate with notebooklm
---

# notebooklm-mcp-cli

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`notebooklm-mcp-cli` provides programmatic access to Google NotebookLM through both a command-line interface (`nlm`) and an MCP server (`notebooklm-mcp`). It enables automation of research workflows, podcast generation, source management, and AI-powered notebook queries.

**Key capabilities:**
- Create and manage NotebookLM notebooks
- Add sources from URLs, files, Google Drive, or text
- Generate studio content (podcasts, videos, slide decks, infographics)
- Query notebooks with AI (responses persist to web UI)
- Download generated artifacts
- Share notebooks publicly or via invite
- Batch operations and cross-notebook queries
- Multi-step pipeline workflows

**Important:** Uses internal NotebookLM APIs that may change without notice. Requires cookie-based authentication.

## Installation

### Quick Install

```bash
# Recommended: uv (fastest)
uv tool install notebooklm-mcp-cli

# Alternative: pip
pip install notebooklm-mcp-cli

# Alternative: pipx (isolated environment)
pipx install notebooklm-mcp-cli
```

This installs both:
- `nlm` — CLI for terminal use
- `notebooklm-mcp` — MCP server for AI agents

### Verify Installation

```bash
nlm --version
nlm --help
```

### Upgrading

```bash
# Using uv
uv tool upgrade notebooklm-mcp-cli

# Using pip
pip install --upgrade notebooklm-mcp-cli
```

After upgrading, restart your AI tool to reconnect to the updated MCP server.

## Authentication

### Initial Setup

```bash
# Auto mode (recommended): launches browser, extracts cookies automatically
nlm login

# Check authentication status
nlm login --check

# Manual mode: import cookies from file
nlm login --manual --file cookies.txt
```

**How auto mode works:**
1. Launches a dedicated browser profile (Chrome, Arc, Brave, Edge, etc.)
2. You log in to Google
3. Cookies are extracted and stored in `~/.notebooklm-mcp-cli/`
4. Browser profile persists for future auth refreshes

### Multi-Account Support

```bash
# Login with named profiles
nlm login --profile work
nlm login --profile personal

# Switch default profile
nlm login switch work

# List all profiles
nlm login profile list

# Delete a profile
nlm login profile delete old-account
```

### Preferred Browser

```bash
# Set preferred browser
nlm config set auth.browser brave

# Supported: chrome, arc, brave, edge, chromium
# Falls back to auto-detection if not found
```

### Troubleshooting Auth

```bash
# Diagnose auth issues
nlm doctor

# Force re-authentication
nlm login --force
```

## CLI Usage

### Notebook Management

```bash
# List all notebooks
nlm notebook list

# List with full details (JSON)
nlm notebook list --full

# Create a new notebook
nlm notebook create "Quantum Computing Research"

# Get notebook details
nlm notebook get <notebook-id>

# Delete a notebook
nlm notebook delete <notebook-id>

# Rename a notebook
nlm notebook rename <notebook-id> "New Name"
```

### Source Management

```bash
# Add a URL source
nlm source add <notebook-id> --url "https://example.com/article"

# Add plain text
nlm source add <notebook-id> --text "Your research notes here"

# Add a local file
nlm source add <notebook-id> --file ./document.pdf

# Add Google Drive file
nlm source add <notebook-id> --drive "1ABC...XYZ"

# Add multiple sources at once
nlm source add <notebook-id> \
  --url "https://site1.com" \
  --url "https://site2.com" \
  --file ./doc.pdf

# List sources in a notebook
nlm source list <notebook-id>

# Delete a source
nlm source delete <notebook-id> <source-id>

# Sync Google Drive sources (refresh content)
nlm source sync <notebook-id> <source-id>
```

### Querying Notebooks

```bash
# Query a notebook (persists to web UI)
nlm notebook query <notebook-id> "What are the key findings?"

# Query without saving to web UI
nlm notebook query <notebook-id> "Quick question?" --ephemeral

# Query across multiple notebooks
nlm cross query "Compare findings" <notebook-id-1> <notebook-id-2>

# Batch query multiple notebooks
nlm batch query "Same question" <id-1> <id-2> <id-3>
```

### Studio Content (Audio, Video, Slides)

```bash
# Generate podcast audio (requires confirmation)
nlm studio create <notebook-id> --type audio --confirm

# Generate slide deck
nlm studio create <notebook-id> --type slides --confirm

# Generate video with cinematic visuals
nlm studio create <notebook-id> --type video --style cinematic --confirm

# Generate infographic
nlm studio create <notebook-id> --type infographic --confirm

# Revise slide deck with instructions
nlm slides revise <notebook-id> <artifact-id> \
  "Add more charts, reduce text on slide 3"

# List studio artifacts
nlm studio list <notebook-id>
```

### Downloading Artifacts

```bash
# Download audio podcast
nlm download audio <notebook-id> <artifact-id>

# Download to specific path
nlm download audio <notebook-id> <artifact-id> --output ./podcast.wav

# Download video
nlm download video <notebook-id> <artifact-id> --output ./video.mp4

# Download slide deck (PDF)
nlm download slides <notebook-id> <artifact-id> --output ./deck.pdf

# Download infographic
nlm download infographic <notebook-id> <artifact-id> --output ./graphic.png
```

### Sharing

```bash
# Enable public link
nlm share public <notebook-id>

# Disable public sharing
nlm share public <notebook-id> --disable

# Get public link
nlm share link <notebook-id>

# Invite specific email addresses
nlm share invite <notebook-id> user@example.com collaborator@example.com
```

### Research Mode

```bash
# Start web/Drive research
nlm research start <notebook-id> "quantum entanglement applications"

# Check research status
nlm research status <notebook-id> <research-id>

# Stop ongoing research
nlm research stop <notebook-id> <research-id>

# List all research sessions
nlm research list <notebook-id>
```

### Tagging and Smart Selection

```bash
# Add tags to notebooks
nlm tag add <notebook-id> ml research priority

# List notebooks by tag
nlm tag list research

# Smart select notebooks for batch operations
nlm tag select --tags ml,priority --operation query "What's new?"
```

### Pipelines (Multi-Step Workflows)

```bash
# List available pipelines
nlm pipeline list

# Run a pipeline
nlm pipeline run research-to-podcast \
  --param topic="AI Safety" \
  --param sources="url1,url2"

# Create custom pipeline (YAML)
nlm pipeline create my-workflow.yaml
```

**Example pipeline YAML:**

```yaml
name: research-to-podcast
description: Create notebook, add sources, generate podcast
steps:
  - action: notebook_create
    params:
      title: "{{ topic }}"
    output: notebook_id
  
  - action: source_add
    params:
      notebook_id: "{{ notebook_id }}"
      urls: "{{ sources }}"
  
  - action: studio_create
    params:
      notebook_id: "{{ notebook_id }}"
      type: audio
      confirm: true
```

### Batch Operations

```bash
# Batch create notebooks
nlm batch create "Topic 1" "Topic 2" "Topic 3"

# Batch query
nlm batch query "Summarize findings" <id-1> <id-2> <id-3>

# Batch delete
nlm batch delete <id-1> <id-2> <id-3>
```

### Configuration

```bash
# View current config
nlm config show

# Set a config value
nlm config set auth.browser arc
nlm config set output.format json

# Reset to defaults
nlm config reset
```

### Diagnostics

```bash
# Run full diagnostic
nlm doctor

# Check authentication only
nlm login --check

# View MCP server logs
nlm logs
```

## MCP Server Setup

### Automatic Setup for AI Tools

```bash
# Claude Code (recommended for Claude Desktop users)
nlm setup add claude-code

# Claude Desktop
nlm setup add claude-desktop

# Gemini CLI
nlm setup add gemini

# GitHub Copilot
nlm setup add github-copilot

# Cursor IDE
nlm setup add cursor

# Windsurf
nlm setup add windsurf

# Cline
nlm setup add cline

# Generate JSON for custom tools
nlm setup add json
```

### Manual MCP Configuration

If you prefer to configure manually, add to your AI tool's MCP config file:

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "notebooklm-mcp-cli": {
      "command": "notebooklm-mcp"
    }
  }
}
```

**Cursor** (`.cursor/mcp_config.json`):

```json
{
  "mcpServers": {
    "notebooklm-mcp-cli": {
      "command": "notebooklm-mcp"
    }
  }
}
```

### Using a Specific Profile with MCP

```json
{
  "mcpServers": {
    "notebooklm-mcp-cli": {
      "command": "notebooklm-mcp",
      "args": ["--profile", "work"]
    }
  }
}
```

### Install AI Skills (Optional)

Install the NotebookLM expert guide for your AI assistant:

```bash
# Cline
nlm skill install cline

# Antigravity
nlm skill install antigravity

# Claude Code
nlm skill install claude-code

# Codex
nlm skill install codex

# Update installed skills
nlm skill update
```

### Verify MCP Setup

```bash
# List configured AI tools
nlm setup list

# Test MCP server
nlm setup test claude-code
```

After setup, restart your AI tool to activate the MCP server.

## MCP Tools Reference

The MCP server provides 35 tools. Key tools:

| Tool | Purpose |
|------|---------|
| `notebook_list` | List all notebooks |
| `notebook_create` | Create new notebook |
| `notebook_get` | Get notebook details |
| `notebook_delete` | Delete notebook |
| `source_add` | Add sources (URL/text/file/Drive) |
| `source_list` | List sources in notebook |
| `source_delete` | Remove a source |
| `notebook_query` | Query notebook with AI |
| `studio_create` | Generate audio/video/slides |
| `studio_revise` | Revise slide decks |
| `download_artifact` | Download generated content |
| `research_start` | Start web/Drive research |
| `notebook_share_public` | Enable public sharing |
| `notebook_share_invite` | Invite collaborators |
| `batch` | Batch operations on multiple notebooks |
| `cross_notebook_query` | Query across notebooks |
| `pipeline` | Multi-step workflows |
| `tag` | Tag and smart select notebooks |

### Example: Natural Language MCP Usage

Once the MCP is configured in Claude Code, Cursor, or another AI tool, you can use natural language:

**User:** "Create a notebook about quantum computing, add sources from arxiv.org, and generate a podcast"

**AI Assistant (using MCP tools):**

```
1. Using notebook_create: "Quantum Computing Research"
2. Using source_add: Adding URL https://arxiv.org/...
3. Using studio_create: Generating podcast audio
4. Using download_artifact: Downloading podcast.wav
```

## Common Workflows

### Research to Podcast Pipeline

```bash
# Step 1: Create notebook
NOTEBOOK_ID=$(nlm notebook create "AI Safety Research" --json | jq -r '.id')

# Step 2: Add sources
nlm source add $NOTEBOOK_ID \
  --url "https://arxiv.org/abs/example1" \
  --url "https://arxiv.org/abs/example2" \
  --file ./notes.pdf

# Step 3: Generate podcast
nlm studio create $NOTEBOOK_ID --type audio --confirm

# Step 4: Wait for completion and download
# (artifact ID returned from step 3)
nlm download audio $NOTEBOOK_ID <artifact-id> --output research-podcast.wav
```

### Batch Research Across Topics

```bash
# Create notebooks for multiple topics
TOPICS=("Machine Learning" "Quantum Computing" "Blockchain")

for topic in "${TOPICS[@]}"; do
  nlm notebook create "$topic Research"
done

# Query all notebooks with same question
nlm batch query "What are the latest developments?" $(nlm notebook list --json | jq -r '.[].id')
```

### Multi-Account Workflow

```bash
# Setup profiles
nlm login --profile work
nlm login --profile personal

# Use work account for research
nlm --profile work notebook create "Work Project"

# Use personal account for learning
nlm --profile personal notebook create "Personal Study"

# Set default profile
nlm login switch work
```

### Automated Content Generation

```python
import subprocess
import json

def create_research_podcast(topic, sources):
    """Create a NotebookLM podcast from research sources."""
    
    # Create notebook
    result = subprocess.run(
        ["nlm", "notebook", "create", topic, "--json"],
        capture_output=True,
        text=True
    )
    notebook = json.loads(result.stdout)
    notebook_id = notebook["id"]
    
    # Add sources
    for source_url in sources:
        subprocess.run([
            "nlm", "source", "add", notebook_id,
            "--url", source_url
        ])
    
    # Generate podcast
    result = subprocess.run([
        "nlm", "studio", "create", notebook_id,
        "--type", "audio",
        "--confirm",
        "--json"
    ], capture_output=True, text=True)
    
    artifact = json.loads(result.stdout)
    artifact_id = artifact["id"]
    
    # Download
    subprocess.run([
        "nlm", "download", "audio",
        notebook_id, artifact_id,
        "--output", f"{topic.replace(' ', '_')}.wav"
    ])
    
    print(f"Podcast created: {topic}.wav")

# Example usage
create_research_podcast(
    "AI Safety",
    [
        "https://arxiv.org/abs/example1",
        "https://arxiv.org/abs/example2"
    ]
)
```

### Pipeline Automation

Create a pipeline for recurring workflows:

```yaml
# pipeline.yaml
name: weekly-research-digest
description: Automated weekly research summary
steps:
  - action: notebook_create
    params:
      title: "Weekly Digest - {{ date }}"
    output: notebook_id
  
  - action: research_start
    params:
      notebook_id: "{{ notebook_id }}"
      query: "{{ topic }} developments this week"
  
  - action: studio_create
    params:
      notebook_id: "{{ notebook_id }}"
      type: audio
      confirm: true
  
  - action: notebook_share_public
    params:
      notebook_id: "{{ notebook_id }}"
```

Run weekly:

```bash
nlm pipeline run weekly-research-digest \
  --param date="2026-01-15" \
  --param topic="AI Safety"
```

## Troubleshooting

### Authentication Issues

```bash
# Check auth status
nlm login --check

# Force re-authentication
nlm login --force

# Try manual cookie import
nlm login --manual --file cookies.txt

# Use different profile
nlm login --profile backup
```

### MCP Connection Issues

```bash
# Run diagnostics
nlm doctor

# Check MCP server logs
nlm logs

# Verify tool configuration
nlm setup list

# Test specific tool
nlm setup test claude-code

# Restart AI tool after reconfiguration
```

### Cookie Expiration

Cookies typically expire after 30-90 days. If you see auth errors:

```bash
# Re-authenticate
nlm login

# Verify authentication
nlm login --check
```

### Studio Content Generation Fails

```bash
# Ensure sources are fully loaded
nlm source list <notebook-id>

# Check notebook has enough content (5000+ characters recommended)
nlm notebook get <notebook-id>

# Try with explicit confirmation
nlm studio create <notebook-id> --type audio --confirm
```

### Profile Issues

```bash
# List all profiles with details
nlm login profile list

# Switch to working profile
nlm login switch <profile-name>

# Delete corrupted profile
nlm login profile delete <profile-name>

# Re-authenticate
nlm login --profile <profile-name>
```

### Browser Selection Issues

```bash
# Check detected browsers
nlm doctor

# Set preferred browser explicitly
nlm config set auth.browser chromium

# Use different browser
nlm config set auth.browser brave
```

### Rate Limiting

If you encounter rate limits:

- Add delays between batch operations
- Use `--ephemeral` for queries that don't need persistence
- Spread operations across time

```bash
# Add delay in batch script
for id in $NOTEBOOK_IDS; do
  nlm notebook query $id "Question?" --ephemeral
  sleep 2
done
```

## Environment Variables

```bash
# Override config directory
export NOTEBOOKLM_CONFIG_DIR=~/.custom-notebooklm

# Override default profile
export NOTEBOOKLM_PROFILE=work

# Enable debug logging
export NOTEBOOKLM_DEBUG=1

# Set preferred browser
export NOTEBOOKLM_BROWSER=arc
```

## Best Practices

1. **Use profiles for multiple accounts:** Keep work and personal research separate
2. **Tag notebooks for organization:** Use `nlm tag add` for easy filtering
3. **Leverage pipelines for recurring tasks:** Define YAML workflows once, run many times
4. **Verify auth regularly:** Run `nlm login --check` weekly
5. **Use ephemeral queries for testing:** Save context window with `--ephemeral`
6. **Enable MCP only when needed:** Toggle with `@notebooklm-mcp` in Claude Code
7. **Keep tools updated:** Run `uv tool upgrade notebooklm-mcp-cli` monthly
8. **Backup important notebooks:** Use `notebook_get` to save full JSON snapshots

## Advanced: Programmatic Usage (Python)

While primarily a CLI/MCP tool, you can call `nlm` from Python scripts:

```python
import subprocess
import json

def get_notebooks():
    """Get all notebooks as Python objects."""
    result = subprocess.run(
        ["nlm", "notebook", "list", "--json"],
        capture_output=True,
        text=True,
        check=True
    )
    return json.loads(result.stdout)

def create_and_populate(title, sources):
    """Create notebook and add sources."""
    # Create
    result = subprocess.run(
        ["nlm", "notebook", "create", title, "--json"],
        capture_output=True,
        text=True,
        check=True
    )
    notebook = json.loads(result.stdout)
    
    # Add sources
    for url in sources:
        subprocess.run([
            "nlm", "source", "add",
            notebook["id"],
            "--url", url
        ], check=True)
    
    return notebook["id"]

# Example
notebook_id = create_and_populate(
    "AI Research",
    [
        "https://arxiv.org/abs/1234.5678",
        "https://example.com/article"
    ]
)

print(f"Created notebook: {notebook_id}")
```

## Resources

- **GitHub:** https://github.com/jacob-bd/notebooklm-mcp-cli
- **PyPI:** https://pypi.org/project/notebooklm-mcp-cli/
- **Documentation:**
  - [Getting Started](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/GETTING_STARTED.md)
  - [CLI Guide](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/CLI_GUIDE.md)
  - [MCP Guide](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/MCP_GUIDE.md)
  - [Authentication](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/AUTHENTICATION.md)
- **Video Demos:** See README for latest demo links

## Summary

`notebooklm-mcp-cli` provides complete programmatic access to Google NotebookLM through:
- **CLI (`nlm`):** Direct terminal commands for automation
- **MCP Server (`notebooklm-mcp`):** AI agent integration for natural language workflows

Key features: notebook management, source control, AI queries, studio content generation (podcasts/videos/slides), sharing, batch operations, pipelines, and multi-account support.

Install with `uv tool install notebooklm-mcp-cli`, authenticate with `nlm login`, and configure AI tools with `nlm setup add <tool>`.
```
