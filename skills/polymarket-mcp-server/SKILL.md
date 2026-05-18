---
name: polymarket-mcp-server
description: Enable Claude to trade, analyze, and manage Polymarket prediction markets with 45 AI-powered tools, real-time monitoring, and enterprise-grade safety features
triggers:
  - "help me trade on Polymarket"
  - "analyze prediction markets"
  - "set up Polymarket trading bot"
  - "monitor my Polymarket positions"
  - "find trending prediction markets"
  - "execute trades on Polymarket with Claude"
  - "manage my Polymarket portfolio"
  - "get real-time Polymarket market data"
---

# Polymarket MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

AI-powered MCP server that enables Claude to autonomously trade, analyze, and manage positions on Polymarket prediction markets. Provides 45 comprehensive tools across market discovery, analysis, trading, portfolio management, and real-time monitoring with WebSocket support.

## Installation

### Quick Start (DEMO Mode - No Wallet Required)

```bash
# Clone and install
git clone https://github.com/caiovicentino/polymarket-mcp-server.git
cd polymarket-mcp-server
./quickstart.sh

# Or automated one-liner
curl -sSL https://raw.githubusercontent.com/caiovicentino/polymarket-mcp-server/main/quickstart.sh | bash
```

### Full Installation (Trading Enabled)

```bash
# Use automated installer
./install.sh

# Or manual installation
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -e .
```

### Configuration

Create `.env` file:

```bash
cp .env.example .env
```

**DEMO Mode (Read-Only):**
```env
DEMO_MODE=true
```

**Full Trading Mode:**
```env
# Required
POLYGON_PRIVATE_KEY=${POLYGON_PRIVATE_KEY}
POLYGON_ADDRESS=${POLYGON_ADDRESS}

# Optional Safety Limits
MAX_ORDER_SIZE_USD=1000
MAX_TOTAL_EXPOSURE_USD=5000
MAX_POSITION_SIZE_PER_MARKET=2000
MIN_LIQUIDITY_REQUIRED=10000
MAX_SPREAD_TOLERANCE=0.05
REQUIRE_CONFIRMATION_ABOVE_USD=500
ENABLE_AUTONOMOUS_TRADING=true
```

### Claude Desktop Integration

Add to `claude_desktop_config.json`:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`  
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "polymarket": {
      "command": "/absolute/path/to/venv/bin/python",
      "args": ["-m", "polymarket_mcp.server"],
      "cwd": "/absolute/path/to/polymarket-mcp-server",
      "env": {
        "POLYGON_PRIVATE_KEY": "${POLYGON_PRIVATE_KEY}",
        "POLYGON_ADDRESS": "${POLYGON_ADDRESS}"
      }
    }
  }
}
```

## Tool Categories

### 1. Market Discovery (8 Tools)

#### Search Markets
```python
# Tool: search_markets
{
  "query": "Trump election",
  "limit": 10,
  "offset": 0
}
```

#### Get Trending Markets
```python
# Tool: get_trending_markets
{
  "period": "24h",  # "24h", "7d", or "30d"
  "limit": 20
}
```

#### Markets Closing Soon
```python
# Tool: get_markets_closing_soon
{
  "hours": 24,
  "limit": 10
}
```

#### Category-Specific Markets
```python
# Tool: get_markets_by_category
{
  "category": "Politics",  # Politics, Sports, Crypto, Pop Culture
  "limit": 20
}
```

### 2. Market Analysis (10 Tools)

#### Get Market Details
```python
# Tool: get_market_details
{
  "market_id": "0x1234..."
}
```

#### Analyze Market Opportunity (AI-Powered)
```python
# Tool: analyze_market_opportunity
{
  "market_id": "0x1234...",
  "analysis_depth": "deep"  # "quick" or "deep"
}
# Returns: BUY/SELL/HOLD recommendation with reasoning
```

#### Get Orderbook Depth
```python
# Tool: get_orderbook_depth
{
  "token_id": "0x5678..."
}
```

#### Compare Markets
```python
# Tool: compare_markets
{
  "market_ids": ["0x1234...", "0x5678...", "0x9abc..."]
}
```

#### Calculate Spread
```python
# Tool: calculate_spread
{
  "token_id": "0x5678..."
}
```

### 3. Trading (12 Tools)

#### Place Limit Order
```python
# Tool: place_limit_order
{
  "token_id": "0x5678...",
  "side": "BUY",  # or "SELL"
  "price": 0.65,
  "size": 100,
  "time_in_force": "GTC"  # GTC, GTD, FOK, FAK
}
```

#### Place Market Order
```python
# Tool: place_market_order
{
  "token_id": "0x5678...",
  "side": "BUY",
  "amount": 50
}
```

#### Smart Trade (AI-Powered)
```python
# Tool: smart_trade
{
  "instruction": "Buy $200 worth of Yes on Trump 2024 election if price below 0.60",
  "strategy": "passive"  # aggressive, passive, or mid
}
# Automatically parses instruction and executes optimal trade
```

#### Get AI Suggested Prices
```python
# Tool: get_ai_suggested_prices
{
  "token_id": "0x5678...",
  "side": "BUY",
  "strategy": "mid"  # aggressive, passive, or mid
}
```

#### Cancel Order
```python
# Tool: cancel_order
{
  "order_id": "0xabcd..."
}
```

#### Get Open Orders
```python
# Tool: get_open_orders
{
  "market_id": "0x1234..."  # Optional filter
}
```

### 4. Portfolio Management (8 Tools)

#### Get Portfolio Summary
```python
# Tool: get_portfolio_summary
{}
# Returns: total value, positions, P&L, risk metrics
```

#### Get Active Positions
```python
# Tool: get_active_positions
{
  "market_id": "0x1234..."  # Optional filter
}
```

#### Calculate P&L
```python
# Tool: calculate_pnl
{
  "position_id": "0x5678...",
  "include_unrealized": true
}
```

#### Analyze Portfolio Risk
```python
# Tool: analyze_portfolio_risk
{}
# Returns: concentration risk, liquidity risk, diversification score
```

#### Optimize Portfolio (AI-Powered)
```python
# Tool: optimize_portfolio
{
  "risk_profile": "balanced",  # conservative, balanced, or aggressive
  "target_allocation": {
    "Politics": 0.4,
    "Sports": 0.3,
    "Crypto": 0.3
  }
}
# Returns: rebalancing suggestions with rationale
```

#### Get Trade History
```python
# Tool: get_trade_history
{
  "start_date": "2024-01-01",
  "end_date": "2024-12-31",
  "market_id": "0x1234..."  # Optional
}
```

### 5. Real-Time Monitoring (7 Tools)

#### Subscribe to Market Updates
```python
# Tool: subscribe_to_market
{
  "market_id": "0x1234...",
  "channels": ["price", "orderbook", "trades"]
}
```

#### Subscribe to User Orders
```python
# Tool: subscribe_to_user_orders
{}
# Monitors all your order status changes
```

#### Get WebSocket Status
```python
# Tool: get_websocket_status
{}
# Returns: connection state, subscriptions, message counts
```

#### Unsubscribe from Market
```python
# Tool: unsubscribe_from_market
{
  "market_id": "0x1234..."
}
```

## Python API Usage

### Direct Python Integration

```python
from polymarket_mcp.client import PolymarketClient
from decimal import Decimal
import os

# Initialize client
client = PolymarketClient(
    private_key=os.getenv("POLYGON_PRIVATE_KEY"),
    polygon_address=os.getenv("POLYGON_ADDRESS")
)

# Search markets
markets = client.search_markets(query="election", limit=5)
for market in markets:
    print(f"{market['question']} - Volume: ${market['volume24h']}")

# Get market analysis
market_id = markets[0]['id']
analysis = client.analyze_market_opportunity(
    market_id=market_id,
    analysis_depth="deep"
)
print(f"Recommendation: {analysis['recommendation']}")
print(f"Reasoning: {analysis['reasoning']}")

# Place order with safety checks
token_id = markets[0]['tokens'][0]['id']
order = client.place_limit_order(
    token_id=token_id,
    side="BUY",
    price=Decimal("0.55"),
    size=100,
    time_in_force="GTC"
)
print(f"Order placed: {order['id']}")

# Monitor portfolio
portfolio = client.get_portfolio_summary()
print(f"Total Value: ${portfolio['total_value_usd']}")
print(f"Unrealized P&L: ${portfolio['unrealized_pnl']}")

# Get real-time updates via WebSocket
client.subscribe_to_market(
    market_id=market_id,
    channels=["price", "orderbook"]
)
```

### Trading Strategies Example

```python
from polymarket_mcp.client import PolymarketClient
from decimal import Decimal

class SimpleArbitrageBot:
    def __init__(self, client: PolymarketClient):
        self.client = client
        self.min_spread = Decimal("0.05")  # 5% minimum spread
    
    def find_arbitrage_opportunities(self):
        """Find markets with high spreads"""
        trending = self.client.get_trending_markets(period="24h", limit=50)
        
        opportunities = []
        for market in trending:
            for token in market['tokens']:
                spread = self.client.calculate_spread(token['id'])
                
                if spread['spread_percentage'] > float(self.min_spread):
                    opportunities.append({
                        'market': market['question'],
                        'token_id': token['id'],
                        'spread': spread['spread_percentage'],
                        'best_bid': spread['best_bid'],
                        'best_ask': spread['best_ask']
                    })
        
        return sorted(opportunities, key=lambda x: x['spread'], reverse=True)
    
    def execute_trade(self, opportunity):
        """Execute trade on identified opportunity"""
        # Get AI-suggested optimal price
        suggested = self.client.get_ai_suggested_prices(
            token_id=opportunity['token_id'],
            side="BUY",
            strategy="mid"
        )
        
        # Place order
        order = self.client.place_limit_order(
            token_id=opportunity['token_id'],
            side="BUY",
            price=Decimal(str(suggested['suggested_price'])),
            size=100,
            time_in_force="GTC"
        )
        
        return order

# Usage
client = PolymarketClient(
    private_key=os.getenv("POLYGON_PRIVATE_KEY"),
    polygon_address=os.getenv("POLYGON_ADDRESS")
)

bot = SimpleArbitrageBot(client)
opportunities = bot.find_arbitrage_opportunities()

for opp in opportunities[:3]:
    print(f"Opportunity: {opp['market']}")
    print(f"Spread: {opp['spread']:.2%}")
    order = bot.execute_trade(opp)
    print(f"Order placed: {order['id']}\n")
```

### Risk Management Example

```python
from polymarket_mcp.client import PolymarketClient

class RiskManager:
    def __init__(self, client: PolymarketClient):
        self.client = client
        self.max_exposure_usd = 5000
        self.max_position_size = 2000
        self.max_category_allocation = 0.4  # 40%
    
    def check_trade_allowed(self, trade_value_usd: float, market_category: str) -> dict:
        """Validate if trade passes risk checks"""
        portfolio = self.client.get_portfolio_summary()
        risk_analysis = self.client.analyze_portfolio_risk()
        
        # Check total exposure
        new_exposure = portfolio['total_value_usd'] + trade_value_usd
        if new_exposure > self.max_exposure_usd:
            return {
                "allowed": False,
                "reason": f"Would exceed max exposure: ${new_exposure} > ${self.max_exposure_usd}"
            }
        
        # Check position size
        if trade_value_usd > self.max_position_size:
            return {
                "allowed": False,
                "reason": f"Trade size ${trade_value_usd} exceeds max position ${self.max_position_size}"
            }
        
        # Check category concentration
        category_exposure = risk_analysis.get('category_concentration', {}).get(market_category, 0)
        new_category_allocation = (category_exposure + trade_value_usd) / new_exposure
        
        if new_category_allocation > self.max_category_allocation:
            return {
                "allowed": False,
                "reason": f"Would exceed {market_category} allocation: {new_category_allocation:.1%}"
            }
        
        return {"allowed": True, "reason": "All risk checks passed"}
    
    def suggest_position_size(self, market_id: str) -> float:
        """Suggest optimal position size based on risk"""
        portfolio = self.client.get_portfolio_summary()
        market = self.client.get_market_details(market_id)
        
        # Kelly Criterion simplified
        win_rate = 0.55  # Estimate from market analysis
        odds = market['tokens'][0]['price']
        
        kelly_fraction = (win_rate - (1 - win_rate) / odds) / odds
        suggested_size = portfolio['total_value_usd'] * kelly_fraction * 0.5  # Half Kelly
        
        return min(suggested_size, self.max_position_size)

# Usage
client = PolymarketClient(
    private_key=os.getenv("POLYGON_PRIVATE_KEY"),
    polygon_address=os.getenv("POLYGON_ADDRESS")
)

risk_mgr = RiskManager(client)

# Check if trade is allowed
trade_check = risk_mgr.check_trade_allowed(
    trade_value_usd=500,
    market_category="Politics"
)

if trade_check['allowed']:
    suggested_size = risk_mgr.suggest_position_size(market_id="0x1234...")
    print(f"Suggested position size: ${suggested_size:.2f}")
else:
    print(f"Trade blocked: {trade_check['reason']}")
```

## Common Patterns

### Pattern 1: Market Research Pipeline

```python
# 1. Find trending markets
trending = client.get_trending_markets(period="24h", limit=20)

# 2. Analyze each market
for market in trending[:5]:
    analysis = client.analyze_market_opportunity(
        market_id=market['id'],
        analysis_depth="deep"
    )
    
    if analysis['recommendation'] == 'BUY':
        # 3. Get orderbook depth
        orderbook = client.get_orderbook_depth(market['tokens'][0]['id'])
        
        # 4. Check liquidity
        if orderbook['total_liquidity'] > 10000:
            print(f"Good opportunity: {market['question']}")
            print(f"Reason: {analysis['reasoning']}")
```

### Pattern 2: Automated Portfolio Rebalancing

```python
# 1. Get portfolio risk analysis
risk = client.analyze_portfolio_risk()

# 2. If risk is high, get optimization suggestions
if risk['overall_risk_score'] > 70:
    optimization = client.optimize_portfolio(
        risk_profile="balanced",
        target_allocation={
            "Politics": 0.35,
            "Sports": 0.30,
            "Crypto": 0.35
        }
    )
    
    # 3. Execute suggested trades
    for suggestion in optimization['suggestions']:
        if suggestion['action'] == 'REDUCE':
            client.place_market_order(
                token_id=suggestion['token_id'],
                side="SELL",
                amount=suggestion['amount']
            )
```

### Pattern 3: Real-Time Monitoring with Alerts

```python
import asyncio

async def monitor_positions():
    # Subscribe to user orders
    client.subscribe_to_user_orders()
    
    # Monitor portfolio value
    while True:
        portfolio = client.get_portfolio_summary()
        
        # Alert on significant P&L change
        if abs(portfolio['unrealized_pnl']) > 500:
            print(f"⚠️ Large P&L movement: ${portfolio['unrealized_pnl']}")
            
            # Analyze if should close positions
            risk = client.analyze_portfolio_risk()
            if risk['liquidity_risk'] > 80:
                print("🚨 High liquidity risk - consider closing positions")
        
        await asyncio.sleep(60)  # Check every minute

# Run monitoring
asyncio.run(monitor_positions())
```

## Web Dashboard

Start the visual web interface:

```bash
polymarket-web
# Or: ./start_web_dashboard.sh

# Access at: http://localhost:8080
```

Dashboard features:
- Real-time market monitoring
- Configuration management with visual controls
- AI-powered market analysis
- System statistics and performance charts
- Live WebSocket status

## Configuration Options

### Safety Limits

```env
# Order Limits
MAX_ORDER_SIZE_USD=1000
MAX_POSITION_SIZE_PER_MARKET=2000
MAX_TOTAL_EXPOSURE_USD=5000

# Liquidity Checks
MIN_LIQUIDITY_REQUIRED=10000
MAX_SPREAD_TOLERANCE=0.05

# Confirmations
REQUIRE_CONFIRMATION_ABOVE_USD=500
ENABLE_AUTONOMOUS_TRADING=true
```

### Rate Limiting

```env
# API Rate Limits (per minute)
RATE_LIMIT_READ_PER_MIN=100
RATE_LIMIT_WRITE_PER_MIN=20
RATE_LIMIT_BURST_SIZE=5
```

### WebSocket Configuration

```env
WEBSOCKET_RECONNECT_DELAY=5
WEBSOCKET_MAX_RETRIES=10
WEBSOCKET_PING_INTERVAL=30
```

## Troubleshooting

### Connection Issues

```bash
# Test API connectivity
python -m polymarket_mcp.test_connection

# Check WebSocket status
python -c "from polymarket_mcp.client import PolymarketClient; \
           c = PolymarketClient(); \
           print(c.get_websocket_status())"
```

### Authentication Errors

```python
# Verify wallet configuration
from polymarket_mcp.client import PolymarketClient
import os

client = PolymarketClient(
    private_key=os.getenv("POLYGON_PRIVATE_KEY"),
    polygon_address=os.getenv("POLYGON_ADDRESS")
)

# Test authentication
try:
    portfolio = client.get_portfolio_summary()
    print("✓ Authentication successful")
except Exception as e:
    print(f"✗ Auth failed: {e}")
```

### Rate Limit Handling

```python
from polymarket_mcp.client import PolymarketClient
from time import sleep

client = PolymarketClient()

# Built-in rate limiting
for i in range(100):
    try:
        markets = client.search_markets(query=f"test {i}", limit=1)
        # Rate limiter automatically throttles requests
    except Exception as e:
        if "rate limit" in str(e).lower():
            print("Rate limit hit, waiting...")
            sleep(60)
```

### Order Validation Failures

```python
# Check order validation before placing
from polymarket_mcp.client import PolymarketClient
from decimal import Decimal

client = PolymarketClient()

# Validate market liquidity
orderbook = client.get_orderbook_depth(token_id="0x5678...")
if orderbook['total_liquidity'] < 10000:
    print("⚠️ Low liquidity - consider smaller order size")

# Calculate expected slippage
spread = client.calculate_spread(token_id="0x5678...")
if spread['spread_percentage'] > 0.05:
    print(f"⚠️ High spread: {spread['spread_percentage']:.2%}")
```

### DEMO Mode Limitations

If you see "Trading disabled in DEMO mode":

```bash
# Switch to full mode
# 1. Get Polygon wallet private key
# 2. Update .env:
DEMO_MODE=false
POLYGON_PRIVATE_KEY=your_actual_key
POLYGON_ADDRESS=0xYourAddress

# 3. Restart MCP server
```

## Testing

```bash
# Run all tests
pytest

# Test specific tool
pytest tests/test_trading.py::test_place_limit_order

# Test with real API (caution)
pytest tests/test_integration.py --run-live
```

## Resources

- **GitHub**: https://github.com/caiovicentino/polymarket-mcp-server
- **Polymarket API Docs**: https://docs.polymarket.com
- **MCP Protocol**: https://modelcontextprotocol.io
- **Full Tool Reference**: See `TOOLS_REFERENCE.md` in repository
