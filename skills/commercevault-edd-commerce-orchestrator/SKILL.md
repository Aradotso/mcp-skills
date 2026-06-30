---
name: commercevault-edd-commerce-orchestrator
description: Unified digital commerce middleware for Easy Digital Downloads and e-commerce platforms with analytics, order management, and product catalog synchronization
triggers:
  - connect to Easy Digital Downloads API
  - integrate EDD with my application
  - fetch sales data from WordPress EDD
  - manage digital product orders and licenses
  - sync product catalog from EDD store
  - query EDD analytics and revenue reports
  - set up CommerceVault middleware
  - orchestrate digital commerce transactions
---

# CommerceVault - EDD Commerce Orchestrator

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

CommerceVault is a precision-engineered middleware layer that provides a unified interface to Easy Digital Downloads (EDD) and other e-commerce platforms. It handles product catalog synchronization, order lifecycle management, customer segmentation, analytics, and license/entitlement management with cryptographic integrity verification and adaptive rate limiting.

## Installation

### Prerequisites

- Node.js 18+ or Python 3.9+
- Easy Digital Downloads WordPress installation with REST API enabled
- API credentials (Consumer Key and Consumer Secret from EDD)

### Setup

1. Clone the repository:
```bash
git clone https://github.com/dhapat3927/mcp-edd-analytics-vantage.git
cd mcp-edd-analytics-vantage
```

2. Install dependencies:
```bash
# For Node.js
npm install

# For Python
pip install -r requirements.txt
```

3. Configure environment variables:
```bash
cp .env.example .env
```

Edit `.env`:
```bash
EDD_SITE_URL=https://yourstore.com
EDD_CONSUMER_KEY=ck_xxxxxxxxxxxxxxxx
EDD_CONSUMER_SECRET=cs_xxxxxxxxxxxxxxxx
COMMERCEVAULT_HMAC_SECRET=your-secure-hmac-secret
REDIS_URL=redis://localhost:6379
LOG_LEVEL=info
```

## Core API Operations

### Product Catalog Management

**Fetch All Products:**
```javascript
const CommerceVault = require('commercevault');

const vault = new CommerceVault({
  siteUrl: process.env.EDD_SITE_URL,
  consumerKey: process.env.EDD_CONSUMER_KEY,
  consumerSecret: process.env.EDD_CONSUMER_SECRET
});

// Retrieve product catalog
const products = await vault.products.list({
  perPage: 50,
  page: 1,
  status: 'publish'
});

console.log(`Found ${products.length} products`);
products.forEach(product => {
  console.log(`${product.id}: ${product.name} - $${product.price}`);
});
```

**Get Single Product:**
```javascript
const product = await vault.products.get(123);
console.log(product.name, product.price, product.downloadUrl);
```

**Delta Sync (Changed Products Only):**
```javascript
const lastSync = new Date('2026-06-29T00:00:00Z');
const changedProducts = await vault.products.getDelta({
  modifiedSince: lastSync
});
```

### Order Management

**Fetch Orders:**
```javascript
const orders = await vault.orders.list({
  status: 'complete',
  dateStart: '2026-06-01',
  dateEnd: '2026-06-30',
  perPage: 100
});

orders.forEach(order => {
  console.log(`Order #${order.id}: ${order.total} - ${order.status}`);
  console.log(`Customer: ${order.customer.email}`);
  console.log(`Items: ${order.items.length}`);
});
```

**Create Order Programmatically:**
```javascript
const newOrder = await vault.orders.create({
  customer: {
    email: 'customer@example.com',
    firstName: 'John',
    lastName: 'Doe'
  },
  items: [
    { productId: 123, quantity: 1, price: 29.99 }
  ],
  payment: {
    gateway: 'stripe',
    transactionId: 'ch_xxxxxxxxx'
  }
});

console.log(`Order created: ${newOrder.id}`);
```

**Update Order Status:**
```javascript
await vault.orders.update(456, {
  status: 'refunded',
  note: 'Customer requested refund'
});
```

**Get Order with Audit Trail:**
```javascript
const orderWithHistory = await vault.orders.getWithAudit(456);
console.log('Status history:', orderWithHistory.auditLog);
// Shows cryptographic fingerprints for each state transition
```

### Customer Management

**Fetch Customer Data:**
```javascript
const customer = await vault.customers.get(789);
console.log('Lifetime Value:', customer.lifetimeValue);
console.log('Total Purchases:', customer.purchaseCount);
console.log('Last Purchase:', customer.lastPurchaseDate);
```

**Customer Segmentation:**
```javascript
const highValueCustomers = await vault.customers.segment({
  minLifetimeValue: 500,
  minPurchaseCount: 5,
  lastPurchaseWithin: 90 // days
});

console.log(`${highValueCustomers.length} high-value customers found`);
```

**Export Customer Data (GDPR Compliance):**
```javascript
const customerData = await vault.customers.exportData(789);
// Returns complete data package for GDPR export request
```

### Analytics & Reporting

**Revenue Analytics:**
```javascript
const revenue = await vault.analytics.revenue({
  dateStart: '2026-06-01',
  dateEnd: '2026-06-30',
  granularity: 'daily' // hourly, daily, monthly
});

console.log('Total Revenue:', revenue.total);
console.log('Average Order Value:', revenue.averageOrderValue);
console.log('Daily Breakdown:', revenue.timeSeries);
```

**Product Performance:**
```javascript
const productStats = await vault.analytics.productPerformance({
  dateStart: '2026-06-01',
  dateEnd: '2026-06-30',
  sortBy: 'revenue',
  limit: 10
});

productStats.forEach(stat => {
  console.log(`${stat.productName}: ${stat.unitsSold} units, $${stat.revenue}`);
});
```

**Cohort Analysis:**
```javascript
const cohorts = await vault.analytics.cohorts({
  cohortBy: 'month', // month of first purchase
  metricPeriods: 6 // track for 6 periods
});

console.log('Retention by cohort:', cohorts);
```

### License & Entitlement Management

**Generate License Key:**
```javascript
const license = await vault.licenses.generate({
  productId: 123,
  customerId: 789,
  expiresAt: '2027-06-30',
  activationLimit: 3
});

console.log('License Key:', license.key);
console.log('Activation URL:', license.activationUrl);
```

**Validate License:**
```javascript
const validation = await vault.licenses.validate({
  key: 'XXXX-XXXX-XXXX-XXXX',
  productId: 123,
  siteUrl: 'https://customer-site.com'
});

if (validation.valid) {
  console.log('License is valid');
  console.log('Activations remaining:', validation.activationsRemaining);
} else {
  console.log('License invalid:', validation.reason);
}
```

**Deactivate License:**
```javascript
await vault.licenses.deactivate({
  key: 'XXXX-XXXX-XXXX-XXXX',
  siteUrl: 'https://old-site.com'
});
```

## Python SDK Usage

```python
from commercevault import CommerceVault
import os

vault = CommerceVault(
    site_url=os.getenv('EDD_SITE_URL'),
    consumer_key=os.getenv('EDD_CONSUMER_KEY'),
    consumer_secret=os.getenv('EDD_CONSUMER_SECRET')
)

# Async operations
import asyncio

async def fetch_products():
    products = await vault.products.list_async(per_page=50)
    for product in products:
        print(f"{product.id}: {product.name}")

asyncio.run(fetch_products())

# Synchronous operations
products = vault.products.list(per_page=50)
```

## Configuration

### Adapter Configuration

Support multiple e-commerce backends by configuring adapters:

```javascript
const vault = new CommerceVault({
  adapters: {
    edd: {
      siteUrl: process.env.EDD_SITE_URL,
      consumerKey: process.env.EDD_CONSUMER_KEY,
      consumerSecret: process.env.EDD_CONSUMER_SECRET
    },
    woocommerce: {
      siteUrl: process.env.WOO_SITE_URL,
      consumerKey: process.env.WOO_CONSUMER_KEY,
      consumerSecret: process.env.WOO_CONSUMER_SECRET
    }
  },
  defaultAdapter: 'edd'
});

// Route request to specific adapter
const eddProducts = await vault.products.list({ adapter: 'edd' });
const wooProducts = await vault.products.list({ adapter: 'woocommerce' });
```

### Rate Limiting

```javascript
const vault = new CommerceVault({
  siteUrl: process.env.EDD_SITE_URL,
  consumerKey: process.env.EDD_CONSUMER_KEY,
  consumerSecret: process.env.EDD_CONSUMER_SECRET,
  rateLimit: {
    maxRequestsPerMinute: 60,
    adaptiveThrottling: true,
    queueOnLimit: true
  }
});
```

### Caching

```javascript
const vault = new CommerceVault({
  siteUrl: process.env.EDD_SITE_URL,
  consumerKey: process.env.EDD_CONSUMER_KEY,
  consumerSecret: process.env.EDD_CONSUMER_SECRET,
  cache: {
    enabled: true,
    type: 'redis',
    url: process.env.REDIS_URL,
    ttl: {
      products: 3600,      // 1 hour
      customers: 1800,     // 30 minutes
      orders: 300          // 5 minutes
    }
  }
});
```

### Webhook Relay

Configure webhook handling:

```javascript
const express = require('express');
const app = express();

vault.webhooks.on('order.completed', async (event) => {
  console.log('New order:', event.orderId);
  // Send notification, update CRM, etc.
});

vault.webhooks.on('customer.updated', async (event) => {
  console.log('Customer updated:', event.customerId);
});

// Mount webhook endpoint
app.use('/webhooks/edd', vault.webhooks.middleware());

app.listen(3000);
```

### Middleware Pipeline

Add custom processing:

```javascript
vault.use('request', async (context, next) => {
  console.log(`Request: ${context.method} ${context.endpoint}`);
  await next();
});

vault.use('response', async (context, next) => {
  // Add HMAC signature
  context.response.signature = vault.crypto.sign(context.response.data);
  await next();
});

vault.use('error', async (context, next) => {
  console.error('Error:', context.error);
  // Send to error tracking service
  await next();
});
```

## Common Patterns

### Daily Revenue Report

```javascript
async function generateDailyReport() {
  const yesterday = new Date();
  yesterday.setDate(yesterday.getDate() - 1);
  
  const dateString = yesterday.toISOString().split('T')[0];
  
  const revenue = await vault.analytics.revenue({
    dateStart: dateString,
    dateEnd: dateString
  });
  
  const topProducts = await vault.analytics.productPerformance({
    dateStart: dateString,
    dateEnd: dateString,
    limit: 5
  });
  
  console.log(`\n=== Daily Report: ${dateString} ===`);
  console.log(`Total Revenue: $${revenue.total}`);
  console.log(`Orders: ${revenue.orderCount}`);
  console.log(`AOV: $${revenue.averageOrderValue}`);
  console.log('\nTop Products:');
  topProducts.forEach((p, i) => {
    console.log(`${i+1}. ${p.productName}: ${p.unitsSold} units, $${p.revenue}`);
  });
}
```

### Abandoned Cart Recovery

```javascript
async function findAbandonedCarts() {
  const twoDaysAgo = new Date();
  twoDaysAgo.setDate(twoDaysAgo.getDate() - 2);
  
  const pendingOrders = await vault.orders.list({
    status: 'pending',
    dateEnd: twoDaysAgo.toISOString()
  });
  
  for (const order of pendingOrders) {
    const customer = await vault.customers.get(order.customerId);
    
    // Send recovery email via your email service
    await sendRecoveryEmail({
      to: customer.email,
      orderId: order.id,
      cartUrl: order.cartRecoveryUrl
    });
  }
}
```

### Inventory Sync

```javascript
async function syncInventory() {
  const products = await vault.products.list({ perPage: 100 });
  
  for (const product of products) {
    // Check your internal inventory system
    const internalStock = await getInternalStock(product.sku);
    
    if (product.stock !== internalStock) {
      await vault.products.update(product.id, {
        stock: internalStock
      });
      console.log(`Updated ${product.name}: ${internalStock} units`);
    }
  }
}
```

### Customer Lifetime Value Calculation

```javascript
async function calculateLTV(customerId) {
  const customer = await vault.customers.get(customerId);
  const orders = await vault.orders.list({
    customerId: customerId,
    status: 'complete'
  });
  
  const totalRevenue = orders.reduce((sum, order) => sum + order.total, 0);
  const avgOrderValue = totalRevenue / orders.length;
  const daysSinceFirst = (new Date() - new Date(orders[0].date)) / (1000 * 60 * 60 * 24);
  const purchaseFrequency = orders.length / (daysSinceFirst / 30); // per month
  
  const predictedLTV = avgOrderValue * purchaseFrequency * 36; // 3 year projection
  
  return {
    currentLTV: totalRevenue,
    predictedLTV: predictedLTV,
    avgOrderValue: avgOrderValue,
    purchaseFrequency: purchaseFrequency
  };
}
```

## Troubleshooting

### Authentication Errors

```javascript
// Test connection
try {
  const test = await vault.test();
  console.log('Connection successful:', test);
} catch (error) {
  if (error.code === 'INVALID_CREDENTIALS') {
    console.error('Check your consumer key and secret');
  } else if (error.code === 'UNAUTHORIZED') {
    console.error('API user lacks required permissions');
  }
}
```

### Rate Limit Handling

```javascript
vault.on('rateLimit', (info) => {
  console.warn(`Rate limit reached. Retrying in ${info.retryAfter}s`);
});

vault.on('rateLimitRecovered', () => {
  console.log('Rate limit recovered, resuming requests');
});
```

### Cache Invalidation

```javascript
// Manual cache clear
await vault.cache.clear('products');
await vault.cache.clear('orders');

// Clear specific item
await vault.cache.delete('product:123');

// Event-driven invalidation
vault.webhooks.on('product.updated', async (event) => {
  await vault.cache.delete(`product:${event.productId}`);
});
```

### Error Logging

```javascript
vault.on('error', (error) => {
  console.error('CommerceVault Error:', {
    code: error.code,
    message: error.message,
    endpoint: error.endpoint,
    timestamp: new Date().toISOString()
  });
  
  // Send to error tracking
  if (process.env.SENTRY_DSN) {
    Sentry.captureException(error);
  }
});
```

### HMAC Signature Verification

```javascript
// Verify response integrity
const response = await vault.orders.get(123);
const isValid = vault.crypto.verify(
  response.data,
  response.signature,
  process.env.COMMERCEVAULT_HMAC_SECRET
);

if (!isValid) {
  throw new Error('Response signature verification failed');
}
```

### Debug Mode

```bash
# Enable verbose logging
DEBUG=commercevault:* node app.js

# Or in code
const vault = new CommerceVault({
  siteUrl: process.env.EDD_SITE_URL,
  consumerKey: process.env.EDD_CONSUMER_KEY,
  consumerSecret: process.env.EDD_CONSUMER_SECRET,
  debug: true
});
```

## Testing

```javascript
// Mock adapter for testing
const mockVault = new CommerceVault({
  adapter: 'mock',
  mockData: {
    products: [
      { id: 1, name: 'Test Product', price: 29.99 }
    ]
  }
});

// Integration test
const products = await mockVault.products.list();
assert.equal(products.length, 1);
assert.equal(products[0].name, 'Test Product');
```

## API Reference

Full API documentation available at: `http://localhost:3000/api-docs` when running locally with Swagger UI enabled.

Key endpoints:
- `/products` - Product catalog operations
- `/orders` - Order management
- `/customers` - Customer data and segmentation
- `/analytics` - Revenue and performance metrics
- `/licenses` - License key management
- `/webhooks` - Event subscriptions

All endpoints support JSON responses and follow the Uniform Commerce Language (UCL) standard for consistent data structures across adapters.
