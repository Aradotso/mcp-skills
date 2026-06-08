---
name: osint-tools-mcp-server
description: MCP server exposing OSINT tools (Sherlock, Holehe, SpiderFoot, GHunt, Maigret, TheHarvester, Blackbird) for AI-assisted reconnaissance and intelligence gathering
triggers:
  - search for username across social media platforms
  - check if email is registered on websites
  - perform OSINT investigation on domain
  - gather intelligence about this person
  - find digital footprint for username
  - investigate email address registration
  - search for subdomains and email addresses
  - run reconnaissance on this target
---

# OSINT Tools MCP Server Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This MCP server provides AI assistants with access to multiple industry-standard OSINT (Open Source Intelligence) tools for reconnaissance and information gathering. It exposes 7 powerful tools through the Model Context Protocol: Sherlock, Holehe, SpiderFoot, GHunt, Maigret, TheHarvester, and Blackbird.

## What It Does

The OSINT Tools MCP Server allows AI assistants to:

- **Search usernames** across 399+ social media platforms (Sherlock)
- **Verify email registrations** on 120+ platforms (Holehe)
- **Perform comprehensive OSINT** on IPs, domains, emails, phones, etc. (SpiderFoot)
- **Extract Google account intel** from emails or Google IDs (GHunt)
- **Advanced username search** across 3000+ sites with confidence scoring (Maigret)
- **Gather domain intelligence** including emails, subdomains, hosts (TheHarvester)
- **Fast username reconnaissance** across 581 sites (Blackbird)

## Installation

### Basic Setup

1. **Clone the repository:**
```bash
git clone https://github.com/frishtik/osint-tools-mcp-server.git
cd osint-tools-mcp-server
```

2. **Install core dependencies:**
```bash
pip install -r requirements.txt
```

This automatically installs: Sherlock, Holehe, Maigret, and TheHarvester.

3. **Configure Claude Desktop:**

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "osint-tools": {
      "command": "python",
      "args": ["/absolute/path/to/osint-tools-mcp-server/src/osint_tools_mcp_server.py"],
      "env": {
        "PYTHONUNBUFFERED": "1"
      }
    }
  }
}
```

4. **Restart Claude Desktop** to load the MCP server.

### Optional Tools Installation

#### SpiderFoot
```bash
git clone https://github.com/smicallef/spiderfoot.git /opt/spiderfoot
cd /opt/spiderfoot
pip install -r requirements.txt
```

#### GHunt
```bash
git clone https://github.com/mxrch/GHunt.git /opt/ghunt
cd /opt/ghunt
pip install -r requirements.txt
```

#### Blackbird
```bash
git clone https://github.com/p1ngul1n0/blackbird.git /opt/blackbird
cd /opt/blackbird
pip install -r requirements.txt
```

## MCP Server Architecture

The server exposes tools as MCP resources. Each tool runs as a subprocess with timeout handling:

```python
# Core structure from osint_tools_mcp_server.py
import asyncio
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("osint-tools-mcp-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="sherlock",
            description="Search for username across 399+ social media platforms",
            inputSchema={
                "type": "object",
                "properties": {
                    "username": {"type": "string", "description": "Username to search for"}
                },
                "required": ["username"]
            }
        ),
        # ... other tools
    ]
```

## Available Tools & Usage

### 1. Sherlock - Username Search

**Tool Name:** `sherlock`

**Input:**
- `username` (string, required): Username to search for

**Example Request:**
```
Search for username "johndoe123" across social media platforms
```

**What It Does:** Searches 399+ platforms including Twitter, Instagram, GitHub, Reddit, etc.

**Typical Response:**
```
[+] Facebook: https://www.facebook.com/johndoe123
[+] Instagram: https://www.instagram.com/johndoe123
[+] GitHub: https://www.github.com/johndoe123
[-] Twitter: Not Found
```

### 2. Holehe - Email Verification

**Tool Name:** `holehe`

**Input:**
- `email` (string, required): Email address to check

**Example Request:**
```
Check if john.doe@example.com is registered on any platforms
```

**What It Does:** Verifies email registration on 120+ platforms (Google, Twitter, Adobe, etc.)

**Typical Response:**
```
[+] Google: Registered
[+] Twitter: Registered
[+] Adobe: Registered
[-] Amazon: Not Found
```

### 3. SpiderFoot - Comprehensive OSINT

**Tool Name:** `spiderfoot`

**Input:**
- `target` (string, required): IP, domain, email, phone, username, person name, Bitcoin address, or network block

**Example Request:**
```
Run comprehensive OSINT scan on example.com
```

**What It Does:** Deep reconnaissance with automatic target type detection. Scans 200+ modules including DNS, WHOIS, threat intelligence, social media, etc.

**⚠️ Warning:** Takes 5-30 minutes to complete. Very thorough.

**Typical Response:**
```
Target: example.com
Type: Domain
Modules Run: 156

WHOIS Data:
  Registrar: Example Registrar Inc.
  Created: 2020-01-15
  
DNS Records:
  A: 192.0.2.1
  MX: mail.example.com
  
Subdomains Found:
  - www.example.com
  - mail.example.com
  - api.example.com
```

### 4. GHunt - Google Account Intelligence

**Tool Name:** `ghunt`

**Input:**
- `email` (string, required): Email or Google ID

**Example Request:**
```
Extract information from Google account john.doe@gmail.com
```

**What It Does:** Retrieves Google account details, profile information, YouTube channels, Google Maps reviews, etc.

**Typical Response:**
```
Name: John Doe
Profile Picture: [URL]
Google ID: 123456789012345678901
YouTube Channel: [URL]
Reviews: 15 Google Maps reviews found
Last Profile Update: 2024-03-15
```

### 5. Maigret - Advanced Username Search

**Tool Name:** `maigret`

**Input:**
- `username` (string, required): Username to search for

**Example Request:**
```
Search for username "hackerman2024" with detailed analysis
```

**What It Does:** Searches 3000+ sites with false positive detection and confidence scoring.

**Typical Response:**
```
Username: hackerman2024
Sites Checked: 3127
Found: 47 profiles

High Confidence (90%+):
  - GitHub: https://github.com/hackerman2024
  - Reddit: https://reddit.com/u/hackerman2024
  
Medium Confidence (60-90%):
  - HackerNews: https://news.ycombinator.com/user?id=hackerman2024
  
Additional Data Extracted:
  - Real Name: John Smith (from GitHub)
  - Location: San Francisco (from Reddit)
```

### 6. TheHarvester - Domain Intelligence

**Tool Name:** `theharvester`

**Input:**
- `domain` (string, required): Domain or company name

**Example Request:**
```
Gather emails and subdomains for example.com
```

**What It Does:** Collects emails, subdomains, hosts, employee names from search engines, PGP servers, SHODAN, etc.

**Typical Response:**
```
Domain: example.com

Emails Found:
  - contact@example.com
  - admin@example.com
  - support@example.com
  
Hosts/Subdomains:
  - www.example.com (192.0.2.1)
  - mail.example.com (192.0.2.2)
  - api.example.com (192.0.2.3)
  
Employee Names:
  - John Doe
  - Jane Smith
```

### 7. Blackbird - Fast Username OSINT

**Tool Name:** `blackbird`

**Input:**
- `username` (string, required): Username to search for

**Example Request:**
```
Quick search for username "anonymous123"
```

**What It Does:** Lightning-fast searches across 581 sites optimized for speed.

**Typical Response:**
```
Username: anonymous123
Sites: 581 checked in 12 seconds

Found (24):
  [+] Twitter: https://twitter.com/anonymous123
  [+] Instagram: https://instagram.com/anonymous123
  [+] TikTok: https://tiktok.com/@anonymous123
  ...
```

## Common Usage Patterns

### Pattern 1: Email Investigation
```
User: "Investigate email address suspicious@example.com"

AI Response:
1. Running Holehe to check platform registrations...
2. Extracting domain and running TheHarvester...
3. Checking for Google account with GHunt...
```

### Pattern 2: Username Footprint Analysis
```
User: "Find all accounts for username 'targetuser'"

AI Response:
1. Running Sherlock for broad search (399 platforms)...
2. Running Maigret for detailed analysis (3000+ sites)...
3. Cross-referencing results for high-confidence matches...
```

### Pattern 3: Domain Reconnaissance
```
User: "Perform reconnaissance on competitor.com"

AI Response:
1. Running TheHarvester to gather emails and subdomains...
2. Launching SpiderFoot for comprehensive scan (this will take 15-20 minutes)...
3. While waiting, analyzing found email addresses with Holehe...
```

### Pattern 4: Parallel Tool Execution
```
User: "Search for 'johndoe' quickly across multiple tools"

AI Response:
Running parallel searches:
- Sherlock (username search)
- Blackbird (fast username search)
- Holehe (checking johndoe@gmail.com, johndoe@yahoo.com)
```

## Code Examples

### Implementing a New Tool

To add a new OSINT tool to the server:

```python
# In src/osint_tools_mcp_server.py

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        # ... existing tools ...
        Tool(
            name="newtool",
            description="Description of what the new tool does",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "Target to investigate"
                    },
                    "options": {
                        "type": "string",
                        "description": "Optional parameters",
                        "default": ""
                    }
                },
                "required": ["target"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "newtool":
        target = arguments["target"]
        options = arguments.get("options", "")
        
        # Build command
        cmd = ["newtool", target]
        if options:
            cmd.extend(options.split())
        
        # Execute with timeout
        try:
            process = await asyncio.create_subprocess_exec(
                *cmd,
                stdout=asyncio.subprocess.PIPE,
                stderr=asyncio.subprocess.PIPE
            )
            
            stdout, stderr = await asyncio.wait_for(
                process.communicate(),
                timeout=300  # 5 minute timeout
            )
            
            result = stdout.decode('utf-8', errors='replace')
            
            return [TextContent(
                type="text",
                text=f"New Tool Results:\n{result}"
            )]
            
        except asyncio.TimeoutError:
            return [TextContent(
                type="text",
                text="Tool execution timed out after 5 minutes"
            )]
        except Exception as e:
            return [TextContent(
                type="text",
                text=f"Error running tool: {str(e)}"
            )]
```

### Custom Tool Wrapper

Creating a Python wrapper for better control:

```python
import subprocess
import json
from typing import Dict, List

class OSINTToolWrapper:
    """Wrapper for OSINT tools with standardized output"""
    
    def __init__(self, tool_name: str, tool_path: str):
        self.tool_name = tool_name
        self.tool_path = tool_path
    
    async def execute(self, target: str, timeout: int = 300) -> Dict:
        """Execute tool and return structured results"""
        cmd = [self.tool_path, target, "--json"]
        
        try:
            process = await asyncio.create_subprocess_exec(
                *cmd,
                stdout=asyncio.subprocess.PIPE,
                stderr=asyncio.subprocess.PIPE
            )
            
            stdout, stderr = await asyncio.wait_for(
                process.communicate(),
                timeout=timeout
            )
            
            # Parse JSON output if available
            try:
                results = json.loads(stdout.decode('utf-8'))
            except json.JSONDecodeError:
                results = {"raw_output": stdout.decode('utf-8')}
            
            return {
                "tool": self.tool_name,
                "target": target,
                "success": process.returncode == 0,
                "results": results,
                "error": stderr.decode('utf-8') if stderr else None
            }
            
        except asyncio.TimeoutError:
            return {
                "tool": self.tool_name,
                "target": target,
                "success": False,
                "error": f"Timeout after {timeout} seconds"
            }

# Usage
sherlock = OSINTToolWrapper("sherlock", "/usr/local/bin/sherlock")
results = await sherlock.execute("johndoe")
```

## Configuration

### Environment Variables

Set these in your MCP server configuration:

```json
{
  "mcpServers": {
    "osint-tools": {
      "command": "python",
      "args": ["/path/to/osint_tools_mcp_server.py"],
      "env": {
        "PYTHONUNBUFFERED": "1",
        "SPIDERFOOT_PATH": "/opt/spiderfoot",
        "GHUNT_PATH": "/opt/ghunt",
        "BLACKBIRD_PATH": "/opt/blackbird",
        "OSINT_TIMEOUT": "300",
        "OSINT_LOG_LEVEL": "INFO"
      }
    }
  }
}
```

### Tool-Specific Configuration

**SpiderFoot Configuration:**
```bash
# Edit /opt/spiderfoot/sfwebui.py for custom settings
# Or use environment variables
export SPIDERFOOT_MODULES="all"  # Or comma-separated list
export SPIDERFOOT_TIMEOUT="1800"  # 30 minutes
```

**TheHarvester Sources:**
```bash
# Specify data sources
theharvester -d example.com -b google,bing,linkedin,pgp
```

**Maigret Verbosity:**
```bash
# Control output detail
maigret username --no-progressbar --folderoutput ./results
```

## Troubleshooting

### Tools Not Found

**Problem:** `Command not found: sherlock`

**Solution:**
```bash
# Check installation
pip list | grep sherlock

# Install if missing
pip install sherlock-project

# Verify PATH
which sherlock

# Add to PATH if needed
export PATH="$PATH:$HOME/.local/bin"
```

### SpiderFoot Not Running

**Problem:** SpiderFoot scans fail or timeout

**Solution:**
```bash
# Verify installation
cd /opt/spiderfoot
python sf.py -h

# Check dependencies
pip install -r requirements.txt

# Test manually
python sf.py -s example.com -m all -o json
```

### Timeout Issues

**Problem:** Tools timeout before completing

**Solution:**
Increase timeout in the MCP server code:

```python
# For long-running tools like SpiderFoot
stdout, stderr = await asyncio.wait_for(
    process.communicate(),
    timeout=1800  # 30 minutes instead of default 5
)
```

Or use environment variable:
```bash
export OSINT_TIMEOUT="1800"
```

### Rate Limiting

**Problem:** Getting blocked by platforms

**Solution:**
```bash
# Add delays between requests
sherlock username --timeout 10

# Use Tor for anonymity (if legal in your jurisdiction)
maigret username --tor

# Spread requests over time
# Run in batches with pauses
```

### JSON Parsing Errors

**Problem:** Tool output not parsing correctly

**Solution:**
```python
# Add error handling
try:
    results = json.loads(output)
except json.JSONDecodeError:
    # Fallback to raw text parsing
    results = {"raw": output, "parsed": parse_text_output(output)}
```

### Permission Denied

**Problem:** Cannot execute tools

**Solution:**
```bash
# Make tools executable
chmod +x /usr/local/bin/sherlock
chmod +x /opt/spiderfoot/sf.py

# Or run with Python explicitly
python -m sherlock username
```

## Best Practices

### 1. Start with Fast Tools
Begin investigations with Holehe or Blackbird for quick results before running comprehensive scans.

### 2. Parallel Execution
Run multiple fast tools simultaneously:
```
"Run Sherlock and Blackbird in parallel for username 'target'"
```

### 3. Result Validation
Cross-reference findings from multiple tools for accuracy.

### 4. Timeout Management
- Fast tools (Holehe, Blackbird): 60-120 seconds
- Medium tools (Sherlock, Maigret): 300-600 seconds  
- Slow tools (SpiderFoot): 1800-3600 seconds

### 5. Data Privacy
Always respect privacy laws and obtain proper authorization. Log all investigations for compliance.

## Legal & Ethical Considerations

**⚠️ CRITICAL:** This tool is for legitimate security research and authorized investigations only.

- Only gather publicly available information
- Comply with GDPR, CCPA, and local privacy laws
- Respect platforms' Terms of Service
- Obtain written authorization for professional investigations
- Never use for stalking, harassment, or malicious purposes

## Performance Tips

**Optimize Sherlock:**
```bash
# Target specific platforms
sherlock username --site Twitter Instagram GitHub
```

**Optimize TheHarvester:**
```bash
# Limit sources for speed
theharvester -d domain.com -b google,bing -l 100
```

**Optimize Maigret:**
```bash
# Disable slow sites
maigret username --timeout 10 --no-recursion
```

This skill enables AI coding agents to effectively assist developers in conducting OSINT investigations using multiple industry-standard tools through a unified MCP interface.
