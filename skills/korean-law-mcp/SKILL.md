---
name: korean-law-mcp
description: MCP server providing access to 41 Korean legal APIs (법제처) with citation verification, impact analysis, and time-travel diff capabilities
triggers:
  - search korean law or legal statute
  - verify legal citations for hallucinations
  - analyze law impact map or dependencies
  - compare law versions between dates
  - get korean precedent or court decision
  - create citizen action plan for legal issue
  - check ordinance compliance with superior law
  - search korean administrative rules
---

# korean-law-mcp

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**korean-law-mcp** is a Model Context Protocol (MCP) server that transforms 41 Korean Ministry of Government Legislation (법제처) APIs into 17 streamlined tools. It provides comprehensive access to Korean legal information including statutes, precedents, administrative rules, local ordinances, treaties, and interpretations with advanced features like:

- **Citation verification** to detect LLM hallucinations in legal responses
- **Impact mapping** showing downstream effects of legal provisions
- **Time-travel diff** comparing statute text between arbitrary dates
- **Citizen action plans** converting natural language problems into 5-step legal guides

The server integrates with Claude Desktop, Cursor, Windsurf, Zed, and other MCP clients. All data comes from official 법제처 (Korea Legislation Research Institute) APIs.

## Installation

### NPX (Recommended)

Add to your MCP client configuration (e.g., `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "korean-law": {
      "command": "npx",
      "args": [
        "-y",
        "korean-law-mcp"
      ],
      "env": {
        "MOLEG_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

### Global Install

```bash
npm install -g korean-law-mcp
korean-law-mcp
```

### Local Development

```bash
git clone https://github.com/chrisryugj/korean-law-mcp.git
cd korean-law-mcp
npm install
npm run build
npm start
```

## Configuration

### Required Environment Variables

```bash
# 법제처 Open API key (required)
MOLEG_API_KEY=your_api_key_here
```

Get your API key from: https://www.law.go.kr/DRF/lawService.do

### Optional Environment Variables

```bash
# Server configuration
PORT=3000                    # HTTP server port (default: 3000)
TRUST_PROXY=1               # Trust X-Forwarded-For (default: 1)
LAW_USER_AGENT=Mozilla/5.0  # Custom User-Agent for API calls

# Rate limiting (per IP)
RATE_LIMIT_WINDOW_MS=60000  # Window in ms (default: 60000)
RATE_LIMIT_MAX_REQUESTS=100 # Max requests per window (default: 100)
```

## Core Tools

### 1. Citation Verification (`verify_citations`)

Detects LLM hallucinations by cross-referencing cited legal provisions against official databases.

```typescript
// Natural language trigger
"민법 제750조에 따라 불법행위 손해배상을 청구하고, 형법 제9999조는 가중처벌을 정한다"

// Tool input
{
  "userText": "민법 제750조에 따라 불법행위 손해배상을 청구하고, 형법 제9999조는 가중처벌을 정한다"
}

// Response structure
{
  "citations": [
    {
      "lawName": "민법",
      "article": "750",
      "status": "verified",
      "exists": true,
      "title": "불법행위의 내용"
    },
    {
      "lawName": "형법", 
      "article": "9999",
      "status": "invalid",
      "exists": false,
      "reason": "해당 조문 없음 (존재 범위: 제1조~제372조)"
    }
  ],
  "summary": "검증 결과: 2건 중 ✓ 1건 실존, ✗ 1건 환각"
}
```

### 2. Impact Map (`impact_map`)

Generates dependency graphs showing which precedents, decisions, and regulations cite a specific provision.

```typescript
// Natural language trigger
"민법 제103조 인용한 판례"

// Tool input
{
  "lawName": "민법",
  "article": "103"
}

// Response includes Mermaid graph code
{
  "graph": "graph LR\n    민법_제103조[\"⚖️ 민법 제103조\"] --> P[\"📚 대법원 판례\"]\n    ...",
  "precedents": [...],
  "constitutionalDecisions": [...],
  "localOrdinances": [...]
}
```

### 3. Time Travel Diff (`time_travel`)

Compares statute text between two dates with automatic article-level diff.

```typescript
// Natural language trigger
"개인정보보호법 2020-01-01 vs 2025-11-01"

// Tool input
{
  "lawName": "개인정보보호법",
  "dateA": "20200101",
  "dateB": "20251101"
}

// Response structure
{
  "metadata": {
    "lawName": "개인정보보호법",
    "dateA": "2020-01-01",
    "dateB": "2025-11-01"
  },
  "diff": [
    {
      "article": "제15조",
      "changeType": "modified",
      "before": "...",
      "after": "...",
      "charDelta": "+120"
    },
    {
      "article": "제39조의2",
      "changeType": "added",
      "after": "..."
    }
  ]
}
```

### 4. Citizen Action Plan (`action_plan`)

Converts natural language legal problems into structured 5-step guides.

```typescript
// Natural language trigger
"전세금 못 받았어"

// Tool input
{
  "situation": "전세금 못 받았어"
}

// Response structure
{
  "steps": [
    {
      "step": 1,
      "title": "상황진단",
      "content": "주택임대차보호법 제3조의2 (임차권등기명령)",
      "relevantLaws": ["주택임대차보호법"]
    },
    {
      "step": 2,
      "title": "권리/구제수단",
      "precedents": [...]
    },
    {
      "step": 3,
      "title": "신청기관/기한",
      "agencies": ["지방법원", "임대차분쟁조정위원회"]
    },
    {
      "step": 4,
      "title": "필요서류/양식",
      "documents": [...]
    },
    {
      "step": 5,
      "title": "함정/주의사항",
      "warnings": ["소멸시효 3년", "법률구조공단 무료 지원"]
    }
  ]
}
```

### 5. Search Laws (`search_law`)

Search Korean statutes by keyword.

```typescript
// Tool input
{
  "query": "개인정보",
  "display": 10
}

// Response
{
  "results": [
    {
      "lawId": "001615",
      "lawName": "개인정보 보호법",
      "lawAbbr": "개인정보보호법",
      "promulgationDate": "20110329",
      "enforcementDate": "20110930"
    }
  ],
  "nextSteps": "💡 다음: get_law_text({lawName: '개인정보 보호법'})"
}
```

### 6. Get Law Text (`get_law_text`)

Retrieve full statute text with hierarchical structure.

```typescript
// Tool input
{
  "lawName": "민법",
  "enforcementDate": "20250101" // optional
}

// Response
{
  "lawName": "민법",
  "articles": [
    {
      "articleNumber": "1",
      "articleTitle": "법원",
      "paragraphs": [
        {
          "paragraphNumber": "1",
          "content": "민사에 관하여 법률에 규정이 없으면..."
        }
      ]
    }
  ]
}
```

### 7. Get Precedents (`get_precedents`)

Search court decisions, constitutional decisions, and administrative appeals.

```typescript
// Tool input
{
  "query": "부당해고",
  "decisionType": "precedent", // or "constitutional", "admin_appeal"
  "display": 5
}

// Response (with compact mode by default)
{
  "decisions": [
    {
      "caseNumber": "2023두12345",
      "court": "대법원",
      "decisionDate": "2023-05-15",
      "caseTitle": "부당해고구제재심판정취소",
      "summary": "판시사항 [800자] ... [중략] ... 주문 [400자]",
      "referenceLaws": ["근로기준법 제23조"],
      "referencePrecedents": ["2020.3.26. 2018두56077"]
    }
  ]
}
```

Add `"full": true` to get complete decision text:

```typescript
{
  "query": "부당해고",
  "decisionType": "precedent",
  "full": true
}
```

## Common Patterns

### Multi-Step Legal Analysis

```typescript
// Step 1: Search relevant law
search_law({ query: "식품위생법" })

// Step 2: Get full text
get_law_text({ lawName: "식품위생법" })

// Step 3: Check precedents
get_precedents({ 
  query: "식품위생법 영업정지", 
  decisionType: "admin_appeal" 
})

// Step 4: Get penalty schedules (별표)
get_annex({ 
  lawName: "식품위생법", 
  annexType: "별표" 
})

// Step 5: Check administrative rules
search_administrative_rules({ query: "식품위생법 시행규칙" })
```

### Contract Review Workflow

```typescript
// 1. Verify all cited provisions
verify_citations({ 
  userText: "[contract text with legal citations]" 
})

// 2. For each valid citation, get impact map
impact_map({ 
  lawName: "민법", 
  article: "103" 
})

// 3. Check if provisions have changed recently
time_travel({ 
  lawName: "민법",
  dateA: "20200101",
  dateB: "20250101"
})
```

### Regulatory Compliance Check

```typescript
// 1. Find all relevant regulations
search_law({ query: "개인정보" })
search_administrative_rules({ query: "개인정보" })

// 2. Get enforcement dates
get_law_text({ 
  lawName: "개인정보 보호법",
  enforcementDate: "20250101" 
})

// 3. Check interpretations
get_interpretations({ query: "개인정보 처리방침" })

// 4. Review recent precedents
get_precedents({ 
  query: "개인정보 유출",
  decisionType: "precedent" 
})
```

## Advanced Features

### Annexes and Forms (`get_annex`)

Retrieve penalty schedules (별표), forms (서식), and attachments.

```typescript
{
  "lawName": "식품위생법",
  "annexType": "별표",
  "annexNumber": "1"
}
```

### Treaty Search (`search_treaty`)

Search international treaties registered in Korea.

```typescript
{
  "query": "FTA",
  "country": "미국" // optional
}
```

### Local Ordinances (`search_ordinance`)

Search municipal and provincial ordinances.

```typescript
{
  "query": "주차",
  "jurisdiction": "서울특별시" // optional
}
```

### Administrative Rules (`search_administrative_rules`)

Search regulations, directives, and notices.

```typescript
{
  "query": "건축법 시행규칙",
  "ruleType": "훈령" // optional: 훈령, 예규, 고시
}
```

## Troubleshooting

### API Key Issues

**Error**: `사용자 정보 검증에 실패하였습니다`

```bash
# Verify API key is set
echo $MOLEG_API_KEY

# Check API key validity at:
# https://www.law.go.kr/DRF/lawService.do
```

### Rate Limiting

**Error**: `Too Many Requests`

```bash
# Increase rate limit in config
RATE_LIMIT_MAX_REQUESTS=200
RATE_LIMIT_WINDOW_MS=60000
```

### User-Agent Blocking

**Error**: `fetch failed` or `[EXTERNAL_API_ERROR]`

법제처 API blocks Node.js default User-Agent. The server automatically uses a browser UA, but you can override:

```bash
LAW_USER_AGENT="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
```

### Citation Verification False Negatives

If `verify_citations` shows valid citations as invalid:

1. **Check law name matching**: Use full official names (e.g., "민법" not "민사법")
2. **Article format**: Use `제123조` or just `123조`, not `123`
3. **Increase search depth**: The tool uses `searchDisplay=100` internally

### Empty Results

**Symptom**: Valid queries return `[NOT_FOUND]`

```typescript
// Check enforcement date
get_law_text({ 
  lawName: "개인정보 보호법",
  enforcementDate: "20110930" // Must be after promulgation
})

// Try broader search
search_law({ 
  query: "개인정보",
  display: 50 
})
```

### Time Travel Diff Issues

**Error**: No diff output or "identical" when laws changed

```bash
# Ensure date format is YYYYMMDD
dateA: "20200101"  # ✓ Correct
dateA: "2020-01-01" # ✗ Wrong

# Check if law existed at both dates
# Use get_law_text with each date first
```

## Integration Examples

### Claude Desktop Configuration

```json
{
  "mcpServers": {
    "korean-law": {
      "command": "npx",
      "args": ["-y", "korean-law-mcp"],
      "env": {
        "MOLEG_API_KEY": "your-key-here",
        "RATE_LIMIT_MAX_REQUESTS": "150"
      }
    }
  }
}
```

### Cursor Configuration

Add to `.cursor/config.json`:

```json
{
  "mcp": {
    "servers": {
      "korean-law": {
        "command": "npx",
        "args": ["-y", "korean-law-mcp"],
        "env": {
          "MOLEG_API_KEY": "your-key-here"
        }
      }
    }
  }
}
```

### Custom MCP Connector (claude.ai)

```bash
# Deploy to fly.io or Vercel
npm run build
fly deploy

# Use in claude.ai custom connector:
# https://korean-law-mcp.fly.dev/mcp?oc=YOUR_API_KEY
```

## Natural Language Usage

The AI will automatically route these queries to appropriate tools:

- "민법 750조가 실제로 존재하나?" → `verify_citations`
- "근로기준법 개정 이력" → `time_travel` + `get_law_text`
- "건축허가 절차 법적 근거" → `search_law` + `search_administrative_rules`
- "전세금 못 받았을 때 대응 방법" → `action_plan`
- "식품위생법 과태료 감경 가능?" → `get_annex` + `get_precedents`
- "주차 조례 상위법 적합성" → `search_ordinance` + `get_precedents`

## Performance Tips

1. **Use compact mode for precedents**: Default compact saves ~74% tokens. Only use `full: true` when you need complete text.

2. **Chain related queries**: The server suggests next steps in `nextSteps` field - copy and execute them.

3. **Cache law text**: If analyzing multiple articles of the same law, call `get_law_text` once and reference locally.

4. **Batch verification**: Include all citations in one `verify_citations` call rather than one per citation.

## References

- **Official API Docs**: https://www.law.go.kr/DRF/lawService.do
- **NPM Package**: https://www.npmjs.com/package/korean-law-mcp
- **GitHub Repository**: https://github.com/chrisryugj/korean-law-mcp
- **MCP Protocol**: https://modelcontextprotocol.io
