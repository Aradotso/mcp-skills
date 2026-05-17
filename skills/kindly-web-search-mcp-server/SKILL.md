---
name: kindly-web-search-mcp-server
description: Web search MCP server with intelligent content extraction for AI coding tools, supporting Serper, Tavily, and SearXNG with specialized parsers for StackOverflow, GitHub, Wikipedia, and arXiv.
triggers:
  - search the web for
  - find documentation about
  - look up recent discussions on
  - get content from this URL
  - search StackOverflow for
  - find GitHub issues about
  - retrieve article content from
  - search for API documentation
---

# Kindly Web Search MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Kindly Web Search is an MCP server that provides intelligent web search and content extraction for AI coding assistants. Unlike basic search wrappers, it returns **full structured content** in a single call—including StackOverflow answers, GitHub issue comments, Wikipedia articles, and arXiv papers—eliminating the need for separate scraping tools.

**Key differentiator**: When searching for solutions, Kindly fetches the entire conversation (questions + answers + comments) in one request, not just a title and URL snippet.

**Eliminates need for**:
- Generic web search MCP servers
- Separate StackOverflow/GitHub/Wikipedia MCP servers
- Web scraping MCP servers (Playwright/Puppeteer)

## Installation

### Prerequisites

1. **Python 3.13+** (3.14 supported)
2. **Chromium-based browser** (Chrome/Chromium/Edge/Brave) installed locally
3. **Search provider API key** (one of):
   - `SERPER_API_KEY` (recommended)
   - `TAVILY_API_KEY`
   - `SEARXNG_BASE_URL` (self-hosted SearXNG)
4. **Optional but recommended**: `GITHUB_TOKEN` (read-only, public repos only)

### Setup Steps

#### 1. Install uvx

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
irm https://astral.sh/uv/install.ps1 | iex
```

#### 2. Install Chromium (if not already installed)

```bash
# macOS
brew install --cask chromium

# Ubuntu/Debian
sudo apt-get update && sudo apt-get install -y chromium

# Windows: Install Chrome or Edge from official sources
```

#### 3. Set environment variables

```bash
# Choose ONE search provider
export SERPER_API_KEY="your-key-here"
# OR
export TAVILY_API_KEY="your-key-here"
# OR
export SEARXNG_BASE_URL="https://searx.example.org"

# Optional: Better GitHub extraction
export GITHUB_TOKEN="your-github-token"

# Optional: Custom browser path (auto-detected by default)
export KINDLY_BROWSER_EXECUTABLE_PATH="/path/to/chrome"
```

#### 4. Configure MCP Client

**Codex:**

```bash
# Using Serper
codex mcp add kindly-web-search \
  --env SERPER_API_KEY="$SERPER_API_KEY" \
  --env GITHUB_TOKEN="$GITHUB_TOKEN" \
  -- uvx --from git+https://github.com/Shelpuk-AI-Technology-Consulting/kindly-web-search-mcp-server \
  kindly-web-search-mcp-server start-mcp-server

# Using Tavily
codex mcp add kindly-web-search \
  --env TAVILY_API_KEY="$TAVILY_API_KEY" \
  --env GITHUB_TOKEN="$GITHUB_TOKEN" \
  -- uvx --from git+https://github.com/Shelpuk-AI-Technology-Consulting/kindly-web-search-mcp-server \
  kindly-web-search-mcp-server start-mcp-server
```

**Claude Desktop / Other MCP Clients:**

Add to your MCP settings JSON (typically `~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "kindly-web-search": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/Shelpuk-AI-Technology-Consulting/kindly-web-search-mcp-server",
        "kindly-web-search-mcp-server",
        "start-mcp-server"
      ],
      "env": {
        "SERPER_API_KEY": "your-key-here",
        "GITHUB_TOKEN": "your-github-token"
      }
    }
  }
}
```

**Windows (PowerShell):**

```powershell
codex mcp add kindly-web-search `
  --env SERPER_API_KEY="$env:SERPER_API_KEY" `
  --env GITHUB_TOKEN="$env:GITHUB_TOKEN" `
  -- uvx --from git+https://github.com/Shelpuk-AI-Technology-Consulting/kindly-web-search-mcp-server `
  kindly-web-search-mcp-server start-mcp-server
```

## MCP Tools

### `web_search`

Search the web and return results with full page content.

**Parameters:**
- `query` (string, required): Search query
- `num_results` (integer, optional): Number of results (default: 3, max: 10)

**Returns:**
- `title`: Page title
- `link`: URL
- `snippet`: Brief excerpt
- `page_content`: Full page content in Markdown (extracted intelligently based on source)

**Example usage in prompt:**

```
Search for "FastAPI async database connection pooling best practices"
```

**What happens behind the scenes:**

```python
# The MCP server executes:
results = web_search(
    query="FastAPI async database connection pooling best practices",
    num_results=5
)

# Returns structured results with full content:
# - StackOverflow threads include all answers and comments
# - GitHub issues include full conversation threads
# - Documentation pages return clean Markdown
# - arXiv papers return full text content
```

### `get_content`

Extract content from a specific URL.

**Parameters:**
- `url` (string, required): URL to extract content from

**Returns:**
- `page_content`: Full page content in Markdown

**Example usage in prompt:**

```
Get the full content from https://github.com/tiangolo/fastapi/issues/12345
```

**What happens behind the scenes:**

```python
# The MCP server executes:
content = get_content(url="https://github.com/tiangolo/fastapi/issues/12345")

# Returns full GitHub issue with:
# - Issue description
# - All comments
# - Reactions and metadata
# - Code snippets in proper format
```

## Specialized Content Extraction

Kindly automatically detects and optimally extracts content from:

### StackOverflow / StackExchange

Returns complete Q&A threads with:
- Question body with code examples
- All answers (sorted by votes/acceptance)
- Comments on questions and answers
- Vote counts and acceptance status

### GitHub Issues & Discussions

Returns full conversation threads:
- Issue/discussion body
- All comments chronologically
- Reactions and labels
- State (open/closed) and metadata

**Requires `GITHUB_TOKEN` for best results** (avoids rate limits, returns richer structure).

### Wikipedia

Returns clean article content:
- Main article text
- Section structure preserved
- Links converted to references
- Tables formatted in Markdown

### arXiv Papers

Returns full paper content:
- Abstract
- Full paper text
- Mathematical equations preserved
- References and citations

### Generic Webpages

Uses headless browser (nodriver) to:
- Handle JavaScript-heavy sites
- Extract main content area
- Convert to clean Markdown
- Remove navigation/ads/clutter

## Configuration

### Search Provider Priority

Kindly checks providers in this order:
1. Serper (if `SERPER_API_KEY` set) — **recommended, most reliable**
2. Tavily (if `TAVILY_API_KEY` set)
3. SearXNG (if `SEARXNG_BASE_URL` set)

### Environment Variables

```bash
# Search providers (set ONE)
SERPER_API_KEY=         # Google Serper API key
TAVILY_API_KEY=         # Tavily API key
SEARXNG_BASE_URL=       # Self-hosted SearXNG instance URL

# Content extraction
GITHUB_TOKEN=           # GitHub personal access token (read-only)
KINDLY_BROWSER_EXECUTABLE_PATH=  # Custom browser path (auto-detected)

# SearXNG advanced config
SEARXNG_HEADERS_JSON=   # JSON string with custom headers
SEARXNG_USER_AGENT=     # Custom user agent for SearXNG
```

### Browser Detection

Kindly auto-detects browsers in this order:
1. `KINDLY_BROWSER_EXECUTABLE_PATH` (if set)
2. Chrome
3. Chromium
4. Edge
5. Brave

**Windows manual path** (if auto-detection fails):

```powershell
$env:KINDLY_BROWSER_EXECUTABLE_PATH="C:\Program Files\Google\Chrome\Application\chrome.exe"
```

## Common Usage Patterns

### Finding Error Solutions

**Prompt:**
```
I'm getting "RuntimeError: CUDA out of memory" in PyTorch. Search for solutions and workarounds.
```

**Result:**
Kindly searches StackOverflow, GitHub issues, and documentation, returning complete threads with:
- Multiple solutions tried by others
- Code examples showing memory management
- Comments explaining tradeoffs
- Links to related discussions

### API Documentation Lookup

**Prompt:**
```
Search for the latest FastAPI dependency injection documentation and examples.
```

**Result:**
Returns official docs with:
- Full API reference
- Complete code examples
- Best practices sections
- Version-specific notes

### Research Paper Content

**Prompt:**
```
Get the full content of the attention mechanism paper from arXiv.
```

**Result:**
Returns complete paper text with:
- Abstract and introduction
- Full methodology section
- Equations and formulas
- References

### GitHub Issue Investigation

**Prompt:**
```
Find GitHub issues about memory leaks in Langchain streaming responses.
```

**Result:**
Returns relevant issues with:
- Full issue descriptions
- All community comments
- Proposed solutions and workarounds
- Current status and fixes

## Real-World Examples

### Example 1: Debugging Cloud Infrastructure

**Prompt:**
```
Search for solutions to "GCP Cloud Batch fails with GPU instance template"
```

**What you get:**
```markdown
# StackOverflow Result 1
## Question (Score: 15)
I am trying to run a GCP Cloud Batch job with K80 GPU...
[full question with code]

## Answer (Accepted, Score: 23)
The issue is related to GPU quota limits...
[complete solution with configuration examples]

## Comments
- User123: "This also works for T4 GPUs"
- OP: "Confirmed, this fixed it. Also needed to..."

# GitHub Issue Result 2
## googleapis/python-batch #145 - GPU instance template failures
[Full issue thread with 12 comments and resolution]
```

### Example 2: Library Usage Pattern

**Prompt:**
```
Search for best practices on using SQLAlchemy async sessions with FastAPI
```

**What you get:**
Multiple results including:
- Official FastAPI documentation on async DB
- StackOverflow threads with production patterns
- GitHub discussions with real codebases
- All with complete code examples inline

## Troubleshooting

### "No search provider configured"

**Problem:** None of the search provider env vars are set.

**Solution:**
```bash
export SERPER_API_KEY="your-key"
# OR
export TAVILY_API_KEY="your-key"
# OR
export SEARXNG_BASE_URL="https://searx.example.org"
```

### "Browser executable not found"

**Problem:** Chromium-based browser not installed or not detected.

**Solution:**
```bash
# Install Chrome/Chromium
brew install --cask chromium  # macOS
sudo apt-get install chromium  # Linux

# Or set custom path
export KINDLY_BROWSER_EXECUTABLE_PATH="/usr/bin/chromium"
```

### First run timeout

**Problem:** First `uvx` invocation takes 30-60 seconds to build environment.

**Solution:** Run once in terminal to prewarm:
```bash
uvx --from git+https://github.com/Shelpuk-AI-Technology-Consulting/kindly-web-search-mcp-server \
  kindly-web-search-mcp-server start-mcp-server
```

Then restart your MCP client.

### GitHub rate limiting

**Problem:** GitHub API returns 403 or rate limit errors.

**Solution:** Set `GITHUB_TOKEN`:
```bash
# Create token at https://github.com/settings/tokens
# Needs: public_repo (classic) or repo read access (fine-grained)
export GITHUB_TOKEN="ghp_..."
```

### Incomplete content extraction

**Problem:** Some pages return minimal content.

**Possible causes:**
1. Site blocks automated browsers
2. Content requires authentication
3. JavaScript errors prevent rendering

**Workarounds:**
- For documentation sites: Use direct API endpoints if available
- For gated content: Provide credentials via custom headers (SearXNG)
- For JavaScript-heavy sites: Content should work; file an issue if persistent

## Integration with AI Coding Agent Workflow

### Recommended Usage

When working with AI coding agents (Claude Code, Codex, Cursor):

1. **Start broad**: Let the agent search first
   ```
   Search for FastAPI WebSocket authentication patterns
   ```

2. **Then drill down**: Use `get_content` for specific URLs
   ```
   Get the content from [URL the agent found]
   ```

3. **Combine with code generation**: The agent now has full context
   ```
   Based on the search results, implement WebSocket auth for our API
   ```

### Part of Shelpuk AI Suite

Kindly works best when combined with:
- **tdd** skill: Enforces TDD workflow
- **Serena**: Semantic code navigation
- **Lad MCP Server**: AI design reviews

Together these improve AI-generated code quality by 15-20%.

## Advanced: SearXNG Self-Hosted

If using self-hosted SearXNG:

```bash
export SEARXNG_BASE_URL="https://searx.example.org"

# If instance requires authentication
export SEARXNG_HEADERS_JSON='{"Authorization":"Bearer your-token"}'

# If instance blocks default user agents
export SEARXNG_USER_AGENT="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
```

## Contributing

GitHub: https://github.com/Shelpuk-AI-Technology-Consulting/kindly-web-search-mcp-server

Issues and PRs welcome. See repository for contribution guidelines.
