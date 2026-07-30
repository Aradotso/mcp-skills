---
name: livetennisapi-mcp-integration
description: Integrate Live Tennis API MCP server to give AI agents real-time tennis scores, odds, and win probabilities
triggers:
  - how do I set up the Live Tennis API MCP server
  - show me live tennis matches and scores
  - get tennis player rankings and stats
  - check tennis match odds and probabilities
  - configure livetennisapi-mcp with Claude
  - what tennis tools are available in MCP
  - how to use tennis API with AI agents
  - set up real-time tennis data for LLM
---

# livetennisapi-mcp Integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The **livetennisapi-mcp** server provides AI agents with access to real-time tennis data through the Model Context Protocol (MCP). It exposes 12 tools for querying live scores, upcoming matches, player profiles, odds, and ML-powered win probabilities across ATP, WTA, Challenger, and ITF tournaments.

## Installation

### Get an API Key

1. Visit [livetennisapi.com/subscribe/free](https://livetennisapi.com/subscribe/free) for a free key (no credit card)
2. Store your key in an environment variable:

```bash
export LIVETENNISAPI_KEY=twjp_your_actual_key_here
```

### Claude Code

```bash
claude mcp add livetennis -e LIVETENNISAPI_KEY=twjp_your_key -- npx -y livetennisapi-mcp
```

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "livetennis": {
      "command": "npx",
      "args": ["-y", "livetennisapi-mcp"],
      "env": { "LIVETENNISAPI_KEY": "twjp_your_key_here" }
    }
  }
}
```

**Config file locations:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

### Cursor / Zed

Same configuration pattern as Claude Desktop. Add the server block to your MCP settings file.

### Codex

```bash
codex mcp add livetennisapi \
  --url https://mcp.livetennisapi.com/mcp \
  --bearer-token-env-var LIVETENNISAPI_KEY
```

Or manually in `~/.codex/config.toml`:

```toml
[mcp_servers.livetennisapi]
url = "https://mcp.livetennisapi.com/mcp"
bearer_token_env_var = "LIVETENNISAPI_KEY"
```

## Available Tools

### FREE Tier

| Tool | Purpose |
|------|---------|
| `get_live_matches` | All matches currently in progress with live scores |
| `get_upcoming_matches` | Matches starting soon |
| `get_match` | Full details for a specific match |
| `get_match_score` | Just the score (fastest) |
| `search_players` | Find players by name |
| `get_player` | Player profile, ranking, country, handedness |
| `get_fixtures` | Forward schedule |

### BASIC Tier

- `get_recent_results` — Completed matches and winners

### PRO Tier

- `get_match_events` — Breaks, games, sets, momentum shifts
- `get_match_odds` — Match-winner betting prices (bid/ask/mid)

### ULTRA Tier

- `get_match_analysis` — ML model win probability and key factors

### Utility

- `check_api_status` — Verify API connectivity and key tier

## Usage Patterns

### Checking Live Matches

**User prompt:**
```
What tennis matches are live right now?
```

The agent will call `get_live_matches` and receive structured data:

```json
{
  "matches": [
    {
      "match_id": 18953,
      "tournament": "Australian Open",
      "round": "QF",
      "player1": "Carlos Alcaraz",
      "player2": "Jannik Sinner",
      "score": "6-3, 2-4",
      "serving": 2,
      "status": "live"
    }
  ]
}
```

### Player Lookup

**User prompt:**
```
Show me Sinner's ranking and recent results
```

The agent will:
1. Call `search_players` with query "Sinner"
2. Call `get_player` with the player_id
3. Call `get_recent_results` (if key has BASIC+)

```json
{
  "player_id": 12345,
  "name": "Jannik Sinner",
  "country": "ITA",
  "ranking": 1,
  "ranking_points": 11830,
  "handedness": "right",
  "age": 23
}
```

### Match Details with Odds

**User prompt:**
```
What are the odds on the Alcaraz vs Sinner match?
```

For PRO+ keys, the agent calls:
1. `get_live_matches` or `get_match` to find match_id
2. `get_match_odds` with that match_id

```json
{
  "match_id": 18953,
  "player1_odds": {
    "bid": 1.85,
    "ask": 1.90,
    "mid": 1.875
  },
  "player2_odds": {
    "bid": 1.95,
    "ask": 2.00,
    "mid": 1.975
  }
}
```

### Win Probability Analysis

**User prompt:**
```
What does the model give Alcaraz in this match?
```

For ULTRA keys, the agent calls `get_match_analysis`:

```json
{
  "match_id": 18953,
  "player1_win_probability": 0.48,
  "player2_win_probability": 0.52,
  "key_factors": [
    "Sinner's serve hold rate on hard courts: 89%",
    "Alcaraz's recent injury recovery affecting movement"
  ],
  "model_thesis": "Close match. Sinner slight favorite due to surface advantage."
}
```

## Tier-Aware Responses

The server returns **human-readable explanations** instead of raw 403 errors when a tool requires a higher tier:

```
This data requires the PRO plan, and the configured API key is on FREE tier.
Upgrade at https://livetennisapi.com/#pricing
```

This prevents the agent from hallucinating reasons or retrying endlessly.

### Check Your Key's Tier

```
What tier is my API key on?
```

The agent calls `check_api_status`, which probes upward to determine:

```json
{
  "status": "reachable",
  "tier": "PRO",
  "message": "API is reachable. Your key is on the PRO plan."
}
```

## HTTP Endpoint (Alternative)

For clients that cannot run stdio MCP servers, use the hosted HTTP endpoint:

```
https://mcp.livetennisapi.com/mcp
```

### With Claude Messages API (Python)

```python
import os
import anthropic

client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

response = client.beta.messages.create(
    model="claude-opus-4-8",
    max_tokens=4096,
    betas=["mcp-client-2025-11-20"],
    mcp_servers=[{
        "type": "url",
        "name": "livetennisapi",
        "url": "https://mcp.livetennisapi.com/mcp",
        "authorization_token": os.environ["LIVETENNISAPI_KEY"],
    }],
    tools=[{"type": "mcp_toolset", "mcp_server_name": "livetennisapi"}],
    messages=[{
        "role": "user",
        "content": "What tennis matches are live right now?"
    }],
)

print(response.content)
```

### With Claude Web Connector

Add a custom connector in Claude with:

```
https://mcp.livetennisapi.com/mcp?token=twjp_your_key
```

**Security note:** Prefer `Authorization: Bearer` header over `?token=` when your client supports it. The query param exists for clients that cannot set headers.

## Development & Self-Hosting

### Local Development

```bash
git clone https://github.com/livetennisapi/livetennisapi-mcp.git
cd livetennisapi-mcp
npm install
npm run build

# Run stdio server
LIVETENNISAPI_KEY=twjp_your_key node dist/index.js

# Run HTTP server (port 8081)
LIVETENNISAPI_KEY=twjp_your_key node dist/http.js
```

### Testing

```bash
npm test               # Protocol + transport isolation + rate limiting
npm run test:mutation  # Proves tests fail when code breaks
```

The mutation tests are important: they reintroduce bugs the tests claim to catch and verify the suite actually detects them.

### Self-Host HTTP Endpoint

See `deploy/install-http.sh` and `deploy/TUNNEL.md` in the repository for instructions on running your own HTTP endpoint.

The hosted endpoint is **multi-tenant** and holds no key of its own — every request builds an isolated server bound to the key provided in that request.

## Troubleshooting

### "Server not found" or "Connection refused"

**Stdio version:**
- Ensure `npx` can reach npm registry: `npx -y livetennisapi-mcp --version`
- Check Node.js version: `node --version` (needs 18+)
- Restart your AI client after adding the server config

**HTTP version:**
- Verify endpoint is reachable: `curl https://mcp.livetennisapi.com/mcp`
- Check firewall/proxy settings

### "Invalid API key" or 401 errors

```bash
# Verify your key is set correctly
echo $LIVETENNISAPI_KEY

# Test key directly against API
curl -H "Authorization: Bearer $LIVETENNISAPI_KEY" \
  https://api.livetennisapi.com/v1/live_matches
```

If the curl works but MCP doesn't, the env var isn't reaching the server process. In Claude Desktop, ensure `env` is in the server config, not at the root level.

### "upgrade_required" responses

Your key works, but the endpoint requires a higher tier. Use `check_api_status` to see your current tier, then upgrade at [pricing](https://livetennisapi.com/#pricing).

### Rate limiting

- Anonymous requests to HTTP endpoint: 60/minute
- Keyed requests to HTTP endpoint: 300/minute
- Per-key quotas enforced upstream by tier

If you hit the limit, the server returns a 429 with `Retry-After` header.

### No live matches returned

Tennis is not 24/7. Grand Slams and peak season provide the most coverage. If `get_live_matches` returns an empty array, try `get_upcoming_matches` or `get_fixtures` to see the schedule.

## Code Examples

### TypeScript: Using the Underlying Client

The MCP server is built on the official `livetennisapi` client. You can use it directly:

```typescript
import { LiveTennisAPI } from 'livetennisapi';

const client = new LiveTennisAPI(process.env.LIVETENNISAPI_KEY!);

// Get live matches
const liveMatches = await client.getLiveMatches();
console.log(liveMatches);

// Search for a player
const players = await client.searchPlayers('Djokovic');
const playerId = players[0]?.player_id;

// Get player details
if (playerId) {
  const player = await client.getPlayer(playerId);
  console.log(`${player.name} is ranked #${player.ranking}`);
}

// Get match odds (PRO tier)
const odds = await client.getMatchOdds(18953);
console.log(`Player 1 mid: ${odds.player1_odds.mid}`);
```

### JavaScript: Programmatic MCP Tool Calls

```javascript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const transport = new StdioClientTransport({
  command: 'npx',
  args: ['-y', 'livetennisapi-mcp'],
  env: { LIVETENNISAPI_KEY: process.env.LIVETENNISAPI_KEY }
});

const client = new Client({
  name: 'tennis-client',
  version: '1.0.0'
}, { capabilities: {} });

await client.connect(transport);

// List available tools
const { tools } = await client.listTools();
console.log(tools.map(t => t.name));

// Call a tool
const result = await client.callTool({
  name: 'get_live_matches',
  arguments: {}
});

console.log(result.content);
```

## Related Projects

- **Python client:** `pip install livetennisapi` — [livetennisapi-python](https://github.com/livetennisapi/livetennisapi-python)
- **JavaScript client:** `npm install livetennisapi` — [livetennisapi-js](https://github.com/livetennisapi/livetennisapi-js)
- **Vercel AI SDK tools:** `npm install livetennisapi-ai` — [livetennisapi-ai](https://github.com/livetennisapi/livetennisapi-ai)
- **OpenAPI spec:** [livetennisapi/openapi](https://github.com/livetennisapi/openapi)

## Resources

- **API Documentation:** [docs.livetennisapi.com](https://docs.livetennisapi.com)
- **Pricing & Plans:** [livetennisapi.com/#pricing](https://livetennisapi.com/#pricing)
- **Terms of Service:** [livetennisapi.com/terms](https://livetennisapi.com/terms)
- **Support:** Open an issue at [github.com/livetennisapi/livetennisapi-mcp](https://github.com/livetennisapi/livetennisapi-mcp)
