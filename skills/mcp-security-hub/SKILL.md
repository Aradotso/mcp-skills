---
name: mcp-security-hub
description: Use FuzzingLabs MCP Security Hub to integrate offensive security tools (Nmap, Nuclei, SQLMap, Ghidra, etc.) with AI assistants via Docker-based MCP servers
triggers:
  - scan a network for vulnerabilities
  - analyze this binary with radare2
  - check for SQL injection vulnerabilities
  - find secrets in this git repository
  - fingerprint web technologies on this domain
  - perform security reconnaissance
  - audit cloud infrastructure for misconfigurations
  - fuzz this network protocol
---

# MCP Security Hub

> Skill by [ara.so](https://ara.so) — MCP Skills collection

## Overview

MCP Security Hub is a collection of 38 production-ready, Dockerized MCP (Model Context Protocol) servers that bring offensive security tools to AI assistants. It enables Claude and other MCP clients to perform security assessments, vulnerability scanning, binary analysis, and penetration testing through natural language interactions.

**Key capabilities:**
- 300+ security tools across 13 categories (reconnaissance, web security, binary analysis, blockchain, cloud, OSINT, etc.)
- Docker-based architecture with security hardening (non-root, capability dropping, read-only mounts)
- Natural language interface to complex security tools
- Multi-tool orchestration via Docker Compose
- CI/CD-ready with automated builds and Trivy scanning

## Installation

### Prerequisites

- Docker and Docker Compose installed
- Claude Desktop or another MCP client
- Git for cloning the repository

### Setup Steps

1. **Clone the repository:**

```bash
git clone https://github.com/FuzzingLabs/mcp-security-hub
cd mcp-security-hub
```

2. **Build all MCP servers:**

```bash
docker-compose build
```

Or build specific servers:

```bash
docker-compose build nmap-mcp nuclei-mcp sqlmap-mcp
```

3. **Configure Claude Desktop:**

Edit your Claude Desktop config file:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "nmap": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "--cap-add=NET_RAW", "nmap-mcp:latest"]
    },
    "nuclei": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "nuclei-mcp:latest"]
    },
    "sqlmap": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "sqlmap-mcp:latest"]
    },
    "gitleaks": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "-v", "${HOME}/projects:/app/target:ro", "gitleaks-mcp:latest"]
    },
    "radare2": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "-v", "${HOME}/binaries:/samples:ro", "radare2-mcp:latest"]
    }
  }
}
```

4. **Restart Claude Desktop** to load the new MCP servers.

## Key MCP Servers by Category

### Reconnaissance

**nmap-mcp** - Port scanning and service detection:
```bash
# Start the server
docker-compose up nmap-mcp -d

# Example prompts:
# "Scan 192.168.1.0/24 for open ports"
# "Perform service detection on 10.0.0.5"
# "Run aggressive scan with OS detection on example.com"
```

**whatweb-mcp** - Web technology fingerprinting:
```bash
docker-compose up whatweb-mcp -d

# Example prompts:
# "Identify technologies used on example.com"
# "Fingerprint the CMS on this website"
```

**masscan-mcp** - High-speed port scanning:
```bash
docker-compose up masscan-mcp -d

# Example prompts:
# "Fast scan ports 1-65535 on 10.0.0.0/16"
# "Scan for web servers across this entire subnet"
```

### Web Security

**nuclei-mcp** - Template-based vulnerability scanning:
```bash
docker-compose up nuclei-mcp -d

# Example prompts:
# "Scan example.com for CVEs and misconfigurations"
# "Check this site for exposed sensitive files"
# "Run nuclei templates for authentication bypass"
```

**sqlmap-mcp** - SQL injection testing:
```bash
docker-compose up sqlmap-mcp -d

# Example prompts:
# "Test https://example.com/page?id=1 for SQL injection"
# "Check if this login form is vulnerable to SQLi"
# "Enumerate databases on this vulnerable endpoint"
```

**ffuf-mcp** - Web fuzzing:
```bash
docker-compose up ffuf-mcp -d

# Example prompts:
# "Fuzz directories on example.com"
# "Find hidden API endpoints on this application"
# "Brute force parameter names for this URL"
```

### Binary Analysis

**radare2-mcp** - Reverse engineering (requires volume mount):
```bash
docker run -i --rm -v /path/to/binaries:/samples:ro radare2-mcp:latest

# Example prompts:
# "Disassemble /samples/malware.exe and find main function"
# "Analyze this binary for suspicious strings"
# "Decompile the authentication routine"
```

**binwalk-mcp** - Firmware analysis:
```bash
docker-compose up binwalk-mcp -d

# Example prompts:
# "Extract filesystem from this firmware image"
# "Scan for embedded files in this binary"
# "Analyze this router firmware for security issues"
```

**yara-mcp** - Malware pattern matching:
```bash
docker-compose up yara-mcp -d

# Example prompts:
# "Scan this file for malware signatures"
# "Check if this binary matches ransomware patterns"
```

### Secrets Detection

**gitleaks-mcp** - Find credentials in repos:
```bash
docker run -i --rm -v /path/to/repo:/app/target:ro gitleaks-mcp:latest

# Example prompts:
# "Scan this repository for hardcoded secrets"
# "Find API keys in the commit history"
# "Check for AWS credentials in the codebase"
```

### Cloud Security

**trivy-mcp** - Container and IaC scanning:
```bash
docker-compose up trivy-mcp -d

# Example prompts:
# "Scan this Docker image for vulnerabilities"
# "Audit my Terraform files for misconfigurations"
# "Check this Kubernetes manifest for security issues"
```

**prowler-mcp** - Cloud security auditing:
```bash
docker-compose up prowler-mcp -d

# Example prompts:
# "Audit my AWS account for security best practices"
# "Check Azure for compliance violations"
# "Scan GCP project for misconfigurations"
```

## Project Structure

```
mcp-security-hub/
├── reconnaissance/
│   ├── nmap-mcp/
│   ├── masscan-mcp/
│   ├── whatweb-mcp/
│   └── ...
├── web-security/
│   ├── nuclei-mcp/
│   ├── sqlmap-mcp/
│   ├── ffuf-mcp/
│   └── ...
├── binary-analysis/
│   ├── radare2-mcp/
│   ├── binwalk-mcp/
│   ├── yara-mcp/
│   └── ...
├── cloud-security/
│   ├── trivy-mcp/
│   ├── prowler-mcp/
│   └── ...
├── secrets/
│   └── gitleaks-mcp/
├── docker-compose.yml
└── examples/
    └── .mcp.json (full config template)
```

## Docker Compose Orchestration

Start multiple servers simultaneously:

```bash
# Reconnaissance stack
docker-compose up nmap-mcp whatweb-mcp masscan-mcp -d

# Web security stack
docker-compose up nuclei-mcp sqlmap-mcp ffuf-mcp -d

# Full suite
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f nmap-mcp

# Stop services
docker-compose down
```

## Common Usage Patterns

### Network Reconnaissance Workflow

```bash
# 1. Build required images
docker-compose build nmap-mcp whatweb-mcp

# 2. Start services
docker-compose up nmap-mcp whatweb-mcp -d

# 3. In Claude Desktop, use natural language:
# "Scan 192.168.1.0/24 for web servers, then fingerprint their technologies"
```

Claude will orchestrate:
1. nmap-mcp scans for ports 80, 443, 8080
2. whatweb-mcp fingerprints each discovered host
3. Consolidates results into a security assessment

### Web Application Assessment

```bash
# Build and start
docker-compose build nuclei-mcp sqlmap-mcp ffuf-mcp
docker-compose up nuclei-mcp sqlmap-mcp ffuf-mcp -d

# Prompt:
# "Assess example.com for vulnerabilities: scan for CVEs, test for SQL injection, and fuzz directories"
```

### Binary Analysis Pipeline

```bash
# Mount your binaries directory
docker run -i --rm -v /path/to/samples:/samples:ro radare2-mcp:latest &
docker-compose up binwalk-mcp yara-mcp capa-mcp -d

# Prompt:
# "Analyze /samples/suspicious.exe: extract strings, identify capabilities, and scan for malware"
```

### Secrets Scanning in CI/CD

```bash
# Scan repository
docker run -i --rm \
  -v $(pwd):/app/target:ro \
  gitleaks-mcp:latest

# In automation:
# "Scan the current repository for hardcoded secrets and API keys"
```

## Configuration Examples

### Volume Mounts for File Analysis

For servers that need to access files (radare2, binwalk, gitleaks):

```json
{
  "mcpServers": {
    "radare2": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-v", "/home/user/malware:/samples:ro",
        "radare2-mcp:latest"
      ]
    },
    "gitleaks": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-v", "/home/user/projects:/app/target:ro",
        "gitleaks-mcp:latest"
      ]
    }
  }
}
```

### Network Capabilities

For servers requiring raw socket access (nmap, masscan):

```json
{
  "mcpServers": {
    "nmap": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "--cap-add=NET_RAW",
        "nmap-mcp:latest"
      ]
    }
  }
}
```

### API-Based Servers

For servers requiring API keys (use environment variables):

```json
{
  "mcpServers": {
    "shodan": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-e", "SHODAN_API_KEY",
        "shodan-mcp:latest"
      ],
      "env": {
        "SHODAN_API_KEY": "${SHODAN_API_KEY}"
      }
    },
    "virustotal": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-e", "VT_API_KEY",
        "virustotal-mcp:latest"
      ],
      "env": {
        "VT_API_KEY": "${VT_API_KEY}"
      }
    }
  }
}
```

## Building Individual Servers

Each server can be built independently:

```bash
# Navigate to server directory
cd reconnaissance/nmap-mcp

# Build with Docker
docker build -t nmap-mcp:latest .

# Run directly
docker run -i --rm --cap-add=NET_RAW nmap-mcp:latest

# Or use docker-compose from root
cd ../..
docker-compose build nmap-mcp
docker-compose run --rm nmap-mcp
```

## Security Hardening Features

All containers implement:

1. **Non-root execution**: Runs as `mcpuser` (UID 1000)
2. **Capability dropping**: `cap_drop: ALL` with selective adds
3. **Read-only mounts**: Sample directories are `:ro`
4. **No new privileges**: `security_opt: no-new-privileges:true`
5. **Resource limits**: CPU and memory constraints in docker-compose
6. **Minimal base images**: Alpine/Debian slim
7. **Health checks**: Built-in container monitoring
8. **Vulnerability scanning**: Trivy scans in CI/CD

Example Dockerfile pattern:

```dockerfile
FROM python:3.11-alpine

# Create non-root user
RUN addgroup -g 1000 mcpuser && \
    adduser -D -u 1000 -G mcpuser mcpuser

# Install tool
RUN apk add --no-cache nmap nmap-scripts

# Switch to non-root
USER mcpuser
WORKDIR /app

# Copy MCP server
COPY --chown=mcpuser:mcpuser server.py .

CMD ["python", "server.py"]
```

## Troubleshooting

### MCP Server Not Appearing in Claude

1. **Verify build completed:**
   ```bash
   docker images | grep mcp
   ```

2. **Check Claude config syntax:**
   ```bash
   # Validate JSON
   cat ~/Library/Application\ Support/Claude/claude_desktop_config.json | python -m json.tool
   ```

3. **Restart Claude Desktop completely** (Quit, not just close window)

4. **Check Docker daemon is running:**
   ```bash
   docker ps
   ```

### Permission Denied Errors

For network scanning tools (nmap, masscan), add `NET_RAW` capability:

```json
{
  "command": "docker",
  "args": ["run", "-i", "--rm", "--cap-add=NET_RAW", "nmap-mcp:latest"]
}
```

### Volume Mount Issues

Ensure paths exist and use absolute paths:

```json
{
  "command": "docker",
  "args": [
    "run", "-i", "--rm",
    "-v", "/absolute/path/to/files:/samples:ro",
    "radare2-mcp:latest"
  ]
}
```

### Container Fails to Start

Check logs:

```bash
docker-compose logs nmap-mcp
docker logs $(docker ps -aq --filter name=nmap-mcp)
```

Verify health:

```bash
docker-compose ps
docker inspect nmap-mcp:latest
```

### API Key Not Working

For API-based servers, ensure environment variables are exported:

```bash
export SHODAN_API_KEY="your-key-here"
export VT_API_KEY="your-key-here"

# Then start Claude Desktop from the same shell
open -a "Claude"
```

Or set in Claude config:

```json
{
  "mcpServers": {
    "shodan": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "-e", "SHODAN_API_KEY", "shodan-mcp:latest"],
      "env": {
        "SHODAN_API_KEY": "${SHODAN_API_KEY}"
      }
    }
  }
}
```

### Build Failures

Update Docker Compose schema version if needed:

```yaml
version: '3.8'  # or higher
```

Clean build cache:

```bash
docker-compose build --no-cache nmap-mcp
docker system prune -a
```

### MCP Protocol Errors

Ensure MCP client (Claude Desktop) is up to date. The servers implement the Model Context Protocol specification and require compatible clients.

## Advanced Usage

### Custom MCP Server Development

Follow the project's structure to add new security tools:

```
new-category/
└── newtool-mcp/
    ├── Dockerfile
    ├── server.py
    ├── requirements.txt
    └── README.md
```

Example `server.py` structure:

```python
#!/usr/bin/env python3
import json
import subprocess
import sys

def handle_request(request):
    """Handle MCP protocol requests"""
    method = request.get("method")
    params = request.get("params", {})
    
    if method == "tools/list":
        return {
            "tools": [
                {
                    "name": "scan_target",
                    "description": "Scan target with tool",
                    "inputSchema": {
                        "type": "object",
                        "properties": {
                            "target": {"type": "string"}
                        },
                        "required": ["target"]
                    }
                }
            ]
        }
    elif method == "tools/call":
        tool_name = params.get("name")
        arguments = params.get("arguments", {})
        
        if tool_name == "scan_target":
            result = subprocess.run(
                ["tool", "scan", arguments["target"]],
                capture_output=True,
                text=True,
                timeout=300
            )
            return {"content": [{"type": "text", "text": result.stdout}]}
    
    return {"error": "Unknown method"}

if __name__ == "__main__":
    for line in sys.stdin:
        request = json.loads(line)
        response = handle_request(request)
        print(json.dumps(response))
        sys.stdout.flush()
```

### Multi-Stage Security Assessments

Combine multiple servers in a single workflow:

```
User: "Perform a full security assessment of example.com"

Claude orchestrates:
1. nmap-mcp: Port scan and service detection
2. whatweb-mcp: Technology fingerprinting
3. nuclei-mcp: CVE and misconfiguration scanning
4. sqlmap-mcp: SQL injection testing on discovered forms
5. ffuf-mcp: Directory and file fuzzing
6. gitleaks-mcp: Check if .git directory exposed and scan for secrets

Results consolidated into comprehensive report.
```

## Resources

- **Documentation**: https://fuzzinglabs.github.io/mcp-security-hub/
- **GitHub**: https://github.com/FuzzingLabs/mcp-security-hub
- **MCP Protocol**: https://modelcontextprotocol.io/
- **Examples**: https://github.com/FuzzingLabs/mcp-security-hub/tree/main/examples

This skill enables AI coding agents to guide developers in deploying and using the MCP Security Hub for offensive security testing through natural language interactions with Claude and other MCP clients.
