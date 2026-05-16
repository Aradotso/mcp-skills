---
name: tradingview-mcp-assistant
description: AI-assisted TradingView chart analysis and automation via Chrome DevTools Protocol for Claude Code
triggers:
  - analyze my TradingView chart
  - help me write a Pine Script indicator
  - automate my TradingView workflow
  - read indicator values from TradingView
  - set up multi-chart layout in TradingView
  - create TradingView price alerts
  - draw support and resistance levels
  - practice trading with replay mode
---

# TradingView MCP Assistant

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Personal AI assistant for TradingView Desktop charts. Connects Claude Code to your locally running TradingView app via Chrome DevTools Protocol for AI-assisted chart analysis, Pine Script development, and workflow automation.

**Important:** This tool requires a valid TradingView subscription and the Desktop app. It communicates only with your local TradingView instance via CDP (Chrome DevTools Protocol) — no external connections, no data transmission.

## What This Skill Enables

- **Pine Script development** — write, compile, debug, and iterate on indicators/strategies
- **Chart navigation** — change symbols, timeframes, zoom to dates, manage layouts
- **Visual analysis** — read indicator values, price levels, and Pine Script drawings
- **Drawing tools** — add trend lines, horizontal lines, rectangles, text annotations
- **Alert management** — create, list, and delete price alerts
- **Replay mode** — step through historical bars for practice trading
- **Multi-pane layouts** — set up 2x2, 3x1 grids with different symbols
- **Chart streaming** — monitor live data with JSONL output
- **CLI access** — every MCP tool available as `tv` command for scripting

## Installation

### Prerequisites

- TradingView Desktop app (paid subscription required)
- Node.js 18+
- Claude Code with MCP support
- macOS, Windows, or Linux

### Setup Steps

```bash
# 1. Clone and install
git clone https://github.com/tradesdontlie/tradingview-mcp.git
cd tradingview-mcp
npm install

# 2. Optional: Install CLI globally
npm link
```

### Launch TradingView with Debug Mode

TradingView Desktop must run with Chrome DevTools Protocol enabled on port 9222.

**macOS:**
```bash
./scripts/launch_tv_debug_mac.sh
```

**Windows:**
```bash
scripts\launch_tv_debug.bat
```

**Linux:**
```bash
./scripts/launch_tv_debug_linux.sh
```

**Or use the MCP tool:**
```javascript
// Claude will execute
tv_launch()
```

### Configure Claude Code

Add to `~/.claude/.mcp.json` or project `.mcp.json`:

```json
{
  "mcpServers": {
    "tradingview": {
      "command": "node",
      "args": ["/absolute/path/to/tradingview-mcp/src/server.js"]
    }
  }
}
```

### Verify Connection

Ask Claude: *"Use tv_health_check to verify TradingView is connected"*

## Key MCP Tools

### Chart Reading (Start Here)

```javascript
// 1. Always start with state to get chart context
const state = await chart_get_state();
// Returns: { symbol: "BTCUSD", timeframe: "5", indicators: [{name, id}], layout: "..." }

// 2. Get current price
const quote = await quote_get();
// Returns: { symbol, close, open, high, low, volume, change, changePercent }

// 3. Read all indicator values (RSI, MACD, BB, etc.)
const values = await data_get_study_values();
// Returns: [{ name: "RSI", plots: [{title: "RSI", value: 54.32}] }]

// 4. Get OHLCV bars (ALWAYS use summary: true first)
const bars = await data_get_ohlcv({ summary: true });
// Returns: { bars: 500, range: "...", high: 45000, low: 42000, avgVolume: 1234 }
```

### Pine Script Data (Custom Indicators)

Read `line.new()`, `label.new()`, `table.new()`, `box.new()` from Pine Scripts:

```javascript
// Read horizontal price levels (support/resistance)
const lines = await data_get_pine_lines({ 
  study_filter: "Session Levels" 
});
// Returns: [{ y: 24500, text: "PDH", color: "blue" }]

// Read text labels with prices
const labels = await data_get_pine_labels({ 
  study_filter: "Market Structure" 
});
// Returns: [{ price: 24450, text: "HH", time: "2025-01-15T10:30:00Z" }]

// Read data tables (session stats, analytics)
const tables = await data_get_pine_tables({ 
  study_filter: "Volume Profile" 
});
// Returns: [{ headers: ["Price", "Volume"], rows: [[24500, 1234], ...] }]

// Read price zones/ranges
const boxes = await data_get_pine_boxes({ 
  study_filter: "Supply Demand" 
});
// Returns: [{ high: 24600, low: 24500, text: "Supply Zone" }]
```

**Always use `study_filter`** to target specific indicators — reduces noise and token usage.

### Chart Control

```javascript
// Change symbol
await chart_set_symbol({ symbol: "AAPL" });

// Change timeframe (1, 5, 15, 60, D, W, M)
await chart_set_timeframe({ timeframe: "D" });

// Change chart type
await chart_set_type({ type: "HeikinAshi" }); // Candles, Line, Area, Renko

// Scroll to date
await chart_scroll_to_date({ date: "2025-01-15" });

// Zoom to exact range (unix timestamps)
await chart_set_visible_range({ 
  from: 1705276800, 
  to: 1705363200 
});

// Add indicator (use FULL name)
await chart_manage_indicator({ 
  action: "add", 
  name: "Relative Strength Index" // NOT "RSI"
});

// Remove indicator
await chart_manage_indicator({ 
  action: "remove", 
  indicator_id: "123abc" // from chart_get_state
});
```

### Pine Script Development

```javascript
// 1. Inject code into Pine Editor
await pine_set_source({ 
  source: `//@version=5
indicator("My RSI", overlay=false)
rsiValue = ta.rsi(close, 14)
plot(rsiValue, "RSI", color=color.blue)
hline(70, "Overbought", color=color.red)
hline(30, "Oversold", color=color.green)`
});

// 2. Compile with auto-detection
const result = await pine_smart_compile();
// Returns: { success: true/false, version_detected: 5, errors: [] }

// 3. Check for errors
if (!result.success) {
  const errors = await pine_get_errors();
  // Returns: [{ line: 4, message: "Undeclared identifier 'foo'" }]
}

// 4. Read console output (log.info)
const logs = await pine_get_console();
// Returns: ["RSI value: 54.32", "Signal: BUY"]

// 5. Save to TradingView cloud
await pine_save({ name: "My Custom RSI" });

// Read current source (WARNING: can be 200KB+ for complex scripts)
const code = await pine_get_source();

// Analyze code structure (get functions, inputs, plots)
const analysis = await pine_analyze_code();
// Returns: { functions: [...], inputs: [...], plots: [...] }
```

### Multi-Pane Layouts

```javascript
// List all panes
const panes = await pane_list();
// Returns: [{ index: 0, symbol: "BTCUSD", active: true }, ...]

// Change layout (s, 2h, 2v, 2x2, 4, 6, 8)
await pane_set_layout({ layout: "2x2" });

// Set symbol on specific pane
await pane_set_symbol({ pane_index: 0, symbol: "ES1!" });
await pane_set_symbol({ pane_index: 1, symbol: "NQ1!" });
await pane_set_symbol({ pane_index: 2, symbol: "AAPL" });
await pane_set_symbol({ pane_index: 3, symbol: "TSLA" });

// Focus a pane
await pane_focus({ pane_index: 1 });
```

### Drawing Tools

```javascript
// Draw horizontal line
await draw_shape({
  shape_type: "horizontal_line",
  price: 24500,
  text: "Key Level",
  color: "#FF0000"
});

// Draw trend line
await draw_shape({
  shape_type: "trend_line",
  price1: 24000,
  time1: "2025-01-10T10:00:00Z",
  price2: 24500,
  time2: "2025-01-15T15:00:00Z",
  color: "#00FF00"
});

// Draw rectangle
await draw_shape({
  shape_type: "rectangle",
  price1: 24000,
  time1: "2025-01-10T10:00:00Z",
  price2: 24500,
  time2: "2025-01-15T15:00:00Z",
  fill_color: "#0000FF33"
});

// List all drawings
const drawings = await draw_list();

// Remove specific drawing
await draw_remove({ drawing_id: "abc123" });

// Clear all drawings
await draw_clear();
```

### Alert Management

```javascript
// List alerts
const alerts = await alert_list();

// Create price alert
await alert_create({
  symbol: "BTCUSD",
  condition: "crossing",
  price: 45000,
  message: "BTC crossed 45k"
});

// Delete alert
await alert_delete({ alert_id: "xyz789" });
```

### Replay Mode

```javascript
// Start replay at date
await replay_start({ date: "2025-03-01" });

// Step forward (bar-by-bar)
await replay_step({ direction: "forward", bars: 1 });

// Step backward
await replay_step({ direction: "backward", bars: 5 });

// Simulate trade entry
await replay_trade({
  action: "long",
  price: 24500,
  quantity: 1,
  stop_loss: 24400,
  take_profit: 24700
});

// Check replay status
const status = await replay_status();

// Auto-play
await replay_autoplay({ speed: 2 }); // 1-5

// Stop replay
await replay_stop();
```

### Screenshot Capture

```javascript
// Capture full chart
await capture_screenshot({ region: "chart" });

// Capture specific region
await capture_screenshot({ 
  region: "custom",
  x: 100, y: 100, width: 800, height: 600
});

// Returns: { path: "/tmp/chart_2025-05-16_12-30-45.png" }
```

## CLI Usage

Every MCP tool is also a `tv` CLI command. All output is JSON.

```bash
# Connection
tv status                          # check connection
tv launch                          # start TradingView in debug mode

# Chart reading
tv quote                           # current price
tv ohlcv --summary                 # price summary
tv state                           # full chart state
tv values                          # all indicator values

# Chart control
tv symbol AAPL                     # change symbol
tv timeframe D                     # change timeframe
tv type HeikinAshi                 # change chart type
tv scroll --date 2025-01-15        # jump to date

# Pine Script
tv pine get                        # read source
tv pine compile                    # compile
tv pine errors                     # show errors
tv pine save "My Indicator"        # save to cloud

# Data extraction
tv data lines --filter "Levels"    # read Pine lines
tv data labels --filter "MSB"      # read Pine labels
tv data tables --filter "Profile"  # read Pine tables

# Drawing
tv draw shape --type horizontal_line --price 24500
tv draw list                       # list drawings
tv draw clear                      # clear all

# Layouts
tv pane layout 2x2                 # 4-chart grid
tv pane symbol 0 ES1!              # set pane symbol
tv pane list                       # list all panes

# Streaming (JSONL output, pipe-friendly)
tv stream quote                    # monitor price
tv stream bars                     # bar updates
tv stream values                   # indicator updates
tv stream all                      # all panes
tv stream quote | jq '.close'      # pipe to jq
```

## Common Patterns

### Full Chart Analysis Workflow

```javascript
// 1. Get chart context
const state = await chart_get_state();
const quote = await quote_get();

// 2. Read all indicators
const values = await data_get_study_values();

// 3. Read custom Pine data
const lines = await data_get_pine_lines({ study_filter: "Levels" });
const labels = await data_get_pine_labels({ study_filter: "Structure" });
const tables = await data_get_pine_tables({ study_filter: "Stats" });

// 4. Get price summary
const ohlcv = await data_get_ohlcv({ summary: true });

// 5. Capture screenshot
const screenshot = await capture_screenshot({ region: "chart" });

// 6. Compile analysis
const analysis = `
Chart: ${state.symbol} ${state.timeframe}
Price: ${quote.close} (${quote.changePercent}%)
RSI: ${values.find(v => v.name.includes('RSI'))?.plots[0]?.value}
Key Levels: ${lines.map(l => `${l.text}: ${l.y}`).join(', ')}
Screenshot: ${screenshot.path}
`;
```

### Pine Script Iteration

```javascript
// User says: "Write a VWAP indicator with bands"

// 1. Generate initial code
const code = `//@version=5
indicator("VWAP Bands", overlay=true)
vwapValue = ta.vwap
stdDev = ta.stdev(close, 20)
upper = vwapValue + stdDev
lower = vwapValue - stdDev
plot(vwapValue, "VWAP", color=color.blue)
plot(upper, "Upper", color=color.red)
plot(lower, "Lower", color=color.green)`;

// 2. Inject and compile
await pine_set_source({ source: code });
const result = await pine_smart_compile();

// 3. Check errors
if (!result.success) {
  const errors = await pine_get_errors();
  // Fix errors, iterate
  // await pine_set_source({ source: fixedCode });
  // await pine_smart_compile();
}

// 4. Test on chart, read console
const logs = await pine_get_console();

// 5. Save when satisfied
await pine_save({ name: "VWAP Bands" });
```

### Multi-Symbol Dashboard

```javascript
// Set up 4-chart grid with futures
await pane_set_layout({ layout: "2x2" });
await pane_set_symbol({ pane_index: 0, symbol: "ES1!" });  // S&P 500
await pane_set_symbol({ pane_index: 1, symbol: "NQ1!" });  // Nasdaq
await pane_set_symbol({ pane_index: 2, symbol: "YM1!" });  // Dow
await pane_set_symbol({ pane_index: 3, symbol: "RTY1!" }); // Russell

// Stream all panes
// CLI: tv stream all | jq '{es: .panes[0].quote.close, nq: .panes[1].quote.close}'
```

### Automated Level Detection

```javascript
// Read Pine Script levels from custom indicator
const lines = await data_get_pine_lines({ 
  study_filter: "Supply Demand Zones" 
});

// Create alerts at each level
for (const line of lines) {
  await alert_create({
    symbol: state.symbol,
    condition: "crossing",
    price: line.y,
    message: `Price crossed ${line.text} at ${line.y}`
  });
}
```

### Replay Practice Session

```javascript
// 1. Start replay at month start
await replay_start({ date: "2025-05-01" });

// 2. Step forward until setup appears
await replay_step({ direction: "forward", bars: 10 });

// 3. Simulate entry
const quote = await quote_get();
await replay_trade({
  action: "long",
  price: quote.close,
  quantity: 1,
  stop_loss: quote.close * 0.98,
  take_profit: quote.close * 1.02
});

// 4. Continue stepping to see outcome
await replay_step({ direction: "forward", bars: 20 });
```

## Troubleshooting

### Connection Issues

```bash
# Check if TradingView is running with debug port
tv status

# If failed, launch manually
./scripts/launch_tv_debug_mac.sh  # or .bat / _linux.sh

# Verify port 9222 is open
lsof -i :9222  # macOS/Linux
netstat -ano | findstr :9222  # Windows
```

### MCP Tools Not Appearing in Claude

1. Check `~/.claude/.mcp.json` has absolute path
2. Restart Claude Code completely
3. Check Claude Code logs: Help → Show Logs
4. Verify Node.js version: `node --version` (must be 18+)

### Pine Script Compilation Fails

```javascript
// Always check errors first
const result = await pine_smart_compile();
if (!result.success) {
  const errors = await pine_get_errors();
  console.log(errors);
}

// Common issues:
// - Wrong version (use //@version=5)
// - Undeclared identifiers (check variable names)
// - Invalid function calls (check Pine Script docs)
```

### Indicator Values Empty

```javascript
// Make sure indicator is visible and loaded
const state = await chart_get_state();
console.log(state.indicators); // check IDs

// Try toggling visibility
await indicator_toggle_visibility({ 
  indicator_id: "abc123", 
  visible: true 
});

// Wait for chart to update, then read values
await new Promise(r => setTimeout(r, 1000));
const values = await data_get_study_values();
```

### Pine Data Returns Empty

```javascript
// Check if Pine Script is actually drawing
const state = await chart_get_state();
// Look for your indicator in state.indicators

// Use exact study_filter match (case-sensitive)
const lines = await data_get_pine_lines({ 
  study_filter: "Session Levels"  // exact name from state
});

// If still empty, the Pine Script may not be drawing anything
// Check Pine console: tv pine console
```

### Streaming Stops

```bash
# Streams poll local CDP connection
# If TradingView crashes or debug port closes, restart:
./scripts/launch_tv_debug_mac.sh
tv stream quote  # restart stream
```

### High Token Usage

```javascript
// ALWAYS use summary: true for OHLCV first
const bars = await data_get_ohlcv({ summary: true }); // 500B
// NOT: data_get_ohlcv({ bars: 100 }) // 8KB

// Use study_filter to target specific indicators
const lines = await data_get_pine_lines({ 
  study_filter: "Levels"  // reduces from 50+ lines to 5-10
});

// Avoid pine_get_source unless necessary (can be 200KB+)
// Use pine_analyze_code for structure instead
```

## Environment Variables

None required — all communication is local via CDP on port 9222.

Optional CLI customization:
```bash
export TV_CDP_PORT=9222  # default
export TV_TIMEOUT=30000  # CDP timeout in ms
```

## Security Considerations

- **No external connections** — only localhost CDP (port 9222)
- **No data storage** — all data stays on your machine
- **No TradingView server access** — requires valid subscription
- **Debug port is local-only** — not exposed to network
- Review TradingView Terms of Use for programmatic access compliance

## References

- [MCP Documentation](https://modelcontextprotocol.io/)
- [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)
- [TradingView Pine Script v5](https://www.tradingview.com/pine-script-docs/)
- [Project Repository](https://github.com/tradesdontlie/tradingview-mcp)
