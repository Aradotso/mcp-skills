---
name: data-go-mcp-servers
description: MCP servers for accessing Korea's public data portal (data.go.kr) APIs including business registration, procurement, financial info, and more
triggers:
  - how do I query Korean business registration data
  - integrate data.go.kr APIs with Claude
  - search Korean national pension enrollment
  - verify Korean business registration number
  - access Korean public procurement data
  - set up data-go-mcp servers
  - query Korean government open data
  - check Korean business tax status
---

# data-go-mcp-servers

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

MCP servers providing standardized access to Korea's public data portal (data.go.kr) APIs. This project wraps various Korean government APIs as Model Context Protocol servers, enabling AI tools like Claude Desktop and Cline to directly query business registrations, procurement data, financial information, and more.

## Available MCP Servers

The project provides six specialized MCP servers:

1. **NPS Business Enrollment** - National Pension Service workplace enrollment data
2. **NTS Business Verification** - National Tax Service business registration verification
3. **PPS Narajangteo** - Public procurement bidding and contract information
4. **FSC Financial Info** - Financial Services Commission corporate financial data
5. **Presidential Speeches** - Presidential archives speech database
6. **MSDS Chemical Info** - Material Safety Data Sheets chemical information

## Installation

### Using UV (Recommended)

```bash
# Install specific servers
uv pip install data-go-mcp.nps-business-enrollment
uv pip install data-go-mcp.nts-business-verification
uv pip install data-go-mcp.pps-narajangteo
uv pip install data-go-mcp.fsc-financial-info
uv pip install data-go-mcp.presidential-speeches
uv pip install data-go-mcp.msds-chemical-info
```

### Using pip

```bash
pip install data-go-mcp.nps-business-enrollment
pip install data-go-mcp.nts-business-verification
pip install data-go-mcp.pps-narajangteo
pip install data-go-mcp.fsc-financial-info
pip install data-go-mcp.presidential-speeches
pip install data-go-mcp.msds-chemical-info
```

## Configuration

### Claude Desktop Setup

Edit the Claude Desktop configuration file:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`  
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "data-go-mcp.nps-business-enrollment": {
      "command": "uvx",
      "args": ["data-go-mcp.nps-business-enrollment@latest"],
      "env": {
        "API_KEY": "${DATA_GO_KR_API_KEY}"
      }
    },
    "data-go-mcp.nts-business-verification": {
      "command": "uvx",
      "args": ["data-go-mcp.nts-business-verification@latest"],
      "env": {
        "API_KEY": "${DATA_GO_KR_API_KEY}"
      }
    },
    "data-go-mcp.pps-narajangteo": {
      "command": "uvx",
      "args": ["data-go-mcp.pps-narajangteo@latest"],
      "env": {
        "API_KEY": "${DATA_GO_KR_API_KEY}"
      }
    },
    "data-go-mcp.fsc-financial-info": {
      "command": "uvx",
      "args": ["data-go-mcp.fsc-financial-info@latest"],
      "env": {
        "API_KEY": "${DATA_GO_KR_API_KEY}"
      }
    }
  }
}
```

### Cline (VS Code) Setup

Create `.vscode/cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "data-go-mcp.nps-business-enrollment": {
      "command": "python",
      "args": ["-m", "data_go_mcp.nps_business_enrollment.server"],
      "env": {
        "API_KEY": "${DATA_GO_KR_API_KEY}"
      }
    },
    "data-go-mcp.nts-business-verification": {
      "command": "python",
      "args": ["-m", "data_go_mcp.nts_business_verification.server"],
      "env": {
        "API_KEY": "${DATA_GO_KR_API_KEY}"
      }
    }
  }
}
```

### Environment Variables

Set your data.go.kr API key as an environment variable:

```bash
export DATA_GO_KR_API_KEY="your-api-key-here"
```

Obtain an API key from [data.go.kr](https://www.data.go.kr/).

## MCP Server Tools Reference

### NPS Business Enrollment Server

**Tool: `search_business`**

Search for businesses enrolled in the National Pension Service.

Parameters:
- `ldong_addr_mgpl_dg_cd` (string, optional): Metropolitan city/province code (2 digits)
- `ldong_addr_mgpl_sggu_cd` (string, optional): City/county/district code (5 digits)
- `ldong_addr_mgpl_sggu_emd_cd` (string, optional): Town/township/neighborhood code (8 digits)
- `wkpl_nm` (string, optional): Workplace name
- `bzowr_rgst_no` (string, optional): Business registration number (first 6 digits)
- `page_no` (integer, optional): Page number (default: 1)
- `num_of_rows` (integer, optional): Results per page (default: 100, max: 100)

Example natural language queries:
- "Find businesses in Seoul Gangnam-gu"
- "Search for Samsung Electronics workplaces"
- "Look up businesses with registration number starting with 123456"

### NTS Business Verification Server

**Tool: `validate_business`**

Verify the authenticity of business registration information.

Parameters:
- `business_number` (string, required): Business registration number (10 digits)
- `start_date` (string, required): Opening date (YYYYMMDD format)
- `representative_name` (string, required): Representative's name
- `representative_name2` (string, optional): Representative's name 2 (foreign Korean name)
- `business_name` (string, optional): Business name
- `corp_number` (string, optional): Corporate registration number (13 digits)
- `business_sector` (string, optional): Main business sector
- `business_type` (string, optional): Main business type
- `business_address` (string, optional): Business address

**Tool: `check_business_status`**

Check the current status of business registrations.

Parameters:
- `business_numbers` (string, required): Comma-separated business registration numbers (max 100)

Returns:
- Business status: 01 (active), 02 (suspended), 03 (closed)
- Tax type: general taxpayer, simplified taxpayer, etc.
- Closure date, tax type conversion date, etc.

**Tool: `batch_validate_businesses`**

Validate multiple businesses at once.

Parameters:
- `businesses_json` (string, required): JSON array of business information (max 100)

JSON format:
```json
[
  {
    "b_no": "1234567890",
    "start_dt": "20200101",
    "p_nm": "Hong Gildong",
    "b_nm": "Test Company"
  }
]
```

Example natural language queries:
- "Verify business registration 123-45-67890 opened on 2020-01-01 by Hong Gildong"
- "Check the status of business number 123-45-67890"
- "Validate multiple businesses: 123-45-67890, 098-76-54321"

### PPS Narajangteo Server

**Tool: `search_bid_announcements`**

Search public procurement bid announcements.

**Tool: `search_successful_bids`**

Search successful bid information.

**Tool: `search_contracts`**

Search contract information.

### FSC Financial Info Server

**Tool: `get_financial_statements`**

Retrieve corporate financial statements.

**Tool: `get_income_statements`**

Retrieve corporate income statements.

### Presidential Speeches Server

**Tool: `search_speeches`**

Search presidential speeches from archives.

### MSDS Chemical Info Server

**Tool: `search_chemical_info`**

Search Material Safety Data Sheet information for chemicals.

## Common Usage Patterns

### Validating Korean Business Registration

```python
# When user asks to verify a Korean business
# The MCP server handles this via natural language:
# "Verify business 123-45-67890 that opened on January 1, 2020 
#  with representative Kim Cheolsu and company name ABC Corp"

# The server will call validate_business with:
# {
#   "business_number": "1234567890",
#   "start_date": "20200101",
#   "representative_name": "김철수",
#   "business_name": "ABC주식회사"
# }
```

### Checking Business Status

```python
# When user asks to check business status:
# "What is the current status of business numbers 
#  123-45-67890 and 098-76-54321?"

# The server will call check_business_status with:
# {
#   "business_numbers": "1234567890,0987654321"
# }

# Returns status codes:
# 01 = Active business
# 02 = Suspended business
# 03 = Closed business
```

### Searching NPS Enrollment Data

```python
# When user asks to find businesses by location:
# "Find all businesses enrolled in NPS in Seoul Gangnam-gu"

# The server will call search_business with:
# {
#   "ldong_addr_mgpl_dg_cd": "11",        # Seoul
#   "ldong_addr_mgpl_sggu_cd": "11680",   # Gangnam-gu
#   "page_no": 1,
#   "num_of_rows": 100
# }
```

### Batch Business Verification

```python
# When user needs to verify multiple businesses:
# "Verify these businesses: 
#  1. 123-45-67890, opened 2020-01-01, rep: Kim
#  2. 098-76-54321, opened 2021-06-15, rep: Lee"

# The server will call batch_validate_businesses with:
# {
#   "businesses_json": '[
#     {"b_no": "1234567890", "start_dt": "20200101", "p_nm": "Kim"},
#     {"b_no": "0987654321", "start_dt": "20210615", "p_nm": "Lee"}
#   ]'
# }
```

## Development Setup

### Clone and Install for Development

```bash
git clone https://github.com/Koomook/data-go-mcp-servers.git
cd data-go-mcp-servers

# Install UV if not already installed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install with dev dependencies
uv sync --dev
```

### Creating a New MCP Server

Use the automated script:

```bash
uv run python scripts/create_mcp_server.py
```

Or manually with Cookiecutter:

```bash
uv pip install cookiecutter
uv run cookiecutter template/ -o src/
```

The template creates:
- Server module structure
- API client implementation
- Pydantic models for type safety
- Tests
- Documentation

### Running Tests

```bash
# Test specific server
uv run pytest src/data_go_mcp/nps_business_enrollment/tests/

# Test all servers
uv run pytest
```

### Building and Publishing

```bash
# Build a specific package
cd src/data_go_mcp/nps_business_enrollment
uv build

# Publish to PyPI
uv publish
```

## Troubleshooting

### API Key Issues

**Problem**: `API_KEY environment variable not set`

**Solution**: Ensure the API key is properly configured:
```bash
export DATA_GO_KR_API_KEY="your-actual-api-key"
```

For Claude Desktop, verify the environment variable is referenced correctly in `claude_desktop_config.json`.

### Server Not Appearing in Claude

**Problem**: MCP server doesn't show up in Claude Desktop

**Solutions**:
1. Restart Claude Desktop completely (quit and relaunch)
2. Check the configuration file syntax is valid JSON
3. Verify the `command` path is correct (`uvx` or `python`)
4. Check Claude Desktop logs for errors

**macOS logs**: `~/Library/Logs/Claude/`  
**Windows logs**: `%APPDATA%\Claude\logs\`

### Connection Errors

**Problem**: `Failed to connect to data.go.kr API`

**Solutions**:
1. Verify your API key is valid and active at data.go.kr
2. Check network connectivity
3. Verify the API endpoint is accessible
4. Check if you've exceeded rate limits

### Invalid Business Numbers

**Problem**: Business number format errors

**Solution**: Korean business registration numbers must be:
- Exactly 10 digits (remove hyphens)
- Format: XXX-XX-XXXXX becomes XXXXXXXXXX
- Valid checksum digit

Example: `123-45-67890` → `1234567890`

### Date Format Issues

**Problem**: Date format validation errors

**Solution**: Use YYYYMMDD format:
- Correct: `20200101`
- Incorrect: `2020-01-01`, `01/01/2020`

### Pagination Limits

**Problem**: Not getting all results

**Solution**: The `num_of_rows` parameter has a maximum of 100. For larger datasets:
1. Use pagination with `page_no`
2. Make multiple requests
3. Iterate through pages until no more results

```python
# Pseudocode for pagination
page = 1
all_results = []
while True:
    results = search_business(page_no=page, num_of_rows=100)
    if not results:
        break
    all_results.extend(results)
    page += 1
```

### Character Encoding

**Problem**: Korean characters not displaying correctly

**Solution**: Ensure UTF-8 encoding:
- API responses are UTF-8 by default
- Check your terminal/editor encoding settings
- For Python files, include: `# -*- coding: utf-8 -*-`

## Best Practices

1. **API Key Security**: Never hardcode API keys. Always use environment variables.

2. **Rate Limiting**: Respect data.go.kr rate limits. Implement exponential backoff for retries.

3. **Error Handling**: Always check response status and handle errors gracefully.

4. **Data Validation**: Use Pydantic models to ensure data integrity.

5. **Caching**: Consider caching responses for frequently accessed data to reduce API calls.

6. **Batch Operations**: Use batch endpoints when validating multiple businesses to minimize API calls.

7. **Testing**: Test with real API responses in development, but use mocks in CI/CD.
