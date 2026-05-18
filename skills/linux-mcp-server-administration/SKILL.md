---
name: linux-mcp-server-administration
description: MCP server for read-only Linux system diagnostics and troubleshooting on RHEL-based systems via SSH
triggers:
  - diagnose Linux system issues remotely
  - check system logs and services on remote servers
  - troubleshoot RHEL server problems
  - inspect systemd services and processes
  - analyze Linux system performance and status
  - connect to remote Linux hosts for diagnostics
  - monitor network and storage on Linux systems
  - examine journal logs and system information
---

# Linux MCP Server Administration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The Linux MCP Server is a Model Context Protocol server that provides read-only diagnostic and troubleshooting capabilities for RHEL-based Linux systems. It enables LLM clients to execute diagnostic commands on local or remote systems via SSH, making it ideal for system administration, monitoring, and troubleshooting tasks.

**Key Features:**
- Strictly read-only operations for safe diagnostics
- Remote SSH execution with key-based authentication
- Multi-host management in a single session
- Comprehensive system diagnostics (services, processes, logs, network, storage)
- RHEL/systemd optimized

## Installation

### Using pip

```bash
pip install linux-mcp-server
```

### Using uv (recommended)

```bash
# Install uv first
pip install uv

# Install the server
uv pip install linux-mcp-server
```

### From source

```bash
git clone https://github.com/rhel-lightspeed/linux-mcp-server.git
cd linux-mcp-server
pip install -e .
```

## Configuration

### MCP Client Configuration

Add to your MCP client settings (e.g., Claude Desktop `config.json`):

```json
{
  "mcpServers": {
    "linux": {
      "command": "uv",
      "args": [
        "--directory",
        "/path/to/linux-mcp-server",
        "run",
        "linux-mcp-server"
      ],
      "env": {
        "LINUX_MCP_ALLOWED_LOGS": "/var/log/messages,/var/log/secure,/var/log/audit/audit.log"
      }
    }
  }
}
```

### Environment Variables

- `LINUX_MCP_ALLOWED_LOGS`: Comma-separated list of log files that can be accessed (default: common system logs)
- `SSH_AUTH_SOCK`: SSH agent socket for key-based authentication
- `HOME`: User home directory (for SSH key location)

### SSH Setup for Remote Hosts

Ensure SSH key-based authentication is configured:

```bash
# Generate SSH key if needed
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy public key to remote host
ssh-copy-id user@remote-host

# Test connection
ssh user@remote-host
```

## Available Tools

### System Information

**get_system_info** - Retrieve basic system information (hostname, OS, kernel, uptime)

```python
# Usage in MCP client
# Ask: "What's the system info for server01?"
# Tool call: get_system_info(host="user@server01")
```

**get_uname** - Get detailed kernel and system information

```python
# Usage: "Show me kernel details"
# Tool call: get_uname(host=None)  # Local system
```

### Service Management

**list_services** - List systemd services and their states

```python
# Usage: "List all services on the production server"
# Tool call: list_services(host="admin@prod-server", state="all")
# States: "all", "running", "failed", "enabled", "disabled"
```

**get_service_status** - Get detailed status of a specific service

```python
# Usage: "Check nginx status on web01"
# Tool call: get_service_status(service_name="nginx", host="root@web01")
```

### Process Management

**list_processes** - List running processes

```python
# Usage: "Show all processes"
# Tool call: list_processes(host=None)
```

**get_process_info** - Get detailed information about a specific process

```python
# Usage: "What's using PID 1234?"
# Tool call: get_process_info(pid=1234, host=None)
```

### Log Analysis

**read_journal** - Read systemd journal logs

```python
# Usage: "Show last 50 journal entries"
# Tool call: read_journal(lines=50, unit=None, priority=None, host=None)
# Priorities: "emerg", "alert", "crit", "err", "warning", "notice", "info", "debug"
```

**read_log_file** - Read specific log files

```python
# Usage: "Show last 100 lines of /var/log/messages"
# Tool call: read_log_file(log_path="/var/log/messages", lines=100, host=None)
# Note: Log must be in LINUX_MCP_ALLOWED_LOGS
```

**search_logs** - Search for patterns in logs

```python
# Usage: "Search for 'error' in system logs"
# Tool call: search_logs(pattern="error", log_path="/var/log/messages", host=None)
```

### Network Diagnostics

**get_network_info** - Get network interface information

```python
# Usage: "What are the network interfaces?"
# Tool call: get_network_info(host=None)
```

**get_listening_ports** - List listening network ports

```python
# Usage: "What ports are listening?"
# Tool call: get_listening_ports(host=None)
```

**test_connectivity** - Test network connectivity to a host

```python
# Usage: "Can we reach google.com?"
# Tool call: test_connectivity(target="google.com", host=None)
```

### Storage Diagnostics

**get_disk_usage** - Get filesystem disk usage

```python
# Usage: "Show disk usage"
# Tool call: get_disk_usage(host=None)
```

**get_mount_points** - List mounted filesystems

```python
# Usage: "What's mounted on the system?"
# Tool call: get_mount_points(host=None)
```

## Common Patterns

### Multi-Host Diagnostics

```python
# Check multiple hosts in sequence
# "Check service status on web01, web02, and db01"

# Tool calls:
# 1. get_service_status(service_name="httpd", host="admin@web01")
# 2. get_service_status(service_name="httpd", host="admin@web02")
# 3. get_service_status(service_name="postgresql", host="admin@db01")
```

### Service Troubleshooting Workflow

```python
# "Troubleshoot why nginx isn't working on web01"

# Typical tool sequence:
# 1. get_service_status(service_name="nginx", host="admin@web01")
# 2. read_journal(lines=100, unit="nginx", priority="err", host="admin@web01")
# 3. get_listening_ports(host="admin@web01")  # Check if port 80/443 open
# 4. test_connectivity(target="localhost:80", host="admin@web01")
```

### System Health Check

```python
# "Run a health check on server01"

# Tool sequence:
# 1. get_system_info(host="admin@server01")
# 2. list_services(host="admin@server01", state="failed")
# 3. get_disk_usage(host="admin@server01")
# 4. read_journal(lines=50, priority="err", host="admin@server01")
# 5. list_processes(host="admin@server01")
```

### Log Analysis for Errors

```python
# "Find authentication errors in the last hour"

# Tool sequence:
# 1. read_journal(lines=1000, priority="err", host=None)
# 2. search_logs(pattern="authentication failure", log_path="/var/log/secure", host=None)
# 3. read_log_file(log_path="/var/log/audit/audit.log", lines=500, host=None)
```

## Running the Server

### Standalone Mode

```bash
# Run locally
linux-mcp-server

# With custom log access
LINUX_MCP_ALLOWED_LOGS="/var/log/myapp.log,/var/log/custom.log" linux-mcp-server
```

### Development Mode

```bash
# Clone and install in editable mode
git clone https://github.com/rhel-lightspeed/linux-mcp-server.git
cd linux-mcp-server
pip install -e .

# Run with debugging
python -m linux_mcp_server
```

## Troubleshooting

### SSH Connection Issues

**Problem:** "Could not connect to remote host"

**Solutions:**
- Verify SSH key authentication: `ssh user@host`
- Check SSH agent is running: `echo $SSH_AUTH_SOCK`
- Start SSH agent: `eval $(ssh-agent)` and `ssh-add ~/.ssh/id_ed25519`
- Verify host in known_hosts: `ssh-keyscan host >> ~/.ssh/known_hosts`

### Log Access Denied

**Problem:** "Log file not in allowed list"

**Solution:**
- Add log path to `LINUX_MCP_ALLOWED_LOGS` environment variable
- Restart the MCP server after updating configuration

```json
{
  "env": {
    "LINUX_MCP_ALLOWED_LOGS": "/var/log/messages,/var/log/secure,/custom/app.log"
  }
}
```

### Permission Denied Errors

**Problem:** "Permission denied" when accessing system resources

**Solutions:**
- Use appropriate user with sudo privileges: `ssh admin@host` not `ssh user@host`
- Verify remote user has read access to logs and system commands
- For systemctl commands, ensure user can run `systemctl status`

### Tool Not Found

**Problem:** Command not found on remote system

**Solutions:**
- Verify RHEL/systemd-based system (tool is optimized for RHEL)
- Check required commands are installed: `systemctl`, `journalctl`, `ss`, `df`
- Ensure PATH includes `/usr/bin`, `/usr/sbin`

## Security Considerations

- **Read-only operations**: All tools are strictly read-only, no system modifications
- **SSH key authentication**: Use key-based auth, never passwords
- **Log access control**: Explicitly whitelist allowed log files
- **Principle of least privilege**: Use service accounts with minimal permissions
- **Audit logging**: All SSH connections are logged on remote systems

## Best Practices

1. **Use service accounts**: Create dedicated accounts for MCP diagnostics
2. **Limit log access**: Only whitelist logs necessary for diagnostics
3. **Test SSH first**: Always verify manual SSH connection before using MCP
4. **Monitor usage**: Review SSH logs to track diagnostic sessions
5. **Keep keys secure**: Use SSH agent, don't embed keys in config files
