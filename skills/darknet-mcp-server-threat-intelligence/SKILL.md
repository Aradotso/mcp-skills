---
name: darknet-mcp-server-threat-intelligence
description: Dark web and threat intelligence MCP server with 66 tools for breach data, ransomware tracking, Tor access, malware analysis, and OSINT
triggers:
  - check if this domain has been breached
  - search for ransomware activity targeting this company
  - analyze this malware hash across threat databases
  - investigate stealer logs for compromised credentials
  - fetch threat intelligence on this IP address
  - search dark web forums for this indicator
  - trace this bitcoin address for abuse reports
  - check if this URL is associated with phishing
---

# darknet-mcp-server Threat Intelligence

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

darknet-mcp-server is a comprehensive MCP server providing 66 tools across 16 data sources for dark web and threat intelligence. It unifies breach databases (HIBP), ransomware tracking, Tor .onion access, malware analysis (Hybrid Analysis, MalwareBazaar), blockchain forensics, exploit databases (Vulners), stealer logs (Hudson Rock), and OSINT platforms into a single interface for AI agents.

**Key capabilities:**
- Breach intelligence: HIBP, IntelligenceX, paste searches
- Ransomware tracking: ransomware.live, ransomlook.io (12 active groups, 1000+ victims)
- Dark web access: 7 Tor tools for .onion fetching, scraping, searching
- Malware analysis: ThreatFox, MalwareBazaar, Hybrid Analysis, URLhaus
- Blockchain: Bitcoin transaction tracing, ChainAbuse reports
- IP/domain intel: AbuseIPDB, GreyNoise, AlienVault OTX, Pulsedive
- Exploit search: Vulners database
- Phishing: PhishTank, OpenPhish

## Installation

### npx (Zero Install)

```bash
# Run directly without installation
npx darknet-mcp-server

# Check Tor connectivity (for .onion tools)
npx darknet-mcp-server --check-tor
```

### Local Clone

```bash
git clone https://github.com/badchars/darknet-mcp-server.git
cd darknet-mcp-server
bun install
bun run src/index.ts
```

### MCP Client Configuration

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "darknet": {
      "command": "npx",
      "args": ["-y", "darknet-mcp-server"],
      "env": {
        "HIBP_API_KEY": "",
        "INTELX_API_KEY": "",
        "ABUSEIPDB_API_KEY": "",
        "HUDSONROCK_API_KEY": "",
        "HYBRID_API_KEY": "",
        "VULNERS_API_KEY": "",
        "OTX_API_KEY": "",
        "PULSEDIVE_API_KEY": "",
        "PHISHTANK_API_KEY": "",
        "ABUSECH_AUTH_KEY": "",
        "TOR_SOCKS_HOST": "127.0.0.1",
        "TOR_SOCKS_PORT": "9050"
      }
    }
  }
}
```

**Claude Code**:

```bash
claude mcp add darknet-mcp-server -- npx darknet-mcp-server
```

**Cursor/Windsurf** (`.cursor/mcp.json` or similar):

```json
{
  "mcpServers": {
    "darknet": {
      "command": "npx",
      "args": ["-y", "darknet-mcp-server"]
    }
  }
}
```

## Environment Variables

All API keys are **optional**. Many tools work without authentication:

```bash
# Breach & credential intelligence
export HIBP_API_KEY=your-key           # Enables breachSearch, pasteSearch
export INTELX_API_KEY=your-key         # Enables intelx_search, intelx_phonebook, etc.

# Threat intelligence
export OTX_API_KEY=your-key            # AlienVault OTX (higher rate limits)
export ABUSEIPDB_API_KEY=your-key      # Enables abuseipdb_check, abuseipdb_report
export ABUSECH_AUTH_KEY=your-key       # abuse.ch (ThreatFox, MalwareBazaar, URLhaus)
export PULSEDIVE_API_KEY=your-key      # Pulsedive threat intel

# Stealer logs & credentials
export HUDSONROCK_API_KEY=your-key     # Enables stealer_domain, stealer_ip, stealer_cve

# Exploit & malware analysis
export VULNERS_API_KEY=your-key        # Enables vulners_search, vulners_exploit
export HYBRID_API_KEY=your-key         # Enables hybrid_search, hybrid_report, hybrid_submit

# Phishing
export PHISHTANK_API_KEY=your-key      # PhishTank (higher rate limits)

# Tor SOCKS5 proxy (for .onion access)
export TOR_SOCKS_HOST=127.0.0.1       # Default: 127.0.0.1
export TOR_SOCKS_PORT=9050            # Default: 9050
```

### Tor Setup (Required for .onion Tools)

```bash
# macOS
brew install tor
brew services start tor

# Linux (Debian/Ubuntu)
sudo apt install tor
sudo systemctl start tor
sudo systemctl enable tor

# Docker
docker run -d -p 9050:9050 dperson/torproxy

# Verify
curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org
```

## Tool Categories & Usage

### 1. Breach Intelligence (HIBP)

**Tools:** `breachList`, `breachSearch`, `pasteSearch`

```typescript
// List all breaches for a domain
await use_mcp_tool("darknet", "breachList", {
  domain: "example.com"
});
// Returns: Array of breach objects with Name, BreachDate, PwnCount, DataClasses

// Search if specific account was breached (requires HIBP_API_KEY)
await use_mcp_tool("darknet", "breachSearch", {
  account: "user@example.com"
});
// Returns: Array of breaches affecting this account

// Search pastes mentioning an email (requires HIBP_API_KEY)
await use_mcp_tool("darknet", "pasteSearch", {
  account: "admin@example.com"
});
// Returns: Array of paste sites with Source, Id, Date
```

**Example investigation:**

```typescript
// Comprehensive breach check
const domain = "target.com";

const breaches = await use_mcp_tool("darknet", "breachList", { domain });
const accountCheck = await use_mcp_tool("darknet", "breachSearch", { 
  account: `admin@${domain}` 
});
const pastes = await use_mcp_tool("darknet", "pasteSearch", { 
  account: `admin@${domain}` 
});

// Cross-reference with stealer logs
const stealerLogs = await use_mcp_tool("darknet", "stealer_domain", { 
  domain 
});

// Summary: breaches.length breaches, pastes.length pastes, stealerLogs compromised machines
```

### 2. Ransomware Tracking

**Tools:** `ransomwareGroups`, `ransomwareRecent`, `ransomwareByGroup`, `ransomwareBySector`, `ransomwareByCountry`, `ransomlookRecent`, `ransomlookGroup`

```typescript
// List all active ransomware groups
await use_mcp_tool("darknet", "ransomwareGroups", {});
// Returns: Array of {name, description, country, activity_since, onion_url}

// Get recent victims (last 7 days by default)
await use_mcp_tool("darknet", "ransomwareRecent", {
  days: 30
});
// Returns: Array of {victim_name, group_name, published_date, country, sector}

// Track specific group
await use_mcp_tool("darknet", "ransomwareByGroup", {
  group: "lockbit3"
});
// Returns: Victims by this group

// Sector-specific tracking
await use_mcp_tool("darknet", "ransomwareBySector", {
  sector: "healthcare"
});
// Returns: All healthcare sector victims

// Cross-source validation (ransomlook.io)
await use_mcp_tool("darknet", "ransomlookRecent", {});
await use_mcp_tool("darknet", "ransomlookGroup", {
  group: "lockbit3"
});
```

**Example: Healthcare sector threat landscape**

```typescript
const healthcareVictims = await use_mcp_tool("darknet", "ransomwareBySector", {
  sector: "healthcare"
});

const recentActivity = await use_mcp_tool("darknet", "ransomwareRecent", {
  days: 90
});

// Identify most active groups targeting healthcare
const groups = new Set(healthcareVictims.map(v => v.group_name));
for (const group of groups) {
  const groupVictims = await use_mcp_tool("darknet", "ransomwareByGroup", { group });
  // Analyze: groupVictims.length victims, average time between attacks
}
```

### 3. Tor & Dark Web Access

**Tools:** `tor_fetch_onion`, `tor_scrape_onion`, `tor_search_onion`, `tor_check_availability`, `tor_exit_nodes`, `tor_check_exit`, `circl_onion_lookup`

**Prerequisites:** Tor SOCKS5 proxy running on `127.0.0.1:9050`

```typescript
// Fetch raw .onion content
await use_mcp_tool("darknet", "tor_fetch_onion", {
  onion_url: "http://example.onion/page.html"
});
// Returns: {content: "raw HTML", status: 200}

// Extract structured data (links, text, metadata)
await use_mcp_tool("darknet", "tor_scrape_onion", {
  onion_url: "http://forum.onion/",
  extract: ["links", "text"]
});
// Returns: {links: [...], text: "...", title: "..."}

// Search content across multiple .onion pages
await use_mcp_tool("darknet", "tor_search_onion", {
  onion_urls: [
    "http://forum1.onion/threads",
    "http://market2.onion/listings"
  ],
  search_term: "stolen credentials"
});
// Returns: Array of {url, matches: [...], snippet}

// Check if .onion is reachable
await use_mcp_tool("darknet", "tor_check_availability", {
  onion_url: "http://example.onion"
});
// Returns: {available: true, response_time_ms: 2341}

// Get current Tor exit nodes
await use_mcp_tool("darknet", "tor_exit_nodes", {});
// Returns: Array of {ip, name, country, exit_policy}

// Check if IP is a Tor exit node
await use_mcp_tool("darknet", "tor_check_exit", {
  ip: "185.220.101.1"
});
// Returns: {is_exit: true, nickname: "...", country: "DE"}

// CIRCL onion lookup (known malicious .onion sites)
await use_mcp_tool("darknet", "circl_onion_lookup", {
  onion_address: "example.onion"
});
// Returns: {known: true, tags: ["phishing", "scam"], first_seen: "2023-01-15"}
```

**Example: Dark web forum monitoring**

```typescript
const forumUrls = [
  "http://darkforum1.onion/breaches",
  "http://marketplace2.onion/credentials"
];

// Check availability first
for (const url of forumUrls) {
  const status = await use_mcp_tool("darknet", "tor_check_availability", { 
    onion_url: url 
  });
  if (!status.available) continue;
  
  // Search for company mentions
  const results = await use_mcp_tool("darknet", "tor_search_onion", {
    onion_urls: [url],
    search_term: "example.com database"
  });
  
  if (results.length > 0) {
    // Fetch full pages for context
    for (const result of results) {
      const content = await use_mcp_tool("darknet", "tor_fetch_onion", {
        onion_url: result.url
      });
      // Analyze content.content for breach details
    }
  }
}
```

### 4. Malware Analysis

**Tools:** `threatfox_ioc`, `threatfox_malware`, `threatfox_tag`, `malwarebazaar_hash`, `malwarebazaar_recent`, `malwarebazaar_tag`, `urlhaus_url`, `urlhaus_host`, `urlhaus_recent`, `hybrid_search`, `hybrid_report`, `hybrid_submit`

```typescript
// ThreatFox: Look up IOC (IP, domain, URL, hash)
await use_mcp_tool("darknet", "threatfox_ioc", {
  ioc: "185.220.101.1"
});
// Returns: {ioc_type: "ip", malware: "emotet", confidence: 90, tags: [...]}

// Search by malware family
await use_mcp_tool("darknet", "threatfox_malware", {
  malware: "emotet"
});
// Returns: Array of IOCs associated with Emotet

// Search by tag
await use_mcp_tool("darknet", "threatfox_tag", {
  tag: "ransomware"
});

// MalwareBazaar: Hash lookup
await use_mcp_tool("darknet", "malwarebazaar_hash", {
  hash: "d41d8cd98f00b204e9800998ecf8427e"
});
// Returns: {sha256, file_type, file_name, signature, tags, first_seen}

// Recent malware samples
await use_mcp_tool("darknet", "malwarebazaar_recent", {
  limit: 50
});

// Search by tag
await use_mcp_tool("darknet", "malwarebazaar_tag", {
  tag: "emotet"
});

// URLhaus: Malicious URL lookup
await use_mcp_tool("darknet", "urlhaus_url", {
  url: "http://malicious.example.com/payload.exe"
});
// Returns: {url_status: "online", threat: "malware_download", tags: [...]}

// Host-based search
await use_mcp_tool("darknet", "urlhaus_host", {
  host: "malicious.example.com"
});

// Recent malicious URLs
await use_mcp_tool("darknet", "urlhaus_recent", {
  limit: 100
});

// Hybrid Analysis: Search samples (requires HYBRID_API_KEY)
await use_mcp_tool("darknet", "hybrid_search", {
  hash: "d41d8cd98f00b204e9800998ecf8427e"
});

// Get full report
await use_mcp_tool("darknet", "hybrid_report", {
  report_id: "64e3f2a1b4c5d6e7f8a9b0c1"
});

// Submit file for analysis
await use_mcp_tool("darknet", "hybrid_submit", {
  file_path: "/path/to/sample.exe",
  environment_id: 120  // Windows 10 64-bit
});
```

**Example: Cross-platform malware correlation**

```typescript
const hash = "d41d8cd98f00b204e9800998ecf8427e";

// Query all malware databases
const threatfox = await use_mcp_tool("darknet", "threatfox_ioc", { ioc: hash });
const bazaar = await use_mcp_tool("darknet", "malwarebazaar_hash", { hash });
const hybrid = await use_mcp_tool("darknet", "hybrid_search", { hash });

// Correlate findings
const malwareFamily = threatfox.malware || bazaar.signature;
const c2Servers = threatfox.ioc_type === "ip" ? [threatfox.ioc] : [];

// Check C2 infrastructure
for (const c2 of c2Servers) {
  const abuse = await use_mcp_tool("darknet", "abuseipdb_check", { ip: c2 });
  const otx = await use_mcp_tool("darknet", "otx_ip", { ip: c2 });
  // Analyze: abuse.abuseConfidenceScore, otx.pulse_count
}

// If not in Hybrid, submit for analysis
if (!hybrid || hybrid.length === 0) {
  await use_mcp_tool("darknet", "hybrid_submit", {
    file_path: "/path/to/sample",
    environment_id: 120
  });
}
```

### 5. Stealer Logs & Credentials (Hudson Rock)

**Tools:** `stealer_domain`, `stealer_ip`, `stealer_cve` (requires `HUDSONROCK_API_KEY`)

```typescript
// Search stealer logs by domain
await use_mcp_tool("darknet", "stealer_domain", {
  domain: "example.com"
});
// Returns: Array of {computer_name, infection_date, passwords: [...], cookies: [...]}

// Search by IP address
await use_mcp_tool("darknet", "stealer_ip", {
  ip: "192.168.1.1"
});

// Search by CVE (machines vulnerable to specific exploit)
await use_mcp_tool("darknet", "stealer_cve", {
  cve: "CVE-2023-12345"
});
// Returns: Compromised machines with this vulnerability
```

**Example: Credential exposure assessment**

```typescript
const domain = "corporate.com";

// Check stealer logs
const stealerResults = await use_mcp_tool("darknet", "stealer_domain", { domain });

// Analyze exposure
const uniqueUsers = new Set(stealerResults.flatMap(r => 
  r.passwords.map(p => p.username)
));

// Cross-check with breaches
const breaches = await use_mcp_tool("darknet", "breachList", { domain });

// Check IntelligenceX for additional leaks
const intelx = await use_mcp_tool("darknet", "intelx_search", {
  term: domain,
  buckets: ["pastes", "darknet.forums"]
});

// Summary: uniqueUsers.size compromised employees, breaches.length known breaches
```

### 6. IP & Domain Intelligence

**Tools:** `abuseipdb_check`, `abuseipdb_report`, `abuseipdb_blacklist`, `abuseipdb_bulk`, `greynoiseRiot`, `greynoiseQuick`, `greynoiseContext`, `greynoiseQuery`, `otx_ip`, `otx_domain`, `otx_hash`, `otx_url`, `pulsedive_indicator`, `pulsedive_threat`

```typescript
// AbuseIPDB: Check IP reputation (requires ABUSEIPDB_API_KEY)
await use_mcp_tool("darknet", "abuseipdb_check", {
  ip: "1.2.3.4"
});
// Returns: {abuseConfidenceScore: 85, usageType: "Data Center", reports: [...]}

// Report abusive IP
await use_mcp_tool("darknet", "abuseipdb_report", {
  ip: "1.2.3.4",
  categories: [18, 22],  // Brute force, SSH abuse
  comment: "SSH brute force attempts from this IP"
});

// Get blacklist (IPs with >90 confidence score)
await use_mcp_tool("darknet", "abuseipdb_blacklist", {
  confidence_minimum: 90,
  limit: 1000
});

// Bulk check multiple IPs
await use_mcp_tool("darknet", "abuseipdb_bulk", {
  ips: ["1.2.3.4", "5.6.7.8", "9.10.11.12"]
});

// GreyNoise: Check if IP is benign scanner (RIOT)
await use_mcp_tool("darknet", "greynoiseRiot", {
  ip: "8.8.8.8"
});
// Returns: {riot: true, name: "Google Public DNS", trust_level: "1"}

// Quick IP classification
await use_mcp_tool("darknet", "greynoiseQuick", {
  ips: ["1.2.3.4", "8.8.8.8"]
});
// Returns: {noise: ["1.2.3.4"], riot: ["8.8.8.8"]}

// Full context
await use_mcp_tool("darknet", "greynoiseContext", {
  ip: "1.2.3.4"
});
// Returns: {seen: true, classification: "malicious", tags: ["tor", "vpn"], last_seen: "2024-01-15"}

// GNQL query
await use_mcp_tool("darknet", "greynoiseQuery", {
  query: "classification:malicious AND tags:ransomware"
});

// AlienVault OTX: IP reputation (no key required, higher limits with OTX_API_KEY)
await use_mcp_tool("darknet", "otx_ip", {
  ip: "1.2.3.4"
});
// Returns: {pulse_count: 12, pulses: [{name: "...", tags: [...]}]}

// Domain reputation
await use_mcp_tool("darknet", "otx_domain", {
  domain: "malicious.com"
});

// Hash reputation
await use_mcp_tool("darknet", "otx_hash", {
  hash: "d41d8cd98f00b204e9800998ecf8427e"
});

// URL reputation
await use_mcp_tool("darknet", "otx_url", {
  url: "http://phishing.example.com/login"
});

// Pulsedive: Indicator lookup
await use_mcp_tool("darknet", "pulsedive_indicator", {
  indicator: "1.2.3.4",
  type: "ip"
});
// Returns: {risk: "high", threats: [...], properties: {...}}

// Threat intelligence feed
await use_mcp_tool("darknet", "pulsedive_threat", {
  threat_name: "emotet"
});
```

**Example: Multi-source IP investigation**

```typescript
const ip = "185.220.101.1";

// Parallel queries across all sources
const [abuseipdb, greynoise, greynoiseContext, otx, pulsedive, torCheck] = await Promise.all([
  use_mcp_tool("darknet", "abuseipdb_check", { ip }),
  use_mcp_tool("darknet", "greynoiseQuick", { ips: [ip] }),
  use_mcp_tool("darknet", "greynoiseContext", { ip }),
  use_mcp_tool("darknet", "otx_ip", { ip }),
  use_mcp_tool("darknet", "pulsedive_indicator", { indicator: ip, type: "ip" }),
  use_mcp_tool("darknet", "tor_check_exit", { ip })
]);

// Correlate findings
const isMalicious = abuseipdb.abuseConfidenceScore > 75 || 
                    greynoiseContext.classification === "malicious" ||
                    pulsedive.risk === "high";

const isTor = torCheck.is_exit;
const pulseCount = otx.pulse_count;

// Summary: IP reputation across 5 sources, Tor exit node status, threat intel pulses
```

### 7. Blockchain & Cryptocurrency

**Tools:** `btc_balance`, `btc_transactions`, `btc_unspent`, `chainabuse_report`

```typescript
// Get Bitcoin balance
await use_mcp_tool("darknet", "btc_balance", {
  address: "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"
});
// Returns: {balance_btc: 68.12345678, total_received: 100.5, total_sent: 32.37}

// Get transaction history
await use_mcp_tool("darknet", "btc_transactions", {
  address: "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa",
  limit: 50
});
// Returns: Array of {txid, time, amount, confirmations}

// Get unspent outputs (UTXO)
await use_mcp_tool("darknet", "btc_unspent", {
  address: "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"
});

// ChainAbuse: Check if address is reported
await use_mcp_tool("darknet", "chainabuse_report", {
  address: "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"
});
// Returns: {reported: true, abuse_type: "ransomware", reports: 5}
```

**Example: Ransomware payment tracking**

```typescript
// Ransomware Bitcoin address from leak site
const btcAddress = "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh";

// Check balance and transactions
const balance = await use_mcp_tool("darknet", "btc_balance", { address: btcAddress });
const txs = await use_mcp_tool("darknet", "btc_transactions", { 
  address: btcAddress,
  limit: 100 
});

// Check abuse reports
const abuse = await use_mcp_tool("darknet", "chainabuse_report", { 
  address: btcAddress 
});

// Analyze payments
const victimPayments = txs.filter(tx => tx.amount > 0);  // Incoming
const totalExtorted = victimPayments.reduce((sum, tx) => sum + tx.amount, 0);

// Summary: totalExtorted BTC from victimPayments.length victims, abuse.reports reports
```

### 8. Exploit Databases

**Tools:** `vulners_search`, `vulners_exploit`, `vulners_cve` (requires `VULNERS_API_KEY`)

```typescript
// Search Vulners database
await use_mcp_tool("darknet", "vulners_search", {
  query: "apache struts rce"
});
// Returns: Array of {id, title, cvss, published, description}

// Get exploit details
await use_mcp_tool("darknet", "vulners_exploit", {
  exploit_id: "EDB-ID:12345"
});
// Returns: {title, cvss, exploit_code: "...", references: [...]}

// CVE lookup
await use_mcp_tool("darknet", "vulners_cve", {
  cve: "CVE-2023-12345"
});
// Returns: {cvss3: 9.8, description: "...", exploits: [{id, title}]}
```

**Example: Vulnerability impact assessment**

```typescript
const cve = "CVE-2023-12345";

// Get CVE details from Vulners
const vulnDetails = await use_mcp_tool("darknet", "vulners_cve", { cve });

// Check if exploits exist
const exploits = vulnDetails.exploits || [];

// Check stealer logs for exposed machines
const stealerLogs = await use_mcp_tool("darknet", "stealer_cve", { cve });

// Cross-reference with OTX
const otx = await use_mcp_tool("darknet", "otx_cve", { cve });

// Summary: CVSS vulnDetails.cvss3, exploits.length exploits available, 
// stealerLogs.length compromised machines, otx.pulse_count threat intel pulses
```

### 9. Phishing & Malicious URLs

**Tools:** `phishtank_check`, `phishtank_recent`, `openphish_recent`

```typescript
// PhishTank: Check if URL is phishing
await use_mcp_tool("darknet", "phishtank_check", {
  url: "http://paypal-verify.example.com/login"
});
// Returns: {in_database: true, valid: true, verified: true, target: "PayPal"}

// Recent phishing URLs
await use_mcp_tool("darknet", "phishtank_recent", {
  limit: 100
});
// Returns: Array of {phish_id, url, target, submission_time}

// OpenPhish: Recent feed
await use_mcp_tool("darknet", "openphish_recent", {
  limit: 500
});
// Returns: Array of URLs
```

**Example: Phishing campaign detection**

```typescript
const targetDomain = "example.com";

// Get recent phishing URLs
const phishtank = await use_mcp_tool("darknet", "phishtank_recent", { limit: 1000 });
const openphish = await use_mcp_tool("darknet", "openphish_recent", { limit: 1000 });

// Filter for URLs targeting our domain
const targetedPhishing = phishtank.filter(p => 
  p.target.toLowerCase().includes(targetDomain.toLowerCase()) ||
  p.url.includes(targetDomain)
);

// Check each URL in URLhaus
for (const phish of targetedPhishing) {
  const urlhaus = await use_mcp_tool("darknet", "urlhaus_url", { url: phish.url });
  // Analyze: urlhaus.url_status, urlhaus.threat
}

// Summary: targetedPhishing.length active phishing URLs targeting example.com
```

### 10. IntelligenceX (Requires API Key)

**Tools:** `intelx_search`, `intelx_phonebook`, `intelx_leaks`, `intelx_darknet`

```typescript
// Search across all IntelX sources
await use_mcp_tool("darknet", "intelx_search", {
  term: "example.com",
  buckets: ["darknet.forums", "pastes", "leaks", "domains"],
  limit: 100
});
// Returns: Array of {systemid, name, bucket, added, size}

// Phonebook search (domains, emails, URLs)
await use_mcp_tool("darknet", "intelx_phonebook", {
  selector: "example.com",
  selector_type: "domain"
});
// Returns: Array of subdomains, emails, URLs associated with domain

// Search leak databases
await use_mcp_tool("darknet", "intelx_leaks", {
  email: "admin@example.com"
});
// Returns: Databases containing this email

// Dark web forum
