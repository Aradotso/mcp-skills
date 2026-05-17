---
name: alpaca-mcp-server-trading
description: Use Alpaca's MCP server to trade stocks, ETFs, crypto, and options through natural language in your IDE or AI assistant.
triggers:
  - "help me set up alpaca trading in my IDE"
  - "how do I configure the alpaca mcp server"
  - "place a stock trade using alpaca"
  - "get real-time market data with alpaca"
  - "configure alpaca for live trading"
  - "filter alpaca tools to only show stock data"
  - "trade options using alpaca mcp"
  - "get my alpaca account balance"
---

# Alpaca MCP Server Trading

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Alpaca MCP Server is an official Model Context Protocol server that enables AI assistants to trade stocks, ETFs, crypto, and options through natural language. Built with FastMCP and OpenAPI, it exposes Alpaca's Trading API as MCP tools in Claude Desktop, Cursor, VS Code, PyCharm, and other MCP clients.

**Key capabilities:**
- Real-time market data (quotes, trades, bars, snapshots)
- Order management (market, limit, stop, trailing-stop orders)
- Options trading (single-leg and multi-leg strategies)
- Crypto trading (BTC, ETH, and more)
- Account and portfolio management
- Watchlists and corporate actions
- Paper trading by default, live trading supported

## Installation

### Prerequisites

- Python 3.10+
- [uv](https://docs.astral.sh/uv/getting-started/installation/) package manager
- Alpaca API keys (free paper trading account at [alpaca.markets](https://alpaca.markets))

### Get API Keys

1. Sign up at [Alpaca Dashboard](https://app.alpaca.markets/paper/dashboard/overview)
2. Create a paper trading account
3. Generate API keys from the dashboard

### Configuration by MCP Client

#### Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "alpaca": {
      "command": "uvx",
      "args": ["alpaca-mcp-server"],
      "env": {
        "ALPACA_API_KEY": "your_alpaca_api_key",
        "ALPACA_SECRET_KEY": "your_alpaca_secret_key"
      }
    }
  }
}
```

Restart Claude Desktop.

#### Cursor

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "alpaca": {
      "command": "uvx",
      "args": ["alpaca-mcp-server"],
      "env": {
        "ALPACA_API_KEY": "your_alpaca_api_key",
        "ALPACA_SECRET_KEY": "your_alpaca_secret_key"
      }
    }
  }
}
```

Restart Cursor.

#### VS Code

Create `.vscode/mcp.json` in your project:

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
          "ALPACA_SECRET_KEY": "your_alpaca_secret_key"
        }
      }
    }
  }
}
```

Reload window.

#### PyCharm

1. Go to File → Settings → Tools → Model Context Protocol (MCP)
2. Add new server:
   - Type: stdio
   - Command: `uvx`
   - Arguments: `alpaca-mcp-server`
3. Set environment variables:
   ```
   ALPACA_API_KEY=your_alpaca_api_key
   ALPACA_SECRET_KEY=your_alpaca_secret_key
   ```

#### Docker

```bash
git clone https://github.com/alpacahq/alpaca-mcp-server.git
cd alpaca-mcp-server
docker build -t mcp/alpaca:latest .
```

Add to MCP client config:

```json
{
  "mcpServers": {
    "alpaca": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "ALPACA_API_KEY",
        "-e", "ALPACA_SECRET_KEY",
        "-e", "ALPACA_PAPER_TRADE=true",
        "mcp/alpaca:latest"
      ]
    }
  }
}
```

## Configuration

All configuration is via environment variables in your MCP client config:

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `ALPACA_API_KEY` | Yes | — | Alpaca API key |
| `ALPACA_SECRET_KEY` | Yes | — | Alpaca secret key |
| `ALPACA_PAPER_TRADE` | No | `true` | Set to `false` for live trading |
| `ALPACA_TOOLSETS` | No | all | Comma-separated toolsets to enable |

### Switching to Live Trading

**⚠️ WARNING: Live trading uses real money. Test thoroughly in paper trading first.**

Update env vars in your MCP client config:

```json
{
  "env": {
    "ALPACA_API_KEY": "your_live_api_key",
    "ALPACA_SECRET_KEY": "your_live_secret_key",
    "ALPACA_PAPER_TRADE": "false"
  }
}
```

Restart your MCP client.

### Toolset Filtering

Limit the server to specific capabilities by setting `ALPACA_TOOLSETS`:

```json
{
  "env": {
    "ALPACA_API_KEY": "your_alpaca_api_key",
    "ALPACA_SECRET_KEY": "your_alpaca_secret_key",
    "ALPACA_TOOLSETS": "stock-data,trading,account"
  }
}
```

**Available toolsets:**

- `account` — Account info, balances, portfolio history
- `trading` — Orders, positions, option exercises
- `watchlists` — Watchlist management
- `assets` — Asset lookup, option contracts, market calendar
- `stock-data` — Stock bars, quotes, trades, snapshots, screeners
- `crypto-data` — Crypto bars, quotes, trades, orderbooks
- `options-data` — Option bars, quotes, Greeks, chain data
- `corporate-actions` — Corporate action announcements
- `news` — News articles for stocks and crypto

## Common Usage Patterns

### Getting Account Information

Natural language prompts that work:
- "What's my account balance?"
- "Show my portfolio history for the last week"
- "What's my buying power?"

The AI will use tools like `get_account_info` and `get_portfolio_history`.

### Trading Stocks

**Market order:**
```
Buy 10 shares of AAPL at market price
```

**Limit order:**
```
Buy 5 shares of TSLA with a limit price of $250
```

**Stop-loss:**
```
Sell 3 shares of NVDA with a stop price of $800
```

**Bracket order:**
```
Buy 10 shares of GOOGL at market with a take-profit at $150 and stop-loss at $140
```

### Trading Crypto

```
Buy 0.1 BTC at market price
```

```
Place a limit order to buy 1 ETH at $3000
```

### Trading Options

**Find contracts:**
```
Show me call options for SPY expiring next Friday with strikes near $450
```

**Place option trade:**
```
Buy 1 call option for AAPL with strike $180 expiring in 30 days
```

**Multi-leg strategy:**
```
Create an iron condor for SPY with the following legs: buy 1 put at $440, sell 1 put at $445, sell 1 call at $455, buy 1 call at $460, all expiring next Friday
```

### Market Data

**Real-time quote:**
```
Get the current price of AAPL
```

**Historical bars:**
```
Show me daily price bars for TSLA for the last month
```

**Snapshot:**
```
Get a snapshot of NVDA including latest quote, trade, and daily bar
```

**Option chain:**
```
Show me the option chain for SPY expiring in 7 days
```

### Watchlists

```
Create a watchlist called "Tech Stocks" with AAPL, MSFT, GOOGL
```

```
Show me all my watchlists
```

```
Add TSLA to my "Tech Stocks" watchlist
```

## Programmatic Usage (Python SDK)

While the MCP server is designed for natural language use, you can also use the Alpaca Python SDK directly:

```python
from alpaca.trading.client import TradingClient
from alpaca.trading.requests import MarketOrderRequest
from alpaca.trading.enums import OrderSide, TimeInForce
import os

# Initialize client
client = TradingClient(
    api_key=os.environ["ALPACA_API_KEY"],
    secret_key=os.environ["ALPACA_SECRET_KEY"],
    paper=True  # Use paper trading
)

# Get account info
account = client.get_account()
print(f"Buying power: ${account.buying_power}")

# Place market order
order_data = MarketOrderRequest(
    symbol="AAPL",
    qty=10,
    side=OrderSide.BUY,
    time_in_force=TimeInForce.DAY
)
order = client.submit_order(order_data)
print(f"Order submitted: {order.id}")

# Get all positions
positions = client.get_all_positions()
for position in positions:
    print(f"{position.symbol}: {position.qty} shares @ ${position.avg_entry_price}")

# Cancel all orders
client.cancel_orders()
```

For market data:

```python
from alpaca.data.historical import StockHistoricalDataClient
from alpaca.data.requests import StockBarsRequest
from alpaca.data.timeframe import TimeFrame
from datetime import datetime, timedelta

# Initialize data client (no auth needed for free data)
data_client = StockHistoricalDataClient(
    api_key=os.environ["ALPACA_API_KEY"],
    secret_key=os.environ["ALPACA_SECRET_KEY"]
)

# Get historical bars
request_params = StockBarsRequest(
    symbol_or_symbols=["AAPL", "TSLA"],
    timeframe=TimeFrame.Day,
    start=datetime.now() - timedelta(days=30),
    end=datetime.now()
)
bars = data_client.get_stock_bars(request_params)

for symbol, bar_set in bars.items():
    print(f"\n{symbol}:")
    for bar in bar_set:
        print(f"  {bar.timestamp}: O=${bar.open} H=${bar.high} L=${bar.low} C=${bar.close}")
```

## Example Prompts

### Account & Portfolio
- "What's my current portfolio value?"
- "Show me my trading activity from yesterday"
- "What's my day trade count?"

### Orders
- "Cancel all my pending orders"
- "Show me all open orders for AAPL"
- "Replace my TSLA order with a new limit price of $245"

### Positions
- "What positions do I currently hold?"
- "Close my entire NVDA position"
- "What's my average entry price for MSFT?"

### Market Data
- "Get the latest trade for BTC/USD"
- "Show me hourly bars for SPY from last week"
- "What's the bid-ask spread for AAPL?"

### Options
- "Show me put options for QQQ with delta around -0.30"
- "Get Greeks for SPY call option with strike 450 expiring next Friday"
- "What are the latest option trade prices for AAPL?"

### Watchlists
- "Delete my 'Old Stocks' watchlist"
- "Remove TSLA from my 'Tech' watchlist"
- "Update my watchlist to include COIN"

## Troubleshooting

### Tools not appearing in MCP client

1. **Restart your MCP client** after updating config
2. **Start a fresh chat/session** to clear cached tool lists
3. **Verify config syntax** — JSON must be valid (use a JSON validator)
4. **Check logs** — Claude Desktop: View → Developer → Server Logs
5. **Test uvx** — Run `uvx alpaca-mcp-server --help` in terminal to verify installation

### Authentication errors

```
Error: Invalid API credentials
```

**Solution:**
- Verify `ALPACA_API_KEY` and `ALPACA_SECRET_KEY` are set correctly
- Ensure no extra quotes or whitespace in env vars
- Check if using paper keys with `ALPACA_PAPER_TRADE=true` or live keys with `false`
- Regenerate keys in Alpaca dashboard if needed

### Live trading not working

```
Error: This endpoint is not available for paper trading
```

**Solution:**
- Set `ALPACA_PAPER_TRADE=false` in env vars
- Use live trading API keys (not paper keys)
- Restart MCP client after changing config

### Specific tools not available

If certain tools are missing (e.g., options tools):

**Solution:**
- Remove or update `ALPACA_TOOLSETS` to include the toolset you need
- Default is all toolsets — if you've filtered, add back missing ones:
  ```json
  {
    "env": {
      "ALPACA_TOOLSETS": "account,trading,stock-data,options-data"
    }
  }
  ```

### Order placement fails

```
Error: Insufficient buying power
```

**Solution:**
- Check account balance with "What's my buying power?"
- Reduce order quantity
- In paper trading, you have $100k virtual cash by default

```
Error: Symbol not found
```

**Solution:**
- Verify symbol is correct (e.g., "AAPL" not "APPLE")
- Check if asset is tradable: "Is AAPL tradable?"
- Some assets may not be available in paper trading

### Rate limiting

```
Error: Rate limit exceeded
```

**Solution:**
- Alpaca has rate limits (200 requests/minute for paper, varies for live)
- Add delays between rapid requests
- Use batch endpoints where available (e.g., get multiple positions at once)

### Upgrading from V1

If you used alpaca-mcp-server V1:

1. **V2 is a complete rewrite** — tool names and parameters have changed
2. **Remove old config** — Delete any `.env` files or `init` commands from V1 setup
3. **Update MCP client config** — Use the V2 config format shown above
4. **Clear caches** — Restart MCP client and start fresh sessions
5. **Update prompts** — Remove references to V1-specific tool names
6. **To stay on V1:** Pin to `uvx alpaca-mcp-server==1.x.x serve` in config

### Docker issues

```
Error: Cannot connect to Docker daemon
```

**Solution:**
- Ensure Docker is running
- On macOS/Windows: Start Docker Desktop
- Test with `docker ps`

## Testing

The MCP server exposes tools dynamically — test by asking your AI assistant:

```
List all available Alpaca tools
```

Or verify directly with the MCP inspector (if your client supports it).

For paper trading, you can freely test all operations without risk. Recommended test flow:

1. "What's my account balance?"
2. "Buy 1 share of AAPL at market price"
3. "Show my current positions"
4. "Get a quote for AAPL"
5. "Cancel all orders"
6. "Close my AAPL position"

## Resources

- [Alpaca Trading API Documentation](https://docs.alpaca.markets/)
- [Alpaca Python SDK](https://alpaca.markets/sdks/python/)
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [FastMCP Framework](https://github.com/jlowin/fastmcp)
- [Alpaca Community Forum](https://forum.alpaca.markets/)
- [GitHub Repository](https://github.com/alpacahq/alpaca-mcp-server)

## Disclosure

This is a trading platform. Use paper trading for testing. Live trading involves real money and risk of loss. Past performance does not guarantee future results. Alpaca Securities LLC is a member of FINRA/SIPC. See [alpaca.markets/disclosures](https://alpaca.markets/disclosures) for important disclosures.
