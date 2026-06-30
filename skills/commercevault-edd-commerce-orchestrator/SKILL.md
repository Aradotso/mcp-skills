---
name: commercevault-edd-commerce-orchestrator
description: Easy Digital Downloads MCP server for sales analytics, product management, and order processing via WordPress REST API
triggers:
  - "connect to Easy Digital Downloads API"
  - "fetch EDD sales data and analytics"
  - "manage digital products in WordPress EDD"
  - "query EDD customer orders and revenue"
  - "set up CommerceVault MCP server"
  - "integrate with Easy Digital Downloads store"
  - "analyze EDD e-commerce metrics"
  - "retrieve digital download statistics"
---

# CommerceVault EDD Commerce Orchestrator

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

CommerceVault (formerly MCP-EDD-Analytics-Vantage) is an MCP server that provides programmatic access to Easy Digital Downloads (EDD) WordPress stores. It enables AI assistants and automation tools to fetch sales analytics, manage products, query orders, retrieve customer data, and handle digital license management through a standardized interface.

The server acts as a middleware layer between your application and the EDD REST API, providing structured access to e-commerce operations with built-in authentication, caching, and data transformation.

## Installation

### Prerequisites

- Node.js 18+ or Python 3.9+
- WordPress site with Easy Digital Downloads plugin installed
- EDD REST API credentials (Consumer Key and Consumer Secret)

### Clone and Install

```bash
git clone https://github.com/dhapat3927/mcp-edd-analytics-vantage.git
cd mcp-edd-analytics-vantage
npm install
```

### MCP Server Configuration

Add to your MCP settings file (e.g., `claude_desktop_config.json` or `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "commercevault": {
      "command": "node",
      "args": ["/path/to/mcp-edd-analytics-vantage/server.js"],
      "env": {
        "EDD_SITE_URL": "https://your-store.com",
        "EDD_CONSUMER_KEY": "ck_xxxxxxxxxxxxx",
        "EDD_CONSUMER_SECRET": "cs_xxxxxxxxxxxxx",
        "EDD_API_VERSION": "v2"
      }
    }
  }
}
```

### Environment Variables

Create a `.env` file in the project root:

```bash
EDD_SITE_URL=https://your-store.com
EDD_CONSUMER_KEY=ck_your_consumer_key_here
EDD_CONSUMER_SECRET=cs_your_consumer_secret_here
EDD_API_VERSION=v2
CACHE_TTL=300
LOG_LEVEL=info
```

## Core Capabilities

### Product Management

Retrieve and manage digital product catalogs:

```javascript
// Fetch all products with pagination
const products = await server.call('edd.getProducts', {
  page: 1,
  per_page: 20,
  status: 'publish'
});

// Get single product details
const product = await server.call('edd.getProduct', {
  product_id: 123
});

// Search products by keyword
const searchResults = await server.call('edd.searchProducts', {
  query: 'premium plugin',
  filter: { category: 'wordpress' }
});

// Update product pricing
await server.call('edd.updateProduct', {
  product_id: 123,
  price: 29.99,
  sale_price: 19.99
});
```

### Order & Sales Analytics

Query order data and revenue metrics:

```javascript
// Get orders within date range
const orders = await server.call('edd.getOrders', {
  start_date: '2026-01-01',
  end_date: '2026-06-30',
  status: 'completed'
});

// Fetch sales statistics
const stats = await server.call('edd.getSalesStats', {
  period: 'month',
  year: 2026,
  month: 6
});

// Get revenue by product
const revenue = await server.call('edd.getRevenueByProduct', {
  product_id: 123,
  start_date: '2026-01-01'
});

// Calculate average order value
const aov = await server.call('edd.getAverageOrderValue', {
  start_date: '2026-06-01',
  end_date: '2026-06-30'
});
```

### Customer Management

Access customer profiles and purchase history:

```javascript
// Get customer details
const customer = await server.call('edd.getCustomer', {
  customer_id: 456
});

// List all customers with filters
const customers = await server.call('edd.getCustomers', {
  per_page: 50,
  orderby: 'purchase_count',
  order: 'desc'
});

// Get customer purchase history
const purchases = await server.call('edd.getCustomerPurchases', {
  customer_id: 456,
  status: 'completed'
});

// Calculate customer lifetime value
const ltv = await server.call('edd.getCustomerLTV', {
  customer_id: 456
});
```

### License & Entitlement Management

Handle digital product licenses and activations:

```javascript
// Generate license key
const license = await server.call('edd.createLicense', {
  product_id: 123,
  customer_email: 'customer@example.com',
  activations_limit: 3
});

// Verify license status
const validation = await server.call('edd.validateLicense', {
  license_key: 'xxxx-xxxx-xxxx-xxxx',
  product_id: 123
});

// Get license activations
const activations = await server.call('edd.getLicenseActivations', {
  license_key: 'xxxx-xxxx-xxxx-xxxx'
});

// Deactivate license
await server.call('edd.deactivateLicense', {
  license_key: 'xxxx-xxxx-xxxx-xxxx',
  site_url: 'https://customer-site.com'
});
```

## Configuration

### Authentication

CommerceVault supports multiple authentication methods:

```javascript
// API Key authentication (recommended)
{
  "auth_type": "api_key",
  "consumer_key": process.env.EDD_CONSUMER_KEY,
  "consumer_secret": process.env.EDD_CONSUMER_SECRET
}

// OAuth 2.0 (for advanced use cases)
{
  "auth_type": "oauth",
  "client_id": process.env.EDD_CLIENT_ID,
  "client_secret": process.env.EDD_CLIENT_SECRET,
  "redirect_uri": "http://localhost:3000/callback"
}
```

### Rate Limiting

Configure adaptive rate limiting to respect API quotas:

```javascript
{
  "rate_limit": {
    "enabled": true,
    "max_requests_per_minute": 60,
    "queue_enabled": true,
    "retry_on_429": true
  }
}
```

### Caching Strategy

Enable Redis-backed caching for frequently accessed data:

```javascript
{
  "cache": {
    "enabled": true,
    "backend": "redis",
    "redis_url": process.env.REDIS_URL,
    "ttl": {
      "products": 300,
      "customers": 180,
      "orders": 60
    }
  }
}
```

## Common Patterns

### Sales Dashboard Data

Aggregate metrics for a dashboard view:

```javascript
async function getDashboardMetrics(startDate, endDate) {
  const [orders, revenue, customers, topProducts] = await Promise.all([
    server.call('edd.getOrders', { 
      start_date: startDate, 
      end_date: endDate,
      status: 'completed'
    }),
    server.call('edd.getSalesStats', { 
      start_date: startDate, 
      end_date: endDate 
    }),
    server.call('edd.getCustomers', { 
      created_after: startDate 
    }),
    server.call('edd.getTopSellingProducts', { 
      start_date: startDate,
      limit: 10 
    })
  ]);

  return {
    total_orders: orders.length,
    total_revenue: revenue.total,
    new_customers: customers.length,
    average_order_value: revenue.total / orders.length,
    top_products: topProducts
  };
}
```

### Customer Segmentation

Segment customers by purchase behavior:

```javascript
async function segmentCustomers() {
  const customers = await server.call('edd.getCustomers', { 
    per_page: 100 
  });

  const segments = {
    high_value: [],
    repeat_buyers: [],
    at_risk: [],
    new: []
  };

  for (const customer of customers) {
    const ltv = await server.call('edd.getCustomerLTV', {
      customer_id: customer.id
    });

    if (ltv > 500) {
      segments.high_value.push(customer);
    } else if (customer.purchase_count > 3) {
      segments.repeat_buyers.push(customer);
    } else if (customer.purchase_count === 1) {
      segments.new.push(customer);
    }
  }

  return segments;
}
```

### Automated Reporting

Generate and export revenue reports:

```javascript
async function generateMonthlyReport(year, month) {
  const stats = await server.call('edd.getSalesStats', {
    period: 'month',
    year: year,
    month: month
  });

  const products = await server.call('edd.getRevenueByProduct', {
    start_date: `${year}-${month}-01`,
    end_date: `${year}-${month}-31`
  });

  const report = {
    period: `${year}-${month}`,
    total_revenue: stats.total,
    total_orders: stats.order_count,
    average_order_value: stats.total / stats.order_count,
    product_breakdown: products.map(p => ({
      name: p.name,
      units_sold: p.quantity,
      revenue: p.revenue
    }))
  };

  // Export to CSV or send via webhook
  return report;
}
```

### Webhook Event Handling

Process EDD webhook events:

```javascript
server.on('webhook.order.completed', async (event) => {
  const order = event.data;
  
  // Send fulfillment email
  await server.call('edd.sendOrderEmail', {
    order_id: order.id,
    template: 'order_confirmation'
  });

  // Update CRM
  await updateCRM({
    customer_email: order.customer_email,
    purchase_amount: order.total,
    products: order.items
  });
});

server.on('webhook.license.activated', async (event) => {
  const license = event.data;
  
  // Log activation for compliance
  await server.call('edd.logLicenseEvent', {
    license_key: license.key,
    event: 'activated',
    site_url: license.site_url
  });
});
```

## Troubleshooting

### Authentication Errors

If you receive `401 Unauthorized`:

```bash
# Verify credentials are correct
curl -u "ck_xxxxx:cs_xxxxx" https://your-store.com/wp-json/edd/v2/products

# Check if REST API is enabled in WordPress
# Navigate to: Settings > Permalinks > Save Changes

# Ensure EDD REST API is enabled
# Go to: Downloads > Settings > Extensions
```

### Rate Limiting Issues

Handle `429 Too Many Requests`:

```javascript
// Enable automatic retry with exponential backoff
{
  "rate_limit": {
    "retry_on_429": true,
    "max_retries": 3,
    "backoff_multiplier": 2
  }
}
```

### Cache Invalidation

Force cache refresh for updated data:

```javascript
// Clear cache for specific resource
await server.call('cache.invalidate', {
  resource: 'products',
  id: 123
});

// Clear all cache
await server.call('cache.flush');
```

### Connection Timeouts

Increase timeout for slow endpoints:

```javascript
{
  "http": {
    "timeout": 30000,  // 30 seconds
    "keepalive": true
  }
}
```

### Debug Logging

Enable verbose logging for troubleshooting:

```bash
LOG_LEVEL=debug node server.js
```

View structured logs:

```javascript
// Logs include request/response details
{
  "timestamp": "2026-06-30T16:45:10Z",
  "level": "debug",
  "method": "GET",
  "endpoint": "/edd/v2/products/123",
  "duration_ms": 245,
  "status": 200
}
```

## API Reference

### Products

- `edd.getProducts({ page, per_page, status })` - List products
- `edd.getProduct({ product_id })` - Get single product
- `edd.searchProducts({ query, filter })` - Search products
- `edd.updateProduct({ product_id, ...fields })` - Update product

### Orders

- `edd.getOrders({ start_date, end_date, status })` - List orders
- `edd.getOrder({ order_id })` - Get single order
- `edd.getSalesStats({ period, year, month })` - Sales statistics

### Customers

- `edd.getCustomers({ per_page, orderby, order })` - List customers
- `edd.getCustomer({ customer_id })` - Get customer details
- `edd.getCustomerLTV({ customer_id })` - Calculate lifetime value

### Licenses

- `edd.createLicense({ product_id, customer_email, activations_limit })` - Generate license
- `edd.validateLicense({ license_key, product_id })` - Verify license
- `edd.getLicenseActivations({ license_key })` - List activations
- `edd.deactivateLicense({ license_key, site_url })` - Deactivate license

### Analytics

- `edd.getRevenueByProduct({ product_id, start_date })` - Product revenue
- `edd.getAverageOrderValue({ start_date, end_date })` - AOV calculation
- `edd.getTopSellingProducts({ start_date, limit })` - Best sellers
