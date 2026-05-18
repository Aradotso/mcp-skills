---
name: mcp-server-12306
description: MCP server for querying China Railway 12306 train tickets, stations, transfers, and schedules in real-time
triggers:
  - how do I query train tickets in China
  - set up 12306 MCP server for train information
  - search for railway stations by name or pinyin
  - find train transfer connections between cities
  - check train ticket prices and availability
  - get train route and schedule information
  - query China railway data through MCP
  - integrate 12306 train queries with AI assistant
---

# MCP Server 12306

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

MCP Server 12306 is a Model Context Protocol server that provides real-time access to China Railway 12306 data including train tickets, station information, transfers, prices, and schedules. Built with FastAPI for high-performance async operations.

## What It Does

This MCP server exposes tools for:

- **Real-time ticket queries**: Search available tickets with seat types, departure/arrival times
- **Station search**: Fuzzy search for stations by Chinese name, pinyin, or abbreviation
- **Transfer planning**: Find one-transfer routes between cities
- **Price queries**: Get real-time ticket prices for trains
- **Route information**: Query train stops and schedules
- **Time utilities**: Get current time and relative dates for booking

## Installation

### For Claude Desktop (Stdio Mode - Recommended)

Add to your Claude Desktop configuration file:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

#### Using uvx (fastest):

```json
{
  "mcpServers": {
    "12306": {
      "command": "uvx",
      "args": ["mcp-server-12306"]
    }
  }
}
```

#### Using pipx:

```json
{
  "mcpServers": {
    "12306": {
      "command": "pipx",
      "args": ["run", "--no-cache", "mcp-server-12306"]
    }
  }
}
```

#### From source (for development):

```json
{
  "mcpServers": {
    "12306": {
      "command": "uv",
      "args": ["run", "python", "-m", "mcp_12306.cli"],
      "cwd": "/path/to/mcp-server-12306"
    }
  }
}
```

### For Remote/HTTP Mode (Streamable HTTP)

#### Local development:

```bash
git clone https://github.com/drfccv/mcp-server-12306.git
cd mcp-server-12306
uv sync
uv run python scripts/start_server.py
```

Then configure your MCP client:

```json
{
  "mcpServers": {
    "12306": {
      "url": "http://localhost:8000/mcp"
    }
  }
}
```

#### Docker deployment:

```bash
docker run -d -p 8000:8000 --name mcp-server-12306 drfccv/mcp-server-12306:latest
```

## Available Tools

### 1. query_tickets

Search for available train tickets between stations.

**Parameters:**
- `from_station` (string, required): Departure station name (Chinese or pinyin)
- `to_station` (string, required): Arrival station name (Chinese or pinyin)
- `train_date` (string, required): Travel date in YYYY-MM-DD format
- `purpose_codes` (string, optional): "ADULT" (default) or "0X00" for student tickets

**Example usage:**

```python
# Search tickets from Beijing to Shanghai on a specific date
result = await mcp_client.call_tool("query_tickets", {
    "from_station": "北京",
    "to_station": "上海",
    "train_date": "2025-06-15",
    "purpose_codes": "ADULT"
})
```

**Response includes:**
- Train number, type (G/D/K/T/Z)
- Departure/arrival times and duration
- Seat availability and types
- Station codes

### 2. query_ticket_price

Get real-time ticket prices for a specific train.

**Parameters:**
- `train_no` (string, required): Train number (e.g., "G101")
- `from_station_no` (string, required): Departure station code
- `to_station_no` (string, required): Arrival station code
- `seat_types` (string, required): Comma-separated seat type codes
- `train_date` (string, required): Travel date in YYYY-MM-DD format

**Example usage:**

```python
# Get prices for train G101
result = await mcp_client.call_tool("query_ticket_price", {
    "train_no": "G101",
    "from_station_no": "BJP",
    "to_station_no": "SHH",
    "seat_types": "9,M,O",
    "train_date": "2025-06-15"
})
```

**Response includes:**
- Price for each seat type
- Seat type codes and names
- Currency (CNY)

### 3. search_stations

Search for railway stations by name, pinyin, or abbreviation.

**Parameters:**
- `keyword` (string, required): Search term (Chinese, pinyin, or abbreviation)

**Example usage:**

```python
# Search stations containing "beijing"
result = await mcp_client.call_tool("search_stations", {
    "keyword": "beijing"
})

# Also works with Chinese
result = await mcp_client.call_tool("search_stations", {
    "keyword": "北京"
})
```

**Response includes:**
- Station name (Chinese)
- Station code
- Pinyin and abbreviation
- Match score

### 4. get_station_info

Get detailed information about a specific station.

**Parameters:**
- `station_name` (string, required): Exact station name or code

**Example usage:**

```python
result = await mcp_client.call_tool("get_station_info", {
    "station_name": "北京南"
})
```

### 5. query_transfer

Find one-transfer routes between cities (when no direct train available).

**Parameters:**
- `from_station` (string, required): Departure station
- `to_station` (string, required): Arrival station
- `train_date` (string, required): Travel date in YYYY-MM-DD format
- `purpose_codes` (string, optional): Ticket type

**Example usage:**

```python
# Find transfer routes from Beijing to Guilin
result = await mcp_client.call_tool("query_transfer", {
    "from_station": "北京",
    "to_station": "桂林",
    "train_date": "2025-06-20",
    "purpose_codes": "ADULT"
})
```

**Response includes:**
- First leg train details
- Transfer station
- Second leg train details
- Total duration
- Layover time at transfer station

### 6. get_train_route_stations

Query all stops and schedule for a specific train.

**Parameters:**
- `train_no` (string, required): Train number
- `from_station_telecode` (string, required): Origin station code
- `to_station_telecode` (string, required): Destination station code
- `depart_date` (string, required): Departure date in YYYY-MM-DD format

**Example usage:**

```python
result = await mcp_client.call_tool("get_train_route_stations", {
    "train_no": "G101",
    "from_station_telecode": "BJP",
    "to_station_telecode": "SHH",
    "depart_date": "2025-06-15"
})
```

**Response includes:**
- Station sequence
- Arrival/departure times
- Stop duration
- Running distance

### 7. get_current_time

Get current time and calculate relative dates for booking.

**Parameters:**
- `timezone` (string, optional): Timezone (default: "Asia/Shanghai")

**Example usage:**

```python
result = await mcp_client.call_tool("get_current_time", {
    "timezone": "Asia/Shanghai"
})
```

**Response includes:**
- Current time
- ISO format timestamp
- Unix timestamp
- Formatted date strings
- Relative dates (today, tomorrow, next week)

## Common Patterns

### Complete ticket search workflow:

```python
# 1. Get current time to determine valid booking dates
time_info = await client.call_tool("get_current_time", {})
travel_date = time_info["dates"]["tomorrow"]

# 2. Search for station codes if needed
stations = await client.call_tool("search_stations", {
    "keyword": "beijing"
})
from_code = stations[0]["code"]

stations = await client.call_tool("search_stations", {
    "keyword": "shanghai"
})
to_code = stations[0]["code"]

# 3. Query available tickets
tickets = await client.call_tool("query_tickets", {
    "from_station": "北京",
    "to_station": "上海",
    "train_date": travel_date,
    "purpose_codes": "ADULT"
})

# 4. Get price for specific train
if tickets["trains"]:
    train = tickets["trains"][0]
    prices = await client.call_tool("query_ticket_price", {
        "train_no": train["station_train_code"],
        "from_station_no": train["from_station_telecode"],
        "to_station_no": train["to_station_telecode"],
        "seat_types": train["seat_types"],
        "train_date": travel_date
    })
```

### Finding transfer routes:

```python
# When no direct trains available, query transfers
transfers = await client.call_tool("query_transfer", {
    "from_station": "拉萨",
    "to_station": "厦门",
    "train_date": "2025-07-01",
    "purpose_codes": "ADULT"
})

# Each result shows:
# - First train to transfer city
# - Layover duration
# - Second train to final destination
for route in transfers["transfer_routes"]:
    print(f"Transfer at: {route['transfer_station']}")
    print(f"Layover: {route['layover_minutes']} minutes")
    print(f"Total duration: {route['total_duration']}")
```

### Checking train route details:

```python
# Get complete route for a train
route = await client.call_tool("get_train_route_stations", {
    "train_no": "G7",
    "from_station_telecode": "BJP",
    "to_station_telecode": "SHH",
    "depart_date": "2025-06-15"
})

# Shows all stops with arrival/departure times
for stop in route["stations"]:
    print(f"{stop['station_name']}: {stop['arrive_time']} - {stop['start_time']}")
```

## Configuration

### Environment Variables

The server uses default 12306 public APIs and doesn't require API keys. However, you can configure:

- **Port** (HTTP mode only): Set via `PORT` environment variable (default: 8000)
- **Timezone**: Pass to `get_current_time` tool (default: "Asia/Shanghai")

### Station Data

The server includes a pre-built station database. To update:

```bash
cd scripts
uv run python update_station_data.py
```

This fetches the latest station list from 12306 and updates `src/mcp_12306/data/station_names.json`.

## Project Structure

```
src/mcp_12306/
├── server.py           # FastAPI main entry point
├── services/
│   ├── ticket_service.py    # Ticket query logic
│   ├── station_service.py   # Station search logic
│   └── http_service.py      # 12306 API client
├── utils/
│   ├── config.py           # Configuration
│   └── logger.py           # Logging
├── data/
│   └── station_names.json  # Station database
scripts/
├── start_server.py         # HTTP mode launcher
└── update_station_data.py  # Station data updater
```

## Troubleshooting

### "Station not found" errors

The station database uses exact matching. Use `search_stations` first to find the correct station name:

```python
# Search before querying
results = await client.call_tool("search_stations", {
    "keyword": "beijing"
})
# Use the exact name from results
station_name = results[0]["name"]
```

### No tickets available

- Check date format is YYYY-MM-DD
- 12306 typically allows booking 15-30 days in advance
- Use `get_current_time` to get valid booking dates
- Try different dates or use `query_transfer` for indirect routes

### HTTP connection errors

- Ensure the server is running (`uv run python scripts/start_server.py`)
- Check port 8000 is not in use
- For Docker, verify port mapping: `-p 8000:8000`
- In China, direct 12306 API access may require proper network access

### Stdio mode not working in Claude Desktop

- Restart Claude Desktop after configuration changes
- Verify JSON syntax in config file
- Check `uvx` or `pipx` is installed: `uvx --version`
- For source mode, ensure `cwd` points to correct directory

### Rate limiting

The 12306 API may rate-limit requests. The server includes basic retry logic, but for production use:

- Add delays between requests
- Cache frequently accessed data (stations, routes)
- Use transfer queries sparingly (they make multiple API calls)

## Best Practices

1. **Always get current time first**: Use `get_current_time` to determine valid booking dates
2. **Search stations before querying**: Verify exact station names with `search_stations`
3. **Check direct routes first**: Try `query_tickets` before `query_transfer`
4. **Cache station data**: Station codes rarely change, cache `get_station_info` results
5. **Handle no-results gracefully**: Not all routes have direct or one-transfer options
6. **Use appropriate date formats**: Always use YYYY-MM-DD for dates

## Legal Notice

This project is for educational and research purposes only. It queries publicly available 12306 APIs and does not store, modify, or distribute official data. Users must comply with Chinese laws and 12306 terms of service. The authors assume no liability for misuse.
