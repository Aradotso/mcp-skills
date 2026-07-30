---
name: patsnap-patent-literature-search
description: Search 200M+ patents and scientific literature using PatSnap's MCP server with natural language queries
triggers:
  - search patents for prior art
  - find academic papers about
  - look up patent information
  - search scientific literature on
  - find patents by assignee
  - research intellectual property for
  - what patents exist for
  - search both patents and papers about
---

# PatSnap Patent & Literature Search MCP

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

PatSnap Patent & Literature Search MCP provides AI agents with direct access to 200M+ patents across 170+ jurisdictions (USPTO, EPO, WIPO) and 216M+ scientific papers. Search using natural language queries without learning Boolean syntax. Supports fusion search (patents + literature), semantic search, keyword/BM25 text search, and precise filtering by assignee, inventor, IPC class, legal status, jurisdiction, date range, and citations.

## Installation

### Claude Desktop

```bash
claude mcp add --transport http search-tool \
  "https://connect.patsnap.com/2b0355/logic-mcp?apikey=${PATSNAP_API_KEY}"
```

### Cursor or Windsurf

Add to MCP configuration file (`~/Library/Application Support/Cursor/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json` or equivalent):

```json
{
  "mcpServers": {
    "patsnap_patent_literature": {
      "url": "https://connect.patsnap.com/2b0355/logic-mcp?apikey=${PATSNAP_API_KEY}",
      "type": "streamableHttp"
    }
  }
}
```

### Docker (stdio bridge)

```bash
docker run --rm -i \
  -e PATSNAP_API_KEY=${PATSNAP_API_KEY} \
  ghcr.io/patsnap/patent-literature-search-mcp:latest
```

MCP client configuration for Docker:

```json
{
  "mcpServers": {
    "patsnap_patent_literature": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-e",
        "PATSNAP_API_KEY",
        "ghcr.io/patsnap/patent-literature-search-mcp:latest"
      ],
      "env": {
        "PATSNAP_API_KEY": "${PATSNAP_API_KEY}"
      }
    }
  }
}
```

### API Key

1. Register at [Patsnap Open Platform](https://open.patsnap.com)
2. Navigate to [Patent & Literature Search MCP page](https://open.patsnap.com/marketplace/mcp-servers/patsnap-search)
3. Generate API key (10,000 free credits for new accounts)
4. Set environment variable: `export PATSNAP_API_KEY=your_key_here`

## Available Tools

### `patsnap_search`

Search patents or scientific literature with natural language or structured queries.

**Parameters:**
- `source` (required): `"patent"` or `"literature"`
- `query` (required): Natural language search query or keywords
- `filters` (optional): Object with filter criteria
  - `assignee`: Company/organization name
  - `inventor`: Inventor name
  - `ipc_class`: International Patent Classification code
  - `legal_status`: Patent legal status
  - `jurisdiction`: Country/region code (e.g., "US", "EP", "CN")
  - `date_from`, `date_to`: Date range (ISO 8601 format)
  - `min_citations`: Minimum citation count
- `page` (optional): Page number (default: 1)
- `page_size` (optional): Results per page (default: 10, max: 50)

**Example: Patent search with filters**

```javascript
{
  "source": "patent",
  "query": "solid state battery lithium metal anode",
  "filters": {
    "assignee": "Toyota",
    "date_from": "2020-01-01",
    "jurisdiction": "US",
    "legal_status": "granted"
  },
  "page": 1,
  "page_size": 20
}
```

**Example: Literature search**

```javascript
{
  "source": "literature",
  "query": "GLP-1 receptor agonist cardiovascular outcomes",
  "filters": {
    "date_from": "2022-01-01",
    "min_citations": 10
  },
  "page_size": 15
}
```

**Example: IPC classification search**

```javascript
{
  "source": "patent",
  "query": "neural network image recognition",
  "filters": {
    "ipc_class": "G06N3/08",
    "date_from": "2023-01-01"
  }
}
```

### `patsnap_fetch`

Retrieve detailed patent or literature records as Markdown.

**Parameters:**
- `urls` (optional): Array of result URLs from `patsnap_search`
- `publication_numbers` (optional): Array of patent publication numbers (e.g., `["US10123456B2", "EP3456789A1"]`)

**Example: Fetch by URLs**

```javascript
{
  "urls": [
    "https://analytics.zhihuiya.com/patent-view?patentId=abc123",
    "https://analytics.zhihuiya.com/patent-view?patentId=def456"
  ]
}
```

**Example: Fetch by publication numbers**

```javascript
{
  "publication_numbers": ["US11234567B2", "WO2023012345A1"]
}
```

## Common Usage Patterns

### Prior Art Search

Search for existing patents before filing:

```javascript
// Step 1: Search patents
{
  "source": "patent",
  "query": "augmented reality waveguide display optical combiner",
  "filters": {
    "date_to": "2023-06-01",
    "jurisdiction": "US"
  },
  "page_size": 50
}

// Step 2: Fetch detailed records of top results
{
  "urls": ["<url1>", "<url2>", "<url3>"]
}
```

### Technology Landscape Analysis

Map patent activity by assignee and technology area:

```javascript
// Search broad technology area
{
  "source": "patent",
  "query": "electric vehicle battery thermal management",
  "filters": {
    "date_from": "2020-01-01"
  },
  "page_size": 50
}

// Follow up with assignee-specific searches
{
  "source": "patent",
  "query": "battery thermal management",
  "filters": {
    "assignee": "Tesla",
    "date_from": "2020-01-01"
  }
}
```

### Fusion Search (Patents + Literature)

Combine patent and academic research:

```javascript
// Search patents
{
  "source": "patent",
  "query": "CRISPR base editing adenine deaminase",
  "filters": {
    "date_from": "2021-01-01",
    "legal_status": "granted"
  }
}

// Search literature on same topic
{
  "source": "literature",
  "query": "CRISPR base editing adenine deaminase efficiency",
  "filters": {
    "date_from": "2021-01-01",
    "min_citations": 5
  }
}
```

### Patent Expiration Monitoring

Find patents expiring soon:

```javascript
{
  "source": "patent",
  "query": "GLP-1 receptor agonist diabetes treatment",
  "filters": {
    "legal_status": "granted",
    "date_to": "2008-12-31" // Filed ~20 years ago
  },
  "page_size": 30
}
```

### Inventor/Assignee Tracking

Monitor specific organizations or inventors:

```javascript
{
  "source": "patent",
  "query": "quantum computing error correction",
  "filters": {
    "assignee": "IBM",
    "date_from": "2023-01-01"
  },
  "page_size": 25
}
```

## Response Structure

### Search Response

```json
{
  "total": 1234,
  "page": 1,
  "page_size": 10,
  "results": [
    {
      "title": "Solid-state battery with lithium metal anode",
      "url": "https://analytics.zhihuiya.com/patent-view?patentId=...",
      "publication_number": "US11234567B2",
      "assignee": "Toyota Motor Corporation",
      "filing_date": "2021-03-15",
      "publication_date": "2023-01-24",
      "abstract": "A solid-state battery comprising...",
      "ipc_classes": ["H01M10/0562", "H01M10/0525"],
      "citations": 12
    }
  ]
}
```

### Fetch Response

Returns Markdown-formatted detailed records including:
- **Patents**: Bibliographic data, claims, description, drawings, citations, legal status, patent family
- **Literature**: Title, authors, journal, DOI, abstract, citations, publication info

## Configuration

### Environment Variables

- `PATSNAP_API_KEY` (required): API key from Patsnap Open Platform

### Docker Build (for self-hosting)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY src ./src
CMD ["node", "src/index.js"]
```

Build and run:

```bash
docker build -t patsnap-search-mcp .
docker run --rm -i -e PATSNAP_API_KEY=${PATSNAP_API_KEY} patsnap-search-mcp
```

## Troubleshooting

### Connection Issues

**Problem**: MCP server not appearing in Claude/Cursor

**Solution**:
- Verify API key is set: `echo $PATSNAP_API_KEY`
- Check MCP configuration file syntax (valid JSON)
- Restart IDE/Claude Desktop after configuration changes
- Test with: `claude mcp list` (Claude Desktop)

### Search Returns No Results

**Problem**: Query returns zero results

**Solution**:
- Simplify query (remove overly specific terms)
- Remove or relax filters (date range, jurisdiction)
- Try different keyword combinations
- Use broader IPC classes (e.g., "H01M10" instead of "H01M10/0562")
- Check spelling of assignee/inventor names

### API Rate Limits

**Problem**: Request fails with rate limit error

**Solution**:
- Free tier: 10,000 credits (check usage on Patsnap Open Platform)
- Reduce `page_size` to consume fewer credits
- Implement exponential backoff for repeated searches
- Upgrade account for higher limits

### Invalid Publication Number

**Problem**: `patsnap_fetch` fails with publication number

**Solution**:
- Use standardized format: `US11234567B2` (country code + number + kind code)
- Common formats: `US10123456B2`, `EP3456789A1`, `WO2023012345A1`
- Extract from `publication_number` field in search results

### Docker Container Exits Immediately

**Problem**: Container starts but exits without output

**Solution**:
- Ensure `-i` flag is present: `docker run --rm -i`
- Verify MCP client sends proper stdio handshake
- Check container logs: `docker logs <container_id>`
- Confirm `PATSNAP_API_KEY` is passed: `docker run -e PATSNAP_API_KEY`

## Best Practices

1. **Start broad, then filter**: Begin with general queries, analyze results, then add filters
2. **Combine sources**: Use both patent and literature searches for comprehensive research
3. **Iterate on queries**: Refine based on initial results (synonyms, related terms)
4. **Use fetch selectively**: Only fetch full records for most relevant results (saves credits)
5. **Leverage IPC codes**: Use classification codes for precise technical searching
6. **Date ranges**: Narrow by date to focus on recent innovations or identify expiring patents
7. **Citation counts**: Filter by citations to find influential work (especially for literature)

## Example Workflows

### Competitive Intelligence

```javascript
// 1. Find all patents by competitor
const competitorPatents = await patsnap_search({
  source: "patent",
  query: "autonomous vehicle perception lidar",
  filters: { assignee: "Waymo" },
  page_size: 50
});

// 2. Analyze their recent focus areas
const recentPatents = await patsnap_search({
  source: "patent",
  query: "autonomous vehicle",
  filters: {
    assignee: "Waymo",
    date_from: "2023-01-01"
  }
});

// 3. Fetch details for key patents
const details = await patsnap_fetch({
  urls: competitorPatents.results.slice(0, 5).map(r => r.url)
});
```

### Freedom to Operate (FTO)

```javascript
// 1. Search active patents in target jurisdiction
const activePatents = await patsnap_search({
  source: "patent",
  query: "mRNA vaccine lipid nanoparticle formulation",
  filters: {
    jurisdiction: "US",
    legal_status: "granted",
    date_from: "2003-01-01" // Within 20-year term
  },
  page_size: 50
});

// 2. Fetch full claims for analysis
const claims = await patsnap_fetch({
  publication_numbers: activePatents.results.map(r => r.publication_number)
});
```

## Resources

- [Patsnap Open Platform](https://open.patsnap.com)
- [API Documentation](https://open.patsnap.com/marketplace/mcp-servers/patsnap-search)
- [Eureka AI Assistant](https://eureka.patsnap.com)
- [GitHub Repository](https://github.com/patsnap/patent-literature-search-mcp)
- [All Patsnap MCP Servers](https://github.com/patsnap/mcp)
