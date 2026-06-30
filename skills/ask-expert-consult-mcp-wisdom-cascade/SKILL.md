---
name: ask-expert-consult-mcp-wisdom-cascade
description: AI-powered expert consultation MCP server for Claude Desktop and Cursor that connects developers to enriched guidance through OpenRouter models
triggers:
  - "consult an expert about this code problem"
  - "ask for expert guidance on this architecture"
  - "get wisdom cascade consultation for this issue"
  - "connect to OpenRouter expert agent for help"
  - "query the expert consultation MCP server"
  - "use ask-expert to analyze this code"
  - "get AI mentor feedback through wisdom cascade"
  - "request expert consult via MCP"
---

# ask-expert-consult-mcp

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

An MCP (Model Context Protocol) server that provides AI-powered expert consultation capabilities through OpenRouter's model routing. This server enables Claude Desktop, Cursor, and other MCP-compatible AI agents to request structured expert guidance, code reviews, and architectural advice by leveraging multiple AI models through a unified interface.

## What It Does

The ask-expert-consult-mcp server acts as a bridge between your coding environment and OpenRouter's AI model ecosystem. It provides:

- **Expert consultation requests** with automatic domain categorization
- **Multi-model AI routing** for specialized guidance (frontend, backend, DevOps, etc.)
- **Context enrichment** that appends relevant documentation and patterns
- **Structured response formats** including code diffs, walkthroughs, and architecture suggestions
- **Knowledge persistence** through a local wisdom cache
- **Multilingual query support** with automatic language detection

## Installation

### Prerequisites

- Node.js 18+ or Python 3.10+
- OpenRouter API key ([get one here](https://openrouter.ai/keys))
- Claude Desktop or Cursor IDE with MCP support

### Setup for Claude Desktop

1. **Clone and install the server:**

```bash
git clone https://github.com/pakgik01/ask-expert-consult-mcp.git
cd ask-expert-consult-mcp
npm install
# or
pip install -r requirements.txt
```

2. **Configure environment variables:**

Create a `.env` file in the project root:

```bash
OPENROUTER_API_KEY=your_openrouter_api_key_here
WISDOM_DB_PATH=./wisdom_cache.db
DEFAULT_MODEL=anthropic/claude-3.5-sonnet
CONFIDENCE_THRESHOLD=0.75
```

3. **Add to Claude Desktop config:**

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%/Claude/claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "ask-expert-consult": {
      "command": "node",
      "args": ["/absolute/path/to/ask-expert-consult-mcp/src/index.js"],
      "env": {
        "OPENROUTER_API_KEY": "your_openrouter_api_key_here"
      }
    }
  }
}
```

For Python implementation:

```json
{
  "mcpServers": {
    "ask-expert-consult": {
      "command": "python",
      "args": ["/absolute/path/to/ask-expert-consult-mcp/src/server.py"],
      "env": {
        "OPENROUTER_API_KEY": "your_openrouter_api_key_here"
      }
    }
  }
}
```

4. **Restart Claude Desktop** to load the MCP server.

### Setup for Cursor IDE

Add to your Cursor MCP settings (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "ask-expert-consult": {
      "command": "node",
      "args": ["/absolute/path/to/ask-expert-consult-mcp/src/index.js"],
      "env": {
        "OPENROUTER_API_KEY": "${OPENROUTER_API_KEY}"
      }
    }
  }
}
```

## Core MCP Tools

The server exposes the following MCP tools that AI agents can invoke:

### 1. `ask_expert`

Request expert guidance on a coding problem or architectural decision.

**Parameters:**
- `question` (string, required): The query or problem description
- `code_snippet` (string, optional): Relevant code context
- `domain` (string, optional): Specific domain (frontend, backend, devops, database, security)
- `language` (string, optional): Programming language or framework
- `difficulty` (string, optional): junior, intermediate, senior

**Example invocation:**

```typescript
// AI agent calls this tool internally
const response = await callTool("ask_expert", {
  question: "How do I prevent race conditions in this async function?",
  code_snippet: `
async function updateUserBalance(userId, amount) {
  const user = await db.users.findById(userId);
  user.balance += amount;
  await user.save();
}
  `,
  domain: "backend",
  language: "javascript",
  difficulty: "intermediate"
});
```

**Response structure:**

```json
{
  "guidance": "Detailed step-by-step solution...",
  "code_diff": "Suggested code changes with before/after...",
  "alternatives": ["Alternative approach 1", "Alternative approach 2"],
  "confidence_score": 0.92,
  "related_patterns": ["Transaction pattern", "Optimistic locking"],
  "wisdom_id": "wsd_a7b3c9d2"
}
```

### 2. `search_wisdom`

Search previously answered questions in the local knowledge cache.

**Parameters:**
- `query` (string, required): Search terms
- `domain` (string, optional): Filter by domain
- `limit` (number, optional): Maximum results (default: 5)

**Example:**

```typescript
const cached = await callTool("search_wisdom", {
  query: "race condition async javascript",
  domain: "backend",
  limit: 3
});
```

### 3. `enrich_context`

Enhance a code question with relevant documentation and patterns before asking.

**Parameters:**
- `code_snippet` (string, required): Code to analyze
- `language` (string, required): Programming language
- `include_docs` (boolean, optional): Append official documentation excerpts

**Example:**

```typescript
const enriched = await callTool("enrich_context", {
  code_snippet: "const result = await fetch(url).then(r => r.json());",
  language: "javascript",
  include_docs: true
});
```

## Usage Patterns

### Pattern 1: Quick Code Review

When a user asks for code feedback:

```typescript
// User: "Can you review this React component?"

// Agent workflow:
// 1. Enrich the context
const context = await callTool("enrich_context", {
  code_snippet: userProvidedCode,
  language: "javascript",
  include_docs: false
});

// 2. Ask expert with enriched context
const review = await callTool("ask_expert", {
  question: "Please review this React component for best practices and potential issues",
  code_snippet: context.enriched_code,
  domain: "frontend",
  language: "react"
});

// 3. Present structured feedback to user
```

### Pattern 2: Architecture Consultation

For high-level design questions:

```typescript
// User: "How should I structure a microservices API gateway?"

// Agent workflow:
// 1. Check wisdom cache first
const cached = await callTool("search_wisdom", {
  query: "microservices API gateway architecture",
  domain: "backend"
});

if (cached.results.length > 0) {
  // Present cached wisdom with citation
} else {
  // 2. Request fresh expert guidance
  const guidance = await callTool("ask_expert", {
    question: "How should I structure a microservices API gateway for a Node.js backend?",
    domain: "backend",
    language: "nodejs",
    difficulty: "senior"
  });
}
```

### Pattern 3: Multi-Domain Consultation

For complex questions spanning multiple domains:

```typescript
// User: "How do I optimize this full-stack feature?"

// Agent workflow:
// 1. Ask frontend expert
const frontendAdvice = await callTool("ask_expert", {
  question: "Optimize this React component rendering",
  code_snippet: reactCode,
  domain: "frontend"
});

// 2. Ask backend expert
const backendAdvice = await callTool("ask_expert", {
  question: "Optimize this database query",
  code_snippet: sqlQuery,
  domain: "database"
});

// 3. Synthesize both responses
```

## Configuration Options

### Environment Variables

```bash
# Required
OPENROUTER_API_KEY=sk-or-v1-...

# Optional - Model Selection
DEFAULT_MODEL=anthropic/claude-3.5-sonnet
FRONTEND_MODEL=anthropic/claude-3-opus
BACKEND_MODEL=openai/gpt-4-turbo
DATABASE_MODEL=anthropic/claude-3.5-sonnet

# Optional - Behavior
CONFIDENCE_THRESHOLD=0.75        # Minimum confidence for auto-responses
WISDOM_DB_PATH=./wisdom_cache.db # Local knowledge cache location
MAX_CONTEXT_LENGTH=8000          # Maximum token context per request
CACHE_TTL_DAYS=90                # Days to keep wisdom entries
AUTO_TRANSLATE=true              # Enable multilingual detection

# Optional - Privacy
ANONYMIZE_AFTER_DAYS=90          # Auto-anonymize old queries
RETAIN_RAW_RESPONSES=false       # Don't store raw OpenRouter responses
```

### Model Routing Configuration

Create a `wisdom.config.json` for advanced model routing:

```json
{
  "domainModels": {
    "frontend": "anthropic/claude-3-opus",
    "backend": "openai/gpt-4-turbo",
    "devops": "anthropic/claude-3.5-sonnet",
    "database": "anthropic/claude-3.5-sonnet",
    "security": "openai/gpt-4-turbo",
    "general": "anthropic/claude-3.5-sonnet"
  },
  "fallbackModel": "anthropic/claude-3-haiku",
  "maxRetries": 2,
  "timeout": 30000
}
```

## Real-World Examples

### Example 1: Debugging Async Issue

```typescript
// User asks: "Why is my async function not waiting?"

const response = await callTool("ask_expert", {
  question: "This async function returns undefined instead of waiting for the promise",
  code_snippet: `
function getData() {
  return fetch('/api/data')
    .then(res => res.json());
}

async function processData() {
  const data = getData();  // Missing await!
  console.log(data);       // Logs Promise, not data
}
  `,
  domain: "backend",
  language: "javascript",
  difficulty: "junior"
});

// Response includes:
// - Explanation of missing await keyword
// - Corrected code with await
// - Alternative using .then() pattern
// - Related patterns: "Promise chaining", "Async/await best practices"
```

### Example 2: Security Review

```typescript
// User asks: "Is this authentication secure?"

const response = await callTool("ask_expert", {
  question: "Review this authentication implementation for security vulnerabilities",
  code_snippet: `
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  const user = db.query('SELECT * FROM users WHERE username = ? AND password = ?', 
    [username, password]);
  if (user) {
    req.session.userId = user.id;
    res.json({ success: true });
  }
});
  `,
  domain: "security",
  language: "javascript",
  difficulty: "intermediate"
});

// Response flags:
// - Missing password hashing
// - SQL injection vulnerability (though parameterized)
// - No rate limiting
// - Suggests bcrypt, prepared statements, express-rate-limit
```

### Example 3: Performance Optimization

```typescript
// User asks: "How do I optimize this database query?"

const response = await callTool("ask_expert", {
  question: "This query is slow on large datasets. How can I optimize it?",
  code_snippet: `
SELECT u.*, p.*, c.*
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
LEFT JOIN comments c ON p.id = c.post_id
WHERE u.created_at > '2024-01-01'
ORDER BY u.created_at DESC;
  `,
  domain: "database",
  language: "sql",
  difficulty: "intermediate"
});

// Response suggests:
// - Add index on users.created_at
// - Paginate results with LIMIT/OFFSET
// - Consider separate queries for N+1 prevention
// - Use EXPLAIN ANALYZE to profile
```

## Troubleshooting

### Server Not Appearing in Claude Desktop

1. **Check config path**: Ensure absolute paths in `claude_desktop_config.json`
2. **Verify API key**: Test key at https://openrouter.ai/
3. **Check logs**: 
   ```bash
   tail -f ~/Library/Logs/Claude/mcp*.log  # macOS
   ```
4. **Restart Claude**: Completely quit and reopen

### "Model not found" Errors

- Verify model name in OpenRouter docs: https://openrouter.ai/docs#models
- Check your OpenRouter account has credits
- Try fallback model: `anthropic/claude-3-haiku`

### Low Confidence Scores

If responses frequently have low confidence (<0.75):

1. Provide more code context in `code_snippet`
2. Specify the `domain` explicitly
3. Enrich context first with `enrich_context` tool
4. Lower `CONFIDENCE_THRESHOLD` in config (not recommended)

### Slow Response Times

- Reduce `MAX_CONTEXT_LENGTH` to decrease token processing
- Use faster models for non-critical queries: `claude-3-haiku`, `gpt-3.5-turbo`
- Check OpenRouter status: https://status.openrouter.ai/

### Cache Not Persisting

- Verify `WISDOM_DB_PATH` is writable
- Check disk space for SQLite database
- Ensure server process has file permissions

## Advanced Usage

### Custom Model Selection Per Query

Override the default model for specific queries:

```typescript
const response = await callTool("ask_expert", {
  question: "Explain quantum computing algorithms",
  domain: "general",
  model: "openai/gpt-4-turbo-preview"  // Override default
});
```

### Batch Wisdom Search

Search multiple domains simultaneously:

```typescript
const domains = ["frontend", "backend", "database"];
const results = await Promise.all(
  domains.map(domain => 
    callTool("search_wisdom", {
      query: "authentication best practices",
      domain: domain,
      limit: 2
    })
  )
);
```

### Export Wisdom Cache

To backup or share your accumulated wisdom:

```bash
# SQLite export
sqlite3 wisdom_cache.db .dump > wisdom_backup.sql

# JSON export (if supported)
node scripts/export-wisdom.js --format json --output wisdom.json
```

## Integration with Other MCPs

Combine with complementary MCP servers:

```json
{
  "mcpServers": {
    "ask-expert-consult": { /* config */ },
    "filesystem": { /* for code context */ },
    "web-search": { /* for documentation lookups */ }
  }
}
```

Then chain tools:

```typescript
// 1. Read file context
const codeFile = await callTool("read_file", { path: "src/app.js" });

// 2. Enrich with web search
const docs = await callTool("web_search", { query: "express.js best practices" });

// 3. Ask expert with full context
const advice = await callTool("ask_expert", {
  question: "Review this Express.js app",
  code_snippet: codeFile.content,
  domain: "backend"
});
```

## Resources

- **OpenRouter API Docs**: https://openrouter.ai/docs
- **MCP Protocol Spec**: https://modelcontextprotocol.io/
- **Project Repository**: https://github.com/pakgik01/ask-expert-consult-mcp
- **Issue Tracker**: https://github.com/pakgik01/ask-expert-consult-mcp/issues
