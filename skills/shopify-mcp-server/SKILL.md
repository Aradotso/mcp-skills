---
name: shopify-mcp-server
description: MCP server connecting AI assistants to Shopify Admin GraphQL API for products, orders, customers, inventory, and metafields management
triggers:
  - connect Claude to my Shopify store
  - manage Shopify products with MCP
  - set up Shopify MCP server
  - query Shopify orders through Claude
  - configure Shopify MCP authentication
  - use Shopify Admin API with AI assistant
  - install shopify-mcp for Claude Desktop
  - create Shopify metafields with MCP
---

# Shopify MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A production-ready Model Context Protocol (MCP) server that exposes Shopify Admin GraphQL API to AI assistants like Claude Desktop. Provides 40 typed tools for managing products, customers, orders, inventory, metafields, and store operations through a structured interface.

## Architecture Overview

The server translates MCP tool calls into Shopify Admin GraphQL operations. It supports:
- Static access tokens (legacy custom apps)
- OAuth client-credentials flow (Dev Dashboard apps)
- Optional Redis token caching
- Structured JSON logging
- Graceful shutdown and error handling

**Request flow:** MCP Client → Tool Registry → Zod Validation → Shopify GraphQL API → Formatted Response

## Installation

### Quick Start with npx

```bash
npx shopify-mcp \
  --clientId YOUR_CLIENT_ID \
  --clientSecret YOUR_CLIENT_SECRET \
  --domain your-store.myshopify.com
```

### From Source

```bash
git clone https://github.com/Cesarjoquin/shopify-mcp.git
cd shopify-mcp
npm install
npm run build
npm start
```

### Claude Desktop Configuration

Add to `claude_desktop_config.json`:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "shopify": {
      "command": "npx",
      "args": [
        "shopify-mcp",
        "--clientId", "<CLIENT_ID>",
        "--clientSecret", "<CLIENT_SECRET>",
        "--domain", "<store>.myshopify.com"
      ]
    }
  }
}
```

## Configuration

### Environment Variables

Create `.env` from `.env.example`:

```bash
# Store domain (required)
MYSHOPIFY_DOMAIN=your-store.myshopify.com

# Authentication Option A: Static token
SHOPIFY_ACCESS_TOKEN=shpat_xxxxxxxxxxxxx

# Authentication Option B: OAuth credentials
SHOPIFY_CLIENT_ID=shp_xxxxxxxxxxxxx
SHOPIFY_CLIENT_SECRET=shpcs_xxxxxxxxxxxxx

# Optional settings
SHOPIFY_API_VERSION=2026-01
LOG_LEVEL=info

# Redis token caching (optional)
REDIS_ENABLED=false
REDIS_URL=redis://127.0.0.1:6379
REDIS_KEY_PREFIX=shopify-mcp:
REDIS_MAX_RETRIES=10
REDIS_CONNECT_TIMEOUT_MS=10000
```

### CLI Flags (Override Environment)

```bash
shopify-mcp \
  --domain store.myshopify.com \
  --accessToken shpat_xxx \
  --apiVersion 2026-01
```

### Required Shopify API Scopes

For full functionality, grant these scopes in your Shopify custom app:
- `read_products`, `write_products`
- `read_customers`, `write_customers`
- `read_orders`, `write_orders`
- `read_inventory`, `write_inventory`
- `read_metaobjects`, `write_metaobjects`

## Key MCP Tools

### Product Management

**List Products:**
```typescript
// Tool: list_products
{
  "first": 10,
  "query": "status:active",
  "sortKey": "TITLE"
}
```

**Create Product:**
```typescript
// Tool: create_product
{
  "title": "New Product",
  "descriptionHtml": "<p>Description</p>",
  "productType": "Apparel",
  "vendor": "My Brand",
  "tags": ["summer", "sale"],
  "variants": [
    {
      "price": "29.99",
      "sku": "PROD-001",
      "inventoryQuantities": {
        "availableQuantity": 100,
        "locationId": "gid://shopify/Location/123"
      }
    }
  ]
}
```

**Update Product:**
```typescript
// Tool: update_product
{
  "id": "gid://shopify/Product/123",
  "title": "Updated Title",
  "status": "ACTIVE"
}
```

### Customer Management

**List Customers:**
```typescript
// Tool: list_customers
{
  "first": 25,
  "query": "email:*@example.com",
  "sortKey": "CREATED_AT",
  "reverse": true
}
```

**Create Customer:**
```typescript
// Tool: create_customer
{
  "email": "customer@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "phone": "+1234567890",
  "acceptsMarketing": true,
  "tags": ["vip", "wholesale"]
}
```

### Order Management

**List Orders:**
```typescript
// Tool: list_orders
{
  "first": 50,
  "query": "fulfillment_status:unfulfilled",
  "sortKey": "CREATED_AT"
}
```

**Get Order Details:**
```typescript
// Tool: get_order
{
  "id": "gid://shopify/Order/123456"
}
```

**Fulfill Order:**
```typescript
// Tool: fulfill_order
{
  "orderId": "gid://shopify/Order/123",
  "lineItems": [
    {
      "id": "gid://shopify/LineItem/456",
      "quantity": 2
    }
  ],
  "trackingInfo": {
    "company": "USPS",
    "number": "1Z999AA10123456784"
  }
}
```

### Metafield Operations

**Set Metafield:**
```typescript
// Tool: set_metafield
{
  "ownerId": "gid://shopify/Product/123",
  "namespace": "custom",
  "key": "instructions",
  "type": "single_line_text_field",
  "value": "Handle with care"
}
```

**Get Metafields:**
```typescript
// Tool: get_metafields
{
  "ownerId": "gid://shopify/Product/123",
  "namespace": "custom"
}
```

### Inventory Management

**Set Inventory Quantity:**
```typescript
// Tool: set_inventory_quantity
{
  "inventoryItemId": "gid://shopify/InventoryItem/789",
  "locationId": "gid://shopify/Location/123",
  "availableQuantity": 50
}
```

**Get Inventory Levels:**
```typescript
// Tool: get_inventory_levels
{
  "inventoryItemId": "gid://shopify/InventoryItem/789"
}
```

### Store Discovery

**Get Shop Info:**
```typescript
// Tool: get_shop_info
{}
```

**List Locations:**
```typescript
// Tool: list_locations
{
  "first": 10
}
```

## Common Patterns

### Pagination

All list tools support cursor-based pagination:

```typescript
// First request
{
  "first": 25,
  "query": "status:active"
}

// Response includes pageInfo.endCursor
// Next request:
{
  "first": 25,
  "after": "eyJsYXN0X2lkIjo...",
  "query": "status:active"
}
```

### Shopify Search Syntax

Use Shopify's query syntax for filtering:

```typescript
// Products by vendor and tag
"query": "vendor:Nike tag:running"

// Orders by status and date
"query": "fulfillment_status:unfulfilled created_at:>2026-01-01"

// Customers by email domain
"query": "email:*@company.com"
```

### Global IDs

Shopify uses GraphQL global IDs:

```typescript
// Format: gid://shopify/{ResourceType}/{numericId}
"id": "gid://shopify/Product/7891234567890"
"id": "gid://shopify/Customer/1234567890"
"id": "gid://shopify/Order/4567890123456"
```

### Error Handling

Tools return `userErrors` arrays from Shopify mutations:

```typescript
// Example error response
{
  "isError": true,
  "error": "Failed to create product: Title can't be blank"
}
```

## Development Workflow

### Build and Test

```bash
# Install dependencies
npm install

# Type checking
npm run typecheck

# Linting
npm run lint

# Run tests
npm test

# Full validation pipeline
npm run validate

# Build for production
npm run build

# Development mode (hot reload)
npm run dev
```

### Adding Custom Tools

Tools use the `createTool()` factory pattern:

```typescript
// src/tools/custom_tool.ts
import { z } from 'zod';
import { createTool } from '../lib/tool-factory.js';

const inputSchema = z.object({
  myParam: z.string().describe('Parameter description')
});

export const myCustomTool = createTool(
  'my_custom_tool',
  'Description of what this tool does',
  inputSchema,
  async (args, shopify) => {
    const query = `
      query MyQuery($param: String!) {
        # GraphQL query here
      }
    `;
    
    const result = await shopify.graphql(query, { param: args.myParam });
    return { success: true, data: result };
  }
);
```

### GraphQL Schema Validation

```bash
# Validate GraphQL queries against Shopify schema
npm run validate:graphql
```

## Troubleshooting

### Authentication Failures

**Error:** `Authentication credentials are required`

**Solution:** Ensure you provide either:
- `SHOPIFY_ACCESS_TOKEN` (static token), OR
- Both `SHOPIFY_CLIENT_ID` and `SHOPIFY_CLIENT_SECRET` (OAuth)

### MCP Connection Issues

**Check Claude Desktop logs:**

```bash
# macOS
tail -f ~/Library/Logs/Claude/mcp*.log

# Windows
# Check %APPDATA%/Claude/logs/
```

**Verify server starts:**

```bash
SHOPIFY_ACCESS_TOKEN=shpat_xxx \
MYSHOPIFY_DOMAIN=store.myshopify.com \
npm start
```

### Redis Connection Errors

**Error:** Redis connection timeout

**Solutions:**
1. Start Redis locally:
```bash
docker run -d -p 6379:6379 redis:7-alpine
```

2. Or disable Redis caching:
```bash
REDIS_ENABLED=false
```

### GraphQL userErrors

Shopify returns field-level validation errors in mutation responses. These are surfaced as:

```
Failed to create product: field: error message
```

Check your input data matches Shopify's validation rules (required fields, format constraints, etc.).

### API Version Compatibility

If tools fail with "Field not found" errors, check API version compatibility:

```bash
# Use a specific API version
SHOPIFY_API_VERSION=2025-01 npm start
```

## Redis Token Caching

For OAuth client-credentials flow, tokens are cached in Redis:

```typescript
// Token cache structure
{
  key: "shopify-mcp:token:store.myshopify.com",
  value: {
    access_token: "shpat_...",
    expires_at: 1234567890
  },
  ttl: 86400 // 24 hours
}
```

**Auto-refresh:** Tokens refresh automatically 5 minutes before expiry.

**Manual cache clear:**
```bash
redis-cli DEL "shopify-mcp:token:your-store.myshopify.com"
```

## Production Deployment

### Environment Variables Checklist

```bash
# Required
MYSHOPIFY_DOMAIN=store.myshopify.com
SHOPIFY_CLIENT_ID=shp_xxx
SHOPIFY_CLIENT_SECRET=shpcs_xxx

# Recommended
LOG_LEVEL=info
SHOPIFY_API_VERSION=2026-01
REDIS_ENABLED=true
REDIS_URL=redis://redis:6379
```

### Docker Deployment

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

### Health Monitoring

The server logs structured JSON to stderr. Monitor for:
- `"level":"error"` entries
- GraphQL API rate limit warnings
- Redis connection failures

## Tool Categories Reference

| Category | Count | Key Tools |
|----------|-------|-----------|
| Products | 12 | list, create, update, delete, variants |
| Customers | 8 | list, create, update, delete, merge |
| Orders | 10 | list, get, cancel, fulfill, refund |
| Metafields | 4 | get, set, delete (any resource) |
| Inventory | 4 | set quantity, get levels/items |
| Discovery | 2 | shop info, locations |

All list tools support: `first`, `after`, `query`, `sortKey`, `reverse` parameters.
