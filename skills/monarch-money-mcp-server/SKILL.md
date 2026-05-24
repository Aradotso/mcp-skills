---
name: monarch-money-mcp-server
description: MCP server for accessing Monarch Money personal finance data through Claude Desktop and Claude Code
triggers:
  - show me my monarch money accounts
  - get my recent transactions from monarch
  - check my budget in monarch money
  - analyze my spending with monarch
  - create a transaction in monarch money
  - categorize my monarch transactions
  - view my net worth from monarch
  - refresh my monarch money accounts
---

# Monarch Money MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection

## Overview

The Monarch Money MCP Server provides seamless integration with the Monarch Money personal finance platform through the Model Context Protocol. It enables Claude Desktop and Claude Code to access your financial accounts, transactions, budgets, analytics, and more through a comprehensive set of tools.

Built on the [MonarchMoneyCommunity Python library](https://github.com/bradleyseanf/monarchmoneycommunity) with full MFA support.

## Installation

### 1. Clone and Install Dependencies

```bash
git clone https://github.com/robcerda/monarch-mcp-server.git
cd monarch-mcp-server

# Using pip
pip install -r requirements.txt
pip install -e .

# OR using uv
uv sync
```

### 2. Configure Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "Monarch Money": {
      "command": "/opt/homebrew/bin/uv",
      "args": [
        "run",
        "--with",
        "mcp[cli]",
        "--with-editable",
        "/absolute/path/to/monarch-mcp-server",
        "mcp",
        "run",
        "/absolute/path/to/monarch-mcp-server/src/monarch_mcp_server/server.py"
      ]
    }
  }
}
```

**If using pip instead of uv:**

```json
{
  "mcpServers": {
    "Monarch Money": {
      "command": "python",
      "args": ["/absolute/path/to/monarch-mcp-server/src/monarch_mcp_server/server.py"]
    }
  }
}
```

### 3. Configure Claude Code (CLI)

**Global config** (`~/.claude.json` or `%USERPROFILE%\.claude.json`):

```json
{
  "mcpServers": {
    "Monarch Money": {
      "command": "/opt/homebrew/bin/uv",
      "args": [
        "run",
        "--with",
        "mcp[cli]",
        "--with-editable",
        "/absolute/path/to/monarch-mcp-server",
        "mcp",
        "run",
        "/absolute/path/to/monarch-mcp-server/src/monarch_mcp_server/server.py"
      ]
    }
  }
}
```

**Project-level config** (`.mcp.json` in project directory):

```json
{
  "Monarch Money": {
    "command": "/opt/homebrew/bin/uv",
    "args": [
      "run",
      "--with",
      "mcp[cli]",
      "--with-editable",
      "/absolute/path/to/monarch-mcp-server",
      "mcp",
      "run",
      "/absolute/path/to/monarch-mcp-server/src/monarch_mcp_server/server.py"
    ]
  }
}
```

### 4. One-Time Authentication

Run the authentication setup script **outside of Claude**:

```bash
cd /absolute/path/to/monarch-mcp-server

# Using python
python login_setup.py

# OR using uv
uv run python login_setup.py
```

Follow prompts to enter:
- Monarch Money email and password
- 2FA code (if MFA is enabled)

Session tokens are stored securely and persist for weeks/months.

**For SSO/Google sign-in users:**
Use the `monarch_login_with_token` function to paste a session token from your browser.

## Core Tools

### Account Management

#### `get_accounts`
Retrieve all linked financial accounts with balances and institution info.

```python
# Returns all accounts with: id, displayName, currentBalance, accountType, institution
get_accounts()
```

**Example prompt:**
```
Show me all my financial accounts
```

#### `get_account_holdings`
View securities and investments in investment accounts.

```python
get_account_holdings(account_id="acc_123456")
```

**Example prompt:**
```
What holdings do I have in my investment account acc_123456?
```

#### `refresh_accounts`
Request real-time data refresh from financial institutions.

```python
refresh_accounts()
```

**Example prompt:**
```
Refresh all my account balances from the banks
```

### Transaction Access

#### `get_transactions`
Fetch transactions with filtering, pagination, and reconciliation fields.

```python
# Basic usage - last 50 transactions
get_transactions(limit=50)

# Filtered by date range
get_transactions(
    start_date="2024-01-01",
    end_date="2024-01-31",
    limit=100
)

# Filter by account and category
get_transactions(
    account_id="acc_123",
    category_ids=["cat_456", "cat_789"],
    limit=50,
    offset=0
)

# Search transactions
get_transactions(
    search="Amazon",
    wide_search=True,
    limit=25
)

# Filter by flags
get_transactions(
    has_notes=False,
    is_split=False,
    limit=100
)
```

**Example prompts:**
```
Show me my last 100 transactions
Get all Amazon transactions from January 2024
Show me uncategorized transactions with no notes
```

#### `create_transaction`
Add a new transaction to an account.

```python
create_transaction(
    account_id="acc_123",
    amount=-45.99,
    description="Coffee shop",
    date="2024-01-15",
    category_id="cat_dining",
    merchant_name="Local Cafe"
)
```

**Example prompt:**
```
Create a transaction for $45.99 at Local Cafe on January 15th in my checking account
```

#### `update_transaction`
Modify existing transaction details.

```python
update_transaction(
    transaction_id="txn_789",
    amount=-52.00,
    description="Grocery store shopping",
    category_id="cat_groceries",
    date="2024-01-16"
)
```

**Example prompt:**
```
Update transaction txn_789 to $52 and change the category to groceries
```

### Budget Management

#### `get_budgets`
Access budget information with spending analysis.

```python
# Current month budgets
get_budgets()

# Specific date range
get_budgets(
    start_date="2024-01-01",
    end_date="2024-12-31"
)
```

**Example prompt:**
```
Show me my budgets for 2024
```

#### `set_budget_amount`
Create or modify budget amounts.

```python
# Set budget for a category
set_budget_amount(
    amount=500.00,
    category_id="cat_dining",
    start_date="2024-02-01",
    apply_to_future=True
)

# Set budget for a category group
set_budget_amount(
    amount=2000.00,
    category_group_id="grp_housing",
    start_date="2024-02-01",
    apply_to_future=False
)
```

**Example prompt:**
```
Set my dining budget to $500 per month starting in February
```

### Category Management

#### `get_categories`
List all transaction categories with groups and metadata.

```python
get_categories()
# Returns: id, name, group, icon, systemCategory, isSystemCategory, isDisabled
```

**Example prompt:**
```
List all my transaction categories
```

#### `get_category_groups`
View category groups with their associated categories.

```python
get_category_groups()
```

**Example prompt:**
```
Show me all category groups
```

#### `set_transaction_category`
Assign a category to a transaction.

```python
set_transaction_category(
    transaction_id="txn_123",
    category_id="cat_groceries",
    mark_reviewed=True
)
```

**Example prompt:**
```
Categorize transaction txn_123 as groceries and mark it reviewed
```

#### `bulk_categorize_transactions`
Apply a category to multiple transactions at once.

```python
bulk_categorize_transactions(
    transaction_ids=["txn_1", "txn_2", "txn_3"],
    category_id="cat_dining"
)
```

**Example prompt:**
```
Categorize all these Amazon transactions as shopping: txn_1, txn_2, txn_3
```

### Transaction Review Workflow

#### `get_transactions_needing_review`
Find transactions requiring attention.

```python
# All transactions needing review
get_transactions_needing_review(needs_review=True)

# Uncategorized transactions only
get_transactions_needing_review(uncategorized=True)

# Transactions without notes
get_transactions_needing_review(no_notes=True)

# Last N days
get_transactions_needing_review(
    needs_review=True,
    days=30
)
```

**Example prompt:**
```
Show me all uncategorized transactions from the last 30 days
```

#### `update_transaction_notes`
Add or update notes on transactions.

```python
update_transaction_notes(
    transaction_id="txn_456",
    notes="Receipt: https://example.com/receipt.pdf"
)
```

**Example prompt:**
```
Add a note to transaction txn_456 with the receipt link
```

#### `mark_transaction_reviewed`
Clear the needs_review flag.

```python
mark_transaction_reviewed(transaction_id="txn_789")
```

**Example prompt:**
```
Mark transaction txn_789 as reviewed
```

### Tag Management

#### `get_tags`
List all available tags with colors and usage counts.

```python
get_tags()
```

**Example prompt:**
```
Show me all my transaction tags
```

#### `set_transaction_tags`
Apply tags to a transaction.

```python
set_transaction_tags(
    transaction_id="txn_123",
    tag_ids=["tag_business", "tag_reimbursable"]
)
```

**Example prompt:**
```
Tag transaction txn_123 as business and reimbursable
```

#### `create_tag`
Create a new tag with custom name and color.

```python
create_tag(
    name="Travel",
    color="#FF5733"
)
```

**Example prompt:**
```
Create a new tag called Travel with color red
```

### Advanced Search

#### `search_transactions`
Comprehensive search with multiple filters.

```python
search_transactions(
    search="Starbucks",
    category_ids=["cat_dining"],
    account_ids=["acc_checking"],
    tag_ids=["tag_business"],
    start_date="2024-01-01",
    end_date="2024-01-31",
    min_amount=5.00,
    max_amount=50.00
)
```

**Example prompt:**
```
Find all Starbucks transactions tagged as business between $5 and $50 in January
```

#### `get_transaction_details`
Retrieve complete details for a single transaction.

```python
get_transaction_details(transaction_id="txn_456")
```

**Example prompt:**
```
Get full details for transaction txn_456
```

#### `delete_transaction`
Remove a transaction.

```python
delete_transaction(transaction_id="txn_789")
```

**Example prompt:**
```
Delete transaction txn_789
```

#### `get_recurring_transactions`
View upcoming recurring transactions.

```python
get_recurring_transactions()
```

**Example prompt:**
```
Show me all my recurring transactions
```

### Transaction Rules (Auto-Categorization)

#### `get_transaction_rules`
List all auto-categorization rules.

```python
get_transaction_rules()
```

**Example prompt:**
```
Show me all my transaction rules
```

#### `create_transaction_rule`
Create rules with merchant/amount conditions.

```python
create_transaction_rule(
    merchant_criteria_operator="contains",
    merchant_criteria_value="Spotify",
    set_category_id="cat_subscriptions",
    add_tag_ids=["tag_recurring"]
)

# Rule with amount criteria
create_transaction_rule(
    merchant_criteria_operator="contains",
    merchant_criteria_value="Gas",
    set_category_id="cat_transportation",
    amount_operator="greater_than",
    amount_value=20.00
)
```

**Example prompt:**
```
Create a rule to categorize all Spotify transactions as subscriptions
```

#### `update_transaction_rule`
Modify existing rules.

```python
update_transaction_rule(
    rule_id="rule_123",
    merchant_criteria_operator="equals",
    merchant_criteria_value="Netflix",
    set_category_id="cat_entertainment"
)
```

**Example prompt:**
```
Update rule rule_123 to use exact match for Netflix
```

#### `delete_transaction_rule`
Remove a rule.

```python
delete_transaction_rule(rule_id="rule_456")
```

**Example prompt:**
```
Delete transaction rule rule_456
```

### Transaction Splits

#### `get_transaction_splits`
View how a transaction has been split.

```python
get_transaction_splits(transaction_id="txn_789")
```

**Example prompt:**
```
Show me the splits for transaction txn_789
```

#### `split_transaction`
Divide a transaction into multiple parts.

```python
split_transaction(
    transaction_id="txn_123",
    splits=[
        {
            "amount": -30.00,
            "category_id": "cat_groceries",
            "merchant_name": "Target - Groceries"
        },
        {
            "amount": -20.00,
            "category_id": "cat_household",
            "merchant_name": "Target - Household"
        }
    ]
)
```

**Example prompt:**
```
Split transaction txn_123: $30 for groceries and $20 for household items
```

### Financial Analysis

#### `get_cashflow`
Analyze income and expenses over time.

```python
get_cashflow(
    start_date="2024-01-01",
    end_date="2024-12-31"
)
```

**Example prompt:**
```
Show me my cashflow for 2024
```

#### `get_net_worth`
Track net worth with daily snapshots and trends.

```python
# All accounts
get_net_worth(
    start_date="2023-01-01",
    end_date="2024-12-31"
)

# Specific account type
get_net_worth(
    start_date="2024-01-01",
    end_date="2024-12-31",
    account_type="investment"
)
```

**Example prompt:**
```
Show me my net worth trend for 2024
```

#### `get_account_balance_history`
View historical balance data for an account.

```python
get_account_balance_history(account_id="acc_checking_123")
```

**Example prompt:**
```
Show me the balance history for my checking account
```

#### `get_net_worth_by_account_type`
Net worth breakdown across account types.

```python
get_net_worth_by_account_type(
    start_date="2024-01-01",
    timeframe="monthly"
)
```

**Example prompt:**
```
Break down my net worth by account type for 2024
```

#### `get_transactions_summary`
Quick high-level statistics about transactions.

```python
get_transactions_summary()
```

**Example prompt:**
```
Give me a summary of my transactions
```

#### `get_spending_summary`
Spending breakdown by category.

```python
get_spending_summary(
    start_date="2024-01-01",
    end_date="2024-01-31",
    limit=10
)
```

**Example prompt:**
```
Show me my top 10 spending categories for January
```

## Common Patterns

### Monthly Budget Review

```python
# 1. Get current month budgets
budgets = get_budgets()

# 2. Check spending summary
summary = get_spending_summary(
    start_date="2024-01-01",
    end_date="2024-01-31",
    limit=20
)

# 3. Review over-budget categories
# Identify categories where spent > budgeted

# 4. Adjust next month's budgets
set_budget_amount(
    amount=600.00,
    category_id="cat_groceries",
    start_date="2024-02-01",
    apply_to_future=True
)
```

### Transaction Cleanup Workflow

```python
# 1. Find transactions needing review
needs_review = get_transactions_needing_review(
    uncategorized=True,
    days=30
)

# 2. Categorize similar transactions in bulk
bulk_categorize_transactions(
    transaction_ids=["txn_1", "txn_2", "txn_3"],
    category_id="cat_groceries"
)

# 3. Add notes where needed
update_transaction_notes(
    transaction_id="txn_4",
    notes="Reimbursable expense"
)

# 4. Mark reviewed
mark_transaction_reviewed(transaction_id="txn_4")
```

### Auto-Categorization Setup

```python
# 1. Review common merchants
transactions = search_transactions(
    search="Spotify",
    limit=10
)

# 2. Create rule for automatic categorization
create_transaction_rule(
    merchant_criteria_operator="contains",
    merchant_criteria_value="Spotify",
    set_category_id="cat_subscriptions",
    add_tag_ids=["tag_recurring", "tag_entertainment"]
)

# 3. Verify rule is working
rules = get_transaction_rules()
```

### Investment Portfolio Review

```python
# 1. Get all investment accounts
accounts = get_accounts()
investment_accounts = [a for a in accounts if a['accountType'] == 'investment']

# 2. Check holdings for each
for acc in investment_accounts:
    holdings = get_account_holdings(account_id=acc['id'])
    
# 3. Review balance history
history = get_account_balance_history(account_id=acc['id'])

# 4. Check overall net worth trend
net_worth = get_net_worth(
    start_date="2023-01-01",
    end_date="2024-12-31",
    account_type="investment"
)
```

### Expense Tracking with Tags

```python
# 1. Create project-specific tags
create_tag(name="Project Alpha", color="#3498db")
create_tag(name="Reimbursable", color="#e74c3c")

# 2. Tag relevant transactions
set_transaction_tags(
    transaction_id="txn_123",
    tag_ids=["tag_project_alpha", "tag_reimbursable"]
)

# 3. Search for all project expenses
project_expenses = search_transactions(
    tag_ids=["tag_project_alpha"],
    start_date="2024-01-01",
    end_date="2024-12-31"
)

# 4. Calculate total reimbursable amount
reimbursable = search_transactions(
    tag_ids=["tag_reimbursable"],
    start_date="2024-01-01",
    end_date="2024-01-31"
)
```

## Authentication

### Check Authentication Status

```python
check_auth_status()
```

**Example prompt:**
```
Check if I'm authenticated with Monarch Money
```

### Get Setup Instructions

```python
setup_authentication()
```

**Example prompt:**
```
How do I set up authentication for Monarch Money?
```

## Troubleshooting

### Authentication Issues

**Problem:** "Session not found" or authentication errors

**Solution:**
1. Run `python login_setup.py` from the terminal
2. Enter credentials and 2FA code if prompted
3. Restart Claude Desktop or Claude Code
4. Verify session file exists in `~/.monarch/session.pickle`

### Missing Transactions

**Problem:** Transactions don't appear or are outdated

**Solution:**
```python
# Force a refresh from financial institutions
refresh_accounts()

# Wait a few minutes, then check again
get_transactions(limit=50)
```

### Tool Not Found

**Problem:** MCP server not showing in Claude

**Solution:**
1. Verify configuration file path (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`)
2. Check absolute paths in config - no `~` or relative paths
3. Restart Claude Desktop completely (quit and reopen)
4. Check Claude Desktop logs for errors

### Performance with Large Result Sets

**Problem:** Timeouts or slow responses with many transactions

**Solution:**
```python
# Use pagination for large datasets
get_transactions(limit=100, offset=0)
get_transactions(limit=100, offset=100)

# Filter by date range
get_transactions(
    start_date="2024-01-01",
    end_date="2024-01-31",
    limit=500
)

# Use search with specific criteria
search_transactions(
    search="Amazon",
    start_date="2024-01-01",
    end_date="2024-01-31"
)
```

### Split Transaction Issues

**Problem:** Split amounts don't sum to original transaction

**Solution:**
```python
# Ensure split amounts sum exactly to transaction amount
# Original transaction: -$100
split_transaction(
    transaction_id="txn_123",
    splits=[
        {"amount": -60.00, "category_id": "cat_1"},
        {"amount": -40.00, "category_id": "cat_2"}
    ]
)
# Sum: -$100 ✓
```

### Budget Not Updating

**Problem:** Budget changes not reflected

**Solution:**
```python
# Ensure apply_to_future is set correctly
set_budget_amount(
    amount=500.00,
    category_id="cat_dining",
    start_date="2024-02-01",
    apply_to_future=True  # Apply to current and future months
)
```

## Environment Variables

The server uses secure session storage. No environment variables are required for normal operation.

**Session storage location:**
- macOS/Linux: `~/.monarch/session.pickle`
- Windows: `%USERPROFILE%\.monarch\session.pickle`

## Additional Resources

- [Monarch Money Official Site](https://www.monarchmoney.com)
- [MonarchMoneyCommunity Library](https://github.com/bradleyseanf/monarchmoneycommunity)
- [MCP Protocol Documentation](https://modelcontextprotocol.io)
- [Project Repository](https://github.com/robcerda/monarch-mcp-server)
