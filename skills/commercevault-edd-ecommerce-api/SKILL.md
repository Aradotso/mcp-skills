---
name: commercevault-edd-ecommerce-api
description: Easy Digital Downloads MCP server for sales analytics, product management, and customer data orchestration
triggers:
  - "connect to Easy Digital Downloads API"
  - "query EDD sales data and analytics"
  - "manage digital products and downloads"
  - "fetch customer purchase history from EDD"
  - "sync EDD store inventory and orders"
  - "analyze revenue streams and cohorts"
  - "retrieve EDD license and entitlement data"
  - "set up MCP server for WordPress ecommerce"
---

# CommerceVault EDD MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

CommerceVault (mcp-edd-analytics-vantage) is an MCP (Model Context Protocol) server that provides structured access to Easy Digital Downloads (EDD) APIs for sales analytics, product management, customer data, and license/entitlement tracking. It acts as a middleware orchestrator between AI agents and WordPress EDD installations, exposing REST endpoints through a standardized interface.

## What It Does

- **Product Catalog Access**: Query, filter, and synchronize digital product listings with pricing, categories, and metadata
- **Order Management**: Retrieve order history, payment status, fulfillment data, and refund information
- **Customer Analytics**: Access customer profiles, lifetime value, purchase frequency, and segmentation data
- **Sales Reporting**: Real-time analytics for revenue, GMV, AOV, churn rate, and cohort analysis
- **License Management**: Track license keys, activation codes, and entitlement tiers for digital products
- **Webhook Integration**: Bidirectional event relay for order completions, refunds, and customer updates

## Installation

### Prerequisites

- Node.js 18+ or Python 3.9+
- Access to an Easy Digital Downloads WordPress installation
- EDD REST API credentials (API key and token)

### MCP Server Setup

1. **Clone the repository**:
```bash
git clone https://github.com/dhapat3927/mcp-edd-analytics-vantage.git
cd mcp-edd-analytics-vantage
```

2. **Install dependencies** (Node.js example):
```bash
npm install
```

3. **Configure environment variables**:
```bash
# .env
EDD_API_URL=https://your-store.com/edd-api/v2
EDD_API_KEY=your_edd_api_key
EDD_API_TOKEN=your_edd_api_token
CACHE_ENABLED=true
REDIS_URL=redis://localhost:6379
WEBHOOK_SECRET=your_webhook_signing_secret
```

4. **Start the MCP server**:
```bash
npm start
# or
python main.py
```

### MCP Client Configuration

Add to your MCP client settings (e.g., Claude Desktop `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "commercevault-edd": {
      "command": "node",
      "args": ["/path/to/mcp-edd-analytics-vantage/dist/index.js"],
      "env": {
        "EDD_API_URL": "https://your-store.com/edd-api/v2",
        "EDD_API_KEY": "${EDD_API_KEY}",
        "EDD_API_TOKEN": "${EDD_API_TOKEN}"
      }
    }
  }
}
```

## Core API Methods

### Products

**List all products**:
```javascript
const products = await mcp.callTool("commercevault-edd", "list_products", {
  per_page: 50,
  page: 1,
  status: "publish",
  category: "software"
});
```

**Get product details**:
```javascript
const product = await mcp.callTool("commercevault-edd", "get_product", {
  product_id: 123
});
```

**Search products**:
```javascript
const results = await mcp.callTool("commercevault-edd", "search_products", {
  query: "wordpress plugin",
  price_min: 19.99,
  price_max: 99.99
});
```

### Orders

**Fetch recent orders**:
```javascript
const orders = await mcp.callTool("commercevault-edd", "list_orders", {
  status: "completed",
  date_start: "2026-01-01",
  date_end: "2026-06-30",
  per_page: 100
});
```

**Get order details**:
```javascript
const order = await mcp.callTool("commercevault-edd", "get_order", {
  order_id: 45678
});
```

**Process refund**:
```javascript
const refund = await mcp.callTool("commercevault-edd", "create_refund", {
  order_id: 45678,
  amount: 29.99,
  reason: "Customer request"
});
```

### Customers

**Get customer data**:
```javascript
const customer = await mcp.callTool("commercevault-edd", "get_customer", {
  customer_id: 789,
  include_purchases: true
});
```

**List customers by segment**:
```javascript
const highValue = await mcp.callTool("commercevault-edd", "list_customers", {
  ltv_min: 500,
  order_count_min: 5,
  sort: "ltv_desc"
});
```

### Analytics

**Revenue report**:
```javascript
const revenue = await mcp.callTool("commercevault-edd", "get_revenue_report", {
  period: "monthly",
  start_date: "2026-01-01",
  end_date: "2026-06-30",
  group_by: "product"
});
```

**Sales metrics**:
```javascript
const metrics = await mcp.callTool("commercevault-edd", "get_sales_metrics", {
  period: "last_30_days",
  metrics: ["gmv", "aov", "conversion_rate", "refund_rate"]
});
```

**Cohort analysis**:
```javascript
const cohorts = await mcp.callTool("commercevault-edd", "analyze_cohorts", {
  cohort_by: "signup_month",
  metric: "retention_rate",
  months: 12
});
```

### Licenses

**Get license info**:
```javascript
const license = await mcp.callTool("commercevault-edd", "get_license", {
  license_key: "abc123def456"
});
```

**Activate license**:
```javascript
const activation = await mcp.callTool("commercevault-edd", "activate_license", {
  license_key: "abc123def456",
  site_url: "https://customer-site.com",
  product_id: 123
});
```

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `EDD_API_URL` | Yes | Base URL for EDD REST API |
| `EDD_API_KEY` | Yes | EDD API public key |
| `EDD_API_TOKEN` | Yes | EDD API secret token |
| `CACHE_ENABLED` | No | Enable Redis caching (default: false) |
| `REDIS_URL` | No | Redis connection string |
| `WEBHOOK_SECRET` | No | HMAC secret for webhook verification |
| `RATE_LIMIT_MAX` | No | Max requests per window (default: 100) |
| `RATE_LIMIT_WINDOW` | No | Rate limit window in seconds (default: 60) |

### Adapter Configuration

Create `config/adapter.json` for custom EDD endpoint mappings:

```json
{
  "adapter": "edd",
  "version": "2.0",
  "endpoints": {
    "products": "/products",
    "orders": "/sales",
    "customers": "/customers",
    "licenses": "/licenses"
  },
  "auth": {
    "type": "bearer",
    "header": "X-EDD-Token"
  },
  "cache": {
    "ttl_products": 3600,
    "ttl_orders": 300,
    "ttl_customers": 600
  }
}
```

## Real-World Examples

### Example 1: Daily Sales Dashboard

```javascript
async function generateDailySalesDashboard(date) {
  // Fetch today's orders
  const orders = await mcp.callTool("commercevault-edd", "list_orders", {
    date_start: date,
    date_end: date,
    status: "completed"
  });

  // Calculate metrics
  const totalRevenue = orders.reduce((sum, order) => sum + order.total, 0);
  const avgOrderValue = totalRevenue / orders.length;

  // Get top products
  const productSales = {};
  orders.forEach(order => {
    order.items.forEach(item => {
      productSales[item.product_id] = (productSales[item.product_id] || 0) + item.quantity;
    });
  });

  // Get refunds
  const refunds = await mcp.callTool("commercevault-edd", "list_orders", {
    date_start: date,
    date_end: date,
    status: "refunded"
  });

  return {
    date,
    totalOrders: orders.length,
    totalRevenue,
    avgOrderValue,
    refundCount: refunds.length,
    topProducts: Object.entries(productSales)
      .sort((a, b) => b[1] - a[1])
      .slice(0, 5)
  };
}
```

### Example 2: Customer Lifetime Value Analysis

```javascript
async function analyzeCustomerLTV(customerIds) {
  const results = [];

  for (const customerId of customerIds) {
    const customer = await mcp.callTool("commercevault-edd", "get_customer", {
      customer_id: customerId,
      include_purchases: true
    });

    const orders = await mcp.callTool("commercevault-edd", "list_orders", {
      customer_id: customerId,
      status: "completed"
    });

    const ltv = orders.reduce((sum, order) => sum + order.total, 0);
    const firstPurchase = orders[orders.length - 1]?.date;
    const daysSinceFirst = Math.floor(
      (new Date() - new Date(firstPurchase)) / (1000 * 60 * 60 * 24)
    );

    results.push({
      customerId,
      email: customer.email,
      ltv,
      orderCount: orders.length,
      avgOrderValue: ltv / orders.length,
      daysSinceFirst,
      purchaseFrequency: orders.length / (daysSinceFirst / 30) // orders per month
    });
  }

  return results.sort((a, b) => b.ltv - a.ltv);
}
```

### Example 3: License Management Automation

```javascript
async function renewExpiredLicenses() {
  const licenses = await mcp.callTool("commercevault-edd", "list_licenses", {
    status: "expired",
    expiry_date_end: new Date().toISOString().split('T')[0]
  });

  const renewalResults = [];

  for (const license of licenses) {
    try {
      // Get customer info
      const customer = await mcp.callTool("commercevault-edd", "get_customer", {
        customer_id: license.customer_id
      });

      // Create renewal order
      const renewal = await mcp.callTool("commercevault-edd", "create_order", {
        customer_id: license.customer_id,
        products: [{
          product_id: license.product_id,
          price: license.renewal_price
        }],
        license_renewal: license.license_key
      });

      renewalResults.push({
        license: license.license_key,
        customer: customer.email,
        status: "renewed",
        orderId: renewal.order_id
      });
    } catch (error) {
      renewalResults.push({
        license: license.license_key,
        status: "failed",
        error: error.message
      });
    }
  }

  return renewalResults;
}
```

## Common Patterns

### Caching Strategy

```javascript
// Enable cache for frequently accessed data
const getCachedProduct = async (productId) => {
  const cacheKey = `product:${productId}`;
  
  const cached = await mcp.callTool("commercevault-edd", "cache_get", {
    key: cacheKey
  });
  
  if (cached) return cached;
  
  const product = await mcp.callTool("commercevault-edd", "get_product", {
    product_id: productId
  });
  
  await mcp.callTool("commercevault-edd", "cache_set", {
    key: cacheKey,
    value: product,
    ttl: 3600
  });
  
  return product;
};
```

### Pagination Handling

```javascript
async function getAllOrders(filters) {
  let page = 1;
  let allOrders = [];
  let hasMore = true;

  while (hasMore) {
    const response = await mcp.callTool("commercevault-edd", "list_orders", {
      ...filters,
      page,
      per_page: 100
    });

    allOrders = allOrders.concat(response.orders);
    hasMore = response.has_more;
    page++;
  }

  return allOrders;
}
```

### Webhook Verification

```javascript
function verifyWebhookSignature(payload, signature) {
  const crypto = require('crypto');
  const hmac = crypto.createHmac('sha256', process.env.WEBHOOK_SECRET);
  const computed = hmac.update(JSON.stringify(payload)).digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(computed)
  );
}

// In webhook handler
app.post('/webhooks/edd', (req, res) => {
  const signature = req.headers['x-edd-signature'];
  
  if (!verifyWebhookSignature(req.body, signature)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Process webhook
  processEDDEvent(req.body);
  res.status(200).json({ received: true });
});
```

## Troubleshooting

### Authentication Errors

**Problem**: `401 Unauthorized` responses

**Solution**: Verify API credentials and endpoint URL
```javascript
// Test authentication
const test = await mcp.callTool("commercevault-edd", "test_connection", {});
console.log(test); // Should return { connected: true, api_version: "2.0" }
```

### Rate Limiting

**Problem**: `429 Too Many Requests` errors

**Solution**: Enable adaptive rate limiting in config
```javascript
// config/adapter.json
{
  "rate_limiting": {
    "enabled": true,
    "strategy": "adaptive",
    "max_retries": 3,
    "backoff_multiplier": 2
  }
}
```

### Cache Invalidation

**Problem**: Stale data returned from cached queries

**Solution**: Manually invalidate cache on updates
```javascript
// After updating a product
await mcp.callTool("commercevault-edd", "update_product", {
  product_id: 123,
  price: 49.99
});

// Invalidate cache
await mcp.callTool("commercevault-edd", "cache_delete", {
  key: `product:123`
});
```

### Webhook Delivery Failures

**Problem**: Webhooks not being received

**Solution**: Check webhook endpoint configuration and firewall rules
```bash
# Test webhook endpoint
curl -X POST https://your-app.com/webhooks/edd \
  -H "Content-Type: application/json" \
  -H "X-EDD-Signature: test" \
  -d '{"event": "test"}'
```

### Data Synchronization Issues

**Problem**: Orders missing or out of sync

**Solution**: Use delta sync with timestamps
```javascript
const lastSync = await getLastSyncTimestamp();

const newOrders = await mcp.callTool("commercevault-edd", "list_orders", {
  date_start: lastSync,
  date_end: new Date().toISOString()
});

await saveLastSyncTimestamp(new Date());
```

## Additional Resources

- [EDD REST API Documentation](https://easydigitaldownloads.com/docs/rest-api/)
- [MCP Protocol Specification](https://modelcontextprotocol.io)
- [GitHub Repository](https://github.com/dhapat3927/mcp-edd-analytics-vantage)
