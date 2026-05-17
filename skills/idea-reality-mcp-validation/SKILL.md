---
name: idea-reality-mcp-validation
description: Pre-build reality check for AI coding agents — scan GitHub, HN, npm, PyPI, Product Hunt to validate ideas before building
triggers:
  - "check if this idea already exists"
  - "validate this project idea before I start building"
  - "is someone already working on this"
  - "run a reality check on this idea"
  - "search for similar projects to"
  - "check market competition for"
  - "scan GitHub and npm for existing solutions"
  - "does this startup idea already exist"
---

# idea-reality-mcp-validation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

idea-reality-mcp is an MCP server that validates project ideas before you write code. It scans GitHub, Hacker News, npm, PyPI, Product Hunt, and Stack Overflow to return a 0–100 reality score, trend detection, top competitors, and pivot suggestions.

## Installation

### Quick Start (uvx)

```bash
uvx idea-reality-mcp
```

### Add to Claude Code

```bash
claude mcp add idea-reality -- uvx idea-reality-mcp
```

### Add to Claude Desktop / Cursor

Edit your MCP config file:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`  
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`  
**Cursor**: `.cursor/mcp.json`

```json
{
  "mcpServers": {
    "idea-reality": {
      "command": "uvx",
      "args": ["idea-reality-mcp"]
    }
  }
}
```

### Smithery (Remote)

```bash
npx -y @smithery/cli install idea-reality-mcp --client claude
```

## Setup & Configuration

### First-Time Setup

```bash
idea-reality setup
```

Interactive wizard that:
1. Shows terms acceptance
2. Detects your platform (Claude Desktop, Cursor, etc.)
3. Generates config snippet
4. Runs health check

### Platform-Specific Config

```bash
idea-reality config              # interactive menu
idea-reality config claude_code  # auto-installs
idea-reality config cursor       # prints Cursor config
idea-reality config raw_json     # generic MCP JSON
```

### Health Check

```bash
idea-reality doctor        # core checks (~2s)
idea-reality doctor --full # + API validation, all 6 sources
```

### Optional Environment Variables

```bash
export GITHUB_TOKEN=ghp_...        # Higher GitHub API rate limits
export PRODUCTHUNT_TOKEN=...       # Enable Product Hunt (deep mode)
```

## Core MCP Tool: `idea_check`

### Tool Schema

**Parameters:**
- `idea_text` (string, required): Natural-language description of your idea
- `depth` (string, optional): `"quick"` (default) or `"deep"`

**Modes:**
- `quick`: GitHub + Hacker News (< 3 seconds)
- `deep`: All 6 sources (GitHub, HN, npm, PyPI, Product Hunt, Stack Overflow)

### Using from AI Agent

When a user says *"check if this idea already exists"*, use the `idea_check` tool:

```json
{
  "tool": "idea_check",
  "arguments": {
    "idea_text": "a CLI tool that converts Figma designs to React components",
    "depth": "deep"
  }
}
```

### Response Structure

```json
{
  "reality_signal": 72,
  "duplicate_likelihood": "high",
  "trend": "accelerating",
  "sub_scores": {
    "market_momentum": 73
  },
  "evidence": [
    {
      "source": "github",
      "type": "repo_count",
      "query": "figma react converter CLI",
      "count": 342
    },
    {
      "source": "github",
      "type": "max_stars",
      "query": "figma react converter",
      "count": 15000
    },
    {
      "source": "hackernews",
      "type": "mention_count",
      "query": "figma react",
      "count": 18
    },
    {
      "source": "npm",
      "type": "package_count",
      "query": "figma-react",
      "count": 56
    },
    {
      "source": "pypi",
      "type": "package_count",
      "query": "figma-react",
      "count": 23
    },
    {
      "source": "producthunt",
      "type": "product_count",
      "query": "figma react",
      "count": 8
    },
    {
      "source": "stackoverflow",
      "type": "question_count",
      "query": "figma react converter",
      "count": 120
    }
  ],
  "top_similars": [
    {
      "name": "figma-to-react/figma-to-react",
      "url": "https://github.com/figma-to-react/figma-to-react",
      "stars": 15000,
      "description": "Convert Figma designs to React components"
    }
  ],
  "pivot_hints": [
    "High competition. Consider a niche differentiator like supporting design tokens or specific component libraries.",
    "The leading project may have gaps in TypeScript support or accessibility features."
  ]
}
```

## REST API (No MCP Required)

### Python Client

```python
import httpx

response = httpx.post(
    "https://idea-reality-mcp.onrender.com/api/check",
    json={
        "idea_text": "AI code review tool",
        "depth": "deep"
    },
    timeout=30.0
)

result = response.json()
print(f"Reality Score: {result['reality_signal']}/100")
print(f"Trend: {result['trend']}")
print(f"Top Competitor: {result['top_similars'][0]['name']} ({result['top_similars'][0]['stars']} ⭐)")
```

### cURL

```bash
curl -X POST https://idea-reality-mcp.onrender.com/api/check \
  -H "Content-Type: application/json" \
  -d '{
    "idea_text": "a markdown-based static site generator with live reload",
    "depth": "quick"
  }'
```

## Common Patterns

### Pre-Build Validation

Before starting a new project:

```python
# User says: "I want to build a CLI tool for GitHub issue management"
# Agent calls:
{
  "tool": "idea_check",
  "arguments": {
    "idea_text": "CLI tool for GitHub issue management with labels and milestones",
    "depth": "deep"
  }
}

# If reality_signal > 80:
#   → Suggest niche differentiation or pivot
# If reality_signal 40-80:
#   → Validate unique features, check top_similars
# If reality_signal < 40:
#   → Green light, low competition
```

### Feature Validation

Check if a feature is already widely implemented:

```json
{
  "tool": "idea_check",
  "arguments": {
    "idea_text": "add real-time collaborative editing to my markdown editor",
    "depth": "quick"
  }
}
```

### Market Trend Analysis

Understand if a space is growing or declining:

```json
{
  "tool": "idea_check",
  "arguments": {
    "idea_text": "browser automation library using Chrome DevTools Protocol",
    "depth": "deep"
  }
}
```

Check the `trend` field:
- `"accelerating"` → Growing market, act fast
- `"stable"` → Mature market, differentiation critical
- `"declining"` → Consider pivoting

### Auto-Trigger in Agent Instructions

Add to `.cursorrules`, `CLAUDE.md`, or `.github/copilot-instructions.md`:

```markdown
When starting a new project, use the idea_check MCP tool to check if similar projects already exist.
```

## CI/CD Integration

### GitHub Action for Pull Requests

Create `.github/workflows/idea-check.yml`:

```yaml
name: Idea Reality Check
on:
  issues:
    types: [opened]

jobs:
  check:
    if: contains(github.event.issue.labels.*.name, 'proposal')
    runs-on: ubuntu-latest
    steps:
      - uses: mnemox-ai/idea-check-action@v1
        with:
          idea: ${{ github.event.issue.title }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

This auto-validates feature proposals labeled `proposal`.

## Interpreting Results

### Reality Signal (0–100)

- **0–30**: Low competition, potentially novel idea
- **31–60**: Moderate competition, validate unique angle
- **61–85**: High competition, niche differentiation required
- **86–100**: Saturated market, strong pivot recommended

### Duplicate Likelihood

- `low`: Few similar projects found
- `medium`: Several similar projects exist
- `high`: Many similar projects, established category
- `very_high`: Extremely crowded space

### Market Momentum (sub_scores)

Measures recent growth in the space:
- **< 40**: Declining interest
- **40–60**: Stable
- **> 60**: Growing/accelerating

## Scoring Weights

### Quick Mode

| Source | Weight |
|--------|--------|
| GitHub repos | 60% |
| GitHub stars | 20% |
| Hacker News | 20% |

### Deep Mode

| Source | Weight |
|--------|--------|
| GitHub repos | 22% |
| GitHub stars | 9% |
| Hacker News | 14% |
| npm | 18% |
| PyPI | 13% |
| Product Hunt | 14% |
| Stack Overflow | 10% |

If a source fails, weights redistribute automatically.

## Troubleshooting

### "MCP server not found"

**Cursor/Claude Desktop:**
1. Restart the application completely
2. Check config file location and syntax
3. Run `idea-reality doctor`

**Claude Code:**
```bash
claude mcp list  # verify installation
claude mcp remove idea-reality
claude mcp add idea-reality -- uvx idea-reality-mcp
```

### "GitHub API rate limit exceeded"

Set a GitHub token:

```bash
export GITHUB_TOKEN=ghp_your_token_here
idea-reality doctor --full  # verify
```

Generate token at: https://github.com/settings/tokens (no scopes needed for public data)

### "Product Hunt data missing (deep mode)"

Product Hunt requires authentication:

```bash
export PRODUCTHUNT_TOKEN=your_token_here
```

Or use `depth: "quick"` which skips Product Hunt.

### "Irrelevant results"

The tool uses 3-stage keyword extraction. If results are off:

1. Be more specific in `idea_text`:
   - ❌ "productivity app"
   - ✅ "CLI time-tracking tool for developers with Git integration"

2. Report the issue: https://github.com/mnemox-ai/idea-reality-mcp/issues/new?template=inaccurate-result.yml

### "Tool call timeout"

Deep mode can take 10–15 seconds. Increase timeout:

```python
# Python httpx client
httpx.post(..., timeout=30.0)
```

Or use `depth: "quick"` (< 3 seconds).

## Example Workflows

### 1. Pre-Project Kickoff

```markdown
User: "I want to build a Rust-based SQL formatter with auto-fix"

Agent:
1. Call idea_check with depth="deep"
2. If reality_signal > 70:
   - Show top competitors (e.g. sqlformat, prettier-plugin-sql)
   - Suggest niches from pivot_hints (e.g. Rust performance angle)
3. If reality_signal < 50:
   - Proceed with project scaffolding
```

### 2. Feature Gap Analysis

```markdown
User: "Should I add Vim keybindings to my editor?"

Agent:
1. Call idea_check: "text editor with vim keybindings"
2. Check evidence[].count for npm/PyPI packages
3. High count → already solved, suggest integration
4. Low count → potential differentiator
```

### 3. Market Validation

```markdown
User: "Is AI code review still worth building?"

Agent:
1. Call idea_check with depth="deep"
2. Check trend field
3. If "accelerating" → market growing, move fast
4. If "declining" → suggest pivot to specific niche
```

## Advanced Usage

### Batch Validation

```python
import httpx

ideas = [
    "AI-powered commit message generator",
    "Real-time Markdown collaboration",
    "GitHub issue templates manager"
]

async with httpx.AsyncClient() as client:
    tasks = [
        client.post(
            "https://idea-reality-mcp.onrender.com/api/check",
            json={"idea_text": idea, "depth": "quick"}
        )
        for idea in ideas
    ]
    results = await asyncio.gather(*tasks)

for idea, resp in zip(ideas, results):
    data = resp.json()
    print(f"{idea}: {data['reality_signal']}/100 ({data['trend']})")
```

### Custom Analysis

```python
def should_build(result: dict) -> str:
    score = result['reality_signal']
    trend = result['trend']
    
    if score < 40:
        return "BUILD: Low competition, novel idea"
    elif score < 70 and trend == "accelerating":
        return "BUILD WITH NICHE: Growing market, differentiate"
    elif score > 85:
        return "PIVOT: Saturated market"
    else:
        return "RESEARCH: Validate unique angle"

response = httpx.post(...)
verdict = should_build(response.json())
```

## Best Practices

1. **Always use `idea_check` before scaffolding** — prevents wasted effort
2. **Start with `depth="quick"`** — faster iteration, upgrade to deep if uncertain
3. **Read `pivot_hints`** — often contains actionable niche suggestions
4. **Check `top_similars`** — study competitors before building
5. **Monitor `trend`** — timing matters as much as uniqueness
6. **Set `GITHUB_TOKEN`** — avoids rate limits on larger scans

## Resources

- GitHub: https://github.com/mnemox-ai/idea-reality-mcp
- PyPI: https://pypi.org/project/idea-reality-mcp/
- Web Demo: https://mnemox.ai/check/
- CI Action: https://github.com/mnemox-ai/idea-check-action
