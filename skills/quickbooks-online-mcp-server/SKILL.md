---
name: quickbooks-online-mcp-server
description: QuickBooks Online MCP server providing 144 tools for CRUD operations on 29 entities and 11 financial reports via OAuth 2.0
triggers:
  - integrate QuickBooks Online with Claude
  - set up QuickBooks MCP server
  - access QuickBooks data through MCP
  - create invoices in QuickBooks
  - query QuickBooks financial reports
  - manage QuickBooks customers and vendors
  - authenticate with QuickBooks Online API
  - search QuickBooks entities with filters
---

# QuickBooks Online MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The QuickBooks Online MCP Server is a comprehensive Model Context Protocol server that exposes the complete QuickBooks Online API through 144 standardized tools. It provides full CRUD operations for 29 entity types (Customer, Invoice, Bill, Vendor, Payment, etc.) and 11 financial reports (Balance Sheet, P&L, Cash Flow, etc.). Built with TypeScript and Zod validation, it enables AI assistants like Claude to interact with QuickBooks data through OAuth 2.0 authenticated API calls.

**Key capabilities:**
- 29 entity types with create, read, update, delete, and search operations
- 11 financial reports with customizable date ranges and parameters
- OAuth 2.0 token management with automatic refresh
- Type-safe operations with Zod schema validation
- Sandbox and production environment support

## Installation

### 1. Clone and Build

```bash
git clone https://github.com/intuit/quickbooks-online-mcp-server.git
cd quickbooks-online-mcp-server
npm install
npm run build
```

### 2. Configure Environment Variables

Create a `.env` file in the project root:

```env
QUICKBOOKS_CLIENT_ID=your_client_id_from_intuit_developer
QUICKBOOKS_CLIENT_SECRET=your_client_secret_from_intuit_developer
QUICKBOOKS_ENVIRONMENT=sandbox
QUICKBOOKS_REFRESH_TOKEN=your_refresh_token_from_oauth_flow
QUICKBOOKS_REALM_ID=your_company_id
```

**Environment values:**
- `QUICKBOOKS_ENVIRONMENT`: `sandbox` for testing, `production` for live data
- `QUICKBOOKS_REALM_ID`: The company ID (obtained during OAuth connection)
- `QUICKBOOKS_REFRESH_TOKEN`: Long-lived token from OAuth 2.0 flow

### 3. Add to Claude Desktop Configuration

Edit your Claude Desktop MCP settings file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`  
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "quickbooks": {
      "command": "node",
      "args": ["/absolute/path/to/quickbooks-online-mcp-server/dist/index.js"],
      "env": {
        "QUICKBOOKS_CLIENT_ID": "your_client_id",
        "QUICKBOOKS_CLIENT_SECRET": "your_client_secret",
        "QUICKBOOKS_REFRESH_TOKEN": "your_refresh_token",
        "QUICKBOOKS_REALM_ID": "your_realm_id",
        "QUICKBOOKS_ENVIRONMENT": "sandbox"
      }
    }
  }
}
```

Restart Claude Desktop to load the server.

## OAuth 2.0 Setup

### Obtaining Credentials

1. **Create an Intuit Developer Account**: [developer.intuit.com](https://developer.intuit.com/)
2. **Create an App**: Dashboard → Create an App → QuickBooks Online API
3. **Get Client ID and Secret**: Keys & OAuth section
4. **Set Redirect URI**: 
   - Sandbox: `http://localhost:8000/callback`
   - Production: Must be HTTPS public URL (localhost rejected)

### Getting a Refresh Token

The server requires a refresh token for ongoing access. You must complete the OAuth flow once to obtain it:

**Quick Method (using Intuit OAuth Playground):**
1. Visit [OAuth 2.0 Playground](https://developer.intuit.com/app/developer/playground)
2. Select your app and scopes: `com.intuit.quickbooks.accounting`
3. Authorize and retrieve the refresh token

**Manual Method (example Node.js script):**

```typescript
import express from 'express';
import axios from 'axios';

const app = express();
const CLIENT_ID = process.env.QUICKBOOKS_CLIENT_ID;
const CLIENT_SECRET = process.env.QUICKBOOKS_CLIENT_SECRET;
const REDIRECT_URI = 'http://localhost:8000/callback';

// Step 1: Authorization URL
app.get('/auth', (req, res) => {
  const authUrl = `https://appcenter.intuit.com/connect/oauth2?` +
    `client_id=${CLIENT_ID}&` +
    `response_type=code&` +
    `scope=com.intuit.quickbooks.accounting&` +
    `redirect_uri=${encodeURIComponent(REDIRECT_URI)}&` +
    `state=security_token`;
  res.redirect(authUrl);
});

// Step 2: Handle callback and exchange code for tokens
app.get('/callback', async (req, res) => {
  const { code, realmId } = req.query;
  
  try {
    const response = await axios.post(
      'https://oauth.platform.intuit.com/oauth2/v1/tokens/bearer',
      new URLSearchParams({
        grant_type: 'authorization_code',
        code: code as string,
        redirect_uri: REDIRECT_URI,
      }),
      {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': 'Basic ' + Buffer.from(`${CLIENT_ID}:${CLIENT_SECRET}`).toString('base64'),
        },
      }
    );
    
    console.log('Refresh Token:', response.data.refresh_token);
    console.log('Realm ID:', realmId);
    res.send(`Success! Save these:\nRefresh Token: ${response.data.refresh_token}\nRealm ID: ${realmId}`);
  } catch (error) {
    console.error('Token exchange failed:', error);
    res.status(500).send('Token exchange failed');
  }
});

app.listen(8000, () => console.log('Visit http://localhost:8000/auth to start OAuth flow'));
```

## Core Entity Operations

### Customer Management

```typescript
// Create a customer
const customer = await use_mcp_tool("quickbooks", "create_customer", {
  DisplayName: "Acme Corporation",
  GivenName: "John",
  FamilyName: "Smith",
  PrimaryEmailAddr: { Address: "john@acme.com" },
  PrimaryPhone: { FreeFormNumber: "555-1234" },
  BillAddr: {
    Line1: "123 Main St",
    City: "San Francisco",
    CountrySubDivisionCode: "CA",
    PostalCode: "94105",
    Country: "USA"
  }
});

// Search customers
const results = await use_mcp_tool("quickbooks", "search_customers", {
  query: "SELECT * FROM Customer WHERE DisplayName LIKE '%Acme%'"
});

// Get customer by ID
const customerDetail = await use_mcp_tool("quickbooks", "get_customer", {
  id: "123"
});

// Update customer
const updated = await use_mcp_tool("quickbooks", "update_customer", {
  Id: "123",
  SyncToken: "0",  // Required for updates (from get_customer response)
  DisplayName: "Acme Corporation Ltd",
  sparse: true  // Partial update
});
```

### Invoice Operations

```typescript
// Create an invoice
const invoice = await use_mcp_tool("quickbooks", "create_invoice", {
  CustomerRef: { value: "123" },
  Line: [
    {
      Amount: 1000.00,
      DetailType: "SalesItemLineDetail",
      SalesItemLineDetail: {
        ItemRef: { value: "1" },  // Item ID from QuickBooks
        Qty: 10,
        UnitPrice: 100
      }
    }
  ],
  DueDate: "2025-06-30",
  TxnDate: "2025-05-20"
});

// Search invoices (unpaid)
const unpaid = await use_mcp_tool("quickbooks", "search_invoices", {
  query: "SELECT * FROM Invoice WHERE Balance > '0' MAXRESULTS 100"
});

// Update invoice (add a line)
const updatedInvoice = await use_mcp_tool("quickbooks", "update_invoice", {
  Id: "456",
  SyncToken: "1",
  Line: [
    {
      Amount: 1000.00,
      DetailType: "SalesItemLineDetail",
      SalesItemLineDetail: {
        ItemRef: { value: "1" },
        Qty: 10,
        UnitPrice: 100
      }
    },
    {
      Amount: 500.00,
      DetailType: "SalesItemLineDetail",
      SalesItemLineDetail: {
        ItemRef: { value: "2" },
        Qty: 5,
        UnitPrice: 100
      }
    }
  ]
});

// Void/delete invoice
await use_mcp_tool("quickbooks", "delete_invoice", {
  id: "456",
  syncToken: "2"
});
```

### Payment Recording

```typescript
// Record a customer payment
const payment = await use_mcp_tool("quickbooks", "create_payment", {
  CustomerRef: { value: "123" },
  TotalAmt: 1000.00,
  TxnDate: "2025-05-20",
  Line: [
    {
      Amount: 1000.00,
      LinkedTxn: [
        {
          TxnId: "456",  // Invoice ID
          TxnType: "Invoice"
        }
      ]
    }
  ]
});

// Search payments in date range
const payments = await use_mcp_tool("quickbooks", "search_payments", {
  query: "SELECT * FROM Payment WHERE TxnDate >= '2025-01-01' AND TxnDate <= '2025-05-31'"
});
```

### Bill and Vendor Management

```typescript
// Create a vendor
const vendor = await use_mcp_tool("quickbooks", "create_vendor", {
  DisplayName: "Office Supplies Co",
  PrimaryEmailAddr: { Address: "billing@officesupplies.com" },
  BillAddr: {
    Line1: "456 Vendor Ave",
    City: "Oakland",
    CountrySubDivisionCode: "CA",
    PostalCode: "94612",
    Country: "USA"
  }
});

// Create a bill
const bill = await use_mcp_tool("quickbooks", "create_bill", {
  VendorRef: { value: "789" },
  Line: [
    {
      Amount: 500.00,
      DetailType: "AccountBasedExpenseLineDetail",
      AccountBasedExpenseLineDetail: {
        AccountRef: { value: "45" }  // Expense account ID
      }
    }
  ],
  DueDate: "2025-06-15",
  TxnDate: "2025-05-20"
});

// Pay a bill
const billPayment = await use_mcp_tool("quickbooks", "create_bill_payment", {
  VendorRef: { value: "789" },
  TotalAmt: 500.00,
  PayType: "Check",
  CheckPayment: {
    BankAccountRef: { value: "35" }  // Bank account ID
  },
  Line: [
    {
      Amount: 500.00,
      LinkedTxn: [
        {
          TxnId: "890",  // Bill ID
          TxnType: "Bill"
        }
      ]
    }
  ]
});
```

## Financial Reports

### Balance Sheet

```typescript
const balanceSheet = await use_mcp_tool("quickbooks", "get_balance_sheet", {
  start_date: "2025-01-01",
  end_date: "2025-05-31",
  accounting_method: "Accrual",  // or "Cash"
  summarize_column_by: "Month"   // Daily, Week, Month, Quarter, Year
});

// Access report data
console.log(balanceSheet.Header.ReportName);
console.log(balanceSheet.Rows);  // Array of report rows
```

### Profit & Loss

```typescript
const profitLoss = await use_mcp_tool("quickbooks", "get_profit_and_loss", {
  start_date: "2025-01-01",
  end_date: "2025-05-31",
  accounting_method: "Accrual",
  summarize_column_by: "Quarter"
});

// Filter by customer
const customerPL = await use_mcp_tool("quickbooks", "get_profit_and_loss", {
  start_date: "2025-01-01",
  end_date: "2025-05-31",
  customer: "123"  // Customer ID
});
```

### Cash Flow Report

```typescript
const cashFlow = await use_mcp_tool("quickbooks", "get_cash_flow", {
  start_date: "2025-01-01",
  end_date: "2025-05-31",
  summarize_column_by: "Month"
});
```

### Aged Receivables

```typescript
const agedReceivables = await use_mcp_tool("quickbooks", "get_aged_receivables", {
  as_of_date: "2025-05-31",
  aging_method: "Current",  // Current, Report_Date
  num_periods: 4,
  aging_period: 30  // Days per aging bucket
});

// Detailed aging
const detailedAging = await use_mcp_tool("quickbooks", "get_aged_receivables_detail", {
  as_of_date: "2025-05-31",
  num_periods: 4,
  aging_period: 30
});
```

### General Ledger

```typescript
const generalLedger = await use_mcp_tool("quickbooks", "get_general_ledger", {
  start_date: "2025-01-01",
  end_date: "2025-05-31",
  accounting_method: "Accrual"
});
```

## Search Query Patterns

QuickBooks uses SQL-like queries for searching entities:

### Basic Queries

```typescript
// All active customers
SELECT * FROM Customer WHERE Active = true

// Invoices in date range
SELECT * FROM Invoice WHERE TxnDate >= '2025-01-01' AND TxnDate <= '2025-05-31'

// Unpaid invoices over $1000
SELECT * FROM Invoice WHERE Balance > '1000' AND Balance > '0'

// Vendors with email
SELECT * FROM Vendor WHERE PrimaryEmailAddr IS NOT NULL

// Limit results
SELECT * FROM Customer WHERE Active = true MAXRESULTS 50
```

### Advanced Queries

```typescript
// Multiple conditions
SELECT * FROM Invoice WHERE CustomerRef = '123' AND TxnDate >= '2025-01-01' ORDER BY TxnDate DESC

// Text search (LIKE)
SELECT * FROM Customer WHERE DisplayName LIKE '%Corporation%'

// Count query
SELECT COUNT(*) FROM Invoice WHERE Balance > '0'

// Specific fields
SELECT Id, DisplayName, Balance FROM Customer WHERE Balance > '100'
```

### Using Search Tools

```typescript
// Search with query
const customers = await use_mcp_tool("quickbooks", "search_customers", {
  query: "SELECT * FROM Customer WHERE Active = true MAXRESULTS 100"
});

// Iterate results
customers.QueryResponse.Customer.forEach(customer => {
  console.log(`${customer.DisplayName}: ${customer.Balance}`);
});
```

## Common Patterns

### Creating Line Items

Most transaction entities (Invoice, Bill, Sales Receipt) use `Line` arrays:

```typescript
// Sales item line (for products/services)
{
  Amount: 100.00,
  DetailType: "SalesItemLineDetail",
  SalesItemLineDetail: {
    ItemRef: { value: "1" },  // Item from QuickBooks
    Qty: 1,
    UnitPrice: 100.00,
    TaxCodeRef: { value: "TAX" }  // Optional
  }
}

// Account-based expense line (for bills/purchases)
{
  Amount: 50.00,
  DetailType: "AccountBasedExpenseLineDetail",
  AccountBasedExpenseLineDetail: {
    AccountRef: { value: "45" },  // Chart of accounts ID
    ClassRef: { value: "200" }    // Optional class tracking
  }
}

// Discount line
{
  Amount: 10.00,
  DetailType: "DiscountLineDetail",
  DiscountLineDetail: {
    PercentBased: true,
    DiscountPercent: 10
  }
}

// Subtotal line
{
  Amount: 100.00,
  DetailType: "SubTotalLineDetail",
  SubTotalLineDetail: {}
}
```

### Handling SyncToken for Updates

All updates require the current `SyncToken` for optimistic locking:

```typescript
// 1. Get current entity
const entity = await use_mcp_tool("quickbooks", "get_customer", { id: "123" });

// 2. Update with SyncToken
const updated = await use_mcp_tool("quickbooks", "update_customer", {
  Id: "123",
  SyncToken: entity.Customer.SyncToken,  // Required!
  DisplayName: "New Name",
  sparse: true  // Only update specified fields
});
```

**If SyncToken is stale, you'll get a 400 error.** Always fetch before updating.

### Sparse vs Full Updates

```typescript
// Sparse update (only changes specified fields)
{
  Id: "123",
  SyncToken: "1",
  DisplayName: "Updated Name",
  sparse: true  // Leave other fields unchanged
}

// Full update (replaces entire object)
{
  Id: "123",
  SyncToken: "1",
  DisplayName: "Updated Name",
  GivenName: "John",
  FamilyName: "Smith",
  // Must include ALL fields you want to keep
  sparse: false  // or omit sparse parameter
}
```

### Reference Objects

QuickBooks uses reference objects for relationships:

```typescript
// Customer reference
CustomerRef: { value: "123" }  // Customer ID

// Item reference
ItemRef: { value: "45", name: "Consulting Service" }  // name is optional

// Account reference
AccountRef: { value: "67" }

// Class reference (for tracking)
ClassRef: { value: "200" }

// Department reference
DepartmentRef: { value: "300" }
```

### Date Formats

All dates use `YYYY-MM-DD` format:

```typescript
TxnDate: "2025-05-20"
DueDate: "2025-06-30"
start_date: "2025-01-01"
```

## Account and Item Setup

### Chart of Accounts

```typescript
// Create an account
const account = await use_mcp_tool("quickbooks", "create_account", {
  Name: "Office Expenses",
  AccountType: "Expense",  // Asset, Liability, Equity, Revenue, Expense
  AccountSubType: "OfficeExpenses"
});

// Search accounts by type
const expenses = await use_mcp_tool("quickbooks", "search_accounts", {
  query: "SELECT * FROM Account WHERE AccountType = 'Expense'"
});
```

### Items (Products/Services)

```typescript
// Create a service item
const item = await use_mcp_tool("quickbooks", "create_item", {
  Name: "Consulting Service",
  Type: "Service",
  IncomeAccountRef: { value: "89" },  // Revenue account
  UnitPrice: 150.00
});

// Create an inventory item
const product = await use_mcp_tool("quickbooks", "create_item", {
  Name: "Widget",
  Type: "Inventory",
  QtyOnHand: 100,
  InvStartDate: "2025-01-01",
  IncomeAccountRef: { value: "89" },
  AssetAccountRef: { value: "90" },
  ExpenseAccountRef: { value: "91" },
  UnitPrice: 25.00
});

// Search items
const services = await use_mcp_tool("quickbooks", "search_items", {
  query: "SELECT * FROM Item WHERE Type = 'Service' AND Active = true"
});
```

## Journal Entries

```typescript
// Create a journal entry
const journalEntry = await use_mcp_tool("quickbooks", "create_journal_entry", {
  TxnDate: "2025-05-20",
  Line: [
    {
      Amount: 1000.00,
      DetailType: "JournalEntryLineDetail",
      JournalEntryLineDetail: {
        PostingType: "Debit",
        AccountRef: { value: "35" }  // Bank account
      }
    },
    {
      Amount: 1000.00,
      DetailType: "JournalEntryLineDetail",
      JournalEntryLineDetail: {
        PostingType: "Credit",
        AccountRef: { value: "89" }  // Revenue account
      }
    }
  ]
});
```

**Journal entries must balance:** Sum of debits = Sum of credits.

## Class and Department Tracking

```typescript
// Create a class
const projectClass = await use_mcp_tool("quickbooks", "create_class", {
  Name: "Project Alpha"
});

// Create a department
const dept = await use_mcp_tool("quickbooks", "create_department", {
  Name: "Marketing"
});

// Use in transaction
const invoice = await use_mcp_tool("quickbooks", "create_invoice", {
  CustomerRef: { value: "123" },
  Line: [
    {
      Amount: 1000.00,
      DetailType: "SalesItemLineDetail",
      SalesItemLineDetail: {
        ItemRef: { value: "1" },
        Qty: 1,
        UnitPrice: 1000.00,
        ClassRef: { value: "400" },      // Project tracking
        DepartmentRef: { value: "500" }  // Department tracking
      }
    }
  ]
});
```

## Troubleshooting

### Authentication Errors

**Problem:** `401 Unauthorized` or `Invalid token`

**Solutions:**
1. Verify refresh token is current (refresh tokens expire after 100 days of non-use)
2. Check `QUICKBOOKS_REALM_ID` matches the connected company
3. Ensure environment (`sandbox` vs `production`) matches your app configuration
4. Re-run OAuth flow to get new refresh token

```typescript
// Check token expiration in QuickBooks API response headers
// X-Rate-Limit-* headers indicate API quota status
```

### Validation Errors

**Problem:** `400 Bad Request` with validation message

**Common causes:**
- Missing required fields (e.g., `CustomerRef` on Invoice)
- Invalid reference IDs (entity doesn't exist)
- Stale `SyncToken` on update
- Line items don't balance (Journal Entry)
- Date format incorrect (must be `YYYY-MM-DD`)

**Debug approach:**
```typescript
// Get current entity first to see valid structure
const existing = await use_mcp_tool("quickbooks", "get_invoice", { id: "123" });
console.log(JSON.stringify(existing, null, 2));

// Use response as template for updates
```

### Query Errors

**Problem:** Search returns empty or error

**Solutions:**
- Check entity name spelling: `SELECT * FROM Customer` (not `Customers`)
- Use single quotes for string values: `WHERE DisplayName = 'Acme'`
- Date comparisons: `WHERE TxnDate >= '2025-01-01'`
- Limit results: `MAXRESULTS 1000` (default is 100)

**Valid entity names:**
```
Account, Bill, BillPayment, Class, CompanyInfo, CreditMemo, Customer, 
Department, Deposit, Employee, Estimate, Invoice, Item, JournalEntry, 
Payment, PaymentMethod, Purchase, PurchaseOrder, RefundReceipt, 
SalesReceipt, TaxAgency, TaxCode, TaxRate, Term, TimeActivity, Transfer, 
Vendor, VendorCredit
```

### Report Date Issues

**Problem:** Report returns no data or unexpected results

**Solutions:**
- Ensure date range is valid: `end_date >= start_date`
- Use correct date format: `YYYY-MM-DD`
- Check accounting method matches company settings: `Accrual` vs `Cash`
- Verify company has data in the date range

```typescript
// Get company info to check fiscal year and accounting method
const companyInfo = await use_mcp_tool("quickbooks", "get_company_info", {});
console.log(companyInfo.CompanyInfo.FiscalYearStartMonth);
```

### Rate Limiting

**Problem:** `429 Too Many Requests`

QuickBooks API limits:
- Sandbox: 100 requests per minute per app
- Production: 500 requests per minute per app per company

**Solution:** Implement exponential backoff:

```typescript
// The MCP server doesn't automatically retry
// Implement retry logic in your calling code
async function retryOperation(operation, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Production Environment Setup

**Problem:** Can't connect to production QBO

**Requirements for production:**
1. Redirect URI must be HTTPS (not localhost)
2. App must be published (not in development mode)
3. May require Intuit security review

**Workaround for development:**
- Use ngrok or similar to tunnel localhost: `https://your-subdomain.ngrok.io/callback`
- Add this URL to your app's redirect URIs
- Update OAuth flow to use the public URL

### SyncToken Conflicts

**Problem:** `Stale object error` on update

**Cause:** Another process updated the entity between your read and write

**Solution:**
```typescript
// Retry with fresh fetch
try {
  const updated = await use_mcp_tool("quickbooks", "update_customer", {...});
} catch (error) {
  if (error.message.includes('stale')) {
    // Refetch and try again
    const fresh = await use_mcp_tool("quickbooks", "get_customer", { id: "123" });
    const retry = await use_mcp_tool("quickbooks", "update_customer", {
      ...updateData,
      SyncToken: fresh.Customer.SyncToken
    });
  }
}
```

## Testing

The project includes comprehensive Jest tests:

```bash
# Run all tests
npm test

# Run specific test suite
npm test -- customer.test.ts

# Run with coverage
npm test -- --coverage
```

Test structure:
- `src/__tests__/tools/` - Individual entity tool tests
- `src/__tests__/reports/` - Report tool tests
- Each entity has 5 test cases (create, get, update, delete, search)

## References

- [QuickBooks Online API Documentation](https://developer.intuit.com/app/developer/qbo/docs/api/accounting/all-entities/account)
- [OAuth 2.0 Guide](https://developer.intuit.com/app/developer/qbo/docs/develop/authentication-and-authorization/oauth-2.0)
- [API Explorer](https://developer.intuit.com/app/developer/qbo/docs/api/accounting/all-entities/account)
- [Intuit Developer Portal](https://developer.intuit.com/)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
