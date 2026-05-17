---
name: alpaca-trading-mcp-server
description: Build AI-powered trading strategies and execute stock, crypto, and options trades using Alpaca's official MCP server with natural language commands
triggers:
  - help me trade stocks with Alpaca
  - set up Alpaca trading bot
  - analyze market data with Alpaca API
  - create trading strategy with MCP
  - execute options trades through Alpaca
  - get real-time crypto prices from Alpaca
  - build algorithmic trading system
  - query stock market data with AI
---

# Alpaca Trading MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Alpaca MCP Server is an official Model Context Protocol server that enables AI assistants to execute trades, analyze market data, and build trading strategies using natural language. It supports stocks, ETFs, crypto, and options trading through Alpaca's Trading API, with built-in support for paper trading and live trading modes.

## Prerequisites

- Python 3.10 or higher
- `uv` package installer
- Alpaca Trading API keys (free paper trading account available)
- An MCP-compatible client (Claude Desktop, Cursor, VS Code, etc.)

## Getting API Keys

1. Visit [Alpaca Dashboard](https://app.alpaca.markets/paper/dashboard/overview)
2. Create a free paper trading account
3. Navigate to API Keys section and generate new keys
4. Save both `API_KEY` and `SECRET_KEY` securely

## Installation & Configuration

### Claude Desktop

Edit your Claude Desktop config file:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

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

### Cursor

Add to `~/.cursor/mcp.json`:

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

### VS Code

Create `.vscode/mcp.json` in your project root:

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

### Docker Deployment

```bash
# Clone repository
git clone https://github.com/alpacahq/alpaca-mcp-server.git
cd alpaca-mcp-server

# Build image
docker build -t mcp/alpaca:latest .

# Add to MCP client config
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

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `ALPACA_API_KEY` | Yes | — | Your Alpaca API key |
| `ALPACA_SECRET_KEY` | Yes | — | Your Alpaca secret key |
| `ALPACA_PAPER_TRADE` | No | `true` | Use paper trading (`true`) or live trading (`false`) |
| `ALPACA_TOOLSETS` | No | all | Comma-separated toolsets to enable |

### Toolset Filtering

Restrict available tools by setting `ALPACA_TOOLSETS`:

```json
{
  "env": {
    "ALPACA_API_KEY": "...",
    "ALPACA_SECRET_KEY": "...",
    "ALPACA_TOOLSETS": "account,trading,stock-data"
  }
}
```

Available toolsets:
- `account` — Account info, balances, portfolio history
- `trading` — Orders, positions, exercise options
- `watchlists` — Manage watchlists
- `assets` — Asset lookup, option contracts, calendar
- `stock-data` — Stock quotes, bars, trades, screeners
- `crypto-data` — Crypto quotes, bars, trades, orderbooks
- `options-data` — Option chains, Greeks, quotes
- `corporate-actions` — Corporate action announcements
- `news` — Stock and crypto news

## Switching to Live Trading

**WARNING**: Live trading uses real money. Test thoroughly in paper trading first.

Update your MCP client config:

```json
{
  "env": {
    "ALPACA_API_KEY": "your_live_api_key",
    "ALPACA_SECRET_KEY": "your_live_secret_key",
    "ALPACA_PAPER_TRADE": "false"
  }
}
```

Restart your MCP client after changing configuration.

## Natural Language Examples

Once configured, you can use natural language prompts with your AI assistant:

### Account Management
- "What's my current account balance and buying power?"
- "Show me my portfolio history for the last week"
- "What trades did I make today?"

### Stock Trading
- "Buy 10 shares of AAPL at market price"
- "Place a limit order to sell 50 shares of TSLA at $250"
- "What's the current price of NVDA?"
- "Show me a 5-minute price chart for SPY from yesterday"

### Crypto Trading
- "What's the current Bitcoin price?"
- "Buy $500 worth of Ethereum"
- "Show me the order book for BTC/USD"

### Options Trading
- "Find call options for AAPL expiring next month with strike price $180"
- "What are the Greeks for SPY 450 calls expiring this Friday?"
- "Buy 1 contract of TSLA 220 call expiring in 30 days"

### Market Analysis
- "Find stocks with high volume today"
- "Get the latest news about Tesla"
- "What corporate actions are upcoming for my portfolio?"

## Key Capabilities

### Market Data Tools

**Stock Data**:
- Real-time and historical bars (1min to 1month timeframes)
- Latest quotes and trades
- Snapshot data for multiple symbols
- Stock screeners for discovery

**Crypto Data**:
- Real-time and historical crypto bars
- Latest quotes and trades
- Order book snapshots
- Support for major crypto pairs

**Options Data**:
- Option chain lookup by expiration/strike/type
- Real-time Greeks and implied volatility
- Latest quotes and trades
- Exchange code lookups

### Trading Operations

**Order Types**:
- Market orders
- Limit orders
- Stop orders
- Stop-limit orders
- Trailing-stop orders

**Order Management**:
- Submit new orders
- Cancel individual or all orders
- Modify existing orders
- Check order status and fills

**Position Management**:
- View open positions
- Close positions
- Exercise option contracts
- Track realized/unrealized P&L

### Account Features

**Portfolio**:
- Current balances and buying power
- Portfolio history with customizable timeframes
- Asset positions and allocations

**Activity Tracking**:
- Trade confirmations
- Account activities
- Order history

**Watchlists**:
- Create and manage watchlists
- Add/remove symbols
- Query watchlist contents

## Working with the Server Programmatically

While the MCP server is designed for AI assistants, you can also interact with it programmatically:

### Direct Python Integration

```python
import asyncio
from alpaca.trading.client import TradingClient
from alpaca.data.historical import StockHistoricalDataClient
from alpaca.data.requests import StockBarsRequest
from alpaca.data.timeframe import TimeFrame
from datetime import datetime, timedelta

# Initialize clients with environment variables
trading_client = TradingClient(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY'),
    paper=os.getenv('ALPACA_PAPER_TRADE', 'true').lower() == 'true'
)

data_client = StockHistoricalDataClient(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY')
)

# Get account information
account = trading_client.get_account()
print(f"Buying Power: ${account.buying_power}")
print(f"Cash: ${account.cash}")

# Fetch historical stock data
request_params = StockBarsRequest(
    symbol_or_symbols=["AAPL", "TSLA"],
    timeframe=TimeFrame.Day,
    start=datetime.now() - timedelta(days=30),
    end=datetime.now()
)

bars = data_client.get_stock_bars(request_params)
for symbol, bar_data in bars.items():
    print(f"{symbol}: {len(bar_data)} bars")
```

### Placing Orders

```python
from alpaca.trading.requests import MarketOrderRequest, LimitOrderRequest
from alpaca.trading.enums import OrderSide, TimeInForce

# Market order
market_order_data = MarketOrderRequest(
    symbol="AAPL",
    qty=10,
    side=OrderSide.BUY,
    time_in_force=TimeInForce.DAY
)
market_order = trading_client.submit_order(market_order_data)
print(f"Market order placed: {market_order.id}")

# Limit order
limit_order_data = LimitOrderRequest(
    symbol="TSLA",
    limit_price=250.00,
    qty=5,
    side=OrderSide.SELL,
    time_in_force=TimeInForce.GTC
)
limit_order = trading_client.submit_order(limit_order_data)
print(f"Limit order placed: {limit_order.id}")
```

### Working with Options

```python
from alpaca.trading.requests import GetOptionContractsRequest
from alpaca.trading.enums import ContractType
from datetime import datetime, timedelta

# Find option contracts
option_request = GetOptionContractsRequest(
    underlying_symbols=["AAPL"],
    expiration_date_gte=datetime.now().date(),
    expiration_date_lte=(datetime.now() + timedelta(days=60)).date(),
    type=ContractType.CALL,
    strike_price_gte=170,
    strike_price_lte=180
)

contracts = trading_client.get_option_contracts(option_request)
for contract in contracts:
    print(f"{contract.symbol}: Strike ${contract.strike_price}, "
          f"Expires {contract.expiration_date}")
```

### Streaming Real-Time Data

```python
from alpaca.data.live import StockDataStream

# Initialize streaming client
stream = StockDataStream(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY')
)

# Define handlers
async def quote_handler(data):
    print(f"{data.symbol}: Bid ${data.bid_price} Ask ${data.ask_price}")

async def trade_handler(data):
    print(f"{data.symbol}: Trade ${data.price} Size {data.size}")

# Subscribe to streams
stream.subscribe_quotes(quote_handler, "AAPL", "TSLA")
stream.subscribe_trades(trade_handler, "AAPL", "TSLA")

# Run stream
stream.run()
```

## Common Patterns

### Building a Simple Trading Bot

```python
from alpaca.trading.client import TradingClient
from alpaca.data.historical import StockHistoricalDataClient
from alpaca.data.requests import StockBarsRequest
from alpaca.data.timeframe import TimeFrame
from alpaca.trading.requests import MarketOrderRequest
from alpaca.trading.enums import OrderSide, TimeInForce
from datetime import datetime, timedelta
import pandas as pd

class SimpleMomentumBot:
    def __init__(self, api_key, secret_key, paper=True):
        self.trading_client = TradingClient(api_key, secret_key, paper=paper)
        self.data_client = StockHistoricalDataClient(api_key, secret_key)
        
    def get_momentum(self, symbol, days=10):
        """Calculate simple momentum indicator"""
        request = StockBarsRequest(
            symbol_or_symbols=[symbol],
            timeframe=TimeFrame.Day,
            start=datetime.now() - timedelta(days=days*2),
            end=datetime.now()
        )
        bars = self.data_client.get_stock_bars(request)
        df = bars.df.reset_index()
        
        # Calculate momentum: % change over period
        current_price = df['close'].iloc[-1]
        past_price = df['close'].iloc[-days]
        return ((current_price - past_price) / past_price) * 100
    
    def execute_strategy(self, symbol, threshold=5.0):
        """Buy if momentum > threshold, sell if momentum < -threshold"""
        momentum = self.get_momentum(symbol)
        position = self.get_position(symbol)
        
        if momentum > threshold and position is None:
            # Buy signal
            order = MarketOrderRequest(
                symbol=symbol,
                qty=10,
                side=OrderSide.BUY,
                time_in_force=TimeInForce.DAY
            )
            return self.trading_client.submit_order(order)
            
        elif momentum < -threshold and position is not None:
            # Sell signal
            return self.trading_client.close_position(symbol)
    
    def get_position(self, symbol):
        """Get current position for symbol"""
        try:
            return self.trading_client.get_open_position(symbol)
        except:
            return None

# Usage
bot = SimpleMomentumBot(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY'),
    paper=True
)
order = bot.execute_strategy("AAPL", threshold=5.0)
```

### Portfolio Analysis

```python
from alpaca.trading.client import TradingClient
import pandas as pd

def analyze_portfolio(api_key, secret_key, paper=True):
    """Analyze current portfolio holdings"""
    client = TradingClient(api_key, secret_key, paper=paper)
    
    # Get account and positions
    account = client.get_account()
    positions = client.get_all_positions()
    
    # Calculate metrics
    total_equity = float(account.equity)
    positions_data = []
    
    for position in positions:
        positions_data.append({
            'Symbol': position.symbol,
            'Qty': float(position.qty),
            'Avg Entry': float(position.avg_entry_price),
            'Current Price': float(position.current_price),
            'Market Value': float(position.market_value),
            'P&L': float(position.unrealized_pl),
            'P&L %': float(position.unrealized_plpc) * 100,
            'Weight %': (float(position.market_value) / total_equity) * 100
        })
    
    df = pd.DataFrame(positions_data)
    
    print(f"\nPortfolio Summary")
    print(f"Total Equity: ${total_equity:,.2f}")
    print(f"Cash: ${float(account.cash):,.2f}")
    print(f"Buying Power: ${float(account.buying_power):,.2f}")
    print(f"\nPositions:")
    print(df.to_string(index=False))
    print(f"\nTotal P&L: ${df['P&L'].sum():,.2f}")
    
    return df

# Usage
df = analyze_portfolio(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY')
)
```

### Risk Management Helper

```python
from alpaca.trading.client import TradingClient

class RiskManager:
    def __init__(self, api_key, secret_key, paper=True):
        self.client = TradingClient(api_key, secret_key, paper=paper)
        
    def check_position_limits(self, symbol, qty, max_position_pct=10):
        """Verify position doesn't exceed maximum portfolio percentage"""
        account = self.client.get_account()
        total_equity = float(account.equity)
        
        # Get current price
        from alpaca.data.historical import StockHistoricalDataClient
        from alpaca.data.requests import StockLatestQuoteRequest
        
        data_client = StockHistoricalDataClient(
            self.client.api_key,
            self.client.secret_key
        )
        request = StockLatestQuoteRequest(symbol_or_symbols=[symbol])
        quote = data_client.get_stock_latest_quote(request)[symbol]
        
        position_value = float(qty) * float(quote.ask_price)
        position_pct = (position_value / total_equity) * 100
        
        if position_pct > max_position_pct:
            raise ValueError(
                f"Position would be {position_pct:.1f}% of portfolio, "
                f"exceeds {max_position_pct}% limit"
            )
        
        return True
    
    def check_buying_power(self, symbol, qty):
        """Verify sufficient buying power for trade"""
        account = self.client.get_account()
        
        # Get current price
        from alpaca.data.historical import StockHistoricalDataClient
        from alpaca.data.requests import StockLatestQuoteRequest
        
        data_client = StockHistoricalDataClient(
            self.client.api_key,
            self.client.secret_key
        )
        request = StockLatestQuoteRequest(symbol_or_symbols=[symbol])
        quote = data_client.get_stock_latest_quote(request)[symbol]
        
        cost = float(qty) * float(quote.ask_price)
        buying_power = float(account.buying_power)
        
        if cost > buying_power:
            raise ValueError(
                f"Insufficient buying power. Cost: ${cost:.2f}, "
                f"Available: ${buying_power:.2f}"
            )
        
        return True

# Usage
risk_mgr = RiskManager(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY')
)

try:
    risk_mgr.check_position_limits("AAPL", 100, max_position_pct=10)
    risk_mgr.check_buying_power("AAPL", 100)
    print("Trade passes risk checks")
except ValueError as e:
    print(f"Risk check failed: {e}")
```

## Troubleshooting

### Server Not Appearing in MCP Client

1. **Verify installation**: Run `uvx alpaca-mcp-server --version` in terminal
2. **Check config syntax**: Validate JSON in your MCP client config file
3. **Restart client**: Completely quit and restart your MCP client
4. **Check logs**: 
   - Claude Desktop: `~/Library/Logs/Claude/mcp*.log` (Mac) or `%APPDATA%\Claude\logs\` (Windows)
   - Cursor/VS Code: Check developer console

### Authentication Errors

```
Error: Invalid API credentials
```

**Solutions**:
- Verify API keys are correct and copied completely
- Check for extra spaces in config file
- Ensure you're using paper trading keys with `ALPACA_PAPER_TRADE=true`
- Regenerate keys in Alpaca dashboard if needed

### Tool Not Found Errors

```
Error: Tool 'xyz' not found
```

**Solutions**:
- You may be using V1 tool names with V2 server
- Clear your MCP client cache and restart
- Start a fresh chat/session
- Check `ALPACA_TOOLSETS` if you've restricted available tools

### Rate Limiting

```
Error: Rate limit exceeded
```

**Solutions**:
- Reduce frequency of requests
- Add delays between operations
- Upgrade Alpaca account tier for higher limits
- Use batch operations where available

### Paper Trading vs Live Trading Confusion

If trades aren't appearing:
- Verify `ALPACA_PAPER_TRADE` setting matches your intention
- Check you're looking at correct dashboard (paper vs live)
- Confirm API keys match the trading mode

### Docker Issues

```
Error: Cannot connect to Docker daemon
```

**Solutions**:
- Ensure Docker Desktop is running
- Check Docker daemon is accessible
- Verify image was built successfully: `docker images | grep alpaca`
- Check container logs: `docker logs <container_id>`

### Order Rejection

```
Error: Order rejected - insufficient buying power
```

**Solutions**:
- Check account buying power before placing order
- Verify order quantity and price are correct
- Ensure market is open for the asset type
- Check for pending orders that may be reserving funds

## Best Practices

1. **Always test in paper trading first**: Never use live trading without thorough testing
2. **Implement risk management**: Set position limits and stop losses
3. **Handle errors gracefully**: Market conditions change; be prepared for rejections
4. **Monitor rate limits**: Space out requests to avoid throttling
5. **Use appropriate timeframes**: Match data granularity to your strategy needs
6. **Validate data**: Check for missing bars or stale quotes
7. **Keep credentials secure**: Never commit API keys to version control
8. **Use environment variables**: Store configuration in env vars, not code
9. **Log operations**: Track all trades and decisions for analysis
10. **Start small**: Begin with small position sizes while learning

## Additional Resources

- [Alpaca Documentation](https://docs.alpaca.markets/)
- [Alpaca Python SDK](https://alpaca.markets/sdks/python/)
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [Alpaca API Reference](https://docs.alpaca.markets/reference/)
- [Alpaca Community Forum](https://forum.alpaca.markets/)
- [Alpaca Slack Community](https://alpaca.markets/slack)
