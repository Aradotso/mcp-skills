---
name: opennews-mcp-news-aggregation
description: Real-time crypto news aggregation with AI ratings and trading signals from 84+ sources across news, listings, on-chain, meme, market, and prediction engines
triggers:
  - get latest crypto news
  - search for SEC regulation news
  - show high impact crypto articles
  - subscribe to real-time blockchain news
  - find bullish trading signals
  - get on-chain whale alerts
  - search news by coin BTC ETH
  - monitor exchange listing announcements
---

# OpenNews MCP News Aggregation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

OpenNews MCP is a Model Context Protocol server providing real-time access to 84+ crypto and financial news sources across 6 categories (News, Listing, OnChain, Meme, Market, Prediction). Every article includes AI-powered impact scores (0-100), trading signals (long/short/neutral), and bilingual summaries.

## Installation

### Prerequisites

1. Get your API token from [https://6551.io/mcp](https://6551.io/mcp)
2. Set the token as an environment variable:

```bash
export OPENNEWS_TOKEN="your-token-here"
```

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "opennews": {
      "command": "uv",
      "args": [
        "--directory",
        "/path/to/opennews-mcp",
        "run",
        "opennews-mcp"
      ],
      "env": {
        "OPENNEWS_TOKEN": "your-token-here"
      }
    }
  }
}
```

### Using `claude mcp add`

```bash
claude mcp add opennews \
  -e OPENNEWS_TOKEN=your-token-here \
  -- uv --directory /path/to/opennews-mcp run opennews-mcp
```

### OpenClaw

```bash
export OPENNEWS_TOKEN="your-token-here"
cp -r openclaw-skill/opennews ~/.openclaw/skills/
```

## Configuration

The server supports both environment variables and a `config.json` file in the project root. Environment variables take precedence.

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENNEWS_TOKEN` | **Yes** | - | API Bearer Token from 6551.io |
| `OPENNEWS_API_BASE` | No | `https://ai.6551.io` | REST API base URL |
| `OPENNEWS_WSS_URL` | No | `wss://ai.6551.io/open/news_wss` | WebSocket URL |
| `OPENNEWS_MAX_ROWS` | No | `100` | Max results per request |

### config.json

```json
{
  "api_base_url": "https://ai.6551.io",
  "wss_url": "wss://ai.6551.io/open/news_wss",
  "api_token": "your-token-here",
  "max_rows": 100
}
```

## Data Sources Overview

The platform aggregates 84+ sources across 6 engine types:

- **news** (53 sources): Bloomberg, Reuters, Financial Times, CNBC, CoinDesk, Cointelegraph, The Block, Twitter/X, Telegram, and more
- **listing** (9 sources): Binance, Coinbase, OKX, Bybit, Upbit, Robinhood, Hyperliquid listings
- **onchain** (3 sources): Hyperliquid whale trades, large positions, KOL trades
- **meme** (1 source): Twitter meme coin sentiment
- **market** (6 sources): Price changes, funding rates, liquidations, OI changes
- **prediction** (12 sources): AI correlation, smart money, whale positions, insider patterns

## Available Tools

### Discovery Tools

#### `get_news_sources`

Retrieves the complete engine tree with all categories and sources.

```python
# Returns hierarchical structure of all 6 engine types and their sources
# Each source includes metadata like description and type
```

**Response structure:**
```json
{
  "news": {
    "Bloomberg": {"description": "Bloomberg — top-tier financial news"},
    "CoinDesk": {"description": "CoinDesk — leading crypto media"}
  },
  "listing": {
    "Binance": {"description": "Binance new token listings"}
  },
  "onchain": {
    "Hyperliquid Whale Trade": {"description": "Hyperliquid whale trade alerts"}
  }
}
```

#### `list_news_types`

Returns a flat list of all available source codes for filtering.

```python
# Returns array of all source codes across all engine types
# Use these codes in other tool filters
```

**Response:**
```json
["Bloomberg", "Reuters", "CoinDesk", "Binance", "Hyperliquid Whale Trade", ...]
```

### Search Tools

#### `get_latest_news`

Fetch the most recent articles across all sources.

**Parameters:**
- `limit` (optional, default: 20): Number of articles to return

```python
# Get latest 50 articles
{"limit": 50}
```

#### `search_news`

Full-text keyword search across all sources.

**Parameters:**
- `keywords` (required): Search query string
- `limit` (optional, default: 20): Number of results

```python
# Search for SEC-related news
{"keywords": "SEC regulation", "limit": 30}
```

#### `search_news_by_coin`

Filter articles by specific cryptocurrencies.

**Parameters:**
- `coins` (required): Array of coin symbols (e.g., `["BTC", "ETH"]`)
- `limit` (optional, default: 20): Number of results

```python
# Get Bitcoin and Ethereum news
{"coins": ["BTC", "ETH"], "limit": 40}
```

#### `get_news_by_source`

Filter by specific source within an engine type.

**Parameters:**
- `engine_type` (required): Engine category (`news`, `listing`, `onchain`, `meme`, `market`, `prediction`)
- `news_type` (required): Specific source code (e.g., `Bloomberg`, `Binance`)
- `limit` (optional, default: 20): Number of results

```python
# Get Bloomberg news only
{"engine_type": "news", "news_type": "Bloomberg", "limit": 25}

# Get Binance listings
{"engine_type": "listing", "news_type": "Binance", "limit": 10}
```

#### `get_news_by_engine`

Filter articles by engine category.

**Parameters:**
- `engine_type` (required): Engine category
- `limit` (optional, default: 20): Number of results

```python
# Get all on-chain events
{"engine_type": "onchain", "limit": 30}

# Get all prediction signals
{"engine_type": "prediction", "limit": 50}
```

#### `search_news_advanced`

Multi-filter search combining coins, keywords, and engine types.

**Parameters:**
- `coins` (optional): Array of coin symbols
- `keywords` (optional): Search query
- `engine_types` (optional): Array of engine categories
- `limit` (optional, default: 20): Number of results

```python
# Bitcoin news from Bloomberg and CoinDesk mentioning "ETF"
{
  "coins": ["BTC"],
  "keywords": "ETF",
  "engine_types": ["news"],
  "limit": 30
}

# Whale activity for SOL and ETH
{
  "coins": ["SOL", "ETH"],
  "engine_types": ["onchain"],
  "limit": 20
}
```

### AI-Powered Tools

#### `get_high_score_news`

Retrieve articles with high AI impact scores.

**Parameters:**
- `min_score` (optional, default: 80): Minimum impact score (0-100)
- `limit` (optional, default: 20): Number of results

```python
# Get articles with score >= 90
{"min_score": 90, "limit": 15}

# Get articles with score >= 70
{"min_score": 70, "limit": 30}
```

#### `get_news_by_signal`

Filter by AI-generated trading signals.

**Parameters:**
- `signal` (required): Trading signal type (`long`, `short`, `neutral`)
- `limit` (optional, default: 20): Number of results

```python
# Get bullish signals
{"signal": "long", "limit": 25}

# Get bearish signals
{"signal": "short", "limit": 25}

# Get neutral signals
{"signal": "neutral", "limit": 20}
```

### Real-Time Tools

#### `subscribe_latest_news`

Subscribe to WebSocket live feed with optional filters.

**Parameters:**
- `engine_types` (optional): Object mapping engine types to source codes
  - Key: Engine type (`news`, `listing`, `onchain`, etc.)
  - Value: Array of source codes (empty array = all sources in that engine)
- `coins` (optional): Array of coin symbols
- `has_coin` (optional, boolean): Only articles tagged with coins

```python
# Subscribe to Bloomberg and CoinDesk for BTC and ETH
{
  "engine_types": {
    "news": ["Bloomberg", "CoinDesk"]
  },
  "coins": ["BTC", "ETH"],
  "has_coin": True
}

# Subscribe to all listing announcements
{
  "engine_types": {
    "listing": []
  }
}

# Subscribe to whale on-chain activity
{
  "engine_types": {
    "onchain": []
  }
}
```

**WebSocket message format (incoming):**
```json
{
  "jsonrpc": "2.0",
  "method": "news.update",
  "params": {
    "id": "article-id",
    "title": "Article title",
    "content": "Full article text",
    "engine_type": "news",
    "news_type": "Bloomberg",
    "coins": ["BTC"],
    "ai_score": 85,
    "ai_signal": "long",
    "summary_en": "English summary",
    "summary_zh": "中文摘要",
    "published_at": "2026-05-16T12:00:00Z"
  }
}
```

## Response Data Structure

All news articles follow this structure:

```json
{
  "id": "unique-article-id",
  "title": "Article headline",
  "content": "Full article text content",
  "engine_type": "news",
  "news_type": "Bloomberg",
  "coins": ["BTC", "ETH"],
  "ai_score": 85,
  "ai_signal": "long",
  "summary_en": "English AI-generated summary",
  "summary_zh": "中文AI生成摘要",
  "published_at": "2026-05-16T10:30:00Z",
  "url": "https://original-source.com/article",
  "metadata": {
    "author": "Jane Doe",
    "tags": ["regulation", "ETF"]
  }
}
```

**Key fields:**
- `ai_score` (0-100): AI impact score, higher = more market-moving
- `ai_signal`: Trading direction (`long`, `short`, `neutral`)
- `coins`: Array of related cryptocurrency symbols
- `engine_type`: Category (news, listing, onchain, meme, market, prediction)
- `news_type`: Specific source code

## Common Usage Patterns

### Pattern 1: Monitor High-Impact News

```python
# Get latest high-impact news (score >= 80)
result = get_high_score_news(min_score=80, limit=10)

# Filter for bullish signals only
bullish = get_news_by_signal(signal="long", limit=15)

# Combine: high-impact bullish Bitcoin news
advanced = search_news_advanced(
    coins=["BTC"],
    engine_types=["news", "prediction"],
    limit=20
)
# Then filter results where ai_score >= 80 and ai_signal == "long"
```

### Pattern 2: Track Specific Assets

```python
# Get all Solana-related news
sol_news = search_news_by_coin(coins=["SOL"], limit=30)

# Get Solana on-chain whale activity
sol_onchain = search_news_advanced(
    coins=["SOL"],
    engine_types=["onchain"],
    limit=15
)

# Get Solana listings across all exchanges
sol_listings = search_news_advanced(
    coins=["SOL"],
    engine_types=["listing"],
    limit=10
)
```

### Pattern 3: Source-Specific Monitoring

```python
# Get all Bloomberg articles
bloomberg = get_news_by_source(
    engine_type="news",
    news_type="Bloomberg",
    limit=25
)

# Get Binance listing announcements
binance_listings = get_news_by_source(
    engine_type="listing",
    news_type="Binance",
    limit=10
)

# Get Hyperliquid whale trades
whale_trades = get_news_by_source(
    engine_type="onchain",
    news_type="Hyperliquid Whale Trade",
    limit=20
)
```

### Pattern 4: Real-Time Alerts

```python
# Subscribe to critical sources for BTC and ETH
subscribe_latest_news(
    engine_types={
        "news": ["Bloomberg", "Reuters", "Financial Times"],
        "listing": ["Binance", "Coinbase"],
        "onchain": []  # All on-chain sources
    },
    coins=["BTC", "ETH"],
    has_coin=True
)

# Subscribe to all prediction signals
subscribe_latest_news(
    engine_types={
        "prediction": []
    }
)
```

### Pattern 5: Thematic Research

```python
# Research regulation topics
regulation_news = search_news(
    keywords="SEC regulation compliance",
    limit=40
)

# ETF-related news with high impact
etf_news = search_news(keywords="ETF", limit=30)
# Filter for ai_score >= 75

# Institutional adoption signals
institutional = search_news(
    keywords="institutional adoption grayscale blackrock",
    limit=25
)
```

## WebSocket Direct Usage

For applications needing direct WebSocket access:

```python
import json
import websockets
import asyncio
import os

async def subscribe_news():
    token = os.getenv("OPENNEWS_TOKEN")
    url = f"wss://ai.6551.io/open/news_wss?token={token}"
    
    async with websockets.connect(url) as ws:
        # Subscribe to specific filters
        subscribe_msg = {
            "jsonrpc": "2.0",
            "id": 1,
            "method": "news.subscribe",
            "params": {
                "engineTypes": {
                    "news": ["Bloomberg", "CoinDesk"],
                    "listing": []
                },
                "coins": ["BTC", "ETH"],
                "hasCoin": True
            }
        }
        
        await ws.send(json.dumps(subscribe_msg))
        
        # Receive confirmation
        response = await ws.recv()
        print(f"Subscription confirmed: {response}")
        
        # Listen for updates
        async for message in ws:
            data = json.loads(message)
            if data.get("method") == "news.update":
                article = data["params"]
                print(f"New article: {article['title']}")
                print(f"AI Score: {article['ai_score']}")
                print(f"Signal: {article['ai_signal']}")
```

## Troubleshooting

### Authentication Errors

**Problem:** `401 Unauthorized` or `Invalid token`

**Solution:**
1. Verify token is correct from https://6551.io/mcp
2. Check environment variable is set: `echo $OPENNEWS_TOKEN`
3. Ensure token has no extra spaces or quotes
4. For Claude Desktop, restart the app after config changes

### No Results Returned

**Problem:** Empty results or `[]`

**Solution:**
1. Check if filters are too restrictive (e.g., coin doesn't have recent news)
2. Increase `limit` parameter
3. Verify `engine_type` and `news_type` codes with `list_news_types`
4. Try broader search (e.g., use `get_latest_news` first)

### WebSocket Connection Issues

**Problem:** Connection fails or disconnects

**Solution:**
1. Verify token is included in URL query parameter
2. Check network allows WebSocket connections (corporate firewalls)
3. Implement reconnection logic with exponential backoff
4. Validate subscription message format matches JSON-RPC 2.0

### Rate Limiting

**Problem:** `429 Too Many Requests`

**Solution:**
1. Reduce request frequency
2. Use WebSocket subscriptions instead of polling
3. Implement request queuing with delays
4. Contact support for higher rate limits if needed

### Unexpected Data Format

**Problem:** Missing fields or unexpected values

**Solution:**
1. Not all articles have `coins` array (check `has_coin` filter)
2. `ai_score` and `ai_signal` are always present but may be null
3. `metadata` object is optional and varies by source
4. Always validate fields exist before accessing

## Example Integration Workflows

### Daily Digest Builder

```python
# Get top 10 highest impact articles from past 24 hours
high_impact = get_high_score_news(min_score=85, limit=10)

# Group by signal
bullish = [a for a in high_impact if a['ai_signal'] == 'long']
bearish = [a for a in high_impact if a['ai_signal'] == 'short']

# Format digest
print("📈 Bullish Signals:")
for article in bullish:
    print(f"  • {article['title']} (Score: {article['ai_score']})")

print("\n📉 Bearish Signals:")
for article in bearish:
    print(f"  • {article['title']} (Score: {article['ai_score']})")
```

### Exchange Listing Tracker

```python
# Get all recent listings
listings = get_news_by_engine(engine_type="listing", limit=50)

# Group by exchange
by_exchange = {}
for article in listings:
    exchange = article['news_type']
    if exchange not in by_exchange:
        by_exchange[exchange] = []
    by_exchange[exchange].append(article)

# Display
for exchange, articles in by_exchange.items():
    print(f"\n{exchange} ({len(articles)} listings):")
    for a in articles:
        coins = ', '.join(a.get('coins', []))
        print(f"  • {coins}: {a['title']}")
```

### Smart Money Tracker

```python
# Combine prediction signals with on-chain data
predictions = get_news_by_engine(engine_type="prediction", limit=30)
onchain = get_news_by_engine(engine_type="onchain", limit=30)

# Filter for high-confidence smart money signals
smart_money = [
    p for p in predictions 
    if p['news_type'] == 'SMART_MONEY_TRADE' and p['ai_score'] >= 80
]

# Cross-reference with whale activity
whale_coins = set()
for article in onchain:
    whale_coins.update(article.get('coins', []))

# Find overlap
for signal in smart_money:
    signal_coins = signal.get('coins', [])
    overlap = set(signal_coins) & whale_coins
    if overlap:
        print(f"⚡ Smart money + whale activity: {', '.join(overlap)}")
        print(f"   {signal['summary_en']}")
```
