---
name: security-detections-mcp
description: Query unified database of 8,200+ Sigma, Splunk, Elastic, KQL, Sublime, and CrowdStrike security detections via MCP with MITRE ATT&CK mapping
triggers:
  - search for security detections covering ransomware
  - find detections for MITRE technique T1059.001
  - analyze my detection coverage gaps
  - generate ATT&CK Navigator layer for APT29
  - show me detections for this CVE
  - compare coverage across multiple threat actors
  - suggest detections for persistence techniques
  - identify weak spots in my detection stack
---

# security-detections-mcp

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

MCP server providing AI agents access to 8,200+ security detection rules across 6 formats (Sigma, Splunk ESCU, Elastic, KQL, Sublime, CrowdStrike CQL) with MITRE ATT&CK mapping, coverage analysis, threat actor emulation, and autonomous detection engineering workflows.

## What It Does

- **Unified Detection Search**: Query across all detection formats with single API
- **MITRE ATT&CK Integration**: 172 threat actors, 784 software, 4,362 relationships from STIX
- **Coverage Analysis**: Identify gaps by tactic, technique, threat actor, or procedure
- **Navigator Layers**: Export ATT&CK Navigator JSON for coverage visualization
- **Autonomous Pipeline**: CTI ingestion → gap analysis → detection generation → Atomic testing
- **81 MCP Tools**: Search, filter, analyze, generate, and compare detections
- **11 Expert Prompts**: Ransomware assessment, APT emulation, purple team workflows

## Installation

### Local (Full Power)

**Quick Start** (npx):
```bash
npx -y security-detections-mcp
```

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):
```json
{
  "mcpServers": {
    "security-detections": {
      "command": "npx",
      "args": ["-y", "security-detections-mcp"],
      "env": {
        "SIGMA_PATHS": "/path/to/sigma/rules,/path/to/sigma/rules-threat-hunting",
        "SPLUNK_PATHS": "/path/to/security_content/detections",
        "ELASTIC_PATHS": "/path/to/detection-rules/rules",
        "KQL_PATHS": "/path/to/kql-rules",
        "SUBLIME_PATHS": "/path/to/sublime-rules/detection-rules",
        "CQL_HUB_PATHS": "/path/to/cql-hub/queries",
        "STORY_PATHS": "/path/to/security_content/stories",
        "ATTACK_STIX_PATH": "/path/to/enterprise-attack.json"
      }
    }
  }
}
```

**Cursor** (`.cursor/config/mcp.json`):
```json
{
  "mcpServers": {
    "security-detections": {
      "command": "npx",
      "args": ["-y", "security-detections-mcp"],
      "env": {
        "SIGMA_PATHS": "/Users/yourname/detections/sigma/rules",
        "SPLUNK_PATHS": "/Users/yourname/detections/security_content/detections"
      }
    }
  }
}
```

**Claude Code CLI**:
```bash
claude mcp add security-detections -- npx -y security-detections-mcp
```

### Hosted (Zero Setup)

1. Get token at https://detect.michaelhaag.org/account/tokens (200 calls/day free)
2. Configure MCP client:

**Claude Code**:
```bash
claude mcp add --transport http security-detections \
  https://detect.michaelhaag.org/api/mcp/mcp \
  --header "Authorization: Bearer $SDMCP_TOKEN"
```

**Claude Desktop** (via mcp-remote):
```json
{
  "mcpServers": {
    "security-detections": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://detect.michaelhaag.org/api/mcp/mcp",
        "--header",
        "Authorization: Bearer $SDMCP_TOKEN"
      ]
    }
  }
}
```

## Getting Detection Content

Download all detection sources (sparse checkout for rules only):

```bash
mkdir -p detections && cd detections

# Sigma (8K+ rules)
git clone --depth 1 --filter=blob:none --sparse https://github.com/SigmaHQ/sigma.git
cd sigma && git sparse-checkout set rules rules-threat-hunting && cd ..

# Splunk ESCU
git clone --depth 1 --filter=blob:none --sparse https://github.com/splunk/security_content.git
cd security_content && git sparse-checkout set detections stories && cd ..

# Elastic
git clone --depth 1 --filter=blob:none --sparse https://github.com/elastic/detection-rules.git
cd detection-rules && git sparse-checkout set rules && cd ..

# KQL
git clone --depth 1 https://github.com/Bert-JanP/Hunting-Queries-Detection-Rules.git kql-bertjanp
git clone --depth 1 https://github.com/jkerai1/KQL-Queries.git kql-jkerai1

# Sublime
git clone --depth 1 --filter=blob:none --sparse https://github.com/sublime-security/sublime-rules.git
cd sublime-rules && git sparse-checkout set detection-rules && cd ..

# CrowdStrike CQL
git clone --depth 1 https://github.com/ByteRay-Labs/Query-Hub.git cql-hub
```

Set paths in MCP config:
```bash
export SIGMA_PATHS="$PWD/sigma/rules,$PWD/sigma/rules-threat-hunting"
export SPLUNK_PATHS="$PWD/security_content/detections"
export ELASTIC_PATHS="$PWD/detection-rules/rules"
export KQL_PATHS="$PWD/kql-bertjanp,$PWD/kql-jkerai1"
export SUBLIME_PATHS="$PWD/sublime-rules/detection-rules"
export CQL_HUB_PATHS="$PWD/cql-hub/queries"
```

## Core MCP Tools

### Search & Retrieval

```typescript
// Full-text search across all fields
await use_mcp_tool('security-detections', 'search', {
  query: 'powershell empire',
  limit: 20
});

// Get single detection by ID
await use_mcp_tool('security-detections', 'get_by_id', {
  id: 'sigma-abc123'
});

// List all detections (paginated)
await use_mcp_tool('security-detections', 'list_all', {
  limit: 50,
  offset: 0
});

// Filter by source type
await use_mcp_tool('security-detections', 'list_by_source', {
  source_type: 'sigma' // sigma, splunk_escu, elastic, kql, sublime, crowdstrike_cql
});
```

### MITRE ATT&CK Filtering

```typescript
// Find detections for specific technique
await use_mcp_tool('security-detections', 'list_by_mitre', {
  technique_id: 'T1059.001' // PowerShell
});

// Filter by tactic
await use_mcp_tool('security-detections', 'list_by_mitre_tactic', {
  tactic: 'execution' // execution, persistence, privilege-escalation, etc.
});

// Find CVE detections
await use_mcp_tool('security-detections', 'list_by_cve', {
  cve_id: 'CVE-2023-36884'
});

// Search by process name
await use_mcp_tool('security-detections', 'list_by_process_name', {
  process_name: 'powershell.exe'
});

// Filter by severity
await use_mcp_tool('security-detections', 'list_by_severity', {
  level: 'critical' // critical, high, medium, low
});
```

### Coverage Analysis

```typescript
// Overall coverage stats (~2KB response)
const coverage = await use_mcp_tool('security-detections', 'analyze_coverage', {
  source_type: 'sigma' // optional, omit for all sources
});
// Returns: tactic coverage %, top 20 techniques, weak spots

// Quick tactic summary (~200B)
const summary = await use_mcp_tool('security-detections', 'get_coverage_summary');
// Returns: { execution: 45%, persistence: 67%, ... }

// Identify gaps for threat profile
const gaps = await use_mcp_tool('security-detections', 'identify_gaps', {
  threat_profile: 'ransomware' // ransomware, apt, persistence, privilege_escalation, lateral_movement
});
// Returns: missing techniques, low-coverage tactics, recommended detections

// Suggest detections for technique
const suggestions = await use_mcp_tool('security-detections', 'suggest_detections', {
  technique_id: 'T1078.004' // Cloud Accounts
});
// Returns: detection ideas, logic patterns, data sources needed
```

### Threat Actor Analysis

```typescript
// Analyze coverage for specific actor
const actorCoverage = await use_mcp_tool('security-detections', 'analyze_actor_coverage', {
  actor: 'APT29'
});
// Returns: covered/missing techniques, software used, gap details

// Compare coverage across multiple actors
const comparison = await use_mcp_tool('security-detections', 'compare_actor_coverage', {
  actors: ['APT29', 'APT28', 'Lazarus Group']
});
// Returns: common gaps, actor-specific weaknesses, prioritized recommendations

// Procedure-level coverage (behavioral clusters)
const procedures = await use_mcp_tool('security-detections', 'analyze_procedure_coverage', {
  technique_id: 'T1059.001',
  actor: 'APT29' // optional
});
// Returns: behavioral patterns, detection mapping, uncovered procedures
```

### ATT&CK Navigator Layers

```typescript
// Generate coverage layer (all sources)
const layer = await use_mcp_tool('security-detections', 'generate_navigator_layer', {
  layer_type: 'coverage', // coverage, gap, source, severity, actor
  source_type: 'all' // optional filter
});
// Returns: ATT&CK Navigator JSON

// Generate gap layer (missing/weak coverage)
const gapLayer = await use_mcp_tool('security-detections', 'generate_navigator_layer', {
  layer_type: 'gap',
  min_coverage: 3 // show techniques with <3 detections
});

// Actor-specific layer
const actorLayer = await use_mcp_tool('security-detections', 'generate_navigator_layer', {
  layer_type: 'actor',
  actor: 'APT29'
});

// Import into Navigator: https://mitre-attack.github.io/attack-navigator/
// File → Upload from Local → paste JSON
```

## Expert Prompts

Invoke by name in chat:

### Ransomware Readiness Assessment
```
Use the ransomware-readiness-assessment prompt
```
Analyzes coverage across ransomware kill chain (initial access → impact), identifies gaps, provides remediation plan.

### APT Emulation Plan
```
Use the apt-emulation-plan prompt for APT29
```
Generates purple team scenario with detection validation steps, coverage analysis, gaps, and Atomic Red Team mapping.

### Purple Team Exercise
```
Use the purple-team-exercise prompt for credential dumping
```
Creates detection-centric exercise plan: attack simulation, detection triggers, validation queries, tuning recommendations.

### Executive Detection Briefing
```
Use the executive-detection-briefing prompt
```
High-level coverage summary for leadership: strengths, risks, ROI of detection investments.

### Other Prompts
- `detection-gap-analysis` - Deep-dive into missing coverage
- `threat-actor-profile` - Intel report for specific actor
- `detection-engineering-plan` - Sprint planning for new detections
- `coverage-improvement-roadmap` - 30/60/90 day improvement plan
- `data-source-analysis` - Log source coverage assessment
- `technique-deep-dive` - Comprehensive technique analysis
- `detection-quality-audit` - Rule quality scoring

## Advanced Usage

### Knowledge Graph Queries

```typescript
// Find related detections
const related = await use_mcp_tool('security-detections', 'get_related_detections', {
  detection_id: 'sigma-abc123',
  limit: 10
});

// Extract detection patterns
const patterns = await use_mcp_tool('security-detections', 'learn_detection_patterns', {
  technique_id: 'T1003.001', // LSASS Memory
  max_rules: 50
});
// Returns: common fields, typical values, logic patterns for ML-assisted authoring
```

### Detection Generation

```typescript
// Generate detection from description
const newDetection = await use_mcp_tool('security-detections', 'generate_detection', {
  technique_id: 'T1078.004',
  target_format: 'sigma', // sigma, splunk, kql, elastic
  description: 'Detect cloud console login from TOR exit nodes',
  data_sources: ['CloudTrail', 'Azure AD']
});
// Returns: YAML/JSON detection rule draft

// Create from template
const fromTemplate = await use_mcp_tool('security-detections', 'create_detection_from_template', {
  template_id: 'sigma-process-creation',
  parameters: {
    process_name: 'malicious.exe',
    parent_process: 'explorer.exe'
  }
});
```

### Dynamic Tables & Sprint Planning

```typescript
// Create markdown table from detections
const table = await use_mcp_tool('security-detections', 'create_dynamic_table', {
  technique_id: 'T1059',
  columns: ['name', 'source', 'severity', 'data_sources']
});
// Use in documentation, reports, or issue tracking

// Sprint planning
const sprint = await use_mcp_tool('security-detections', 'generate_sprint_plan', {
  focus_area: 'credential_access',
  sprint_duration_weeks: 2,
  team_capacity: 5 // engineer-days
});
// Returns: prioritized detection tasks, effort estimates, acceptance criteria
```

## Configuration

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `SIGMA_PATHS` | Comma-separated Sigma rule directories | `/path/to/sigma/rules,/path/to/sigma/rules-threat-hunting` |
| `SPLUNK_PATHS` | Splunk ESCU detection directories | `/path/to/security_content/detections` |
| `ELASTIC_PATHS` | Elastic rule directories | `/path/to/detection-rules/rules` |
| `KQL_PATHS` | KQL hunting query directories | `/path/to/kql-rules` |
| `SUBLIME_PATHS` | Sublime Security rule directories | `/path/to/sublime-rules/detection-rules` |
| `CQL_HUB_PATHS` | CrowdStrike CQL directories | `/path/to/cql-hub/queries` |
| `JAMF_PROTECT_PATHS` | Jamf Protect analytic directories | `/path/to/jamf-protect/analytics` |
| `STORY_PATHS` | Splunk analytic story directories (optional) | `/path/to/security_content/stories` |
| `ATTACK_STIX_PATH` | Path to `enterprise-attack.json` (optional) | `/path/to/enterprise-attack.json` |

Download MITRE ATT&CK STIX:
```bash
curl -o enterprise-attack.json https://raw.githubusercontent.com/mitre-attack/attack-stix-data/master/enterprise-attack/enterprise-attack.json
```

### Rebuilding Index

After updating detection repos:
```typescript
await use_mcp_tool('security-detections', 'rebuild_index');
```

## Common Patterns

### Ransomware Coverage Check
```typescript
// 1. Analyze overall coverage
const coverage = await use_mcp_tool('security-detections', 'analyze_coverage');

// 2. Identify ransomware-specific gaps
const gaps = await use_mcp_tool('security-detections', 'identify_gaps', {
  threat_profile: 'ransomware'
});

// 3. Generate Navigator layer for visualization
const layer = await use_mcp_tool('security-detections', 'generate_navigator_layer', {
  layer_type: 'gap',
  min_coverage: 2
});

// 4. Get actionable suggestions
for (const gap of gaps.missing_techniques) {
  const suggestions = await use_mcp_tool('security-detections', 'suggest_detections', {
    technique_id: gap.technique_id
  });
}
```

### APT Threat Hunt
```typescript
// 1. Profile threat actor
const actorCoverage = await use_mcp_tool('security-detections', 'analyze_actor_coverage', {
  actor: 'APT29'
});

// 2. Find existing detections for actor's techniques
for (const technique of actorCoverage.techniques) {
  const detections = await use_mcp_tool('security-detections', 'list_by_mitre', {
    technique_id: technique.id
  });
}

// 3. Export actor layer for emulation planning
const actorLayer = await use_mcp_tool('security-detections', 'generate_navigator_layer', {
  layer_type: 'actor',
  actor: 'APT29'
});
```

### Detection Development Workflow
```typescript
// 1. Learn patterns from existing rules
const patterns = await use_mcp_tool('security-detections', 'learn_detection_patterns', {
  technique_id: 'T1003.001',
  max_rules: 30
});

// 2. Generate draft detection
const draft = await use_mcp_tool('security-detections', 'generate_detection', {
  technique_id: 'T1003.001',
  target_format: 'sigma',
  description: 'Detect LSASS memory dump via Mimikatz',
  data_sources: ['Sysmon', 'Windows Security']
});

// 3. Validate against similar detections
const related = await use_mcp_tool('security-detections', 'search', {
  query: 'lsass memory dump mimikatz'
});
```

## Troubleshooting

### "No detections found"
- Verify environment variables point to correct directories
- Check paths exist and contain YAML/TOML/JSON files
- Run `rebuild_index` after updating paths

### "MITRE data not loaded"
- Set `ATTACK_STIX_PATH` to `enterprise-attack.json`
- Download STIX data: `curl -o enterprise-attack.json https://raw.githubusercontent.com/mitre-attack/attack-stix-data/master/enterprise-attack/enterprise-attack.json`

### Slow search performance
- Index builds on first run (~30 seconds for 8K+ rules)
- Subsequent searches are fast (in-memory cache)
- Use filters (`list_by_mitre`, `list_by_source`) instead of broad `search`

### Claude Desktop not showing tools
- Restart Claude Desktop after config changes
- Check logs: `~/Library/Logs/Claude/mcp*.log` (macOS)
- Validate JSON syntax in `claude_desktop_config.json`

### Hosted MCP rate limits
- Free tier: 200 calls/day
- Pro: 2,000 calls/day
- Rate limit headers in response: `X-RateLimit-Remaining`

## Resources

- **Web UI**: https://detect.michaelhaag.org
- **API Docs**: https://detect.michaelhaag.org/api/docs
- **Setup Guide**: https://github.com/MHaggis/Security-Detections-MCP/blob/main/SETUP.md
- **Hosted MCP**: https://github.com/MHaggis/Security-Detections-MCP/blob/main/docs/HOSTED_MCP.md
- **Autonomous Workflows**: https://github.com/MHaggis/Security-Detections-MCP/blob/main/docs/AUTONOMOUS.md
- **Tools Reference**: https://github.com/MHaggis/Security-Detections-MCP/blob/main/docs/wiki/Tools-Reference.md
