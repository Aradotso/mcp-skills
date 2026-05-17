---
name: alpaca-trading-mcp
description: Trade stocks, ETFs, crypto, and options through Alpaca's MCP server using natural language in AI assistants
triggers:
  - "place a trade on Alpaca"
  - "check my Alpaca portfolio"
  - "get real-time stock quotes"
  - "create an options strategy"
  - "analyze market data with Alpaca"
  - "manage my trading watchlist"
  - "check crypto prices on Alpaca"
  - "place a limit order"
---

# Alpaca Trading MCP

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Alpaca MCP Server is an official Model Context Protocol server that enables natural language trading operations through AI assistants. It provides comprehensive access to Alpaca's Trading API for stocks, options, crypto, portfolio management, and real-time market data. Built with FastMCP and OpenAPI, this is a complete v2 rewrite with spec-aligned tooling.

## What It Does

- **Market Data Access**: Real-time and historical quotes, trades, bars for stocks, crypto, and options
- **Trading Operations**: Place market, limit, stop, trailing-stop orders for any supported asset
- **Options Trading**: Search contracts, place single/multi-leg strategies, get Greeks and IV
- **Portfolio Management**: View account info, positions, buying power, portfolio history
- **Watchlist Management**: Create, update, and manage custom watchlists
- **News & Corporate Actions**: Access market news and corporate action announcements

## Installation

### Prerequisites

- Python 3.10+
- uv package manager: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Alpaca API keys (free paper trading account at https://app.alpaca.markets)
- An MCP-compatible client (Claude Desktop, Cursor, VS Code, etc.)

### Configuration

Add to your MCP client configuration file:

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json` on Mac):

```json
{
  "mcpServers": {
    "alpaca": {
      "command": "uvx",
      "args": ["alpaca-mcp-server"],
      "env": {
        "ALPACA_API_KEY": "your_alpaca_api_key",
        "ALPACA_SECRET_KEY": "your_alpaca_secret_key",
        "ALPACA_PAPER_TRADE": "true"
      }
    }
  }
}
```

**Cursor** (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "alpaca": {
      "command": "uvx",
      "args": ["alpaca-mcp-server"],
      "env": {
        "ALPACA_API_KEY": "your_alpaca_api_key",
        "ALPACA_SECRET_KEY": "your_alpaca_secret_key",
        "ALPACA_PAPER_TRADE": "true"
      }
    }
  }
}
```

**VS Code** (`.vscode/mcp.json` in project root):

```json
{
  "mcp": {
    "servers": {
      "alpaca": {
        "type": "stdio",
        "command": "uvx",
        "args": ["alpaca-mcp-server"],
        "env": {
          "ALPACA_API_KEY": "your_alpaca_api_key",
          "ALPACA_SECRET_KEY": "your_alpaca_secret_key",
          "ALPACA_PAPER_TRADE": "true"
        }
      }
    }
  }
}
```

**Claude Code CLI**:

```bash
claude mcp add alpaca --scope user --transport stdio uvx alpaca-mcp-server \
  --env ALPACA_API_KEY=your_alpaca_api_key \
  --env ALPACA_SECRET_KEY=your_alpaca_secret_key \
  --env ALPACA_PAPER_TRADE=true
```

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `ALPACA_API_KEY` | Yes | — | Your Alpaca API key |
| `ALPACA_SECRET_KEY` | Yes | — | Your Alpaca secret key |
| `ALPACA_PAPER_TRADE` | No | `true` | Set to `false` for live trading |
| `ALPACA_TOOLSETS` | No | all | Comma-separated toolsets to enable |

### Toolset Filtering

Restrict available tools by setting `ALPACA_TOOLSETS`:

```json
{
  "env": {
    "ALPACA_TOOLSETS": "account,trading,stock-data"
  }
}
```

Available toolsets:
- `account`: Account info, portfolio history, activities
- `trading`: Orders, positions, options exercise
- `watchlists`: Watchlist CRUD operations
- `assets`: Asset lookup, option contracts, market calendar
- `stock-data`: Stock bars, quotes, trades, snapshots
- `crypto-data`: Crypto bars, quotes, trades, orderbooks
- `options-data`: Option bars, quotes, Greeks, chain
- `corporate-actions`: Corporate action announcements
- `news`: News articles

## Core Trading Operations

### Check Account Status

```python
# Natural language prompts:
# "What's my current account balance?"
# "Show my buying power"
# "Am I approved for options trading?"

# The MCP server exposes get_account_info tool
# Returns: account_number, status, buying_power, cash, portfolio_value, etc.
```

### Place Stock Orders

```python
# Natural language prompts:
# "Buy 100 shares of AAPL at market price"
# "Place a limit order to buy TSLA at $250"
# "Sell 50 shares of NVDA with a trailing stop at 2%"

# Market order
# Tool: post_orders
# Parameters: {
#   "symbol": "AAPL",
#   "qty": 100,
#   "side": "buy",
#   "type": "market",
#   "time_in_force": "day"
# }

# Limit order
# Parameters: {
#   "symbol": "TSLA",
#   "qty": 10,
#   "side": "buy",
#   "type": "limit",
#   "limit_price": 250.00,
#   "time_in_force": "gtc"
# }

# Trailing stop
# Parameters: {
#   "symbol": "NVDA",
#   "qty": 50,
#   "side": "sell",
#   "type": "trailing_stop",
#   "trail_percent": 2.0,
#   "time_in_force": "gtc"
# }
```

### Get Real-Time Market Data

```python
# Natural language prompts:
# "What's the current price of AAPL?"
# "Show me the latest BTC quote"
# "Get today's price bars for MSFT in 5-minute intervals"

# Latest quote
# Tool: get_stocks_quotes_latest
# Parameters: {"symbols": "AAPL,MSFT,GOOGL"}

# Crypto quote
# Tool: get_crypto_quotes_latest
# Parameters: {"symbols": "BTC/USD,ETH/USD"}

# Historical bars
# Tool: get_stocks_bars
# Parameters: {
#   "symbols": "MSFT",
#   "timeframe": "5Min",
#   "start": "2025-05-01T09:30:00Z",
#   "end": "2025-05-01T16:00:00Z"
# }
```

### Options Trading

```python
# Natural language prompts:
# "Find call options for AAPL expiring next month"
# "Place a covered call on my 100 shares of NVDA"
# "Get the Greeks for SPY 500 calls expiring this Friday"

# Search option contracts
# Tool: get_options_contracts
# Parameters: {
#   "underlying_symbols": "AAPL",
#   "expiration_date_gte": "2025-06-01",
#   "expiration_date_lte": "2025-06-30",
#   "type": "call"
# }

# Place option order
# Tool: post_orders
# Parameters: {
#   "symbol": "AAPL250620C00150000",
#   "qty": 1,
#   "side": "sell_to_open",
#   "type": "limit",
#   "limit_price": 5.50,
#   "time_in_force": "day",
#   "class": "option"
# }

# Get option snapshot with Greeks
# Tool: get_options_snapshots
# Parameters: {"symbols": "SPY250516C00500000"}
# Returns: greeks (delta, gamma, theta, vega), implied_volatility, latest_quote
```

### Manage Positions

```python
# Natural language prompts:
# "Show all my open positions"
# "What's my position in AAPL?"
# "Close half of my TSLA position"

# Get all positions
# Tool: get_positions

# Get specific position
# Tool: get_positions_by_symbol_or_asset_id
# Parameters: {"symbol_or_asset_id": "AAPL"}

# Close position
# Tool: delete_positions_by_symbol_or_asset_id
# Parameters: {
#   "symbol_or_asset_id": "TSLA",
#   "percentage": 50  # Optional: close percentage
# }
```

### Watchlist Management

```python
# Natural language prompts:
# "Create a tech stocks watchlist with AAPL, MSFT, GOOGL"
# "Add TSLA to my tech watchlist"
# "Show me all my watchlists"

# Create watchlist
# Tool: post_watchlists
# Parameters: {
#   "name": "Tech Stocks",
#   "symbols": ["AAPL", "MSFT", "GOOGL"]
# }

# Get all watchlists
# Tool: get_watchlists

# Add symbol to watchlist
# Tool: post_watchlists_by_watchlist_id
# Parameters: {
#   "watchlist_id": "uuid-here",
#   "symbol": "TSLA"
# }
```

## Advanced Market Data Queries

### Screen Stocks

```python
# Natural language prompts:
# "Find the most active stocks today"
# "Show me top gainers with volume over 1M"

# Tool: get_screener_stocks_most_actives
# Parameters: {
#   "by": "volume",
#   "top": 10
# }

# Tool: get_screener_stocks_movers
# Parameters: {
#   "market_type": "stocks",
#   "top": 20
# }
```

### Get News

```python
# Natural language prompts:
# "Show me latest news about AAPL"
# "Get crypto news for Bitcoin"

# Stock news
# Tool: get_news
# Parameters: {
#   "symbols": "AAPL",
#   "limit": 10,
#   "sort": "desc"
# }

# Crypto news
# Tool: get_crypto_news
# Parameters: {
#   "symbols": "BTC",
#   "limit": 10
# }
```

### Corporate Actions

```python
# Natural language prompts:
# "Are there any upcoming stock splits?"
# "Show dividend announcements for next week"

# Tool: get_corporate_actions_announcements
# Parameters: {
#   "ca_types": "dividend,split",
#   "since": "2025-05-01",
#   "until": "2025-05-31"
# }
```

## Portfolio Analysis

### Get Portfolio History

```python
# Natural language prompts:
# "Show my portfolio performance over the last month"
# "Graph my portfolio value for the past year"

# Tool: get_account_portfolio_history
# Parameters: {
#   "period": "1M",
#   "timeframe": "1D"
# }
# Returns: timestamp, equity, profit_loss, profit_loss_pct arrays
```

### Account Activities

```python
# Natural language prompts:
# "Show my recent trades"
# "List all dividend payments this month"

# Tool: get_account_activities
# Parameters: {
#   "activity_types": "FILL,DIV",
#   "after": "2025-05-01T00:00:00Z",
#   "direction": "desc"
# }
```

## Common Patterns

### Multi-Symbol Quotes

```python
# Get quotes for multiple symbols at once
# "Get current prices for AAPL, MSFT, GOOGL, AMZN"
# Tool: get_stocks_quotes_latest
# Parameters: {"symbols": "AAPL,MSFT,GOOGL,AMZN"}
# Use feed parameter for realtime: "feed": "sip"
```

### Batch Order Placement

```python
# Place multiple orders together
# "Buy 10 shares each of AAPL, MSFT, GOOGL at market"
# Call post_orders tool 3 times or use a loop pattern
# Each order is submitted individually through the API
```

### Conditional Order Logic

```python
# Complex strategies require multiple steps
# "If AAPL drops below $150, buy 100 shares"
# 1. Monitor current price with get_stocks_quotes_latest
# 2. When condition met, call post_orders
# Note: Alpaca supports bracket/OCO orders natively:
# Use order_class: "bracket" with take_profit and stop_loss
```

### Options Strategies

```python
# Multi-leg options spreads
# "Create a bull call spread on SPY"
# Tool: post_orders with order_class: "multileg"
# Parameters: {
#   "symbol": "SPY",
#   "type": "limit",
#   "time_in_force": "day",
#   "order_class": "multileg",
#   "legs": [
#     {"symbol": "SPY250620C00500000", "side": "buy_to_open", "qty": 1},
#     {"symbol": "SPY250620C00510000", "side": "sell_to_open", "qty": 1}
#   ]
# }
```

## Troubleshooting

### Server Not Showing in MCP Client

1. **Restart your MCP client** after editing config
2. **Check JSON syntax** in your config file (trailing commas cause issues)
3. **Verify uv is installed**: `uv --version`
4. **Check logs** in client's developer console/logs

### Authentication Errors

```
Error: 401 Unauthorized
```

- Verify `ALPACA_API_KEY` and `ALPACA_SECRET_KEY` are correct
- Ensure you're using paper trading keys if `ALPACA_PAPER_TRADE=true`
- Check if keys have necessary permissions in Alpaca dashboard

### Tool Not Found

```
Error: Tool 'old_tool_name' not found
```

- **You may be using v1 tool names**. V2 is a complete rewrite.
- Clear MCP client cache and restart
- Start a fresh chat session
- Check [Available Tools](#available-tools) for current tool names

### Rate Limiting

```
Error: 429 Too Many Requests
```

- Alpaca API has rate limits (200 requests/minute for paper trading)
- Batch symbol queries when possible (comma-separated symbols)
- Add delays between rapid tool calls
- Upgrade to live account for higher limits

### Order Rejected

```
Error: insufficient buying power / position limit exceeded
```

- Check account buying power: `get_account_info`
- Verify position limits haven't been hit
- For options, ensure account has appropriate trading level
- Check if market is open: `get_clock`

### Market Data Gaps

```
Error: no data found for timeframe
```

- Market may be closed (check `get_clock`)
- Symbol may not have data for requested timeframe
- Use `feed: "sip"` for consolidated US market data
- Crypto trades 24/7, stocks only during market hours (9:30-16:00 ET)

### V1 to V2 Migration

If upgrading from v1:
1. **Remove any .env files** — v2 uses env vars in MCP config only
2. **Delete v1 tool references** in custom instructions/rules
3. **Restart MCP client** to clear tool cache
4. **Start fresh session** so LLM discovers new tools
5. **Update toolset filtering** using `ALPACA_TOOLSETS` env var

### Docker Issues

```bash
# If server fails to start in Docker
docker logs <container-id>

# Check environment variables are passed
docker run --rm -e ALPACA_API_KEY=test mcp/alpaca:latest env | grep ALPACA

# Test connection manually
docker run --rm -i \
  -e ALPACA_API_KEY=your_key \
  -e ALPACA_SECRET_KEY=your_secret \
  -e ALPACA_PAPER_TRADE=true \
  mcp/alpaca:latest
```

## Live Trading

To switch from paper to live trading:

```json
{
  "env": {
    "ALPACA_API_KEY": "your_live_api_key",
    "ALPACA_SECRET_KEY": "your_live_secret_key",
    "ALPACA_PAPER_TRADE": "false"
  }
}
```

**⚠️ WARNING**: Live trading uses real money. Test thoroughly in paper trading first. Markets are volatile and losses can exceed expectations.

## Additional Resources

- **Alpaca API Docs**: https://docs.alpaca.markets/
- **Paper Trading Dashboard**: https://app.alpaca.markets/paper/dashboard/overview
- **Live Trading Dashboard**: https://app.alpaca.markets/brokerage/dashboard/overview
- **Python SDK**: https://alpaca.markets/sdks/python/
- **MCP Specification**: https://modelcontextprotocol.io/
- **Community Forum**: https://forum.alpaca.markets/
- **Slack Community**: https://alpaca.markets/slack

## License

MIT License - Alpaca Markets
