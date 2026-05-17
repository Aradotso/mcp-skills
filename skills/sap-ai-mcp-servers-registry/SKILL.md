---
name: sap-ai-mcp-servers-registry
description: Navigate and discover SAP-related MCP servers, AI skills, and Claude plugins for SAP development workflows
triggers:
  - "Find MCP servers for SAP development"
  - "What SAP MCP servers are available?"
  - "Show me ABAP MCP servers"
  - "List SAP AI skills and plugins"
  - "Which MCP server should I use for SAP Fiori?"
  - "Find SAP BTP documentation MCP server"
  - "Search for SAP integration MCP servers"
  - "What's available for SAP HANA MCP?"
---

# SAP AI MCP Servers Registry

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill helps you navigate the comprehensive registry of SAP-related MCP servers, AI skills, and Claude plugins. The registry is maintained at `marianfoo/sap-ai-mcp-servers` and provides a curated list of open-source tools for SAP development workflows with AI coding assistants.

## What This Registry Provides

The SAP AI MCP Servers registry catalogs:

- **Official SAP MCP Servers**: Fiori, CAP, UI5, MDK, UI5 Web Components
- **SAP Skills Libraries**: Reusable AI skills for digital assistants
- **Claude Plugins**: SAP-specific plugins for Claude Code
- **Community MCP Servers**: 
  - ABAP/ADT servers
  - Documentation search servers
  - Integration Suite servers
  - OData proxy servers
  - Datasphere servers
  - And many more specialty servers

## Key Categories

### 1. Official SAP MCP Servers

**SAP Fiori MCP Server** (`SAP/open-ux-tools`)
- Fiori app generation and modification
- Apache-2.0 licensed
- 147 stars

**CAP MCP Server** (`cap-js/mcp-server`)
- AI-assisted CAP development with CDS-aware context
- Apache-2.0 licensed
- 96 stars

**UI5 MCP Server** (`UI5/mcp-server`)
- UI5-aware development for OpenUI5 and SAPUI5
- Apache-2.0 licensed
- 85 stars

**SAP MDK MCP Server** (`SAP/mdk-mcp-server`)
- AI-assisted Mobile Development Kit workflows
- Apache-2.0 licensed
- 31 stars

**UI5 Web Components MCP Server** (`UI5/webcomponents-mcp-server`)
- Component API, guidelines, and docs
- Apache-2.0 licensed
- 18 stars

### 2. SAP Skills

**SAP AI Skills Library** (`SAP/ai-skills-library`)
- SAP-maintained library with skills CLI
- Discover, search, filter, and install
- Apache-2.0 licensed

**CAP Agentic Engineered Skills** (`SAP-samples/cap-agentic-engineered`)
- Reusable MCP-routing AGENTS.md
- SAP skills for CAP and Fiori Elements
- Apache-2.0 licensed

### 3. ABAP and ADT MCP Servers

**Vibing Steampunk** (`oisee/vibing-steampunk`)
- ADT-to-MCP bridge for ABAP and AMDP
- 342 stars, MIT licensed
- Most popular ABAP MCP server

**ARC-1 SAP ADT MCP Server** (`marianfoo/arc-1`)
- Enterprise-ready, secure-by-default
- BTP OAuth and ADT REST API
- 82 stars, MIT licensed

**ABAP MCP Server SDK** (`abap-ai/mcp`)
- Build MCP servers directly in ABAP
- 68 stars, MIT licensed

**ABAP Accelerator MCP Server** (AWS Solutions Library)
- Enterprise-grade for Amazon Q Developer
- Create, test, document, transform ABAP code
- 42 stars, MIT-0 licensed

**BW Modeling MCP Server** (`dnic-dev/bw-modeling-mcp`)
- Read/create/modify SAP BW/4HANA objects
- BWMT REST API integration
- 33 stars, MIT licensed

**SAP Cloudification Repository MCP** (`ClementRingot/sap-released-objects-mcp-server`)
- Check which SAP objects are released for ABAP Cloud
- Clean Core guidance
- 24 stars, MIT licensed

### 4. Documentation Search Servers

**MCP SAP Docs** (`marianfoo/mcp-sap-docs`)
- Unified SAP developer docs search
- 177 stars, Apache-2.0 licensed

**ABAP MCP Server** (`marianfoo/abap-mcp-server`)
- ABAP-focused doc variant
- 73 stars, Apache-2.0 licensed

**SAP Notes MCP Server** (`marianfoo/mcp-sap-notes`)
- Search SAP Notes and KB content
- 49 stars, Apache-2.0 licensed

### 5. Integration Servers

**MCP Integration Suite** (`1nbuc/mcp-integration-suite`)
- SAP Integration Suite/CPI operations
- 22 stars

**CPI MCP Server** (`vadimklimov/cpi-mcp-server`)
- SAP Cloud Integration via MCP
- 17 stars, MIT licensed

**SAP CPI MCP Server** (`prudvigit/mcp-sap-cpi`)
- Monitor messages, iFlows, security artifacts
- MIT licensed

### 6. OData MCP Proxy Ecosystem

**OData MCP Proxy** (`lemaiwo/odata-mcp-proxy`)
- Foundation: config-driven MCP server
- Exposes OData/REST as MCP tools
- BTP destinations, dual transport
- 19 stars, MIT licensed

**CI MCP Server** (`lemaiwo/ci-mcp-server`)
- SAP Cloud Integration OData API
- 8 stars, MIT licensed

**AI Core MCP Server** (`lemaiwo/ai-core-mcp-server`)
- SAP AI Core lifecycle and admin APIs
- 4 stars, MIT licensed

**BTP MCP Server** (`lemaiwo/btp-mcp-server`)
- BTP Core Services as MCP tools
- 1 star, MIT licensed

### 7. Datasphere Servers

**SAP Datasphere MCP** (`MarioDeFelipe/sap-datasphere-mcp`)
- Feature-rich Datasphere API interaction
- 28 stars, MIT licensed

**SAP Datasphere MCP** (`rahulsethi/SAPDatasphereMCP`)
- Alternative Datasphere implementation

## How to Use This Registry

### Finding the Right Server

```javascript
// Decision tree for selecting an MCP server

// For Fiori development
if (task === 'fiori-app-generation') {
  use('SAP/open-ux-tools/packages/fiori-mcp-server');
}

// For CAP development
if (task === 'cap-development') {
  use('cap-js/mcp-server');
}

// For ABAP development
if (task === 'abap-development') {
  if (needs === 'enterprise-security') {
    use('marianfoo/arc-1'); // BTP OAuth
  } else if (needs === 'most-popular') {
    use('oisee/vibing-steampunk'); // 342 stars
  } else if (needs === 'cloud-core-guidance') {
    use('ClementRingot/sap-released-objects-mcp-server');
  } else if (needs === 'aws-integration') {
    use('aws-solutions-library-samples/guidance-for-deploying-sap-abap-accelerator-for-amazon-q-developer');
  }
}

// For documentation search
if (task === 'search-sap-docs') {
  use('marianfoo/mcp-sap-docs'); // Most comprehensive
}

// For SAP Notes
if (task === 'search-sap-notes') {
  use('marianfoo/mcp-sap-notes');
}

// For integration
if (task === 'sap-integration') {
  use('vadimklimov/cpi-mcp-server'); // Most stars in category
}

// For OData/REST APIs
if (task === 'expose-odata-as-mcp') {
  use('lemaiwo/odata-mcp-proxy'); // Config-driven foundation
}
```

### Installation Pattern (Generic)

```bash
# Clone the repository
git clone https://github.com/[owner]/[repo].git
cd [repo]

# Install dependencies
npm install

# Configure (typically in MCP client config)
# Example for Claude Desktop
```

```json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["/path/to/server/index.js"],
      "env": {
        "SAP_USERNAME": "${SAP_USERNAME}",
        "SAP_PASSWORD": "${SAP_PASSWORD}",
        "SAP_BASE_URL": "${SAP_BASE_URL}"
      }
    }
  }
}
```

### Common Configuration Patterns

#### ABAP/ADT Servers (OAuth/Basic Auth)

```json
{
  "mcpServers": {
    "abap-adt": {
      "command": "node",
      "args": ["/path/to/server/dist/index.js"],
      "env": {
        "ADT_BASE_URL": "${ADT_BASE_URL}",
        "ADT_USERNAME": "${ADT_USERNAME}",
        "ADT_PASSWORD": "${ADT_PASSWORD}",
        "ADT_CLIENT": "${ADT_CLIENT}"
      }
    }
  }
}
```

#### BTP Destination-based Servers

```json
{
  "mcpServers": {
    "odata-proxy": {
      "command": "node",
      "args": ["/path/to/server/dist/index.js"],
      "env": {
        "BTP_DESTINATION_NAME": "${BTP_DESTINATION_NAME}",
        "BTP_SERVICE_KEY": "${BTP_SERVICE_KEY}"
      }
    }
  }
}
```

#### Documentation Search Servers

```json
{
  "mcpServers": {
    "sap-docs": {
      "command": "npx",
      "args": ["-y", "mcp-sap-docs"]
    }
  }
}
```

## Repository Structure Patterns

Most SAP MCP servers follow these patterns:

```
project-root/
├── src/
│   ├── index.ts          # Entry point
│   ├── tools/            # MCP tool definitions
│   ├── client.ts         # SAP API client
│   └── types.ts          # TypeScript types
├── package.json
├── tsconfig.json
└── README.md
```

## Key Metrics for Selection

When choosing a server, consider:

1. **Stars**: Popularity indicator
   - 100+ stars = widely used
   - 50-100 = established
   - <50 = emerging or niche

2. **License**: Most use MIT or Apache-2.0
   - Watch for "NO LICENSE FOUND" entries

3. **Last Update**: Check "Last Change" date
   - Recent updates indicate active maintenance

4. **Scope**: Match your specific need
   - Official SAP servers: production-ready
   - Community servers: specialized use cases

## Contributing New Entries

```yaml
# Issue template: Add new entry
Category: SAP Community MCP Servers > [Subcategory]
Repository: https://github.com/owner/repo
Purpose: One-line description of what it does
Note: |
  Additional context about the server,
  unique features, or usage notes.
```

## Example: Setting Up Multiple SAP Servers

```json
{
  "mcpServers": {
    "sap-fiori": {
      "command": "node",
      "args": ["./sap-fiori-mcp/dist/index.js"]
    },
    "sap-cap": {
      "command": "node",
      "args": ["./cap-mcp-server/dist/index.js"]
    },
    "sap-docs": {
      "command": "npx",
      "args": ["-y", "mcp-sap-docs"]
    },
    "abap-adt": {
      "command": "node",
      "args": ["./arc-1/dist/index.js"],
      "env": {
        "ADT_BASE_URL": "${SAP_ADT_URL}",
        "ADT_USERNAME": "${SAP_USERNAME}",
        "ADT_PASSWORD": "${SAP_PASSWORD}",
        "ADT_CLIENT": "${SAP_CLIENT}"
      }
    },
    "sap-integration": {
      "command": "node",
      "args": ["./cpi-mcp-server/dist/index.js"],
      "env": {
        "CPI_HOST": "${CPI_HOST}",
        "CPI_USERNAME": "${CPI_USERNAME}",
        "CPI_PASSWORD": "${CPI_PASSWORD}"
      }
    }
  }
}
```

## Filtering by Category

```javascript
// JavaScript helper to filter registry by category
const categories = {
  official: ['SAP/open-ux-tools', 'cap-js/mcp-server', 'UI5/mcp-server'],
  abap: ['oisee/vibing-steampunk', 'marianfoo/arc-1', 'abap-ai/mcp'],
  docs: ['marianfoo/mcp-sap-docs', 'marianfoo/abap-mcp-server'],
  integration: ['vadimklimov/cpi-mcp-server', '1nbuc/mcp-integration-suite'],
  odata: ['lemaiwo/odata-mcp-proxy', 'lemaiwo/ci-mcp-server'],
  datasphere: ['MarioDeFelipe/sap-datasphere-mcp']
};

// Get servers by category
function getServersByCategory(category) {
  return categories[category] || [];
}
```

## Troubleshooting Common Issues

### Issue: Server Not Found in Registry

```bash
# Check CHANGELOG.md for recently added entries
curl https://raw.githubusercontent.com/marianfoo/sap-ai-mcp-servers/main/CHANGELOG.md

# Registry is updated regularly; last generated: 2026-05-17
```

### Issue: License Concerns

```javascript
// Servers marked "NO LICENSE FOUND" should be used with caution
// Check repository directly before production use

const safeLicenses = ['MIT', 'Apache-2.0', 'MIT-0'];

function isSafeLicense(license) {
  return safeLicenses.includes(license);
}
```

### Issue: Choosing Between Similar Servers

```javascript
// Example: Multiple ABAP ADT servers available
const abapServers = [
  { name: 'vibing-steampunk', stars: 342, feature: 'Most popular' },
  { name: 'arc-1', stars: 82, feature: 'Enterprise OAuth' },
  { name: 'abap-ai/mcp', stars: 68, feature: 'Build in ABAP' },
  { name: 'abap-accelerator', stars: 42, feature: 'AWS Q Developer' }
];

// Sort by stars for popularity
abapServers.sort((a, b) => b.stars - a.stars);
```

## Best Practices

1. **Start with Official Servers**: SAP-maintained servers are production-ready
2. **Check Activity**: Use servers with recent updates (Last Change < 3 months)
3. **Review Stars**: Higher star count usually indicates better community support
4. **Match Your Stack**: 
   - Fiori → `SAP/open-ux-tools`
   - CAP → `cap-js/mcp-server`
   - ABAP → Multiple options based on needs
5. **Combine Servers**: Use multiple servers for comprehensive SAP development support
6. **Environment Variables**: Always use env vars for credentials, never hardcode

## Registry Maintenance

The registry is:
- Auto-generated regularly
- Last generated: **2026-05-17**
- Excludes forks automatically
- Accepts non-GitHub git hosts
- Community-driven via issue templates

To stay updated:
```bash
# Watch the repository for updates
# Check CHANGELOG.md for new entries
# Review updated_at dates in metadata
```
