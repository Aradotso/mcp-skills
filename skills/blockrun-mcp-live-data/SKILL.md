---
name: blockrun-mcp-live-data
description: Real-time markets, research, crypto, X/Twitter data for AI agents via BlockRun MCP with pay-per-call micropayments
triggers:
  - "get live crypto prices and market data"
  - "search for research papers or web content"
  - "check Polymarket prediction market odds"
  - "generate images or videos with AI"
  - "query on-chain data and DEX prices"
  - "run code in isolated sandbox with GPU"
  - "make AI voice calls or text-to-speech"
  - "access 60+ LLM models with pay-per-token"
---

# BlockRun MCP Live Data

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

BlockRun MCP provides real-time data access for AI agents through pay-per-call micropayments. No API keys, no subscriptions — just a USDC wallet and fractions-of-a-cent per query. Access markets, research, crypto, X/Twitter, on-chain data, LLMs, image/video generation, voice calls, and more.

## What It Solves

Before BlockRun, Claude and other AI agents cannot answer:
- "What's the current Polymarket probability for Bitcoin hitting $100k?"
- "Find top 5 RAG papers from the last 30 days"
- "What's the 24h volume on PEPE/ETH on Uniswap?"
- "What are people saying about @sama on X?"

After installing BlockRun MCP, all these queries work — billed from a local USDC wallet at $0.001–$0.10 per call depending on the tool.

## Installation

### Prerequisites

- Node.js ≥ 18
- ~$5 USDC on Base or Solana (auto-wallet created on first run)
- MCP client (Claude Code, Claude Desktop, Cursor, Windsurf, ChatGPT Desktop)

### Claude Code (Recommended)

```bash
claude mcp add blockrun -s user -- npx -y @blockrun/mcp@latest
```

The `-s user` flag installs globally. The `--` separator ensures `-y` is passed to `npx`.

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "blockrun": {
      "command": "npx",
      "args": ["-y", "@blockrun/mcp"]
    }
  }
}
```

### Cursor

Add to `~/.cursor/mcp.json` (macOS/Linux) or `%APPDATA%\Cursor\mcp.json` (Windows):

```json
{
  "mcpServers": {
    "blockrun": {
      "command": "npx",
      "args": ["-y", "@blockrun/mcp@latest"]
    }
  }
}
```

### Windsurf

Same JSON format in:
- macOS: `~/.codeium/windsurf/mcp_config.json`
- Linux: `~/.config/.codeium/windsurf/mcp_config.json`
- Windows: `%APPDATA%\Codeium\windsurf\mcp_config.json`

## Wallet Setup & Funding

On first run, BlockRun auto-creates a wallet. Get your address:

```
Use blockrun_wallet tool with action: "balance"
```

Response includes your wallet address and balance. Send USDC on Base network:

| Method | Instructions |
|--------|-------------|
| Coinbase | Send → USDC → Base network → paste address |
| Bridge | Use [bridge.base.org](https://bridge.base.org) |

**Budget guide:**
- $5 = ~5,000 market queries
- $5 = ~500 Exa searches
- $5 = ~250 image generations
- $5 = ~10 Seedance video clips (5s @ 720p)

## Available Tools

### blockrun_wallet

Manage wallet balance, spending, agent budgets.

```typescript
// Check balance
{
  "action": "balance"
}

// Get setup QR code
{
  "action": "setup"
}

// Delegate budget to child agent
{
  "action": "delegate",
  "agent_id": "research-agent-1",
  "agent_limit": 1.00  // $1 max spend
}
```

**Cost:** Free

### blockrun_price

Realtime prices + OHLC data via Pyth. Crypto/FX/commodities are free; stocks are paid.

```typescript
// Get BTC price
{
  "symbol": "BTC/USD"
}

// Get OHLC for EUR/USD
{
  "symbol": "EUR/USD",
  "interval": "1h",
  "limit": 24
}

// Stock price (paid)
{
  "symbol": "AAPL",
  "market": "NASDAQ"
}
```

**Cost:** Free (crypto/FX/commodity), $0.001/call (stocks)

### blockrun_markets

Polymarket, Kalshi, Limitless, Opinion, Predict.Fun, dFlow, Binance Futures data.

**CRITICAL:** The full 80+ endpoint catalog is in `skills/markets/SKILL.md`, NOT in the tool description.

```typescript
// Search Polymarket events
{
  "path": "polymarket/events",
  "query": {
    "search": "Fed rate decision",
    "limit": 5
  }
}

// Get market candles
{
  "path": "polymarket/candles",
  "query": {
    "market_id": "0xabc...",
    "interval": "1d",
    "limit": 30
  }
}

// Leaderboard
{
  "path": "polymarket/leaderboard",
  "query": {
    "period": "7d"
  }
}

// Cross-platform search
{
  "path": "search",
  "query": {
    "q": "Bitcoin $100k",
    "platforms": ["polymarket", "kalshi"]
  }
}
```

**Cost:** $0.001–$0.005/query

### blockrun_surf

Surf (asksurf.ai) — 84 endpoints for CEX data, on-chain SQL (13 chains, 80+ tables), labeled wallets, social mindshare, news.

**CRITICAL:** Full endpoint catalog in `skills/surf/SKILL.md`.

```typescript
// Get wallet labels
{
  "path": "wallet/labels/batch",
  "body": {
    "addresses": ["0xabc...", "0xdef..."]
  }
}

// On-chain SQL query
{
  "path": "onchain/sql",
  "body": {
    "sql": "SELECT token_address, SUM(volume_usd) as vol FROM base.dex_trades WHERE block_time > now() - interval '24 hours' GROUP BY token_address ORDER BY vol DESC LIMIT 10"
  }
}

// Market price
{
  "path": "market/price",
  "query": {
    "symbol": "BTC"
  }
}

// Wallet net worth
{
  "path": "wallet/net-worth",
  "query": {
    "address": "0xabc..."
  }
}
```

**Cost:** $0.001–$0.02/call

### blockrun_exa

Neural web search for research, competitors, papers, URL content.

```typescript
// Search for research papers
{
  "action": "search",
  "query": "speculative decoding",
  "type": "papers",
  "num_results": 10,
  "start_published_date": "2025-01-01"
}

// Get full content from URLs
{
  "action": "contents",
  "urls": ["https://arxiv.org/abs/2401.12345"],
  "text": true
}

// Find similar to URL
{
  "action": "similar",
  "url": "https://example.com/article",
  "num_results": 5
}
```

**Cost:** $0.01/query

### blockrun_search

Grok Live Search — web + news with citations.

```typescript
{
  "query": "latest AI regulation EU",
  "max_results": 10  // default, adjust as needed
}
```

**Cost:** $0.025 × max_results (default $0.25)

### blockrun_chat

Access 66+ LLMs (GPT, Claude, Gemini, DeepSeek, Kimi K2.6, GLM, NVIDIA free tier, etc.).

**CRITICAL:** `routing:"smart"` (ClawRouter) only works on Base wallets. On Solana, use `mode:` or `model:`.

```typescript
// Smart routing (Base only)
{
  "messages": [
    {"role": "user", "content": "Explain quantum computing"}
  ],
  "routing": "smart"
}

// Specific model
{
  "messages": [
    {"role": "user", "content": "Write a haiku"}
  ],
  "model": "openai/gpt-4o"
}

// Mode-based tier
{
  "messages": [
    {"role": "user", "content": "Draft email"}
  ],
  "mode": "free"  // or "fast", "balanced", "quality"
}

// With agent budget tracking
{
  "messages": [
    {"role": "user", "content": "Analyze this data"}
  ],
  "model": "anthropic/claude-3.5-sonnet",
  "agent_id": "research-agent-1"
}
```

**Cost:** Per token (varies by model)

### blockrun_image

GPT Image 1/2, Nano Banana/Pro (up to 4K), Grok Imagine, CogView-4 — generation + img2img editing.

```typescript
// Generate image
{
  "action": "generate",
  "prompt": "Retro-futuristic poster with headline 'NOW LIVE', neon colors, BlockRun branding",
  "model": "openai/gpt-image-2",
  "size": "1024x1024"
}

// Image-to-image editing
{
  "action": "edit",
  "image_url": "https://example.com/input.png",
  "prompt": "Add cyberpunk elements and neon glow",
  "model": "nano-banana/pro"
}
```

**Tip:** See `skills/image-prompting/SKILL.md` for 5-section framework (subject, style, composition, technical, text).

**Cost:** $0.015–$0.15/image

### blockrun_video

Sora 2, xAI Grok Imagine Video, ByteDance Seedance 1.5/2.0/2.0-fast (720p + audio defaults), RealFace-driven videos.

**CRITICAL:** Async payment-on-completion. Failures do NOT charge. Don't retry-loop — may take 60–180s.

```typescript
// Generate video
{
  "action": "generate",
  "prompt": "A serene mountain landscape at sunrise, camera slowly panning right",
  "model": "seedance/1.5-pro",
  "duration": 5,
  "resolution": "720p",
  "audio": true
}

// RealFace video (use enrolled ta_xxxx asset)
{
  "action": "generate",
  "prompt": "Product testimonial, excited expression",
  "model": "seedance/2.0",
  "realface_asset": "ta_abc123def456",
  "duration": 10
}
```

**Cost:** $0.05–$0.30/second

### blockrun_realface

Enroll a real person (phone liveness) or AI character (Virtual Portrait, no liveness) as a `ta_xxxx` asset for Seedance video.

```typescript
// Enroll real person
{
  "action": "enroll",
  "type": "real",
  "image_url": "https://example.com/headshot.jpg",
  "phone": "+14155551234"
}

// Enroll AI character (no liveness)
{
  "action": "enroll",
  "type": "virtual",
  "image_url": "https://example.com/ai-avatar.png"
}
```

**Cost:** Free enroll, $0.01 verification

### blockrun_music

MiniMax music generation.

```typescript
{
  "prompt": "Upbeat electronic track with synth leads, 140 BPM",
  "duration": 30
}
```

**Cost:** Per track (varies by duration)

### blockrun_speech

ElevenLabs text-to-speech (Flash/Turbo/Multilingual/v3) + cinematic sound effects.

```typescript
// Text-to-speech
{
  "action": "speak",  // default
  "input": "Welcome to BlockRun. Your balance is five dollars.",
  "voice": "sarah",  // or adam, bella, charlie, dan, emily, fin, grace
  "model": "eleven_flash_v2_5"
}

// Sound effect
{
  "action": "sound_effect",
  "input": "Futuristic spaceship door opening with hydraulic hiss"
}

// List available voices
{
  "action": "voices"
}
```

**Cost:** $0.05–$0.10/1k chars, $0.0525/effect

### blockrun_dex

Live DEX prices via DexScreener.

```typescript
{
  "pair": "PEPE/ETH",
  "chain": "ethereum"
}
```

**Cost:** Free

### blockrun_rpc

Raw JSON-RPC on 40+ chains (Ethereum, Base, Solana, Bitcoin, Sui, NEAR) via Tatum gateway.

```typescript
// Ethereum balance
{
  "chain": "ethereum",
  "method": "eth_getBalance",
  "params": ["0xabc...", "latest"]
}

// Solana account info
{
  "chain": "solana",
  "method": "getAccountInfo",
  "params": ["ABC..."]
}

// Smart contract call
{
  "chain": "base",
  "method": "eth_call",
  "params": [
    {
      "to": "0xcontract...",
      "data": "0x70a08231000000000000000000000000abc..."
    },
    "latest"
  ]
}
```

**Cost:** $0.002/call

### blockrun_defi

DefiLlama — protocol TVL, chain TVL, yield pools (APY), token prices.

```typescript
// Protocol TVL
{
  "action": "protocol",
  "protocol": "uniswap"
}

// Chain TVL
{
  "action": "chain",
  "chain": "ethereum"
}

// Yield pools
{
  "action": "pools",
  "chain": "base",
  "min_apy": 10
}

// Token price
{
  "action": "price",
  "token": "ethereum:0xabc..."
}
```

**Cost:** $0.001–$0.005/call

### blockrun_modal

Isolated code execution in BlockRun-hosted Modal sandbox — disposable container, optional GPU (T4 → H100).

```typescript
// Create sandbox
{
  "action": "create",
  "gpu": "t4"  // optional: t4, a10g, a100, h100
}
// Returns: { sandbox_id: "sb_xxx" }

// Execute code
{
  "action": "exec",
  "sandbox_id": "sb_xxx",
  "code": "import torch; print(torch.cuda.is_available())"
}

// Install package
{
  "action": "install",
  "sandbox_id": "sb_xxx",
  "package": "transformers"
}

// Destroy sandbox
{
  "action": "destroy",
  "sandbox_id": "sb_xxx"
}
```

**Cost:** $0.01 create, $0.001/op

### blockrun_phone

Outbound AI voice calls (Bland) + wallet-owned US/CA numbers (Twilio), carrier + fraud lookups.

```typescript
// Buy a phone number
{
  "path": "phone/numbers/buy",
  "body": {
    "area_code": "415"
  }
}

// Make voice call
{
  "path": "voice/call",
  "body": {
    "to": "+14155551234",
    "task": "Confirm the appointment on Friday at 3pm. If they can't make it, ask for an alternative time.",
    "from": "+14155559999"  // your wallet-owned number
  }
}
// Returns: { call_id: "call_xxx" }

// Poll call status
{
  "path": "voice/call/call_xxx"
}

// Carrier lookup
{
  "path": "phone/lookup",
  "query": {
    "phone": "+14155551234"
  }
}
```

**Cost:** $0.54/call, $5/number/month

### blockrun_models

Live catalog of every LLM/image/video/music model + pricing.

```typescript
{
  "category": "chat"  // or "image", "video", "music"
}
```

**Cost:** Free

## Common Patterns

### Multi-Agent Research with Budget Caps

Before spawning child agents, allocate per-agent budget to prevent runaway costs:

```typescript
// Parent allocates budget
{
  "tool": "blockrun_wallet",
  "action": "delegate",
  "agent_id": "crypto-researcher",
  "agent_limit": 0.50
}

// Child agent queries with agent_id
{
  "tool": "blockrun_chat",
  "messages": [{"role": "user", "content": "Analyze Base L2 metrics"}],
  "model": "anthropic/claude-3.5-sonnet",
  "agent_id": "crypto-researcher"
}

{
  "tool": "blockrun_surf",
  "path": "onchain/sql",
  "body": {
    "sql": "SELECT COUNT(*) FROM base.transactions WHERE block_time > now() - interval '7 days'"
  },
  "agent_id": "crypto-researcher"
}
```

Child is auto-blocked when budget hits zero.

### Free-Tier Drafting

Use free tools for scaffolding before paying for premium:

```typescript
// Free price check
{
  "tool": "blockrun_price",
  "symbol": "ETH/USD"
}

// Free DEX data
{
  "tool": "blockrun_dex",
  "pair": "ETH/USDC",
  "chain": "base"
}

// Free LLM draft
{
  "tool": "blockrun_chat",
  "messages": [{"role": "user", "content": "Draft summary"}],
  "mode": "free"  // NVIDIA free tier
}

// Then upgrade to paid model
{
  "tool": "blockrun_chat",
  "messages": [{"role": "user", "content": "Refine this summary: ..."}],
  "model": "openai/gpt-4o"
}
```

### Research Paper → Summary Flow

```typescript
// 1. Search for papers
const searchResult = {
  "tool": "blockrun_exa",
  "action": "search",
  "query": "retrieval-augmented generation improvements",
  "type": "papers",
  "num_results": 5,
  "start_published_date": "2025-03-01"
}
// Returns: [{ url, title, published_date, ... }]

// 2. Get full content
const contentResult = {
  "tool": "blockrun_exa",
  "action": "contents",
  "urls": [/* URLs from search */],
  "text": true
}
// Returns: [{ url, text }]

// 3. Summarize with LLM
{
  "tool": "blockrun_chat",
  "messages": [
    {"role": "user", "content": `Summarize key improvements in these papers:\n\n${contentResult.map(c => c.text).join('\n---\n')}`}
  ],
  "model": "anthropic/claude-3.5-sonnet"
}
```

### Prediction Market Odds → Decision

```typescript
// 1. Search events
const events = {
  "tool": "blockrun_markets",
  "path": "polymarket/events",
  "query": {
    "search": "Bitcoin 100k 2026",
    "limit": 3
  }
}
// Returns: [{ id, title, markets: [{ id, outcome, price }] }]

// 2. Get historical candles
const candles = {
  "tool": "blockrun_markets",
  "path": "polymarket/candles",
  "query": {
    "market_id": events[0].markets[0].id,
    "interval": "1d",
    "limit": 30
  }
}
// Returns: [{ time, open, high, low, close, volume }]

// 3. Analyze with LLM
{
  "tool": "blockrun_chat",
  "messages": [
    {"role": "user", "content": `Current odds: ${events[0].markets[0].price}. 30-day trend: ${JSON.stringify(candles)}. Should I hedge?`}
  ],
  "model": "openai/gpt-4o"
}
```

### Image Generation with Text Overlay

Use the 5-section framework from `skills/image-prompting/SKILL.md`:

```typescript
{
  "tool": "blockrun_image",
  "action": "generate",
  "prompt": `
Subject: BlockRun logo + "GPT-5.5 NOW LIVE" headline
Style: Retro-futuristic, synthwave, neon pink/blue gradients
Composition: Centered logo, bold sans-serif headline below, dark space background
Technical: 4K resolution, high contrast, sharp text edges
Text: Ensure "GPT-5.5 NOW LIVE" is perfectly legible, no typography errors
  `,
  "model": "openai/gpt-image-2",
  "size": "1024x1024"
}
```

### Voice Call Workflow

```typescript
// 1. Buy number (one-time)
const numberResult = {
  "tool": "blockrun_phone",
  "path": "phone/numbers/buy",
  "body": { "area_code": "415" }
}
// Returns: { phone_number: "+14155559999" }

// 2. Make call
const callResult = {
  "tool": "blockrun_phone",
  "path": "voice/call",
  "body": {
    "to": "+14155551234",
    "task": "Remind them about tomorrow's meeting at 10am. Ask if they need directions.",
    "from": numberResult.phone_number
  }
}
// Returns: { call_id: "call_abc123" }

// 3. Poll status (repeat until completed/failed)
{
  "tool": "blockrun_phone",
  "path": `voice/call/${callResult.call_id}`
}
// Returns: { status: "in-progress" | "completed" | "failed", transcript, ... }
```

## Critical Best Practices for LLM Agents

1. **Payment/Balance Errors:** When ANY `blockrun_*` tool returns a 402 payment error, call `blockrun_wallet` action `"balance"` FIRST to check status, then `"setup"` to get funding instructions. Do NOT retry the failing tool blindly.

2. **Endpoint Catalogs:** For `blockrun_markets` and `blockrun_surf`, the 80+ endpoint catalogs live in `skills/markets/SKILL.md` and `skills/surf/SKILL.md`, NOT in the tool description. Browse the skill before guessing paths.

3. **Smart Routing:** `blockrun_chat routing:"smart"` (ClawRouter) ONLY works on Base wallets. On Solana, pass `mode:` or `model:` to pick a model directly.

4. **Async Jobs:** `blockrun_music` and `blockrun_video` are payment-on-completion async. Failures or client timeouts do NOT charge. Don't retry-loop — they may take 60–180s.

5. **Agent Budgets:** Before spawning child agents, allocate per-agent budget via `blockrun_wallet action:"delegate"`. Pass `agent_id:"X"` to every downstream `blockrun_*` call — child auto-blocked at zero.

6. **Free Tier First:** Use `blockrun_chat mode:"free"` (NVIDIA), `blockrun_dex`, `blockrun_price` (crypto/FX/commodity), and `blockrun_models` (all $0) for drafts before paying for premium models.

## Troubleshooting

### "Insufficient balance" Error

```typescript
// Check wallet
{
  "tool": "blockrun_wallet",
  "action": "balance"
}
// Fund via displayed address

// Get QR code
{
  "tool": "blockrun_wallet",
  "action": "setup"
}
```

### "Unknown endpoint" on blockrun_markets or blockrun_surf

The endpoint catalog is NOT in the tool description. Check:
- `skills/markets/SKILL.md` for Polymarket/Kalshi/etc paths
- `skills/surf/SKILL.md` for Surf's 84 endpoints

### Smart Routing Fails on Solana Wallet

`routing:"smart"` only works on Base. Use explicit `mode:` or `model:`:

```typescript
{
  "tool": "blockrun_chat",
  "messages": [{"role": "user", "content": "Hello"}],
  "mode": "balanced"  // not routing: "smart"
}
```

### Video/Music Generation Times Out

This is normal for 60–180s jobs. Do NOT retry immediately — the job may still be processing and you won't be charged for client-side timeouts. Wait or poll status if the tool supports it.

### Agent Exceeds Budget

Check delegation:

```typescript
{
  "tool": "blockrun_wallet",
  "action": "balance"
}
// Shows per-agent spending
```

Increase or reset:

```typescript
{
  "tool": "blockrun_wallet",
  "action": "delegate",
  "agent_id": "research-agent-1",
  "agent_limit": 2.00  // new limit
}
```

## Cost Optimization Tips

1. **Use free tools first:** `blockrun_price` (crypto/FX/commodity), `blockrun_dex`, `blockrun_models`, `blockrun_chat mode:"free"`
2. **Batch requests:** Single `blockrun_surf` SQL query can replace multiple `blockrun_rpc` calls
3. **Cache aggressively:** Don't re-fetch static data (e.g., model catalog, historical candles)
4. **Set agent budgets:** Prevent runaway loops with `blockrun_wallet delegate`
5. **Preview before video/music:** Use `blockrun_image` for storyboards (~$0.05) before committing to `blockrun_video` (~$1–3)

## When NOT to Use BlockRun

- **High-volume single-API (≥10k calls/day):** Direct subscriptions amortize better
- **Compliance/invoice requirements:** USDC settlement isn't a tax invoice
- **Sub-100ms latency critical:** x402 adds ~200–500ms overhead
- **Single source forever:** If you only need Polymarket, skip the indirection

Use BlockRun for exploration, aggregation, or agent-driven workloads where you can't predict which source you'll need.

## Additional Resources

- **Endpoint catalogs:** `skills/markets/SKILL.md`, `skills/surf/SKILL.md`
- **Image prompting guide:** `skills/image-prompting/SKILL.md`
- **Model catalog:** Use `blockrun_models` tool
- **npm package:** [@blockrun/mcp](https://www.npmjs.com/package/@blockrun/mcp)
- **License:** MIT
