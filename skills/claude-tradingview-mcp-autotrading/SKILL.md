---
name: claude-tradingview-mcp-autotrading
description: Automated cryptocurrency trading bot connecting Claude Code to TradingView charts and executing trades via BitGet with built-in safety checks
triggers:
  - set up automated trading bot with TradingView
  - connect Claude to BitGet for trading
  - deploy crypto trading bot to VPS
  - create automated trading strategy from YouTube transcripts
  - execute trades automatically from TradingView signals
  - configure BitGet API for automated trading
  - run trading bot with safety checks and logging
  - backtest trading strategy with Claude
---

# Claude TradingView MCP Auto-Trading

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What This Does

This project creates a fully automated cryptocurrency trading system that:

1. **Connects Claude Code to TradingView** — reads live chart data and indicator values
2. **Executes trades on BitGet** (or Binance, Bybit, OKX, Coinbase, Kraken, KuCoin, Gate.io, MEXC, Bitfinex)
3. **Enforces safety checks** — every condition in your strategy must pass before execution
4. **Runs 24/7 in the cloud** — deploy to a VPS with cron scheduling
5. **Logs trades for taxes** — automatic CSV generation with date, price, fees, net amounts

The bot runs one check per execution, evaluates your strategy rules against live market data, and executes trades only when all conditions align.

## Installation

### Prerequisites

- Node.js 18+
- Claude Code installed
- TradingView MCP set up (see [first video](https://youtu.be/vIX6ztULs4U))
- Exchange account (BitGet recommended, $1,000 bonus: https://bonus.bitget.com/LewisJackson)

### Clone and Setup

```bash
git clone https://github.com/jackson-video-resources/claude-tradingview-mcp-trading
cd claude-tradingview-mcp-trading
npm install
cp .env.example .env
```

### Configure Environment Variables

Edit `.env`:

```bash
# Exchange API credentials (NEVER commit these)
BITGET_API_KEY=${BITGET_API_KEY}
BITGET_SECRET_KEY=${BITGET_SECRET_KEY}
BITGET_PASSPHRASE=${BITGET_PASSPHRASE}

# Risk management
PORTFOLIO_VALUE_USD=1000
MAX_TRADE_SIZE_USD=100
MAX_TRADES_PER_DAY=3

# Trading mode
PAPER_TRADING=true  # Set to false for live trading

# Market settings
SYMBOL=BTCUSDT
TIMEFRAME=4H
```

**Security rules for ALL exchanges:**
- ✅ Withdrawals OFF
- ✅ IP whitelist ON
- ✅ Never share API keys

### Exchange Setup Guides

| Exchange | Guide Path |
|----------|-----------|
| BitGet | `docs/exchanges/bitget.md` |
| Binance | `docs/exchanges/binance.md` |
| Bybit | `docs/exchanges/bybit.md` |
| OKX | `docs/exchanges/okx.md` |
| Coinbase Advanced | `docs/exchanges/coinbase.md` |
| Kraken | `docs/exchanges/kraken.md` |
| KuCoin | `docs/exchanges/kucoin.md` |
| Gate.io | `docs/exchanges/gateio.md` |
| MEXC | `docs/exchanges/mexc.md` |
| Bitfinex | `docs/exchanges/bitfinex.md` |

## Core Workflow

### The One-Shot Prompt

The fastest way to start trading is to use the one-shot prompt. Copy the entire contents of `prompts/02-one-shot-trade.md` and paste into Claude Code:

```bash
# What the one-shot prompt does:
# 1. Clones repo
# 2. Connects BitGet API
# 3. Sets trading preferences
# 4. Connects TradingView MCP
# 5. Builds/validates strategy
# 6. Optionally deploys to VPS
# 7. Executes first trade check
```

### Manual Execution

```bash
# Connect TradingView MCP (Mac)
tv_launch
tv_health_check

# Run the bot
node bot.js

# Check tax summary
node bot.js --tax-summary
```

### Execution Flow

| Step | Action | Details |
|------|--------|---------|
| 1 | Load strategy | Reads `rules.json` |
| 2 | Fetch live data | TradingView MCP or Binance API |
| 3 | Calculate indicators | MACD, RSI, moving averages from raw candles |
| 4 | Evaluate bias | Bullish/bearish/neutral market state |
| 5 | Check limits | Daily cap, max trade size |
| 6 | Run safety check | All `entry_rules` must pass |
| 7 | Execute trade | Only if ALL conditions met |
| 8 | Log trade | Append to `trades.csv` |
| 9 | Save decision log | Write to `safety-check-log.json` |

## Strategy Configuration

### rules.json Structure

```json
{
  "strategy_name": "Van de Poppe + Tone Vays BTC Strategy",
  "timeframe": "4H",
  "symbol": "BTCUSDT",
  "indicators": {
    "macd": {
      "fast_period": 12,
      "slow_period": 26,
      "signal_period": 9
    },
    "rsi": {
      "period": 14
    },
    "moving_averages": {
      "ma_50": 50,
      "ma_200": 200
    }
  },
  "entry_rules": {
    "long": [
      {
        "condition": "macd_histogram > 0",
        "description": "MACD histogram positive (bullish momentum)"
      },
      {
        "condition": "rsi > 50 AND rsi < 70",
        "description": "RSI in bullish range but not overbought"
      },
      {
        "condition": "close > ma_50",
        "description": "Price above 50-period MA"
      },
      {
        "condition": "ma_50 > ma_200",
        "description": "Golden cross - 50 MA above 200 MA"
      }
    ]
  },
  "risk_management": {
    "max_risk_per_trade_percent": 1,
    "position_sizing_method": "fixed_percentage"
  }
}
```

### Building a Custom Strategy from YouTube

Use the Apify YouTube Transcript Scraper to extract trader methodology:

```bash
# 1. Get transcripts from Apify (https://apify.com?fpr=3ly3yd)
# 2. Paste output into prompts/01-extract-strategy.md
# 3. Run in Claude Code:

# Claude will:
# - Parse trader's methodology
# - Extract entry/exit rules
# - Generate rules.json
# - Validate indicator requirements
```

Example prompt structure in `prompts/01-extract-strategy.md`:

```markdown
Analyze these YouTube transcripts and extract a trading strategy.

[PASTE APIFY OUTPUT HERE]

Generate a rules.json that includes:
- Timeframe preferences
- Required indicators with parameters
- Entry conditions (long/short)
- Exit conditions
- Risk management rules
```

## Key Code Patterns

### Reading Live Market Data (Local with TradingView MCP)

```javascript
// Assuming TradingView MCP is running
const marketData = await fetchMarketData('BTCUSDT', '4H');

console.log(marketData);
// {
//   symbol: 'BTCUSDT',
//   timeframe: '4H',
//   close: 45230.50,
//   macd: { value: 123.45, signal: 98.32, histogram: 25.13 },
//   rsi: 62.5,
//   ma_50: 44500.00,
//   ma_200: 43000.00
// }
```

### Evaluating Entry Conditions

```javascript
function evaluateSafetyCheck(rules, marketData, tradeLimits) {
  const results = {
    passed: true,
    conditions: [],
    tradeSize: 0,
    reason: null
  };

  // Check daily trade limit
  if (tradeLimits.tradesExecutedToday >= tradeLimits.maxTradesPerDay) {
    results.passed = false;
    results.reason = `Daily trade limit reached (${tradeLimits.maxTradesPerDay})`;
    return results;
  }

  // Evaluate each entry rule
  for (const rule of rules.entry_rules.long) {
    const conditionMet = evaluateCondition(rule.condition, marketData);
    
    results.conditions.push({
      description: rule.description,
      condition: rule.condition,
      passed: conditionMet,
      actualValues: extractValues(rule.condition, marketData)
    });

    if (!conditionMet) {
      results.passed = false;
      results.reason = `Failed: ${rule.description}`;
    }
  }

  // Calculate position size
  if (results.passed) {
    const portfolioValue = parseFloat(process.env.PORTFOLIO_VALUE_USD);
    const riskPercent = rules.risk_management.max_risk_per_trade_percent;
    results.tradeSize = Math.min(
      (portfolioValue * riskPercent) / 100,
      parseFloat(process.env.MAX_TRADE_SIZE_USD)
    );
  }

  return results;
}
```

### Executing a Trade

```javascript
async function executeTrade(exchange, symbol, side, quantity) {
  const paperTrading = process.env.PAPER_TRADING === 'true';
  
  if (paperTrading) {
    console.log('[PAPER TRADE] Would execute:', { symbol, side, quantity });
    logTrade({
      date: new Date().toISOString(),
      symbol,
      side,
      quantity,
      mode: 'PAPER'
    });
    return { orderId: 'PAPER-' + Date.now() };
  }

  // Live trading execution
  const order = await exchange.createMarketOrder(symbol, side, quantity);
  
  logTrade({
    date: new Date().toISOString(),
    symbol: order.symbol,
    side: order.side,
    quantity: order.filled,
    price: order.average,
    totalUSD: order.cost,
    fee: order.fee?.cost || 0,
    netAmount: order.cost - (order.fee?.cost || 0),
    orderId: order.id,
    mode: 'LIVE'
  });

  return order;
}
```

### Trade Logging (Tax-Ready CSV)

```javascript
function logTrade(tradeData) {
  const csvLine = [
    tradeData.date,
    new Date(tradeData.date).toISOString().split('T')[1].split('.')[0],
    'BitGet',
    tradeData.symbol,
    tradeData.side,
    tradeData.quantity,
    tradeData.price || 'N/A',
    tradeData.totalUSD || 'N/A',
    tradeData.fee || 0,
    tradeData.netAmount || 'N/A',
    tradeData.orderId,
    tradeData.mode
  ].join(',');

  fs.appendFileSync('trades.csv', csvLine + '\n');
}
```

## VPS Deployment (24/7 Cloud Execution)

### Hostinger VPS Setup

```bash
# SSH into VPS
ssh root@YOUR_VPS_IP

# Install dependencies
apt update && apt install -y nodejs npm git

# Clone and setup
git clone https://github.com/jackson-video-resources/claude-tradingview-mcp-trading bot
cd bot
npm install

# Create .env (use vim/nano)
cat > .env << EOF
BITGET_API_KEY=${BITGET_API_KEY}
BITGET_SECRET_KEY=${BITGET_SECRET_KEY}
BITGET_PASSPHRASE=${BITGET_PASSPHRASE}
PORTFOLIO_VALUE_USD=1000
MAX_TRADE_SIZE_USD=100
MAX_TRADES_PER_DAY=3
PAPER_TRADING=true
SYMBOL=BTCUSDT
TIMEFRAME=4H
EOF
```

### Cron Scheduling

Cloud mode uses Binance's free market API instead of TradingView Desktop. Strategy logic is identical.

```bash
# Edit crontab
crontab -e

# Add schedule based on timeframe:
```

| Timeframe | Crontab Line |
|-----------|-------------|
| 4H | `0 */4 * * * cd /root/bot && /usr/bin/node bot.js >> bot.log 2>&1` |
| 1D | `0 9 * * * cd /root/bot && /usr/bin/node bot.js >> bot.log 2>&1` |
| 1H | `0 * * * * cd /root/bot && /usr/bin/node bot.js >> bot.log 2>&1` |
| 15m | `*/15 * * * * cd /root/bot && /usr/bin/node bot.js >> bot.log 2>&1` |

### Monitoring VPS Logs

```bash
# Watch live bot execution
tail -f /root/bot/bot.log

# Check last 50 lines
tail -n 50 /root/bot/bot.log

# View trades CSV
cat /root/bot/trades.csv

# Check safety check log
cat /root/bot/safety-check-log.json | jq
```

## File Structure

```
claude-tradingview-mcp-trading/
├── bot.js                      # Main execution script
├── rules.json                  # Your trading strategy
├── .env                        # API credentials (GITIGNORED)
├── trades.csv                  # Auto-generated trade log
├── safety-check-log.json       # Decision log with indicator values
├── prompts/
│   ├── 01-extract-strategy.md  # Build rules.json from transcripts
│   └── 02-one-shot-trade.md    # One-shot setup prompt
├── docs/
│   ├── setup-windows.md        # Windows MCP setup
│   ├── setup-linux.md          # Linux MCP setup
│   └── exchanges/              # Exchange-specific guides
│       ├── bitget.md
│       ├── binance.md
│       ├── bybit.md
│       └── ...
└── package.json
```

## Safety Checks and Risk Management

### Built-in Guardrails

```javascript
// Position sizing (from rules.json)
const maxRiskPercent = rules.risk_management.max_risk_per_trade_percent; // 1%
const portfolioValue = parseFloat(process.env.PORTFOLIO_VALUE_USD);
const calculatedSize = (portfolioValue * maxRiskPercent) / 100;

// Hard caps (from .env)
const maxTradeSize = parseFloat(process.env.MAX_TRADE_SIZE_USD);
const maxTradesPerDay = parseInt(process.env.MAX_TRADES_PER_DAY);

// Final position size
const positionSize = Math.min(calculatedSize, maxTradeSize);
```

### Safety Check Output Example

```json
{
  "timestamp": "2026-06-14T10:30:00Z",
  "symbol": "BTCUSDT",
  "timeframe": "4H",
  "marketData": {
    "close": 45230.50,
    "macd_histogram": 25.13,
    "rsi": 62.5,
    "ma_50": 44500.00,
    "ma_200": 43000.00
  },
  "conditions": [
    {
      "description": "MACD histogram positive",
      "condition": "macd_histogram > 0",
      "passed": true,
      "actualValue": 25.13
    },
    {
      "description": "RSI in bullish range",
      "condition": "rsi > 50 AND rsi < 70",
      "passed": true,
      "actualValue": 62.5
    },
    {
      "description": "Price above 50 MA",
      "condition": "close > ma_50",
      "passed": true,
      "actualValue": "45230.50 > 44500.00"
    },
    {
      "description": "Golden cross",
      "condition": "ma_50 > ma_200",
      "passed": true,
      "actualValue": "44500.00 > 43000.00"
    }
  ],
  "result": "PASS",
  "tradeSize": 100,
  "action": "EXECUTE_LONG"
}
```

## Troubleshooting

### TradingView MCP Connection Issues

```bash
# Check MCP health
tv_health_check

# Expected output:
# {
#   "cdp_connected": true,
#   "tradingview_loaded": true
# }

# If cdp_connected is false, relaunch:
tv_launch
```

**Windows users:** See `docs/setup-windows.md` for platform-specific MCP setup.

### Exchange API Errors

```bash
# Test API connection
node -e "
const ccxt = require('ccxt');
const exchange = new ccxt.bitget({
  apiKey: process.env.BITGET_API_KEY,
  secret: process.env.BITGET_SECRET_KEY,
  password: process.env.BITGET_PASSPHRASE
});
exchange.fetchBalance().then(console.log).catch(console.error);
"
```

Common issues:
- **Invalid signature:** Check API secret and passphrase
- **IP not whitelisted:** Add VPS IP to exchange whitelist
- **Insufficient balance:** Check `PORTFOLIO_VALUE_USD` matches actual funds
- **Withdrawals enabled:** Disable withdrawals in API settings

### Paper Trading Not Logging

```bash
# Verify PAPER_TRADING flag
grep PAPER_TRADING .env

# Should show:
# PAPER_TRADING=true

# Check if trades.csv exists and is writable
ls -la trades.csv
```

### VPS Cron Not Running

```bash
# Check cron service
systemctl status cron

# View cron logs
grep CRON /var/log/syslog

# Test manual execution
cd /root/bot && node bot.js

# Check for errors in bot.log
tail -n 100 /root/bot/bot.log
```

### Missing Indicator Data

If safety check fails due to missing indicators:

```javascript
// Verify rules.json matches available data sources
// TradingView MCP provides: MACD, RSI, MA, volume, OHLCV
// Binance API (cloud mode) provides: OHLCV only (indicators calculated)

// Check safety-check-log.json for actual values:
cat safety-check-log.json | jq '.marketData'
```

## Tax Reporting

### trades.csv Format

```csv
Date,Time,Exchange,Symbol,Side,Quantity,Price,Total USD,Fee (est.),Net Amount,Order ID,Mode
2026-06-14T10:30:00Z,10:30:00,BitGet,BTCUSDT,Buy,0.00221,45230.50,100.00,0.10,99.90,12345678,LIVE
```

### Tax Summary Command

```bash
node bot.js --tax-summary

# Output:
# Total trades: 47
# Total volume: $4,700.00
# Total fees: $4.70
# Winning trades: 28 (59.6%)
# Losing trades: 19 (40.4%)
```

### Importing to Accounting Software

Most platforms (QuickBooks, Xero, CoinTracker, Koinly) accept CSV imports:

1. Export `trades.csv`
2. Import as "custom CSV"
3. Map columns: Date → Date, Total USD → Amount, Fee → Fee, etc.

## Best Practices

1. **Always start with paper trading** — Run `PAPER_TRADING=true` for at least a week
2. **Verify exchange security** — Withdrawals OFF, IP whitelist ON
3. **Monitor safety-check-log.json** — Understand why trades do/don't execute
4. **Start with small position sizes** — Max 1-2% risk per trade
5. **Backtest your strategy** — Use historical data before live deployment
6. **Keep API keys secure** — Never commit `.env`, use VPS environment variables
7. **Review trades.csv regularly** — Catch unexpected behavior early
8. **Set realistic limits** — `MAX_TRADES_PER_DAY=3` prevents overtrading

## Resources

- [First video — Connect Claude to TradingView](https://youtu.be/vIX6ztULs4U)
- [TradingView MCP repo](https://github.com/jackson-video-resources/tradingview-mcp-jackson)
- [Apify YouTube Transcript Scraper](https://apify.com?fpr=3ly3yd)
- [BitGet signup ($1,000 bonus)](https://bonus.bitget.com/LewisJackson)
- [Hostinger VPS](https://hostinger.com/lewisjackson10)

**Disclaimer:** This is not financial advice. Automated trading carries significant risk. Never invest more than you can afford to lose. Test thoroughly in paper trading mode before using real funds.
