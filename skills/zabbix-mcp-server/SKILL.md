---
name: zabbix-mcp-server
description: Complete MCP server for Zabbix integration providing AI assistants access to 100% of Zabbix APIs through 3 unified tools
triggers:
  - connect to zabbix monitoring
  - query zabbix hosts and problems
  - integrate zabbix with AI assistant
  - access zabbix api through mcp
  - monitor infrastructure with zabbix
  - get zabbix metrics and alerts
  - configure zabbix mcp server
  - read zabbix monitoring data
---

# Zabbix MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A lightweight Model Context Protocol (MCP) server that provides complete access to the entire Zabbix API (100+ methods) through just 3 tools. Compatible with Zabbix 6.0+.

## What It Does

The Zabbix MCP Server bridges AI assistants with Zabbix monitoring infrastructure, enabling:
- Query hosts, items, triggers, templates, problems, and metrics
- Create and manage monitoring configurations
- Access historical data and analytics
- Control Zabbix through natural language
- Minimal LLM context usage (3 tools vs 50+ individual endpoints)

## Installation

### Quick Install with uvx (Recommended)

```bash
# Install from GitHub
uvx --from git+https://github.com/mpeirone/zabbix-mcp-server@main zabbix-mcp
```

### Clone and Install with uv

```bash
git clone https://github.com/mpeirone/zabbix-mcp-server.git
cd zabbix-mcp-server
uv sync
```

### Docker Installation

```bash
git clone https://github.com/mpeirone/zabbix-mcp-server.git
cd zabbix-mcp-server
docker compose up -d
```

## Configuration

### Environment Variables

**Required:**
```bash
export ZABBIX_URL=https://your-zabbix-server.com
```

**Authentication (choose one):**
```bash
# API Token (recommended)
export ZABBIX_TOKEN=${ZABBIX_API_TOKEN}

# Or username/password
export ZABBIX_USER=${ZABBIX_USERNAME}
export ZABBIX_PASSWORD=${ZABBIX_PASSWORD}
```

**Security:**
```bash
export READ_ONLY=true  # Restrict to read operations only
export VERIFY_SSL=true  # Enable SSL verification
export ZABBIX_API_WHITELIST="host\..*,item\.get,problem\..*"  # Allow only specific methods
export ZABBIX_API_BLACKLIST=".*\.delete,.*\.create"  # Block specific methods
export ZABBIX_API_TIMEOUT=30  # API request timeout in seconds
```

**Transport:**
```bash
export ZABBIX_MCP_TRANSPORT=stdio  # or streamable-http
export ZABBIX_MCP_HOST=127.0.0.1  # HTTP server host
export ZABBIX_MCP_PORT=8000  # HTTP server port
```

**Debug:**
```bash
export DEBUG=true  # Enable verbose logging
```

### MCP Client Configuration

**Claude Desktop (macOS):**

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "zabbix": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/mpeirone/zabbix-mcp-server@main",
        "zabbix-mcp"
      ],
      "env": {
        "ZABBIX_URL": "https://zabbix.example.com",
        "ZABBIX_TOKEN": "${ZABBIX_API_TOKEN}",
        "READ_ONLY": "true"
      }
    }
  }
}
```

**Claude Desktop (Windows):**

Edit `%APPDATA%\Claude\claude_desktop_config.json` with the same configuration.

**Claude Code Integration:**

```bash
claude mcp add zabbix \
  --env ZABBIX_URL=https://your-zabbix-server.com \
  --env ZABBIX_TOKEN=${ZABBIX_API_TOKEN} \
  --env READ_ONLY=true \
  -- uvx --from git+https://github.com/mpeirone/zabbix-mcp-server@main zabbix-mcp
```

## The 3 Core Tools

### 1. zabbix_api - Execute Any Zabbix API Method

The main tool for interacting with Zabbix. Supports all 100+ API methods.

**Signature:**
```python
zabbix_api(method: str, params: dict = {}) -> dict
```

### 2. zabbix_api_docs - Get API Documentation

Retrieve documentation for any Zabbix API method.

**Signature:**
```python
zabbix_api_docs(method: str) -> str
```

### 3. zabbix_api_list - Discover Available Methods

List all available API objects and their methods.

**Signature:**
```python
zabbix_api_list(object: str = None) -> list
```

## Common Usage Patterns

### Querying Hosts

**Get all hosts with basic info:**
```python
result = zabbix_api(
    method='host.get',
    params={
        'output': ['hostid', 'host', 'name', 'status'],
        'selectInterfaces': ['interfaceid', 'ip']
    }
)
```

**Get hosts in specific group:**
```python
result = zabbix_api(
    method='host.get',
    params={
        'output': 'extend',
        'groupids': '2',
        'selectGroups': 'extend'
    }
)
```

**Search hosts by name:**
```python
result = zabbix_api(
    method='host.get',
    params={
        'output': ['hostid', 'host', 'name'],
        'search': {'name': 'web-server'},
        'searchWildcardsEnabled': True
    }
)
```

### Monitoring Problems and Triggers

**Get recent problems:**
```python
result = zabbix_api(
    method='problem.get',
    params={
        'output': 'extend',
        'recent': True,
        'sortfield': ['eventid'],
        'sortorder': 'DESC'
    }
)
```

**Get active triggers:**
```python
result = zabbix_api(
    method='trigger.get',
    params={
        'output': 'extend',
        'filter': {'value': 1},  # Problem state
        'skipDependent': 1,
        'monitored': True,
        'active': True,
        'selectHosts': ['hostid', 'host']
    }
)
```

**Get problems for specific host:**
```python
result = zabbix_api(
    method='problem.get',
    params={
        'output': 'extend',
        'hostids': '10084',
        'recent': True,
        'selectAcknowledges': 'extend'
    }
)
```

### Working with Items and Metrics

**Get items for a host:**
```python
result = zabbix_api(
    method='item.get',
    params={
        'output': 'extend',
        'hostids': '10084',
        'sortfield': 'name'
    }
)
```

**Get historical data:**
```python
result = zabbix_api(
    method='history.get',
    params={
        'output': 'extend',
        'history': 0,  # Float values
        'itemids': '23296',
        'sortfield': 'clock',
        'sortorder': 'DESC',
        'limit': 100
    }
)
```

**Get latest values:**
```python
result = zabbix_api(
    method='item.get',
    params={
        'output': ['itemid', 'name', 'lastvalue', 'lastclock'],
        'hostids': '10084',
        'monitored': True
    }
)
```

### Creating and Managing Resources

**Create a new host:**
```python
result = zabbix_api(
    method='host.create',
    params={
        'host': 'server-01.example.com',
        'name': 'Production Server 01',
        'groups': [{'groupid': '2'}],
        'interfaces': [{
            'type': 1,  # Agent
            'main': 1,
            'useip': 1,
            'ip': '192.168.1.100',
            'dns': '',
            'port': '10050'
        }]
    }
)
```

**Create a trigger:**
```python
result = zabbix_api(
    method='trigger.create',
    params={
        'description': 'CPU load too high on {HOST.NAME}',
        'expression': 'last(/server-01/system.cpu.load[percpu,avg1])>5',
        'priority': 3  # Average
    }
)
```

**Create an item:**
```python
result = zabbix_api(
    method='item.create',
    params={
        'name': 'CPU Load',
        'key_': 'system.cpu.load[percpu,avg1]',
        'hostid': '10084',
        'type': 0,  # Zabbix agent
        'value_type': 0,  # Float
        'delay': '60s'
    }
)
```

### Templates and Groups

**Get templates:**
```python
result = zabbix_api(
    method='template.get',
    params={
        'output': ['templateid', 'name'],
        'selectGroups': 'extend'
    }
)
```

**Link template to host:**
```python
result = zabbix_api(
    method='host.update',
    params={
        'hostid': '10084',
        'templates': [{'templateid': '10001'}]
    }
)
```

**Get host groups:**
```python
result = zabbix_api(
    method='hostgroup.get',
    params={
        'output': 'extend',
        'selectHosts': ['hostid', 'host']
    }
)
```

### Discovery and Documentation

**List all available API objects:**
```python
result = zabbix_api_list()
# Returns: ['action', 'alert', 'apiinfo', 'application', ...]
```

**List methods for specific object:**
```python
result = zabbix_api_list(object='host')
# Returns: ['host.create', 'host.delete', 'host.get', 'host.update', ...]
```

**Get documentation for a method:**
```python
docs = zabbix_api_docs(method='host.create')
# Returns: Detailed documentation with parameters and examples
```

**Get API version:**
```python
result = zabbix_api(method='apiinfo.version', params={})
# Returns: {'result': '7.0.0'}
```

## Security Patterns

### Read-Only Mode

For safe exploration and querying without modification risk:

```bash
export READ_ONLY=true
```

This allows only: `*.get`, `*.version`, `*.check`, `*.export` methods.

### Method Whitelisting

Allow only specific API methods:

```bash
# Only allow host and item queries
export ZABBIX_API_WHITELIST="host\.get,item\.get,problem\.get"
```

### Method Blacklisting

Block dangerous operations:

```bash
# Block all delete and update operations
export ZABBIX_API_BLACKLIST=".*\.delete,.*\.update,.*\.create"
```

### Combined Security

```bash
export READ_ONLY=true
export ZABBIX_API_WHITELIST="host\..*,item\..*,problem\..*,trigger\..*"
export ZABBIX_API_BLACKLIST="host\.delete,item\.delete"
export VERIFY_SSL=true
```

## Testing and Debugging

### Test Server Connection

```bash
uv run python scripts/test_server.py
```

### Start Server Manually

```bash
export ZABBIX_URL=https://zabbix.example.com
export ZABBIX_TOKEN=${ZABBIX_API_TOKEN}
export DEBUG=true
uv run python scripts/start_server.py
```

### Debug API Calls

Enable debug mode to see detailed request/response logs:

```bash
export DEBUG=true
```

### Test API Access

```python
# Test basic connectivity
result = zabbix_api(method='apiinfo.version', params={})

# Test authentication
result = zabbix_api(method='user.get', params={'output': ['userid', 'alias']})

# Test permissions
result = zabbix_api(method='host.get', params={'limit': 1})
```

## Troubleshooting

### Connection Errors

**Problem:** Cannot connect to Zabbix server

**Solutions:**
- Verify `ZABBIX_URL` is accessible: `curl -k $ZABBIX_URL`
- Check SSL verification: `export VERIFY_SSL=false` (not recommended for production)
- Ensure Zabbix API is enabled in server configuration

### Authentication Failures

**Problem:** Authentication error or invalid token

**Solutions:**
- Verify token is valid in Zabbix UI (Administration → API tokens)
- Check user permissions in Zabbix
- Test with username/password if token fails
- Ensure token hasn't expired

### Permission Denied

**Problem:** User has insufficient permissions

**Solutions:**
- Check Zabbix user role and permissions
- Verify user has API access enabled
- Check if `READ_ONLY=true` is blocking the operation
- Review user group permissions in Zabbix

### Method Blocked

**Problem:** "Method is not in whitelist" or "Method is blacklisted"

**Solutions:**
- Check `ZABBIX_API_WHITELIST` pattern: `echo $ZABBIX_API_WHITELIST`
- Check `ZABBIX_API_BLACKLIST` pattern: `echo $ZABBIX_API_BLACKLIST`
- Update patterns to match desired methods (use regex)
- Remove restrictions: `unset ZABBIX_API_WHITELIST ZABBIX_API_BLACKLIST`

### Invalid Parameters

**Problem:** API returns parameter errors

**Solutions:**
- Use `zabbix_api_docs(method='your.method')` to check parameters
- Verify required fields are provided
- Check parameter types (strings vs integers)
- Review Zabbix version compatibility (some parameters added in newer versions)

### Version Compatibility

**Problem:** Features not available or methods failing

**Solutions:**
- Check Zabbix version: `zabbix_api(method='apiinfo.version')`
- Verify Zabbix 6.0+ is running
- Review method documentation for version requirements
- Skip version check if needed: `export ZABBIX_SKIP_VERSION_CHECK=true`

## Advanced Patterns

### Batch Operations

**Get multiple object types in sequence:**
```python
# Get hosts
hosts = zabbix_api(method='host.get', params={'output': ['hostid', 'host']})

# For each host, get problems
for host in hosts['result']:
    problems = zabbix_api(
        method='problem.get',
        params={'hostids': host['hostid'], 'recent': True}
    )
```

### Complex Queries with Filtering

**Get hosts with specific conditions:**
```python
result = zabbix_api(
    method='host.get',
    params={
        'output': ['hostid', 'host', 'name', 'status'],
        'selectInterfaces': ['ip', 'dns', 'port'],
        'selectGroups': ['name'],
        'selectTriggers': ['triggerid', 'description', 'priority'],
        'filter': {'status': 0},  # Enabled hosts only
        'search': {'name': 'prod'},
        'searchWildcardsEnabled': True,
        'sortfield': 'name'
    }
)
```

### Error Handling Pattern

```python
try:
    result = zabbix_api(
        method='host.get',
        params={'output': 'extend'}
    )
    if 'error' in result:
        print(f"API Error: {result['error']}")
    else:
        hosts = result['result']
        print(f"Found {len(hosts)} hosts")
except Exception as e:
    print(f"Connection error: {e}")
```

### Dynamic Method Discovery

```python
# Find all available trigger methods
methods = zabbix_api_list(object='trigger')
print("Available trigger methods:", methods)

# Get docs for each
for method in methods:
    docs = zabbix_api_docs(method=method)
    print(f"\n{method}:\n{docs}")
```

## Real-World Examples

### Dashboard Data Collection

```python
# Get overview for monitoring dashboard
hosts = zabbix_api(method='host.get', params={'output': ['hostid', 'host', 'status']})
problems = zabbix_api(method='problem.get', params={'recent': True, 'output': 'extend'})
triggers = zabbix_api(method='trigger.get', params={'filter': {'value': 1}, 'output': 'extend'})

print(f"Total Hosts: {len(hosts['result'])}")
print(f"Active Problems: {len(problems['result'])}")
print(f"Triggered Alerts: {len(triggers['result'])}")
```

### Host Inventory Report

```python
# Generate host inventory with interfaces
hosts = zabbix_api(
    method='host.get',
    params={
        'output': ['hostid', 'host', 'name'],
        'selectInterfaces': ['ip', 'dns', 'type', 'port'],
        'selectGroups': ['name'],
        'selectParentTemplates': ['name']
    }
)

for host in hosts['result']:
    print(f"\nHost: {host['name']}")
    print(f"  Hostname: {host['host']}")
    print(f"  Groups: {', '.join([g['name'] for g in host['groups']])}")
    print(f"  Interfaces:")
    for iface in host['interfaces']:
        print(f"    - {iface['ip']}:{iface['port']} (type: {iface['type']})")
```

### Problem Alert System

```python
# Get critical problems with host details
problems = zabbix_api(
    method='problem.get',
    params={
        'output': 'extend',
        'selectAcknowledges': 'extend',
        'selectTags': 'extend',
        'severities': [4, 5],  # High and Disaster only
        'recent': True,
        'sortfield': ['severity', 'clock'],
        'sortorder': 'DESC'
    }
)

for problem in problems['result']:
    print(f"[{problem['severity']}] {problem['name']}")
    print(f"  Time: {problem['clock']}")
    print(f"  Acknowledged: {'Yes' if problem['acknowledged'] == '1' else 'No'}")
```
