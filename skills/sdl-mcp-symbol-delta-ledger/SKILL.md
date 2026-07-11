---
name: sdl-mcp-symbol-delta-ledger
description: Policy-centered context budget layer that turns sprawling codebases into compact, high-signal context for AI coding agents using symbol graphs and precision tools
triggers:
  - "index this codebase for better context"
  - "get symbol information without reading the whole file"
  - "analyze code dependencies and call graphs"
  - "show me the blast radius of this change"
  - "give me compact context for this task"
  - "search the codebase semantically"
  - "what's the impact of this PR"
  - "build a token-efficient code slice"
---

# SDL-MCP: Symbol Delta Ledger

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

SDL-MCP is a policy-centered context budget layer for coding agents that transforms sprawling codebases into compact, high-signal context. It indexes code into a searchable symbol graph, providing 4-20x token savings by serving precisely the right amount of context through the Iris Gate Ladder escalation system.

## What SDL-MCP Does

- **Symbol Cards**: Every function, class, interface, type, and variable becomes a ~100 token metadata record instead of ~2,000 tokens of raw code
- **Graph Slicing**: Follow dependency graphs (not directory boundaries) to get the N most relevant symbols within a token budget
- **Iris Gate Ladder**: Four-rung context escalation from compact cards to full source (policy-gated)
- **Delta Packs & Blast Radius**: Semantic change intelligence showing what changes mean and who's affected
- **Live Indexing**: Real-time code intelligence reflecting unsaved editor changes
- **Task-Shaped Retrieval**: Context engine that selects the right rungs and evidence for debug/review/implement/explain tasks

## Installation

**Prerequisites**: Node.js 24+ required

### Global Installation

```bash
# Install globally
npm install -g sdl-mcp

# Non-interactive setup with auto-indexing
sdl-mcp init -y --auto-index

# Start MCP server
sdl-mcp serve --stdio
```

### Using npx

```bash
# Quick start with wrapper
npx create-sdl-mcp

# Or direct npx usage
npx --yes sdl-mcp@latest init -y --auto-index
npx --yes sdl-mcp@latest serve --stdio
```

### MCP Client Configuration

Add to your MCP client configuration (e.g., Claude Desktop):

```json
{
  "mcpServers": {
    "sdl-mcp": {
      "command": "sdl-mcp",
      "args": ["serve", "--stdio"]
    }
  }
}
```

Or with npx:

```json
{
  "mcpServers": {
    "sdl-mcp": {
      "command": "npx",
      "args": ["--yes", "sdl-mcp@latest", "serve", "--stdio"]
    }
  }
}
```

## Key Commands (CLI)

```bash
# Initialize repository
sdl-mcp init                          # Interactive setup
sdl-mcp init -y --auto-index         # Non-interactive with auto-index
sdl-mcp init --config-only           # Generate config without indexing

# Indexing
sdl-mcp index                        # Full index
sdl-mcp index --incremental          # Only changed files
sdl-mcp index --force                # Force re-index all files

# Server
sdl-mcp serve --stdio                # Start MCP server
sdl-mcp serve --stdio --verbose      # Verbose logging

# Health & Diagnostics
sdl-mcp health                       # Run health checks
sdl-mcp summary --task "debug auth"  # Generate portable context summary

# Configuration
sdl-mcp config get                   # Show current config
sdl-mcp config set key=value         # Update config value
```

## MCP Tools API

### Symbol Cards & Search

**Get Symbol Card** (Rung 1: ~100 tokens)
```typescript
// Request minimal symbol metadata
{
  "name": "sdl.symbol.card",
  "arguments": {
    "symbolId": "src/auth/validate.ts::validateToken",
    "format": "compact"  // or "full"
  }
}
```

**Search Symbols**
```typescript
{
  "name": "sdl.symbol.search",
  "arguments": {
    "query": "authenticate user",
    "limit": 10,
    "confidence": 0.7,
    "filters": {
      "kind": ["function", "class"],
      "filePath": "src/auth/**"
    }
  }
}
```

### Graph Slicing

**Build Token-Budgeted Slice**
```typescript
{
  "name": "sdl.slice.build",
  "arguments": {
    "entrySymbols": ["src/auth/validate.ts::validateToken"],
    "tokenBudget": 800,
    "direction": "both",  // "upstream" | "downstream" | "both"
    "weights": {
      "call": 1.0,
      "config": 0.8,
      "import": 0.6
    }
  }
}
```

**Auto-Discovery from Task**
```typescript
{
  "name": "sdl.slice.build",
  "arguments": {
    "taskText": "Fix authentication timeout issue",
    "tokenBudget": 1000,
    "autoDiscover": true
  }
}
```

**Refresh Slice (Delta Updates)**
```typescript
{
  "name": "sdl.slice.refresh",
  "arguments": {
    "sliceHandle": "slice_abc123",
    "deltaOnly": true  // Only return changed symbols
  }
}
```

### Iris Gate Ladder Escalation

**Rung 2: Skeleton (signature + structure)**
```typescript
{
  "name": "sdl.iris.skeleton",
  "arguments": {
    "symbolId": "src/auth/AuthService.ts::AuthService",
    "includeSignatures": true,
    "includeTypes": true
  }
}
```

**Rung 3: Hot Path (specific lines/blocks)**
```typescript
{
  "name": "sdl.iris.hotPath",
  "arguments": {
    "symbolId": "src/auth/validate.ts::validateToken",
    "targetIdentifiers": ["cache", "decrypt"],
    "contextLines": 3
  }
}
```

**Rung 4: Raw Source (policy-gated)**
```typescript
{
  "name": "sdl.iris.rawSource",
  "arguments": {
    "symbolId": "src/auth/validate.ts::validateToken",
    "reason": "Need to see complete error handling logic for timeout bug",
    "expectedIdentifiers": ["TimeoutError", "retry", "backoff"],
    "expectedLineCount": 45,
    "bypassPolicy": false
  }
}
```

### Task-Shaped Context

**Get Context for Task**
```typescript
{
  "name": "sdl.context",
  "arguments": {
    "task": "debug",  // "debug" | "review" | "implement" | "explain"
    "description": "User authentication failing after token refresh",
    "tokenBudget": 2000,
    "focusPaths": ["src/auth/**"],
    "options": {
      "semantic": true,  // Use hybrid retrieval
      "includeTests": true
    }
  }
}
```

### Delta & Blast Radius

**Semantic Diff**
```typescript
{
  "name": "sdl.delta.pack",
  "arguments": {
    "baseRef": "main",
    "headRef": "HEAD",
    "options": {
      "includeBlastRadius": true,
      "maxBlastDepth": 3
    }
  }
}
```

**PR Risk Analysis**
```typescript
{
  "name": "sdl.pr.risk.analyze",
  "arguments": {
    "baseRef": "main",
    "headRef": "feature/auth-refactor",
    "options": {
      "includeFanInTrend": true,
      "testRecommendations": true
    }
  }
}
```

### Live Indexing

**Push Buffer Changes**
```typescript
{
  "name": "sdl.buffer.push",
  "arguments": {
    "filePath": "src/auth/validate.ts",
    "content": "export async function validateToken(token: string): Promise<User> {\n  // ...\n}",
    "version": 42
  }
}
```

**Clear Buffer**
```typescript
{
  "name": "sdl.buffer.clear",
  "arguments": {
    "filePath": "src/auth/validate.ts"
  }
}
```

### Runtime Execution (Sandboxed)

**Execute Code**
```typescript
{
  "name": "sdl.runtime.execute",
  "arguments": {
    "executable": "npm",
    "args": ["test", "auth.test.ts"],
    "cwd": "./",
    "outputMode": "minimal",  // "minimal" | "summary" | "intent"
    "timeout": 30000
  }
}
```

**Query Execution Output**
```typescript
{
  "name": "sdl.runtime.queryOutput",
  "arguments": {
    "executionId": "exec_xyz789",
    "query": "Show me the failing test assertions"
  }
}
```

### Feedback Loop

**Record Symbol Usefulness**
```typescript
{
  "name": "sdl.agent.feedback",
  "arguments": {
    "contextId": "ctx_abc123",
    "useful": ["src/auth/validate.ts::validateToken"],
    "missing": ["src/auth/TokenCache.ts::get"],
    "irrelevant": ["src/utils/logger.ts::debug"]
  }
}
```

## Configuration

SDL-MCP stores configuration in `.sdl-mcp/config.json`:

```json
{
  "repoRoot": "/path/to/repo",
  "languages": {
    "typescript": { "enabled": true },
    "python": { "enabled": true },
    "go": { "enabled": true }
  },
  "indexing": {
    "exclude": ["node_modules/**", "dist/**", "*.test.ts"],
    "maxFileSize": 1048576,
    "parallelism": 4
  },
  "governance": {
    "rawSourceGate": {
      "enabled": true,
      "requireReason": true,
      "requireIdentifiers": true,
      "maxLineCount": 500
    },
    "runtime": {
      "enabled": true,
      "allowedExecutables": ["npm", "node", "python3"],
      "cwdJail": true,
      "timeout": 30000
    }
  },
  "codeMode": {
    "exclusive": true,
    "autoSlicing": true,
    "defaultBudget": 1500
  },
  "retrieval": {
    "hybridThreshold": 0.7,
    "semanticDefault": "auto"
  }
}
```

### Update Configuration

```bash
# Set specific values
sdl-mcp config set governance.rawSourceGate.maxLineCount=1000
sdl-mcp config set codeMode.defaultBudget=2000

# Enable/disable features
sdl-mcp config set governance.runtime.enabled=false
sdl-mcp config set codeMode.exclusive=false
```

## Common Patterns

### Pattern 1: Efficient Code Understanding

Instead of reading entire files:

```typescript
// ❌ Old way: Read whole file (~2000 tokens)
const fileContent = await readFile('src/auth/validate.ts');

// ✅ SDL-MCP way: Get symbol card (~100 tokens)
const card = await mcp.call('sdl.symbol.card', {
  symbolId: 'src/auth/validate.ts::validateToken',
  format: 'compact'
});
// Returns: signature, parameters, return type, dependencies, ~100 tokens
```

### Pattern 2: Dependency-Aware Context

Instead of directory-based context:

```typescript
// ❌ Old way: Read all files in directory (~16,000 tokens)
const authFiles = await glob('src/auth/**/*.ts');

// ✅ SDL-MCP way: Graph slice within budget (~800 tokens)
const slice = await mcp.call('sdl.slice.build', {
  taskText: 'Fix token validation timeout',
  tokenBudget: 800,
  autoDiscover: true
});
// Returns: Only relevant symbols following dependency graph
```

### Pattern 3: Controlled Escalation

Only read raw code when necessary:

```typescript
// Start with card (Rung 1)
const card = await mcp.call('sdl.symbol.card', {
  symbolId: 'src/auth/validate.ts::validateToken'
});

// If needed, escalate to skeleton (Rung 2)
const skeleton = await mcp.call('sdl.iris.skeleton', {
  symbolId: 'src/auth/validate.ts::validateToken',
  includeSignatures: true
});

// If still needed, hot path (Rung 3)
const hotPath = await mcp.call('sdl.iris.hotPath', {
  symbolId: 'src/auth/validate.ts::validateToken',
  targetIdentifiers: ['cache', 'decrypt'],
  contextLines: 3
});

// Only as last resort, raw source (Rung 4, policy-gated)
const raw = await mcp.call('sdl.iris.rawSource', {
  symbolId: 'src/auth/validate.ts::validateToken',
  reason: 'Need complete error handling for timeout analysis',
  expectedIdentifiers: ['TimeoutError', 'retry'],
  expectedLineCount: 45
});
```

### Pattern 4: Task-Shaped Retrieval

Let SDL-MCP plan the context:

```typescript
// Debugging task
const debugContext = await mcp.call('sdl.context', {
  task: 'debug',
  description: 'Authentication timeout after 30 seconds',
  tokenBudget: 2000,
  focusPaths: ['src/auth/**'],
  options: { includeTests: true }
});

// Code review task
const reviewContext = await mcp.call('sdl.context', {
  task: 'review',
  description: 'Refactor token validation to use Redis cache',
  tokenBudget: 1500,
  options: { semantic: true }
});

// Implementation task
const implContext = await mcp.call('sdl.context', {
  task: 'implement',
  description: 'Add rate limiting to login endpoint',
  tokenBudget: 2500,
  focusSymbols: ['src/auth/login.ts::handleLogin']
});
```

### Pattern 5: Change Impact Analysis

Understand semantic impact of changes:

```typescript
// Get semantic diff
const deltaPack = await mcp.call('sdl.delta.pack', {
  baseRef: 'main',
  headRef: 'feature/cache-refactor',
  options: {
    includeBlastRadius: true,
    maxBlastDepth: 3
  }
});

// Analyze PR risk
const riskAnalysis = await mcp.call('sdl.pr.risk.analyze', {
  baseRef: 'main',
  headRef: 'feature/cache-refactor',
  options: {
    includeFanInTrend: true,
    testRecommendations: true
  }
});

// Results include:
// - Semantic changes (not just line diffs)
// - Ranked blast radius (who's affected)
// - Fan-in trends (amplifier symbols)
// - Test re-run recommendations
```

### Pattern 6: Live Development Flow

Work with unsaved changes:

```typescript
// Push buffer as you type
await mcp.call('sdl.buffer.push', {
  filePath: 'src/auth/validate.ts',
  content: currentEditorContent,
  version: editorVersion
});

// Search reflects live changes
const liveSearch = await mcp.call('sdl.symbol.search', {
  query: 'validateToken',
  limit: 5
});
// Returns results including unsaved buffer changes

// Build slice with live state
const liveSlice = await mcp.call('sdl.slice.build', {
  taskText: 'Review my current auth changes',
  tokenBudget: 1000
});

// Clear buffer when done
await mcp.call('sdl.buffer.clear', {
  filePath: 'src/auth/validate.ts'
});
```

### Pattern 7: Feedback-Driven Improvement

Help SDL-MCP learn what's useful:

```typescript
// After using context
await mcp.call('sdl.agent.feedback', {
  contextId: 'ctx_abc123',
  useful: [
    'src/auth/validate.ts::validateToken',
    'src/cache/TokenCache.ts::get'
  ],
  missing: [
    'src/auth/errors.ts::TimeoutError'
  ],
  irrelevant: [
    'src/utils/logger.ts::debug'
  ]
});
// Future slices will prioritize useful symbols and include missing ones
```

## Real-World Example: Debugging Authentication Issue

```typescript
// 1. Start with task-shaped context
const context = await mcp.call('sdl.context', {
  task: 'debug',
  description: 'Users getting 401 errors after token refresh',
  tokenBudget: 2000,
  focusPaths: ['src/auth/**'],
  options: { includeTests: true }
});

// 2. Search for relevant symbols
const symbols = await mcp.call('sdl.symbol.search', {
  query: 'token refresh validate',
  limit: 10,
  confidence: 0.7
});

// 3. Build dependency slice from top candidates
const slice = await mcp.call('sdl.slice.build', {
  entrySymbols: symbols.results.slice(0, 3).map(r => r.symbolId),
  tokenBudget: 800,
  direction: 'both'
});

// 4. Get skeleton of key symbol
const skeleton = await mcp.call('sdl.iris.skeleton', {
  symbolId: 'src/auth/TokenService.ts::refreshToken',
  includeSignatures: true
});

// 5. If needed, hot path for specific logic
const hotPath = await mcp.call('sdl.iris.hotPath', {
  symbolId: 'src/auth/TokenService.ts::refreshToken',
  targetIdentifiers: ['verify', 'expire', 'blacklist'],
  contextLines: 5
});

// 6. Run tests to verify hypothesis
const testRun = await mcp.call('sdl.runtime.execute', {
  executable: 'npm',
  args: ['test', 'auth/token.test.ts'],
  outputMode: 'summary',
  timeout: 30000
});

// 7. Provide feedback
await mcp.call('sdl.agent.feedback', {
  contextId: context.id,
  useful: [
    'src/auth/TokenService.ts::refreshToken',
    'src/auth/TokenService.ts::verifyToken'
  ],
  missing: [
    'src/auth/blacklist.ts::isBlacklisted'
  ]
});
```

**Token Comparison:**
- Reading all auth files: ~16,000 tokens
- SDL-MCP approach: ~1,500 tokens
- **Savings: 10.6x**

## Troubleshooting

### Installation Issues

**Peer dependency warnings:**
```bash
# Suppress harmless ERESOLVE warnings
npm install -g sdl-mcp --legacy-peer-deps
```

**Tree-sitter grammar issues:**
```bash
# Rebuild grammars
npm rebuild tree-sitter tree-sitter-kotlin
```

**Node version mismatch:**
```bash
# Check version (must be 24+)
node -v

# Install Node 24+ from https://nodejs.org
```

### Indexing Issues

**Index fails on large repo:**
```bash
# Increase file size limit
sdl-mcp config set indexing.maxFileSize=2097152

# Reduce parallelism if memory-constrained
sdl-mcp config set indexing.parallelism=2
```

**Files not indexed:**
```bash
# Check exclusion patterns
sdl-mcp config get indexing.exclude

# Force re-index
sdl-mcp index --force
```

**Language not detected:**
```bash
# Enable language explicitly
sdl-mcp config set languages.python.enabled=true

# Re-run auto-index
sdl-mcp init --auto-index
```

### Query Issues

**Low search quality:**
```typescript
// Enable semantic search
{
  "name": "sdl.symbol.search",
  "arguments": {
    "query": "authentication logic",
    "options": { "semantic": true }
  }
}

// Or update default
sdl-mcp config set retrieval.semanticDefault=true
```

**Slice too small/large:**
```typescript
// Adjust token budget
{
  "name": "sdl.slice.build",
  "arguments": {
    "entrySymbols": ["..."],
    "tokenBudget": 1500  // Increase from default 800
  }
}
```

**Raw source denied:**
```typescript
// Check policy requirements
sdl-mcp config get governance.rawSourceGate

// Provide required justification
{
  "name": "sdl.iris.rawSource",
  "arguments": {
    "symbolId": "...",
    "reason": "Specific reason why card/skeleton/hotPath insufficient",
    "expectedIdentifiers": ["specific", "names", "needed"],
    "expectedLineCount": 45  // Within maxLineCount limit
  }
}
```

### Runtime Issues

**Execution denied:**
```bash
# Check allowed executables
sdl-mcp config get governance.runtime.allowedExecutables

# Add executable to allowlist
sdl-mcp config set governance.runtime.allowedExecutables='["npm","node","python3","go"]'
```

**Execution timeout:**
```typescript
// Increase timeout
{
  "name": "sdl.runtime.execute",
  "arguments": {
    "executable": "npm",
    "args": ["test"],
    "timeout": 60000  // 60 seconds
  }
}
```

### Health Checks

```bash
# Run full health check
sdl-mcp health

# Common issues reported:
# - Database corruption: Run `sdl-mcp index --force`
# - Missing grammars: Run `npm rebuild tree-sitter*`
# - Config errors: Run `sdl-mcp config get` to inspect
# - Index stale: Run `sdl-mcp index --incremental`
```

## Environment Variables

```bash
# Database location (default: .sdl-mcp/db)
export SDL_DB_PATH=/custom/path/to/db

# Log level
export SDL_LOG_LEVEL=debug  # error | warn | info | debug

# Disable telemetry
export SDL_TELEMETRY=false

# Max workers for indexing
export SDL_MAX_WORKERS=8
```

## Performance Tips

1. **Use incremental indexing** during development: `sdl-mcp index --incremental`
2. **Set appropriate token budgets** (800-1500 typical, 2000+ for complex tasks)
3. **Use focusPaths/focusSymbols** to scope searches when possible
4. **Enable deltaOnly** on slice refreshes to minimize re-fetching
5. **Leverage live indexing** instead of re-indexing after every edit
6. **Provide feedback** to improve future slice quality
7. **Use Code Mode** (`codeMode.exclusive: true`) for most efficient agent workflow

## Learn More

- [GitHub Repository](https://github.com/GlitterKill/sdl-mcp)
- [Full Documentation](https://github.com/GlitterKill/sdl-mcp/tree/main/docs)
- [Iris Gate Ladder Deep Dive](https://github.com/GlitterKill/sdl-mcp/blob/main/docs/feature-deep-dives/iris-gate-ladder.md)
- [Graph Slicing Deep Dive](https://github.com/GlitterKill/sdl-mcp/blob/main/docs/feature-deep-dives/graph-slicing.md)
