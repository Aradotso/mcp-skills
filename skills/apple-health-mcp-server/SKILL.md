---
name: apple-health-mcp-server
description: MCP server for querying Apple Health data with natural language using DuckDB, Elasticsearch, or ClickHouse backends
triggers:
  - how do I set up Apple Health MCP server
  - query my Apple Health data with natural language
  - analyze Apple Health XML export
  - configure apple health mcp with DuckDB
  - search health records using MCP
  - import Apple Health data into database
  - build health data analysis with MCP
  - troubleshoot Apple Health MCP connection
---

# Apple Health MCP Server Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Apple Health MCP Server is a Model Context Protocol (MCP) server built with FastMCP that enables natural language querying of Apple Health data. It parses Apple Health XML exports and loads them into DuckDB, Elasticsearch, or ClickHouse for efficient querying. The server exposes MCP tools that allow LLMs to analyze health records, generate statistics, and identify trends without requiring SQL knowledge.

**Key capabilities:**
- Parse and import Apple Health XML exports
- Natural language queries over health data
- Multiple backend support (DuckDB, Elasticsearch, ClickHouse)
- Structured data analysis and trend detection
- Docker-ready deployment

## Installation

### Prerequisites

- Python 3.10+
- Apple Health export XML file
- MCP-compatible client (Claude Desktop, Cursor, etc.)

### Setup Steps

1. **Clone the repository:**

```bash
git clone https://github.com/the-momentum/apple-health-mcp-server.git
cd apple-health-mcp-server
```

2. **Install dependencies:**

```bash
pip install -e .
```

3. **Configure environment variables:**

Create a `.env` file:

```bash
# Required: Path to Apple Health export XML
APPLE_HEALTH_EXPORT_PATH=/path/to/apple_health_export/export.xml

# Storage backend (duckdb, elasticsearch, or clickhouse)
STORAGE_BACKEND=duckdb

# DuckDB configuration (default backend)
DUCKDB_PATH=./data/health.db

# Optional: Elasticsearch configuration
# ELASTICSEARCH_URL=http://localhost:9200
# ELASTICSEARCH_INDEX=health_records
# ELASTICSEARCH_API_KEY=${ELASTICSEARCH_API_KEY}

# Optional: ClickHouse configuration
# CLICKHOUSE_HOST=localhost
# CLICKHOUSE_PORT=9000
# CLICKHOUSE_DATABASE=health
# CLICKHOUSE_USER=${CLICKHOUSE_USER}
# CLICKHOUSE_PASSWORD=${CLICKHOUSE_PASSWORD}

# Logging
LOG_LEVEL=INFO
```

4. **Import your Apple Health data:**

```bash
python -m apple_health_mcp_server.import_data
```

### MCP Client Configuration

Add to your MCP client configuration (e.g., Claude Desktop `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "apple-health": {
      "command": "python",
      "args": ["-m", "apple_health_mcp_server"],
      "env": {
        "APPLE_HEALTH_EXPORT_PATH": "/path/to/export.xml",
        "STORAGE_BACKEND": "duckdb",
        "DUCKDB_PATH": "./data/health.db"
      }
    }
  }
}
```

### Docker Installation

```bash
# Build the image
docker build -t apple-health-mcp-server .

# Run with volume mounts
docker run -v /path/to/export.xml:/data/export.xml \
  -v ./data:/app/data \
  -e APPLE_HEALTH_EXPORT_PATH=/data/export.xml \
  -e STORAGE_BACKEND=duckdb \
  -e DUCKDB_PATH=/app/data/health.db \
  apple-health-mcp-server
```

## Available MCP Tools

The server exposes the following MCP tools:

### 1. `analyze_structure`

Analyzes the structure and available data types in your Apple Health export.

```python
# Tool call example (invoked by LLM)
{
  "name": "analyze_structure",
  "arguments": {}
}

# Returns: Summary of record types, counts, and date ranges
```

### 2. `search_records`

Search health records using flexible filters.

```python
# Tool call with filters
{
  "name": "search_records",
  "arguments": {
    "record_type": "HKQuantityTypeIdentifierStepCount",
    "start_date": "2024-01-01",
    "end_date": "2024-01-31",
    "limit": 100,
    "source_name": "iPhone"
  }
}
```

**Parameters:**
- `record_type`: Apple Health record type identifier
- `start_date`: ISO format date (YYYY-MM-DD)
- `end_date`: ISO format date (YYYY-MM-DD)
- `limit`: Maximum records to return (default: 100)
- `source_name`: Filter by data source device/app
- `unit`: Filter by measurement unit

### 3. `get_records_by_type`

Extract all records of a specific type.

```python
{
  "name": "get_records_by_type",
  "arguments": {
    "record_type": "HKQuantityTypeIdentifierHeartRate",
    "limit": 500
  }
}
```

### 4. `generate_statistics`

Generate statistical summaries for a specific metric.

```python
{
  "name": "generate_statistics",
  "arguments": {
    "record_type": "HKQuantityTypeIdentifierDistanceWalkingRunning",
    "start_date": "2024-06-01",
    "end_date": "2024-06-30",
    "aggregation": "daily"
  }
}
```

**Aggregation options:** `daily`, `weekly`, `monthly`

### 5. `analyze_trends`

Identify trends and patterns in health data.

```python
{
  "name": "analyze_trends",
  "arguments": {
    "record_type": "HKQuantityTypeIdentifierStepCount",
    "time_period": "last_30_days",
    "trend_type": "moving_average"
  }
}
```

## Common Usage Patterns

### Pattern 1: Initial Data Exploration

```python
# First, analyze what data is available
analyze_structure()

# Then search for specific metrics
search_records(
    record_type="HKQuantityTypeIdentifierStepCount",
    start_date="2024-01-01",
    end_date="2024-12-31"
)
```

### Pattern 2: Monthly Activity Summary

```python
# Get daily step statistics for a month
generate_statistics(
    record_type="HKQuantityTypeIdentifierStepCount",
    start_date="2024-06-01",
    end_date="2024-06-30",
    aggregation="daily"
)

# Analyze workout data for the same period
search_records(
    record_type="HKWorkoutTypeIdentifier",
    start_date="2024-06-01",
    end_date="2024-06-30"
)
```

### Pattern 3: Heart Rate Analysis

```python
# Get heart rate readings
get_records_by_type(
    record_type="HKQuantityTypeIdentifierHeartRate",
    limit=1000
)

# Analyze trends
analyze_trends(
    record_type="HKQuantityTypeIdentifierHeartRate",
    time_period="last_90_days",
    trend_type="moving_average"
)
```

## Python API Usage

While primarily an MCP server, you can use the underlying modules directly:

```python
from apple_health_mcp_server.parser import AppleHealthParser
from apple_health_mcp_server.storage.duckdb import DuckDBStorage

# Parse Apple Health XML
parser = AppleHealthParser("/path/to/export.xml")
records = parser.parse()

# Store in DuckDB
storage = DuckDBStorage("./health.db")
storage.import_records(records)

# Query directly
results = storage.query(
    record_type="HKQuantityTypeIdentifierStepCount",
    start_date="2024-01-01",
    end_date="2024-12-31"
)
```

### Custom Query Execution

```python
from apple_health_mcp_server.storage.duckdb import DuckDBStorage

storage = DuckDBStorage("./health.db")

# Execute raw SQL
sql = """
    SELECT 
        DATE(start_date) as date,
        SUM(CAST(value AS DOUBLE)) as total_steps
    FROM health_records
    WHERE type = 'HKQuantityTypeIdentifierStepCount'
    AND start_date >= '2024-01-01'
    GROUP BY DATE(start_date)
    ORDER BY date
"""
results = storage.execute_query(sql)
```

## Common Health Record Types

Key Apple Health record type identifiers:

- **Activity:** `HKQuantityTypeIdentifierStepCount`, `HKQuantityTypeIdentifierDistanceWalkingRunning`
- **Heart:** `HKQuantityTypeIdentifierHeartRate`, `HKQuantityTypeIdentifierRestingHeartRate`
- **Sleep:** `HKCategoryTypeIdentifierSleepAnalysis`
- **Workouts:** `HKWorkoutTypeIdentifier`
- **Body:** `HKQuantityTypeIdentifierBodyMass`, `HKQuantityTypeIdentifierHeight`
- **Vitals:** `HKQuantityTypeIdentifierBloodPressureSystolic`, `HKQuantityTypeIdentifierOxygenSaturation`

## Configuration Options

### Storage Backends

**DuckDB (default - local file):**
```bash
STORAGE_BACKEND=duckdb
DUCKDB_PATH=./data/health.db
```

**Elasticsearch (scalable search):**
```bash
STORAGE_BACKEND=elasticsearch
ELASTICSEARCH_URL=http://localhost:9200
ELASTICSEARCH_INDEX=health_records
ELASTICSEARCH_API_KEY=${ELASTICSEARCH_API_KEY}
```

**ClickHouse (analytics):**
```bash
STORAGE_BACKEND=clickhouse
CLICKHOUSE_HOST=localhost
CLICKHOUSE_PORT=9000
CLICKHOUSE_DATABASE=health
CLICKHOUSE_USER=${CLICKHOUSE_USER}
CLICKHOUSE_PASSWORD=${CLICKHOUSE_PASSWORD}
```

### Performance Tuning

```bash
# Batch size for imports
IMPORT_BATCH_SIZE=10000

# Query result limits
DEFAULT_QUERY_LIMIT=100
MAX_QUERY_LIMIT=10000

# Connection pooling
DB_POOL_SIZE=5
```

## Troubleshooting

### Issue: Import fails with "File not found"

**Solution:** Verify the XML path and ensure the export is unzipped:

```bash
# Check file exists
ls -lh "$APPLE_HEALTH_EXPORT_PATH"

# Export should be in: export_cda.xml or export.xml
unzip export.zip -d ./export
export APPLE_HEALTH_EXPORT_PATH=./export/apple_health_export/export.xml
```

### Issue: MCP server not connecting

**Solution:** Check MCP client logs and verify configuration:

```bash
# Test server directly
python -m apple_health_mcp_server

# Check environment variables
env | grep APPLE_HEALTH

# Validate JSON configuration
cat ~/.config/Claude/claude_desktop_config.json | jq .
```

### Issue: Slow queries on large datasets

**Solution:** Switch to Elasticsearch or ClickHouse for better performance:

```bash
# Use ClickHouse for analytics workloads
STORAGE_BACKEND=clickhouse

# Or Elasticsearch for full-text search
STORAGE_BACKEND=elasticsearch

# Index optimization
CREATE INDEX idx_type_date ON health_records(type, start_date);
```

### Issue: Memory errors during import

**Solution:** Reduce batch size and use streaming:

```bash
# Reduce batch size
IMPORT_BATCH_SIZE=5000

# Use Docker with memory limits
docker run --memory=2g apple-health-mcp-server
```

### Issue: Missing data after import

**Solution:** Verify XML structure and check parser logs:

```python
from apple_health_mcp_server.parser import AppleHealthParser

parser = AppleHealthParser("/path/to/export.xml")
parser.validate()  # Check for parsing errors

# Check what was imported
from apple_health_mcp_server.storage.duckdb import DuckDBStorage
storage = DuckDBStorage("./health.db")
print(storage.get_record_type_counts())
```

### Issue: Date filtering not working

**Solution:** Ensure ISO 8601 date format (YYYY-MM-DD):

```python
# Correct format
search_records(
    record_type="HKQuantityTypeIdentifierStepCount",
    start_date="2024-01-01",  # ISO format
    end_date="2024-12-31"
)

# Incorrect: "01/01/2024", "2024-1-1"
```

## Advanced Usage

### Custom Tool Extension

```python
from apple_health_mcp_server.server import app
from fastmcp import Context

@app.tool()
async def custom_health_analysis(
    ctx: Context,
    metric: str,
    threshold: float
) -> str:
    """Custom analysis tool."""
    storage = ctx.app_context["storage"]
    
    results = storage.query(
        record_type=metric,
        filters={"value": {"$gte": threshold}}
    )
    
    return f"Found {len(results)} records above threshold"
```

### Batch Data Export

```python
from apple_health_mcp_server.storage.duckdb import DuckDBStorage
import pandas as pd

storage = DuckDBStorage("./health.db")

# Export to CSV
df = pd.DataFrame(storage.query(
    record_type="HKQuantityTypeIdentifierStepCount"
))
df.to_csv("steps_export.csv", index=False)

# Export to Parquet for efficient storage
df.to_parquet("steps_export.parquet")
```

## Related Projects

- **[Open Wearables](https://github.com/the-momentum/open-wearables)** - Evolution of this project with continuous sync support
- **[FastMCP](https://github.com/jlowin/fastmcp)** - Framework used to build this MCP server

## Resources

- [Apple Health XML Schema](https://developer.apple.com/documentation/healthkit)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Project Documentation](https://github.com/the-momentum/apple-health-mcp-server/tree/main/docs)
