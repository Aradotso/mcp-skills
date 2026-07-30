---
name: unifi-mcp-server
description: MCP server for UniFi Network Controller API - manage devices, clients, networks, WiFi, firewall rules, QoS, backups, and multi-site deployments
triggers:
  - manage my UniFi network
  - configure UniFi devices
  - set up UniFi firewall rules
  - create UniFi WiFi networks
  - monitor UniFi clients
  - backup UniFi configuration
  - manage UniFi QoS policies
  - configure UniFi VLANs
---

# unifi-mcp-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server exposing the UniFi Network Controller API. Provides 74+ tools for managing UniFi devices, clients, networks, WiFi, firewall rules, QoS, backups, port profiles, RADIUS, guest portals, and multi-site orchestration.

## What It Does

- **Device Management**: List, monitor, restart, locate, upgrade UniFi devices (APs, switches, gateways)
- **Network Configuration**: Create/update/delete networks, VLANs, subnets with DHCP
- **Client Management**: Query, block, unblock, reconnect clients with analytics
- **WiFi/SSID Management**: Create wireless networks with WPA2/WPA3, guest networks, VLAN isolation
- **Firewall Rules**: Create/manage firewall rules, port forwarding, traffic filtering
- **QoS Management**: Traffic prioritization, bandwidth control, traffic shaping
- **Backup & Restore**: Automated scheduling, verification, download/upload
- **Port Profiles**: Switch port configuration with PoE, VLAN trunking, 802.1X
- **RADIUS & Guest Portal**: 802.1X authentication, hotspot packages
- **Multi-Site**: Cross-site aggregation, analytics, site provisioning
- **Network Topology**: Complete topology mapping and visualization

## Installation

### Via pip (Recommended)

```bash
pip install unifi-mcp-server
```

### Via uv

```bash
uv pip install unifi-mcp-server
```

### From source

```bash
git clone https://github.com/enuno/unifi-mcp-server.git
cd unifi-mcp-server
pip install -e .
```

## API Modes

Three distinct API modes with different capabilities:

### Local Gateway API (Recommended) ✅

Full feature support with real-time data and configuration changes.

**Required Environment Variables:**
```bash
export UNIFI_API_TYPE=local
export UNIFI_LOCAL_HOST=192.168.2.1  # Your gateway IP
export UNIFI_USERNAME=admin
export UNIFI_PASSWORD=your_password
export UNIFI_SITE=default
```

### Cloud Early Access API ⚠️

Site-centric access with limited read-only capabilities.

**Required Environment Variables:**
```bash
export UNIFI_API_TYPE=cloud-ea
export UNIFI_API_KEY=your_cloud_api_key
export UNIFI_SITE_MANAGER_ENABLED=true  # Optional: multi-site aggregation
```

### Cloud V1 API ⚠️

Limited to aggregate statistics only.

**Required Environment Variables:**
```bash
export UNIFI_API_TYPE=cloud-v1
export UNIFI_API_KEY=your_cloud_api_key
```

## Transport Modes

### STDIO (Default)

For local AI clients (Claude Desktop, Cursor):

```bash
export MCP_SERVER_TRANSPORT=stdio
unifi-mcp-server
```

### SSE (Server-Sent Events)

For network access and MCP gateways:

```bash
export MCP_SERVER_TRANSPORT=sse
export MCP_SERVER_PORT=3000
unifi-mcp-server
```

### HTTP / Streamable HTTP

```bash
export MCP_SERVER_TRANSPORT=http  # or streamable_http
export MCP_SERVER_PORT=3000
unifi-mcp-server
```

## Configuration

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "unifi": {
      "command": "unifi-mcp-server",
      "env": {
        "UNIFI_API_TYPE": "local",
        "UNIFI_LOCAL_HOST": "192.168.2.1",
        "UNIFI_USERNAME": "admin",
        "UNIFI_PASSWORD": "your_password",
        "UNIFI_SITE": "default",
        "MCP_SERVER_TRANSPORT": "stdio"
      }
    }
  }
}
```

### MCP Gateway (SSE Mode)

```json
{
  "mcpServers": {
    "unifi": {
      "url": "http://your-server-ip:3000/sse"
    }
  }
}
```

### Docker Compose

```yaml
services:
  unifi-mcp:
    image: ghcr.io/enuno/unifi-mcp-server:latest
    environment:
      UNIFI_API_TYPE: local
      UNIFI_LOCAL_HOST: 192.168.2.1
      UNIFI_USERNAME: admin
      UNIFI_PASSWORD: ${UNIFI_PASSWORD}
      UNIFI_SITE: default
      MCP_SERVER_TRANSPORT: sse
      MCP_SERVER_PORT: 3000
    ports:
      - "3000:3000"
```

## Core MCP Tools

### Device Management

```python
# List all devices
devices = await call_tool("list_devices")

# Get device details
device = await call_tool("get_device", {"device_id": "abc123"})

# Restart device
result = await call_tool("restart_device", {"device_id": "abc123"})

# Locate device (blink LEDs)
await call_tool("locate_device", {
    "device_id": "abc123",
    "enabled": True
})

# Upgrade device firmware
await call_tool("upgrade_device", {"device_id": "abc123"})

# Force provision device
await call_tool("force_provision_device", {"device_id": "abc123"})
```

### Client Management

```python
# List all clients
clients = await call_tool("list_clients")

# Get client details
client = await call_tool("get_client", {"client_id": "aa:bb:cc:dd:ee:ff"})

# Block client
await call_tool("block_client", {
    "client_id": "aa:bb:cc:dd:ee:ff",
    "blocked": True
})

# Reconnect client
await call_tool("reconnect_client", {"client_id": "aa:bb:cc:dd:ee:ff"})

# Get client statistics
stats = await call_tool("get_client_stats", {"client_id": "aa:bb:cc:dd:ee:ff"})
```

### Network Configuration

```python
# Create network
network = await call_tool("create_network", {
    "name": "IoT Network",
    "vlan_id": 20,
    "subnet": "192.168.20.0/24",
    "dhcp_enabled": True,
    "dhcp_start": "192.168.20.10",
    "dhcp_stop": "192.168.20.250",
    "gateway": "192.168.20.1"
})

# Update network
await call_tool("update_network", {
    "network_id": "abc123",
    "name": "IoT Network Updated",
    "dhcp_dns": ["8.8.8.8", "8.8.4.4"]
})

# Delete network
await call_tool("delete_network", {"network_id": "abc123"})

# List networks
networks = await call_tool("list_networks")
```

### WiFi/SSID Management

```python
# Create SSID
ssid = await call_tool("create_ssid", {
    "name": "Guest WiFi",
    "password": "SecurePass123!",
    "security": "wpapsk",
    "vlan_id": 30,
    "guest_network": True,
    "hide_ssid": False
})

# Update SSID
await call_tool("update_ssid", {
    "ssid_id": "abc123",
    "name": "Guest WiFi Updated",
    "enabled": True
})

# Delete SSID
await call_tool("delete_ssid", {"ssid_id": "abc123"})

# List SSIDs
ssids = await call_tool("list_ssids")
```

### Firewall Rules

```python
# Create firewall rule
rule = await call_tool("create_firewall_rule", {
    "name": "Block IoT to LAN",
    "action": "drop",
    "rule_index": 2000,
    "protocol": "all",
    "src_network_id": "iot_network_id",
    "dst_network_id": "lan_network_id",
    "enabled": True
})

# Update firewall rule
await call_tool("update_firewall_rule", {
    "rule_id": "abc123",
    "enabled": False
})

# Delete firewall rule
await call_tool("delete_firewall_rule", {"rule_id": "abc123"})

# List firewall rules
rules = await call_tool("list_firewall_rules")
```

### Port Forwarding

```python
# Create port forward
forward = await call_tool("create_port_forward", {
    "name": "Web Server",
    "enabled": True,
    "protocol": "tcp",
    "src_port": "80",
    "dst_ip": "192.168.1.100",
    "dst_port": "8080",
    "log": True
})

# Update port forward
await call_tool("update_port_forward", {
    "forward_id": "abc123",
    "enabled": False
})

# Delete port forward
await call_tool("delete_port_forward", {"forward_id": "abc123"})
```

### QoS Management

```python
# Create QoS rule
qos = await call_tool("create_qos_rule", {
    "name": "Video Conference Priority",
    "enabled": True,
    "priority": "high",
    "bandwidth_limit": 10000,
    "protocol": "tcp",
    "dst_port": "443"
})

# Update QoS rule
await call_tool("update_qos_rule", {
    "rule_id": "abc123",
    "priority": "medium"
})

# Delete QoS rule
await call_tool("delete_qos_rule", {"rule_id": "abc123"})

# List QoS rules
rules = await call_tool("list_qos_rules")
```

### Port Profile Management

```python
# Create port profile
profile = await call_tool("create_port_profile", {
    "name": "VoIP Phones",
    "poe_mode": "auto",
    "vlan_id": 40,
    "native_network_id": "voice_network_id",
    "dot1x_ctrl": "auto",
    "lldpmed_enabled": True,
    "lldpmed_notify_enabled": True
})

# Update port profile
await call_tool("update_port_profile", {
    "profile_id": "abc123",
    "poe_mode": "pasv24"
})

# Delete port profile
await call_tool("delete_port_profile", {"profile_id": "abc123"})

# List port profiles
profiles = await call_tool("list_port_profiles")
```

### Device Port Overrides

```python
# Override device port
override = await call_tool("override_device_port", {
    "device_id": "switch123",
    "port_idx": 5,
    "port_profile_id": "voip_profile_id",
    "poe_mode": "auto",
    "name": "Conference Room Phone"
})

# Remove port override
await call_tool("remove_device_port_override", {
    "device_id": "switch123",
    "port_idx": 5
})

# Get device port configuration
config = await call_tool("get_device_port_config", {
    "device_id": "switch123",
    "port_idx": 5
})
```

### Backup & Restore

```python
# Create backup
backup = await call_tool("create_backup", {
    "name": "weekly-backup",
    "include_settings": True
})

# List backups
backups = await call_tool("list_backups")

# Download backup
content = await call_tool("download_backup", {"backup_id": "abc123"})

# Restore from backup
await call_tool("restore_backup", {"backup_id": "abc123"})

# Schedule backup
schedule = await call_tool("schedule_backup", {
    "frequency": "daily",
    "time": "02:00",
    "enabled": True
})
```

### RADIUS & Guest Portal

```python
# Create RADIUS account
account = await call_tool("create_radius_account", {
    "name": "employee1",
    "password": "SecurePass123!",
    "vlan_id": 10,
    "enabled": True
})

# Update RADIUS account
await call_tool("update_radius_account", {
    "account_id": "abc123",
    "enabled": False
})

# Create hotspot package
package = await call_tool("create_hotspot_package", {
    "name": "1 Hour Guest",
    "duration": 3600,
    "bandwidth_up": 5000,
    "bandwidth_down": 10000
})
```

### Multi-Site Management

```python
# List sites
sites = await call_tool("list_sites")

# Get site details
site = await call_tool("get_site", {"site_id": "default"})

# Create site
new_site = await call_tool("create_site", {
    "name": "Branch Office",
    "description": "Remote branch location"
})

# Get cross-site statistics
stats = await call_tool("get_cross_site_stats")
```

### Network Topology

```python
# Get network topology
topology = await call_tool("get_network_topology")

# Get topology map (visualization data)
map_data = await call_tool("get_topology_map")

# Get device connections
connections = await call_tool("get_device_connections", {
    "device_id": "abc123"
})

# Trace route between clients
trace = await call_tool("trace_route", {
    "source_client_id": "aa:bb:cc:dd:ee:ff",
    "dest_client_id": "11:22:33:44:55:66"
})
```

### DPI Statistics

```python
# Get DPI statistics
dpi_stats = await call_tool("get_dpi_stats", {
    "type": "by_app",
    "start_time": 1640000000,
    "end_time": 1640086400
})

# Get DPI categories
categories = await call_tool("get_dpi_categories")
```

## Common Patterns

### Creating a Complete Network Setup

```python
# 1. Create network
network = await call_tool("create_network", {
    "name": "IoT Network",
    "vlan_id": 20,
    "subnet": "192.168.20.0/24",
    "dhcp_enabled": True,
    "dhcp_start": "192.168.20.10",
    "dhcp_stop": "192.168.20.250"
})

# 2. Create WiFi for the network
ssid = await call_tool("create_ssid", {
    "name": "IoT WiFi",
    "password": "SecurePass123!",
    "security": "wpapsk",
    "vlan_id": 20
})

# 3. Create firewall rule to isolate
rule = await call_tool("create_firewall_rule", {
    "name": "Isolate IoT",
    "action": "drop",
    "rule_index": 2000,
    "protocol": "all",
    "src_network_id": network["_id"],
    "dst_network_id": "lan_network_id"
})

# 4. Create QoS rule for bandwidth limiting
qos = await call_tool("create_qos_rule", {
    "name": "IoT Bandwidth Limit",
    "enabled": True,
    "bandwidth_limit": 1000,
    "network_id": network["_id"]
})
```

### Managing Guest Access

```python
# 1. Create guest network
guest_net = await call_tool("create_network", {
    "name": "Guest Network",
    "vlan_id": 99,
    "subnet": "192.168.99.0/24",
    "dhcp_enabled": True,
    "dhcp_start": "192.168.99.10",
    "dhcp_stop": "192.168.99.250"
})

# 2. Create guest SSID
guest_ssid = await call_tool("create_ssid", {
    "name": "Guest WiFi",
    "password": "GuestPass123",
    "security": "wpapsk",
    "vlan_id": 99,
    "guest_network": True
})

# 3. Create hotspot package
package = await call_tool("create_hotspot_package", {
    "name": "Daily Guest",
    "duration": 86400,
    "bandwidth_up": 5000,
    "bandwidth_down": 10000
})

# 4. Isolate guest network
await call_tool("create_firewall_rule", {
    "name": "Isolate Guests",
    "action": "drop",
    "protocol": "all",
    "src_network_id": guest_net["_id"],
    "dst_network_id": "lan_network_id"
})
```

### Switch Port Configuration

```python
# 1. Create port profile for VoIP
voip_profile = await call_tool("create_port_profile", {
    "name": "VoIP Phones",
    "poe_mode": "auto",
    "vlan_id": 40,
    "dot1x_ctrl": "auto",
    "lldpmed_enabled": True
})

# 2. Apply to specific switch port
await call_tool("override_device_port", {
    "device_id": "switch123",
    "port_idx": 5,
    "port_profile_id": voip_profile["_id"],
    "name": "Conference Room Phone"
})

# 3. Verify configuration
config = await call_tool("get_device_port_config", {
    "device_id": "switch123",
    "port_idx": 5
})
```

### Monitoring and Analytics

```python
# 1. Get all device statistics
devices = await call_tool("list_devices")
for device in devices:
    stats = await call_tool("get_device_stats", {
        "device_id": device["_id"]
    })

# 2. Get client bandwidth usage
clients = await call_tool("list_clients")
for client in clients:
    stats = await call_tool("get_client_stats", {
        "client_id": client["mac"]
    })

# 3. Get DPI analytics
dpi_stats = await call_tool("get_dpi_stats", {
    "type": "by_app"
})

# 4. Get topology for visualization
topology = await call_tool("get_network_topology")
```

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to UniFi controller

**Solution:**
```bash
# Verify environment variables
echo $UNIFI_API_TYPE
echo $UNIFI_LOCAL_HOST
echo $UNIFI_USERNAME

# Test connectivity
ping $UNIFI_LOCAL_HOST

# Check credentials
curl -k https://$UNIFI_LOCAL_HOST:8443/api/login \
  -d "{\"username\":\"$UNIFI_USERNAME\",\"password\":\"$UNIFI_PASSWORD\"}"
```

### SSL Certificate Errors

**Problem:** SSL verification failures

**Solution:**
```bash
# Disable SSL verification (local only)
export UNIFI_VERIFY_SSL=false

# Or provide custom CA bundle
export UNIFI_SSL_CERT_PATH=/path/to/ca-bundle.crt
```

### API Rate Limiting

**Problem:** Rate limit exceeded errors

**Solution:**
```python
# Cloud-EA: 100 req/min, Cloud-V1: 10,000 req/min
# Use local API for high-frequency operations
export UNIFI_API_TYPE=local

# Add delays between bulk operations
import asyncio
for device in devices:
    await call_tool("restart_device", {"device_id": device["_id"]})
    await asyncio.sleep(1)  # 1 second delay
```

### Site Not Found

**Problem:** Site ID mismatch errors

**Solution:**
```python
# List all sites first
sites = await call_tool("list_sites")

# Find correct site ID
for site in sites:
    print(f"Site: {site['name']}, ID: {site['_id']}")

# Use correct site in environment
export UNIFI_SITE=correct_site_id
```

### Empty Response Data

**Problem:** Tools return empty arrays or None

**Solution:**
```bash
# Cloud APIs have limited visibility
# Switch to local API for full access
export UNIFI_API_TYPE=local
export UNIFI_LOCAL_HOST=192.168.2.1

# Verify site parameter
export UNIFI_SITE=default
```

### Transport Mode Issues

**Problem:** MCP client cannot connect in SSE mode

**Solution:**
```bash
# Verify server is listening
netstat -an | grep 3000

# Check firewall rules
sudo ufw allow 3000/tcp

# Test endpoint
curl http://localhost:3000/sse

# Use correct URL in gateway config
{
  "url": "http://server-ip:3000/sse"
}
```

## Advanced Configuration

### Environment Variables Reference

```bash
# API Configuration
UNIFI_API_TYPE=local|cloud-ea|cloud-v1
UNIFI_API_KEY=                    # Cloud API key
UNIFI_LOCAL_HOST=192.168.2.1     # Gateway IP (local mode)
UNIFI_USERNAME=admin              # Username (local mode)
UNIFI_PASSWORD=                   # Password (local mode)
UNIFI_SITE=default                # Site ID
UNIFI_PORT=8443                   # Controller port
UNIFI_VERIFY_SSL=true             # SSL verification
UNIFI_SSL_CERT_PATH=              # Custom CA bundle

# Transport Configuration
MCP_SERVER_TRANSPORT=stdio|sse|http|streamable_http
MCP_SERVER_PORT=3000              # Network transport port
MCP_SERVER_HOST=0.0.0.0           # Bind address

# Optional Features
UNIFI_SITE_MANAGER_ENABLED=true   # Multi-site aggregation
```

### Logging Configuration

```python
import logging

# Enable debug logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Filter specific loggers
logging.getLogger('unifi_mcp_server').setLevel(logging.DEBUG)
logging.getLogger('httpx').setLevel(logging.WARNING)
```

## Documentation References

- **Architecture**: See `SPEC.md` for architecture target
- **Roadmap**: See `DEVELOPMENT_PLAN.md` for phase roadmap
- **API Details**: See `API.md` and `docs/UNIFI_API.md`
- **Runbooks**: See `NETWORK_PLAYBOOK.md`, `HARBOR_SETUP.md`, `MULTI_CONTROLLER.md`, `METRICS.md`, `WEBHOOK_SETUP.md`, `A2A.md`
- **Releases**: See `RELEASE_CHECKLIST.md` and `docs/RELEASE_PROCESS.md`
