---
name: commercevault-edd-mcp-server
description: Easy Digital Downloads MCP server for sales analytics, order management, and product data via natural language
triggers:
  - "connect to my Easy Digital Downloads store"
  - "get EDD sales analytics and revenue reports"
  - "fetch product catalog from WordPress EDD"
  - "query customer orders and download history"
  - "analyze EDD sales data and trends"
  - "manage digital product licenses in EDD"
  - "retrieve EDD transaction and payment data"
  - "synchronize Easy Digital Downloads inventory"
---

# CommerceVault EDD MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

CommerceVault (mcp-edd-analytics-vantage) is an MCP (Model Context Protocol) server that provides natural language access to Easy Digital Downloads (EDD) WordPress e-commerce data. It enables AI assistants to query sales analytics, manage products, retrieve customer orders, and perform revenue analysis through a structured API interface.

## What It Does

This MCP server acts as a middleware layer between AI coding agents and Easy Digital Downloads REST API, providing:

- **Sales Analytics**: Revenue reports, GMV, AOV, cohort analysis
- **Product Management**: Catalog synchronization, pricing, digital downloads
- **Order Operations**: Lifecycle management, fulfillment tracking, refunds
- **Customer Data**: Profiles, purchase history, LTV calculations
- **License Management**: Key generation, activation tracking, entitlements
- **Webhook Relay**: Event forwarding with transformation and enrichment

## Installation

### Prerequisites

- Node.js 18+ or Python 3.9+
- Easy Digital Downloads installed on WordPress site
- EDD REST API credentials (API key and token)
- MCP-compatible client (Claude Desktop, Cursor, etc.)

### Install as MCP Server

1. Clone or download the repository:

```bash
git clone https://github.com/dhapat3927/mcp-edd-analytics-vantage.git
cd mcp-edd-analytics-vantage
```

2. Install dependencies:

```bash
npm install
# or
pip install -r requirements.txt
```

3. Configure MCP client settings (e.g., Claude Desktop `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "commercevault-edd": {
      "command": "node",
      "args": ["/path/to/mcp-edd-analytics-vantage/server.js"],
      "env": {
        "EDD_API_URL": "https://yoursite.com/wp-json/edd/v2",
        "EDD_API_KEY": "${EDD_API_KEY}",
        "EDD_API_TOKEN": "${EDD_API_TOKEN}",
        "CACHE_ENABLED": "true",
        "CACHE_TTL": "300"
      }
    }
  }
}
```

For Python implementation:

```json
{
  "mcpServers": {
    "commercevault-edd": {
      "command": "python",
      "args": ["/path/to/mcp-edd-analytics-vantage/server.py"],
      "env": {
        "EDD_API_URL": "https://yoursite.com/wp-json/edd/v2",
        "EDD_API_KEY": "${EDD_API_KEY}",
        "EDD_API_TOKEN": "${EDD_API_TOKEN}"
      }
    }
  }
}
```

## Configuration

### Environment Variables

| Variable | Required | Description | Default |
|----------|----------|-------------|---------|
| `EDD_API_URL` | Yes | Full URL to EDD REST API endpoint | - |
| `EDD_API_KEY` | Yes | EDD API key from WordPress admin | - |
| `EDD_API_TOKEN` | Yes | EDD API token for authentication | - |
| `CACHE_ENABLED` | No | Enable Redis-backed response caching | `false` |
| `CACHE_TTL` | No | Cache time-to-live in seconds | `300` |
| `REDIS_URL` | No | Redis connection string | `redis://localhost:6379` |
| `RATE_LIMIT_MAX` | No | Max requests per minute | `60` |
| `LOG_LEVEL` | No | Logging verbosity (debug/info/warn/error) | `info` |
| `HMAC_SECRET` | No | Shared secret for payload signatures | - |

### Getting EDD API Credentials

1. Log into WordPress admin
2. Navigate to **Downloads → Settings → API**
3. Click **Generate API Keys**
4. Copy the Public Key (API_KEY) and Token (API_TOKEN)
5. Set appropriate permissions for the key (read/write)

## Key API Tools

The MCP server exposes these tools for AI agents:

### Products

**`edd_get_products`** - Retrieve product catalog

```typescript
// Parameters
{
  "category": "software",      // Filter by category slug
  "tag": "featured",          // Filter by tag
  "search": "premium",        // Search product titles
  "per_page": 20,            // Results per page (default: 10)
  "page": 1                  // Page number
}
```

**`edd_get_product`** - Get single product details

```typescript
{
  "product_id": 123          // Product ID
}
```

### Orders

**`edd_get_orders`** - Retrieve order history

```typescript
{
  "status": "complete",      // Order status filter
  "customer": 456,          // Customer ID
  "start_date": "2026-01-01",
  "end_date": "2026-06-30",
  "per_page": 50
}
```

**`edd_get_order`** - Get single order details

```typescript
{
  "order_id": 789
}
```

**`edd_create_order`** - Create new order

```typescript
{
  "customer_id": 456,
  "products": [
    {
      "product_id": 123,
      "price": 49.99,
      "quantity": 1
    }
  ],
  "status": "pending",
  "gateway": "stripe"
}
```

### Customers

**`edd_get_customers`** - Retrieve customer list

```typescript
{
  "email": "user@example.com",
  "order_by": "purchase_count",
  "order": "desc"
}
```

**`edd_get_customer_stats`** - Get customer analytics

```typescript
{
  "customer_id": 456
}
// Returns: total_spent, purchase_count, avg_order_value, lifetime_value
```

### Analytics

**`edd_get_sales_stats`** - Revenue and sales metrics

```typescript
{
  "start_date": "2026-01-01",
  "end_date": "2026-06-30",
  "interval": "day",         // day, week, month
  "product_id": 123         // Optional: filter by product
}
// Returns: revenue, orders, avg_order_value, gross_merchandise_value
```

**`edd_get_top_products`** - Best-selling products report

```typescript
{
  "period": "30days",        // 7days, 30days, 90days, year
  "limit": 10
}
```

### Licenses

**`edd_get_licenses`** - Retrieve license keys

```typescript
{
  "product_id": 123,
  "status": "active",        // active, inactive, expired
  "customer_id": 456
}
```

**`edd_activate_license`** - Activate license key

```typescript
{
  "license_key": "XXXX-XXXX-XXXX-XXXX",
  "site_url": "https://customer-site.com",
  "item_id": 123
}
```

## Code Examples

### Example 1: Fetch Monthly Revenue Report

```javascript
// server.js implementation snippet
async function getSalesStats(params) {
  const { start_date, end_date, interval = 'day' } = params;
  
  const response = await fetch(
    `${process.env.EDD_API_URL}/reports/sales?` +
    `start_date=${start_date}&end_date=${end_date}&interval=${interval}`,
    {
      headers: {
        'Authorization': `Basic ${Buffer.from(
          `${process.env.EDD_API_KEY}:${process.env.EDD_API_TOKEN}`
        ).toString('base64')}`,
        'Content-Type': 'application/json'
      }
    }
  );
  
  if (!response.ok) {
    throw new Error(`EDD API error: ${response.statusText}`);
  }
  
  const data = await response.json();
  
  return {
    revenue: data.totals.revenue,
    orders: data.totals.orders,
    avg_order_value: data.totals.revenue / data.totals.orders,
    interval_data: data.data
  };
}
```

### Example 2: Create Order with License

```python
# server.py implementation snippet
import requests
import os
from base64 import b64encode

def create_order_with_license(customer_id, product_id, price):
    auth_string = f"{os.getenv('EDD_API_KEY')}:{os.getenv('EDD_API_TOKEN')}"
    auth_header = b64encode(auth_string.encode()).decode()
    
    headers = {
        'Authorization': f'Basic {auth_header}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'customer': customer_id,
        'products': [{
            'product_id': product_id,
            'price': price,
            'quantity': 1,
            'options': {
                'generate_license': True
            }
        }],
        'status': 'complete',
        'gateway': 'manual'
    }
    
    response = requests.post(
        f"{os.getenv('EDD_API_URL')}/orders",
        json=payload,
        headers=headers
    )
    
    response.raise_for_status()
    order_data = response.json()
    
    return {
        'order_id': order_data['id'],
        'license_key': order_data['licenses'][0]['key'],
        'total': order_data['total']
    }
```

### Example 3: Customer Lifetime Value Analysis

```javascript
async function analyzeCustomerLTV(customer_id) {
  const orders = await eddRequest('GET', '/orders', {
    customer: customer_id,
    status: 'complete'
  });
  
  const stats = orders.reduce((acc, order) => {
    acc.total_spent += parseFloat(order.total);
    acc.order_count += 1;
    acc.products_purchased += order.items.length;
    
    const orderDate = new Date(order.date_created);
    if (!acc.first_purchase || orderDate < acc.first_purchase) {
      acc.first_purchase = orderDate;
    }
    if (!acc.last_purchase || orderDate > acc.last_purchase) {
      acc.last_purchase = orderDate;
    }
    
    return acc;
  }, {
    total_spent: 0,
    order_count: 0,
    products_purchased: 0,
    first_purchase: null,
    last_purchase: null
  });
  
  const daysSinceFirst = Math.floor(
    (new Date() - stats.first_purchase) / (1000 * 60 * 60 * 24)
  );
  
  return {
    lifetime_value: stats.total_spent,
    avg_order_value: stats.total_spent / stats.order_count,
    purchase_frequency: stats.order_count / (daysSinceFirst / 30),
    customer_age_days: daysSinceFirst,
    predicted_12mo_value: (stats.total_spent / daysSinceFirst) * 365
  };
}
```

## Common Patterns

### Pattern 1: Product Catalog Sync with Caching

```javascript
const CACHE = new Map();
const CACHE_TTL = parseInt(process.env.CACHE_TTL || '300') * 1000;

async function getCachedProducts(params) {
  const cacheKey = JSON.stringify(params);
  const cached = CACHE.get(cacheKey);
  
  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    return cached.data;
  }
  
  const products = await eddRequest('GET', '/products', params);
  
  CACHE.set(cacheKey, {
    data: products,
    timestamp: Date.now()
  });
  
  return products;
}
```

### Pattern 2: Webhook Event Processing

```javascript
function processWebhook(event) {
  const enriched = {
    ...event,
    processed_at: new Date().toISOString(),
    source: 'edd-webhook'
  };
  
  switch (event.event_type) {
    case 'edd_complete_purchase':
      return handleOrderComplete(enriched);
    case 'edd_customer_created':
      return handleCustomerCreated(enriched);
    case 'edd_subscription_status_changed':
      return handleSubscriptionChange(enriched);
    default:
      return { status: 'ignored', event_type: event.event_type };
  }
}
```

### Pattern 3: Rate-Limited Batch Operations

```javascript
async function batchUpdateProducts(updates) {
  const RATE_LIMIT = parseInt(process.env.RATE_LIMIT_MAX || '60');
  const DELAY = (60 / RATE_LIMIT) * 1000;
  
  const results = [];
  
  for (const update of updates) {
    try {
      const result = await eddRequest('PUT', `/products/${update.id}`, update);
      results.push({ success: true, id: update.id, result });
    } catch (error) {
      results.push({ success: false, id: update.id, error: error.message });
    }
    
    await new Promise(resolve => setTimeout(resolve, DELAY));
  }
  
  return results;
}
```

## Troubleshooting

### Authentication Failures

**Symptom**: `401 Unauthorized` errors

**Solution**:
- Verify `EDD_API_KEY` and `EDD_API_TOKEN` are correct
- Check API key permissions in WordPress admin
- Ensure key hasn't been revoked or expired
- Test credentials with curl:

```bash
curl -u "PUBLIC_KEY:TOKEN" https://yoursite.com/wp-json/edd/v2/products
```

### Rate Limiting Issues

**Symptom**: `429 Too Many Requests` errors

**Solution**:
- Reduce `RATE_LIMIT_MAX` in configuration
- Implement request queuing with delays
- Enable caching to reduce API calls
- Use batch endpoints where available

### Missing Data in Responses

**Symptom**: Expected fields are null or missing

**Solution**:
- Verify EDD version supports the endpoint
- Check if required EDD extensions are installed (e.g., Software Licensing)
- Update API version in `EDD_API_URL` (try `/v2` or `/v3`)
- Review EDD plugin settings for disabled features

### Connection Timeouts

**Symptom**: Requests hang or timeout

**Solution**:
- Increase timeout in fetch/requests configuration
- Check WordPress site performance and hosting
- Verify firewall rules allow MCP server IP
- Enable WordPress caching (Redis/Memcached)

### Cache Staleness

**Symptom**: Outdated data returned after updates

**Solution**:
- Reduce `CACHE_TTL` value
- Implement cache invalidation on write operations
- Use Redis with TTL instead of in-memory cache
- Add cache-busting query parameters

### HMAC Verification Errors

**Symptom**: Signature mismatch warnings

**Solution**:
- Ensure `HMAC_SECRET` matches across client and server
- Verify payload hasn't been modified in transit
- Check for character encoding issues
- Use webhook signature verification from EDD settings

## Advanced Usage

### Custom Middleware Chain

```javascript
const middleware = [
  validateRequest,
  enrichWithGeoData,
  applyTaxCalculation,
  logAuditTrail
];

async function processRequest(tool, params) {
  let context = { tool, params, timestamp: Date.now() };
  
  for (const fn of middleware) {
    context = await fn(context);
  }
  
  return executeEDDTool(context.tool, context.params);
}
```

### Multi-Store Support

```javascript
const stores = {
  'store1': {
    url: process.env.EDD_STORE1_URL,
    key: process.env.EDD_STORE1_KEY,
    token: process.env.EDD_STORE1_TOKEN
  },
  'store2': {
    url: process.env.EDD_STORE2_URL,
    key: process.env.EDD_STORE2_KEY,
    token: process.env.EDD_STORE2_TOKEN
  }
};

function getStoreConfig(store_id) {
  return stores[store_id] || stores['store1'];
}
```

This skill enables AI coding agents to interact with Easy Digital Downloads through natural language, performing complex e-commerce operations, analytics, and automation tasks seamlessly.
