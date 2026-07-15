---
name: solana-mcp-vybe
description: Use Vybe's public Solana MCP server to query Solana blockchain data, tokens, NFTs, and accounts from Claude, Cursor, or any MCP client
triggers:
  - connect to Solana blockchain using MCP
  - query Solana accounts with Vybe MCP
  - set up Solana MCP server in Cursor
  - get SPL token data through MCP
  - configure Vybe Solana MCP client
  - browse Solana schemas with MCP
  - make Solana API calls from Claude
  - integrate Token-2022 with MCP
---

# solana-mcp-vybe

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What It Does

Solana MCP by Vybe provides a **public Model Context Protocol (MCP) server** that enables Claude Desktop, Cursor, VS Code, and other MCP clients to interact with the Solana blockchain. It exposes Solana RPC methods, SPL token queries, Token-2022 operations, and account lookups through a standardized MCP interface at `https://mcp.vybenetwork.xyz`.

Instead of writing custom Solana integration code, AI agents can connect to this MCP server and immediately query balances, transaction history, token metadata, NFTs, and execute read operations against Solana mainnet/devnet.

## Installation

### Cursor Setup

Add to your Cursor MCP configuration file (`~/.cursor/mcp.json` or workspace `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "solana-vybe": {
      "url": "https://mcp.vybenetwork.xyz"
    }
  }
}
```

Authentication uses OAuth via the client's flow for the MCP host.

### Claude Desktop Setup

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "solana-vybe": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.vybenetwork.xyz"
      ]
    }
  }
}
```

### VS Code with Cline/Roo-Cline

Add to MCP settings:

```json
{
  "solana-vybe": {
    "url": "https://mcp.vybenetwork.xyz"
  }
}
```

### Verify Connection

After configuration, restart your MCP client. The Solana MCP tools should appear in your client's tool/resource list. Check for tools like:

- `solana_get_account_info`
- `solana_get_balance`
- `solana_get_token_accounts`
- `solana_get_transaction`

## Key Capabilities

### Available MCP Tools

The server exposes Solana RPC methods as MCP tools. Common operations include:

**Account Operations:**
- Get account balance (SOL)
- Fetch account info and data
- Query token accounts by owner
- Get program accounts

**Token Operations:**
- Get SPL token balance
- Query token metadata
- List token accounts
- Token-2022 extensions support

**Transaction Operations:**
- Get transaction details
- Query transaction history
- Check transaction status

**Blockchain Queries:**
- Get block information
- Query slot/epoch data
- Network status

## Usage Examples

### Query SOL Balance

Ask your AI agent:

```
What's the SOL balance of address 7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU?
```

The agent will call the MCP server's `solana_get_balance` tool with:

```json
{
  "address": "7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU",
  "network": "mainnet-beta"
}
```

### Get Token Accounts

```
Show me all SPL token accounts owned by CuieVDEDtLo7FypA9SbLM9saXFdb1dsshEkyErMqkRQq
```

The agent uses `solana_get_token_accounts`:

```json
{
  "owner": "CuieVDEDtLo7FypA9SbLM9saXFdb1dsshEkyErMqkRQq",
  "network": "mainnet-beta"
}
```

### Fetch Transaction Details

```
Get details for transaction 5wHu1qwD7q5ifaN5nwdcDqNFo53GJqa7nLp2BeeEpcnkKoLkJGNK3omvLsEvVpHndv2QgHCzZQmAy6Qx8NXKvPRQ
```

Agent calls `solana_get_transaction`:

```json
{
  "signature": "5wHu1qwD7q5ifaN5nwdcDqNFo53GJqa7nLp2BeeEpcnkKoLkJGNK3omvLsEvVpHndv2QgHCzZQmAy6Qx8NXKvPRQ",
  "network": "mainnet-beta"
}
```

### Query Token Metadata

```
What's the metadata for token mint EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v?
```

Agent uses `solana_get_token_metadata`:

```json
{
  "mint": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
  "network": "mainnet-beta"
}
```

### Switch Networks

All tools support `network` parameter:
- `mainnet-beta` (default)
- `devnet`
- `testnet`

```
Check the devnet balance of 7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU
```

## Common Patterns

### Building a Token Portfolio Viewer

```typescript
// Agent workflow pseudocode
const owner = "USER_WALLET_ADDRESS";

// 1. Get SOL balance
const solBalance = await mcp_call("solana_get_balance", {
  address: owner,
  network: "mainnet-beta"
});

// 2. Get all token accounts
const tokenAccounts = await mcp_call("solana_get_token_accounts", {
  owner: owner,
  network: "mainnet-beta"
});

// 3. For each token, fetch metadata
for (const account of tokenAccounts) {
  const metadata = await mcp_call("solana_get_token_metadata", {
    mint: account.mint,
    network: "mainnet-beta"
  });
  // Display name, symbol, balance
}
```

### Transaction History Analysis

```typescript
// Get recent transactions for an address
const transactions = await mcp_call("solana_get_signatures_for_address", {
  address: "USER_ADDRESS",
  limit: 10,
  network: "mainnet-beta"
});

// Fetch details for each
for (const sig of transactions) {
  const txDetail = await mcp_call("solana_get_transaction", {
    signature: sig.signature,
    network: "mainnet-beta"
  });
  // Parse transfers, program interactions
}
```

### NFT Collection Query

```typescript
// Get token accounts (NFTs are tokens)
const nftAccounts = await mcp_call("solana_get_token_accounts", {
  owner: "COLLECTOR_ADDRESS",
  network: "mainnet-beta"
});

// Filter for NFTs (amount = 1, decimals = 0)
const nfts = nftAccounts.filter(acc => 
  acc.amount === "1" && acc.decimals === 0
);

// Fetch metadata for each NFT mint
for (const nft of nfts) {
  const metadata = await mcp_call("solana_get_token_metadata", {
    mint: nft.mint,
    network: "mainnet-beta"
  });
  // metadata contains name, image URI, attributes
}
```

## Configuration

### Network Selection

Always specify the `network` parameter in tool calls:

```json
{
  "network": "mainnet-beta"  // or "devnet", "testnet"
}
```

### Authentication

The public MCP server at `https://mcp.vybenetwork.xyz` uses **OAuth authentication** managed by your MCP client. No API keys need to be stored in your configuration files.

When you first connect, your client will prompt for OAuth authorization. Follow the client's authentication flow.

### Rate Limiting

The public server has rate limits. For production applications requiring high throughput:

1. Contact Vybe Network for enterprise access
2. Consider hosting a private instance
3. Implement request batching in your agent logic

### Environment Variables

If you need to override the default endpoint (e.g., for testing):

```bash
export VYBE_MCP_URL="https://custom-mcp.example.com"
```

Then reference in config:

```json
{
  "url": "$VYBE_MCP_URL"
}
```

## Troubleshooting

### Connection Refused

**Symptom:** Client can't connect to `https://mcp.vybenetwork.xyz`

**Solutions:**
1. Check network connectivity: `curl https://mcp.vybenetwork.xyz`
2. Verify MCP config syntax (valid JSON)
3. Restart your MCP client after config changes
4. Check client logs for OAuth errors

### Tool Not Found

**Symptom:** Agent says "Tool solana_get_balance not available"

**Solutions:**
1. Confirm MCP server is connected (check client status)
2. List available tools in your client's MCP inspector
3. Verify you're using the correct tool name (snake_case)
4. Restart client to refresh tool registry

### Authentication Errors

**Symptom:** 401/403 errors or "Authentication required"

**Solutions:**
1. Complete OAuth flow in your client
2. Check if OAuth token expired (re-authenticate)
3. For organization repos, grant OAuth app access to `vybenetwork` org
4. Verify SAML/SSO compliance if using enterprise GitHub

### Invalid Address/Signature

**Symptom:** "Invalid base58 string" or "Invalid public key"

**Solutions:**
1. Verify Solana addresses are valid base58
2. Check address length (32-44 characters typical)
3. Ensure transaction signatures are valid base58 (87-88 chars)
4. Don't confuse mainnet/devnet addresses

### Network Timeout

**Symptom:** Requests hang or timeout

**Solutions:**
1. Check Solana network status (mainnet-beta/devnet)
2. Retry with exponential backoff
3. Reduce concurrent requests (rate limiting)
4. Switch to `devnet` if mainnet is congested

### Missing Token Metadata

**Symptom:** Token metadata returns null or incomplete

**Solutions:**
1. Not all tokens have metadata (especially older SPL tokens)
2. Check if the mint address is correct
3. Verify token uses Metaplex standard
4. Query on-chain account data directly as fallback

## Advanced Usage

### Custom Queries with Program Filters

```typescript
// Get accounts owned by a specific program
const programAccounts = await mcp_call("solana_get_program_accounts", {
  programId: "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
  filters: [
    {
      memcmp: {
        offset: 0,
        bytes: "OWNER_ADDRESS_BASE58"
      }
    }
  ],
  network: "mainnet-beta"
});
```

### Block Explorer Integration

```typescript
// Build a block explorer URL from transaction signature
const signature = "5wHu1qwD...";
const explorerUrl = `https://explorer.solana.com/tx/${signature}`;

// Or use Solscan
const solscanUrl = `https://solscan.io/tx/${signature}`;
```

## Resources

- **Documentation:** [docs.vybenetwork.com/docs/mcp](https://docs.vybenetwork.com/docs/mcp)
- **MCP Endpoint:** `https://mcp.vybenetwork.xyz`
- **Registry:** [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io)
- **GitHub:** [github.com/vybenetwork/solana-mcp-vybe](https://github.com/vybenetwork/solana-mcp-vybe)

## Example Agent Prompts

Use these prompts to test your Solana MCP integration:

- "What's the current SOL price in the latest block?"
- "Show me the top 5 token holdings for address X"
- "Analyze the transaction history for this NFT mint"
- "Compare token balances between mainnet and devnet for this wallet"
- "List all Token-2022 accounts with transfer fees enabled"
- "What programs did this transaction interact with?"
