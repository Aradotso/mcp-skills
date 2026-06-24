---
name: codebase-memory-mcp-pro-knowledge-graph
description: Community fork of codebase-memory-mcp with incremental-reindex fixes — pure-C knowledge-graph MCP server for AI code exploration
triggers:
  - index this codebase
  - build a knowledge graph of this project
  - find all callers of this function
  - trace the call path between these symbols
  - query the code graph with Cypher
  - explore the blast radius of this change
  - detect changes since last commit
  - show me the architecture of this module
---

# codebase-memory-mcp-pro Knowledge Graph

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**codebase-memory-mcp-pro** is a community fork of DeusData/codebase-memory-mcp that provides a pure-C knowledge graph MCP server for AI code exploration. It indexes codebases using tree-sitter AST analysis across 158 languages, building a persistent graph of functions, classes, call chains, and cross-file references.

**Key improvements in this fork:**
- **Incremental-reindex correctness** — preserves inbound cross-file `CALLS` edges; editing a file no longer orphans calls into its symbols
- **Enhanced `explore` tool** — single-call blast-radius analysis with callers, neighbors, inline hotspot flags, and line-numbered source
- **Swift type fidelity** — `struct`/`enum`/`actor` are distinct graph labels; enum cases extracted as `EnumCase` nodes
- **Cypher aggregation fix** — non-aggregate functions mixed with aggregates now group correctly
- **`detect_changes` blast radius** — `depth` parameter produces transitive caller impact analysis

The fork ships **no prebuilt binaries** — you build from source to get all integrated fixes.

## Installation

### Build from Source

```bash
# Clone the fork
git clone https://github.com/win4r/codebase-memory-mcp-pro.git
cd codebase-memory-mcp-pro

# Build (first build compiles 158 tree-sitter grammars — takes a few minutes)
./scripts/build.sh
# → build/c/codebase-memory-mcp (reports version: dev)

# Install to PATH
cp build/c/codebase-memory-mcp ~/.local/bin/

# Add as stdio MCP server (available in all projects)
claude mcp add codebase-memory -s user -- ~/.local/bin/codebase-memory-mcp
```

**Iterative rebuilds** (much faster after first build):
```bash
make -j -f Makefile.cbm cbm
```

### Verify Integration

Confirm the integrated fixes are live:

```bash
# Index a repository
codebase-memory-mcp cli index_repository '{"repo_path":"/path/to/repo"}'

# Test PR #465 — node properties survive WITH aggregation
codebase-memory-mcp cli query_graph '{
  "project": "<name>",
  "query": "MATCH (a)-[:CALLS]->(b) WITH b, count(a) AS c RETURN b.file_path, c LIMIT 1"
}'
# Non-empty file_path means you're running the cherry-picked build
```

**⚠️ Do not run `codebase-memory-mcp update`** — it pulls upstream and overwrites the integrated build. Use `./scripts/build.sh` to update instead.

## Core MCP Tools

The server exposes 15 MCP tools. Key tools in this fork:

### `explore` (Fork Enhancement)

**One-call blast-radius analysis** — returns callers, neighbors, and line-numbered source:

```typescript
// AI agent invocation
{
  "name": "explore",
  "arguments": {
    "project": "my-project",
    "symbol_name": "processOrder",
    "include_source": true,
    "max_callers": 20,
    "max_neighbors": 10
  }
}
```

Returns:
- **Blast radius**: attributed callers + inline fan-in hotspot flags
- **Neighbors**: 1-hop callees + same-file siblings
- **Source**: verbatim line-numbered code grouped by file
- **Cypher escape-hatch**: optional `cypher_query` parameter for custom graph traversal

### `index_repository`

Build the knowledge graph:

```typescript
{
  "name": "index_repository",
  "arguments": {
    "repo_path": "/path/to/repo",
    "project_name": "my-project"  // optional, defaults to dir name
  }
}
```

**Incremental re-index** (fork fix: preserves cross-file CALLS edges):
```bash
# Edit files, then re-run index_repository
# Inbound calls to edited symbols are preserved
```

### `query_graph`

Execute Cypher queries against the knowledge graph:

```typescript
{
  "name": "query_graph",
  "arguments": {
    "project": "my-project",
    "query": "MATCH (f:Function)-[:CALLS]->(g:Function) WHERE f.file_path =~ '.*service.*' RETURN f.name, g.name, g.file_path LIMIT 10"
  }
}
```

**Common patterns:**

```cypher
// Find all callers of a function
MATCH (caller)-[:CALLS]->(target:Function)
WHERE target.name = 'processPayment'
RETURN caller.name, caller.file_path

// Functions with most callers (hotspots)
MATCH (caller)-[:CALLS]->(target)
WITH target, count(caller) AS fan_in
WHERE fan_in > 5
RETURN target.name, target.file_path, fan_in
ORDER BY fan_in DESC

// Call chain between two symbols
MATCH path = shortestPath(
  (a:Function {name: 'handleRequest'})-[:CALLS*..10]->(b:Function {name: 'saveToDatabase'})
)
RETURN [n in nodes(path) | n.name] AS call_chain

// Dead code detection (no inbound calls)
MATCH (f:Function)
WHERE NOT ()-[:CALLS]->(f)
  AND f.visibility = 'public'
RETURN f.name, f.file_path

// Swift enum cases (fork feature)
MATCH (e:Enum)-[:CONTAINS]->(case:EnumCase)
WHERE e.name = 'AppError'
RETURN case.name, case.line_start
```

**Fork fix**: Aggregations now group correctly:
```cypher
// This returns one row per edge type (not collapsed into one row)
MATCH (a)-[r]->(b)
RETURN type(r), count(*) AS edge_count
```

### `detect_changes`

Detect modified files and impacted symbols:

```typescript
{
  "name": "detect_changes",
  "arguments": {
    "project": "my-project",
    "since": "HEAD~5",  // fork fix: honors since parameter
    "depth": 2          // fork feature: transitive caller blast radius
  }
}
```

**Fork enhancement**: `depth` parameter produces transitive **caller** blast radius:
- `impacted_symbols` includes callers up to `depth` hops
- Each symbol tagged with `hop` (0 = changed, 1+ = caller) and `transitive` flag
- `impacted_count` deduplicated across hops

Example response:
```json
{
  "changed_files": ["src/payment/processor.ts"],
  "impacted_symbols": [
    {
      "name": "processPayment",
      "file_path": "src/payment/processor.ts",
      "hop": 0,
      "transitive": false
    },
    {
      "name": "handleCheckout",
      "file_path": "src/checkout/handler.ts",
      "hop": 1,
      "transitive": true
    },
    {
      "name": "completeOrder",
      "file_path": "src/order/service.ts",
      "hop": 2,
      "transitive": true
    }
  ],
  "impacted_count": 12
}
```

### `trace_path`

Find call paths between two symbols:

```typescript
{
  "name": "trace_path",
  "arguments": {
    "project": "my-project",
    "from_symbol": "handleRequest",
    "to_symbol": "saveToDatabase",
    "max_depth": 10
  }
}
```

### `get_code_snippet`

Retrieve source code with line numbers:

```typescript
{
  "name": "get_code_snippet",
  "arguments": {
    "project": "my-project",
    "file_path": "src/utils/validator.ts",
    "start_line": 45,
    "end_line": 60
  }
}
```

**Fork fix**: Returns valid UTF-8 (handles non-UTF-8 source files gracefully).

## Swift Enhancements (Fork)

### Distinct Type Labels

Stock upstream lumps Swift `struct`/`enum`/`actor` as `Class`. This fork emits distinct labels:

```cypher
// Find all Swift structs
MATCH (s:Struct)
WHERE s.file_path =~ '.*\\.swift$'
RETURN s.name, s.file_path

// Find all actors
MATCH (a:Actor)
RETURN a.name

// Enum with cases
MATCH (e:Enum)-[:CONTAINS]->(case:EnumCase)
WHERE e.name = 'NetworkError'
RETURN case.name
```

### Enum Case Extraction

Enum cases (including multi-name `case a, b, c` lines) are extracted as `EnumCase` nodes:

```swift
// Source
enum Status {
    case pending, processing  // Multi-name case
    case completed(Date)
    case failed(Error)
}
```

```cypher
// Query
MATCH (e:Enum {name: 'Status'})-[:CONTAINS]->(case:EnumCase)
RETURN case.name
// Returns: pending, processing, completed, failed
```

### Static Method Dedup Fix

Fork fix: An `enum`'s `static func` is no longer double-emitted as both Method and Function nodes.

## CLI Usage

The binary supports both MCP stdio mode (for agents) and direct CLI invocation:

```bash
# MCP stdio mode (agent communication)
codebase-memory-mcp

# Direct CLI tool invocation
codebase-memory-mcp cli <tool_name> '<json_args>'

# Examples
codebase-memory-mcp cli index_repository '{"repo_path": "/path/to/repo"}'

codebase-memory-mcp cli query_graph '{
  "project": "my-project",
  "query": "MATCH (f:Function) RETURN f.name LIMIT 5"
}'

codebase-memory-mcp cli explore '{
  "project": "my-project",
  "symbol_name": "parseConfig",
  "include_source": true
}'
```

## Configuration

### MCP Server Configuration

Add to `~/.config/claude/claude_desktop_config.json` (or agent-specific config):

```json
{
  "mcpServers": {
    "codebase-memory": {
      "command": "/home/user/.local/bin/codebase-memory-mcp",
      "args": [],
      "env": {}
    }
  }
}
```

### Environment Variables

```bash
# Optional: custom database location
export CBM_DB_PATH=/path/to/custom/db

# Optional: log level (debug, info, warn, error)
export CBM_LOG_LEVEL=info
```

### Project-Specific Configuration

Create `.codebase-memory.json` in repository root:

```json
{
  "exclude_paths": [
    "node_modules",
    "vendor",
    "build",
    "dist",
    ".git",
    "*.test.ts"
  ],
  "include_extensions": [
    ".ts", ".tsx", ".js", ".jsx",
    ".py", ".go", ".rs", ".c", ".cpp", ".swift"
  ],
  "max_file_size_kb": 1024
}
```

## Real-World Patterns

### Pattern 1: Impact Analysis Before Refactoring

```typescript
// 1. Find the function
const exploreResult = await explore({
  project: "my-api",
  symbol_name: "validateUser",
  include_source: true,
  max_callers: 50
});

// 2. Check blast radius
const callerCount = exploreResult.callers.length;
const hasHighFanIn = callerCount > 10;

// 3. Query transitive impact
const impactResult = await query_graph({
  project: "my-api",
  query: `
    MATCH (caller)-[:CALLS*1..3]->(target:Function)
    WHERE target.name = 'validateUser'
    RETURN DISTINCT caller.name, caller.file_path
  `
});

// 4. Make informed decision
if (hasHighFanIn) {
  // High-risk refactor: write comprehensive tests first
} else {
  // Low-risk: proceed with confidence
}
```

### Pattern 2: Dead Code Detection

```typescript
// Find public functions with no callers
const deadCode = await query_graph({
  project: "my-api",
  query: `
    MATCH (f:Function)
    WHERE NOT ()-[:CALLS]->(f)
      AND f.visibility = 'public'
      AND f.file_path =~ '.*src/.*'
    RETURN f.name, f.file_path, f.line_start
    ORDER BY f.file_path
  `
});

// Cross-reference with exports
const exports = await query_graph({
  project: "my-api",
  query: `
    MATCH (m:Module)-[:EXPORTS]->(f)
    RETURN f.name, m.file_path
  `
});

// Candidates for deletion: deadCode NOT IN exports
```

### Pattern 3: Incremental Re-index Workflow

```bash
#!/bin/bash
# safe-refactor.sh

# 1. Index current state
codebase-memory-mcp cli index_repository '{"repo_path": "."}'

# 2. Snapshot call graph
codebase-memory-mcp cli query_graph '{
  "project": "my-api",
  "query": "MATCH (a)-[:CALLS]->(b {name: \"targetFunc\"}) RETURN a.name, a.file_path"
}' > before.json

# 3. Make changes
vim src/core/target.ts

# 4. Re-index (fork: preserves inbound CALLS edges)
codebase-memory-mcp cli index_repository '{"repo_path": "."}'

# 5. Verify call graph integrity
codebase-memory-mcp cli query_graph '{
  "project": "my-api",
  "query": "MATCH (a)-[:CALLS]->(b {name: \"targetFunc\"}) RETURN a.name, a.file_path"
}' > after.json

# 6. Compare
diff before.json after.json
```

### Pattern 4: Architecture Documentation

```typescript
// Generate module dependency graph
const modules = await query_graph({
  project: "my-api",
  query: `
    MATCH (m1:Module)-[:IMPORTS]->(m2:Module)
    RETURN m1.file_path AS from, m2.file_path AS to
  `
});

// Detect circular dependencies
const cycles = await query_graph({
  project: "my-api",
  query: `
    MATCH path = (m:Module)-[:IMPORTS*2..10]->(m)
    RETURN [n in nodes(path) | n.file_path] AS cycle
    LIMIT 10
  `
});

// Find architectural layers (no upward imports)
const layers = await query_graph({
  project: "my-api",
  query: `
    MATCH (m:Module)
    WHERE m.file_path =~ '.*/domain/.*'
      AND NOT (m)-[:IMPORTS]->(:Module {file_path: ~'.*/infrastructure/.*'})
    RETURN m.file_path AS clean_domain_module
  `
});
```

## Troubleshooting

### Issue: "No such project"

**Cause**: Project not indexed or wrong name.

```bash
# List indexed projects
codebase-memory-mcp cli query_graph '{
  "project": "any",
  "query": "MATCH (p:Project) RETURN p.name"
}'

# Re-index with explicit name
codebase-memory-mcp cli index_repository '{
  "repo_path": "/path/to/repo",
  "project_name": "exact-name"
}'
```

### Issue: Incremental re-index loses edges (upstream bug)

**Solution**: You're running stock upstream, not the fork. Rebuild from source:

```bash
cd codebase-memory-mcp-pro
./scripts/build.sh
cp build/c/codebase-memory-mcp ~/.local/bin/
```

Verify integration:
```bash
codebase-memory-mcp cli query_graph '{
  "project": "test",
  "query": "MATCH (a)-[:CALLS]->(b) WITH b, count(a) AS c RETURN b.file_path, c LIMIT 1"
}'
# Non-empty file_path = fork build
```

### Issue: Cypher query returns unexpected single row

**Cause**: Aggregation bug in upstream (fixed in fork).

```cypher
// This should return one row per edge type
MATCH (a)-[r]->(b)
RETURN type(r), count(*)
// Upstream: collapses into single row
// Fork: groups correctly by type(r)
```

Rebuild from fork source if you're seeing this.

### Issue: Swift enums/structs labeled as Class

**Solution**: Fork emits distinct labels. Query with specific types:

```cypher
// Fork
MATCH (s:Struct) RETURN s.name

// Upstream workaround (not recommended)
MATCH (s:Class)
WHERE s.kind = 'struct'  // Property may not exist
RETURN s.name
```

### Issue: Build fails on macOS with libgit2 ≥ 1.8

**Solution**: Already fixed in fork (PR #512 integrated). If still failing:

```bash
brew info libgit2  # Check version
# Ensure you're building from fork, not upstream
git remote -v  # Should show win4r/codebase-memory-mcp-pro
./scripts/build.sh
```

### Issue: UTF-8 errors in get_code_snippet

**Solution**: Fixed in fork (PR #526). Rebuild from source.

### Issue: detect_changes ignores `since` parameter

**Solution**: Fixed in fork (PR #464). Rebuild from source.

## Advanced: Custom Cypher Patterns

### Find HTTP Route Handlers

```cypher
MATCH (route:Route)-[:HANDLED_BY]->(handler:Function)
WHERE route.method = 'POST'
RETURN route.path, handler.name, handler.file_path
ORDER BY route.path
```

### Cross-Service Call Analysis

```cypher
MATCH (caller:Function)-[:HTTP_CALL]->(route:Route)
WHERE caller.file_path =~ '.*service-a/.*'
  AND route.service = 'service-b'
RETURN caller.name, route.path, route.method
```

### Complexity Analysis

```cypher
MATCH (f:Function)
WHERE f.cyclomatic_complexity > 10
RETURN f.name, f.file_path, f.cyclomatic_complexity
ORDER BY f.cyclomatic_complexity DESC
LIMIT 20
```

### Ownership Mapping (by directory)

```cypher
MATCH (f:Function)
WITH split(f.file_path, '/')[0..3] AS module, count(f) AS function_count
RETURN module, function_count
ORDER BY function_count DESC
```

## Integration with AI Agents

### Claude Code / Cursor

```json
// ~/.config/claude/claude_desktop_config.json
{
  "mcpServers": {
    "codebase-memory": {
      "command": "/home/user/.local/bin/codebase-memory-mcp"
    }
  }
}
```

### Aider

```bash
# .aider.conf.yml
mcp:
  servers:
    - name: codebase-memory
      command: /home/user/.local/bin/codebase-memory-mcp
```

### Generic MCP Client

```typescript
import { MCPClient } from '@modelcontextprotocol/sdk';

const client = new MCPClient({
  command: '/home/user/.local/bin/codebase-memory-mcp',
  args: []
});

await client.connect();

const result = await client.callTool('explore', {
  project: 'my-api',
  symbol_name: 'processOrder'
});

console.log(result);
```

## Performance Notes

- **Linux kernel** (28M LOC, 75K files): 3 minutes to index
- **Average repository**: milliseconds
- **Graph queries**: <1ms for simple patterns, <100ms for complex traversals
- **Memory**: RAM-first pipeline with LZ4 compression; memory released after indexing

## Contributing to the Fork

This is a community fork tracking upstream. To contribute:

1. **Upstream PRs first**: Submit fixes to [DeusData/codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)
2. **Fork-specific enhancements**: Submit PRs to [win4r/codebase-memory-mcp-pro](https://github.com/win4r/codebase-memory-mcp-pro)
3. **Integration requests**: Open an issue if an upstream PR should be cherry-picked into the fork

## License

MIT License (unchanged from upstream). See [LICENSE](https://github.com/win4r/codebase-memory-mcp-pro/blob/main/LICENSE).

All credit for the original engine: DeusData. This fork exists to integrate fixes faster than upstream merge cycles.
