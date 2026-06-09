---
name: codedb-mcp-fast-code-intelligence
description: Fast local-first code intelligence MCP server with tree-sitter indexing, semantic search, dependency analysis, and millisecond-latency tools for repository understanding
triggers:
  - search my codebase quickly
  - find symbol definitions and callers
  - analyze code dependencies and modules
  - get fast code context without dumping files
  - index my repository for intelligent search
  - explore code architecture with semantic search
  - find related code files and references
  - build a code module atlas
---

# codedb-mcp Fast Code Intelligence

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`codedb-mcp` (also called `codebase-mcp`) is a Rust-based MCP server that turns any local repository into a persistent code intelligence service. It maintains a tree-sitter indexed database under `.codedb-mcp/` with symbols, references, dependencies, graph metadata, lexical indexes, and vector embeddings for fast semantic search.

**Key capabilities:**
- **Millisecond-latency warm queries** — 20,000x faster than `rg` on indexed searches
- **Answer-oriented context tools** — `codedb_context` and `codedb_explore` return ranked relevant code without dumping entire files
- **Dependency-aware module discovery** — Automatic grouping of related files based on dependency graphs
- **LSP-like features** — Symbol definitions, callers, outlines, and reference navigation
- **Code Module Atlas** — Visual 3D browser showing file relationships and module boundaries
- **DeepWiki** — Auto-generated repository documentation with cited sources
- **Local-first** — All data stored in project `.codedb-mcp/` directory, no cloud dependencies

**Performance benchmarks (u3dclient Unity C# project):**
- Index 18,852 files in 13.8s cold, 0.741s warm cache-hit
- Symbol lookup: 0.088-0.351ms
- Text search: 0.103-0.442ms warm (vs 77-5,007ms for `rg`)
- Callers lookup: 8-17ms avg
- Token savings: 43% reduction vs shell-based code lookup (254k tokens saved across 3 feature analysis sessions)

## Installation

### Prerequisites

- Rust toolchain (for building from source)
- Node.js (for skills and atlas viewer)
- Git repository to index

### Build from Source

```bash
git clone https://github.com/killop/codedb-mcp.git
cd codedb-mcp
cargo build --release
```

The binary will be at `target/release/codedb-mcp` (or `codedb-mcp.exe` on Windows).

### Project Setup

1. **Create configuration file** in your target repository:

```bash
cd /path/to/your/project
mkdir -p .codedb-mcp
```

2. **Create `.codedb-mcp/codedb-mcp.toml`**:

```toml
# Basic configuration
[project]
name = "myproject"
root = "."

# Include/exclude patterns
[scan]
include = ["src/**/*.rs", "lib/**/*.rs"]
exclude = ["target/**", "node_modules/**", ".git/**"]

# Language-specific settings
[languages]
rust = { enabled = true }
typescript = { enabled = true }
python = { enabled = true }

# Vector search (optional, lazy-loaded on first use)
[vector]
model = "minishlab/potion-code-16M"  # Model2Vec model from HuggingFace
```

**Unity C# example config:**

```toml
[project]
name = "u3dclient"
root = "."

[scan]
include = ["Assets/**/*.cs", "Packages/**/*.cs"]
exclude = [
    "Library/**",
    "Temp/**",
    "obj/**",
    "Build/**",
    "Logs/**"
]

[languages]
csharp = { enabled = true }
```

3. **Build initial index**:

```bash
/path/to/codedb-mcp --project-root . index
```

### MCP Server Configuration

Add to your MCP settings file (e.g., Claude Desktop config):

```json
{
  "mcpServers": {
    "codedb-myproject": {
      "command": "/path/to/codedb-mcp",
      "args": ["--project-root", "/path/to/your/project", "serve"],
      "env": {
        "RUST_LOG": "info"
      }
    }
  }
}
```

**Windows PowerShell example:**

```json
{
  "mcpServers": {
    "codedb-u3dclient": {
      "command": "D:\\tools\\codedb-mcp\\target\\release\\codedb-mcp.exe",
      "args": ["--project-root", "D:\\projects\\u3dclient", "serve"]
    }
  }
}
```

## Core MCP Tools

### Index Management

#### `codedb_index`
Build or rebuild the code database.

```typescript
// Trigger full rebuild
await use_mcp_tool("codedb-myproject", "codedb_index", {
  force_rebuild: true
});

// Incremental update (default)
await use_mcp_tool("codedb-myproject", "codedb_index", {});
```

**When to use:**
- After cloning a repository
- When adding new files
- After significant code changes
- When index appears stale

#### `codedb_status`
Check index health and statistics.

```typescript
const status = await use_mcp_tool("codedb-myproject", "codedb_status", {});
// Returns: file counts, chunk counts, graph stats, cache state
```

#### `codedb_version`
Get server version info.

```typescript
const version = await use_mcp_tool("codedb-myproject", "codedb_version", {});
```

### Symbol Navigation

#### `codedb_symbol`
Find symbol definitions across the codebase.

```typescript
// Find class/function/variable definition
const defs = await use_mcp_tool("codedb-myproject", "codedb_symbol", {
  symbol: "PoolManager"
});
// Returns: [{path, line, column, context}]

// Find multiple symbols
const defs = await use_mcp_tool("codedb-myproject", "codedb_symbol", {
  symbol: "UserService|AuthController"
});
```

**Use cases:**
- Jump to definition
- Find class/interface declarations
- Locate function implementations

#### `codedb_outline`
Get symbol outline for a single file.

```typescript
const outline = await use_mcp_tool("codedb-myproject", "codedb_outline", {
  path: "src/services/user_service.rs"
});
// Returns: hierarchical list of classes, functions, fields with line numbers
```

**Performance:** 0.088-0.351ms per file (after first load)

#### `codedb_callers`
Find all call sites for a symbol definition.

```typescript
const callers = await use_mcp_tool("codedb-myproject", "codedb_callers", {
  path: "src/pool.rs",
  line: 42,  // definition line
  symbol: "PoolManager"
});
// Returns: [{caller_path, caller_line, context}]
```

**Performance:** 8-17ms avg

**Use cases:**
- Find all usages before refactoring
- Understand impact of API changes
- Trace call chains

### Search Tools

#### `codedb_text_search`
Fast full-text and regex search using trigram indexing.

```typescript
// Exact string search
const results = await use_mcp_tool("codedb-myproject", "codedb_text_search", {
  query: "NetworkListenerManager",
  case_sensitive: false
});

// Regex search
const results = await use_mcp_tool("codedb-myproject", "codedb_text_search", {
  query: "class\\s+(\\w+)Manager",
  is_regex: true
});

// Scoped search
const results = await use_mcp_tool("codedb-myproject", "codedb_text_search", {
  query: "PoolManager",
  scope: ["src/**/*.rs"]
});
```

**Performance:** 0.103-0.442ms warm (vs 77-5,007ms for `rg` on same queries)

**Parameters:**
- `query`: search string or regex pattern
- `is_regex`: treat query as regex (default: false)
- `case_sensitive`: case-sensitive matching (default: false)
- `scope`: file path patterns to limit search
- `limit`: max results (default: 100)

#### `codedb_search`
Unified semantic + symbol + text search.

```typescript
// Symbol-aware search
const results = await use_mcp_tool("codedb-myproject", "codedb_search", {
  query: "PoolManager",
  mode: "symbol"
});

// Natural language semantic search (uses vector embeddings)
const results = await use_mcp_tool("codedb-myproject", "codedb_search", {
  query: "alliance rally join logic",
  mode: "semantic",
  limit: 20
});

// Combined search
const results = await use_mcp_tool("codedb-myproject", "codedb_search", {
  query: "user authentication",
  mode: "fusion"  // symbol + word-trigram + vector
});
```

**Modes:**
- `symbol`: exact symbol matching
- `text`: trigram full-text search
- `semantic`: vector similarity search
- `fusion`: weighted combination of all methods

**Performance:**
- Symbol-aware: 22ms
- Semantic: 36ms (after vector model load)

#### `codedb_word`
Exact identifier inverted index lookup.

```typescript
const results = await use_mcp_tool("codedb-myproject", "codedb_word", {
  word: "initialize",
  limit: 50
});
// Returns: all files containing exact identifier "initialize"
```

**Performance:** ~100ms including lazy word sidecar access

### Answer-Oriented Context Tools

#### `codedb_context`
Get ranked relevant code context WITHOUT dumping full files. Returns summaries, symbols, and snippets optimized for answering questions.

```typescript
// Architecture question
const context = await use_mcp_tool("codedb-myproject", "codedb_context", {
  query: "how does the pool management system work?",
  max_tokens: 2000  // output budget
});

// Feature investigation
const context = await use_mcp_tool("codedb-myproject", "codedb_context", {
  query: "alliance rally and join logic",
  focus_paths: ["src/alliance/**"],
  max_tokens: 3000
});
```

**Performance:** 6.5-30ms depending on query complexity

**Output:** ~1.5-2.0k tokens of ranked context (vs potentially 100k+ from naive file dumps)

**When to use:**
- Answering "how does X work?" questions
- Feature area exploration
- Architecture understanding
- Before making changes to unfamiliar code

**Parameters:**
- `query`: natural language question or keyword
- `max_tokens`: output budget (default: 2000)
- `focus_paths`: limit to specific directories
- `include_deps`: include dependency information

#### `codedb_explore`
Budgeted source-context excerpts with token limit.

```typescript
const excerpts = await use_mcp_tool("codedb-myproject", "codedb_explore", {
  query: "PoolManager initialization",
  max_chars: 10000  // character budget
});

// Returns: ranked code excerpts staying within max_chars limit
// Output: ~2.5-3.0k tokens
```

**Performance:** 7-29ms

**Difference from `codedb_context`:**
- `codedb_context`: returns summaries + symbols + minimal snippets
- `codedb_explore`: returns larger source excerpts with explicit char budget

### Dependency Analysis

#### `codedb_deps`
Analyze file dependencies (imports, includes, references).

```typescript
// Direct dependencies (what this file imports)
const deps = await use_mcp_tool("codedb-myproject", "codedb_deps", {
  path: "src/services/user.rs",
  direction: "depends_on"
});

// Reverse dependencies (what imports this file)
const rdeps = await use_mcp_tool("codedb-myproject", "codedb_deps", {
  path: "src/models/user.rs",
  direction: "imported_by"
});

// Transitive dependencies (full closure)
const transitive = await use_mcp_tool("codedb-myproject", "codedb_deps", {
  path: "src/core/engine.rs",
  direction: "depends_on",
  transitive: true,
  max_depth: 3
});
```

**Performance:**
- Direct: 0.096ms
- Reverse (first sidecar): 170ms, then sub-ms
- Transitive: depends on depth

**Directions:**
- `depends_on`: outgoing dependencies
- `imported_by`: incoming dependents

### File Operations

#### `codedb_read`
Read indexed file or line range.

```typescript
// Read entire file
const content = await use_mcp_tool("codedb-myproject", "codedb_read", {
  path: "src/main.rs"
});

// Read line range
const snippet = await use_mcp_tool("codedb-myproject", "codedb_read", {
  path: "src/services/auth.rs",
  start_line: 42,
  end_line: 67
});
```

**Performance:** 0.562ms avg

**When to use:**
- After using `codedb_context` to identify relevant files
- Reading specific functions identified by `codedb_symbol`
- NOT as first step for exploration (use `codedb_context` instead)

#### `codedb_tree`
Get indexed file tree with metadata.

```typescript
const tree = await use_mcp_tool("codedb-myproject", "codedb_tree", {
  path_pattern: "src/**/*.rs"
});
// Returns: [{path, language, lines, symbol_count}]
```

**Performance:** 8.782ms

#### `codedb_hot`
Get recently modified indexed files.

```typescript
const recent = await use_mcp_tool("codedb-myproject", "codedb_hot", {
  limit: 20
});
// Returns: files sorted by modification time
```

**Performance:** 2.116ms

### Advanced Features

#### `codedb_bundle`
Execute multiple queries in a single MCP call (reduces round-trips).

```typescript
const bundle = await use_mcp_tool("codedb-myproject", "codedb_bundle", {
  calls: [
    {tool: "codedb_symbol", args: {symbol: "UserService"}},
    {tool: "codedb_deps", args: {path: "src/user.rs", direction: "depends_on"}},
    {tool: "codedb_outline", args: {path: "src/user.rs"}}
  ]
});
// Returns: array of results matching call order
```

**When to use:**
- Gathering related information in one shot
- Reducing MCP overhead
- Building complex queries

#### Module Discovery & Atlas

Generate dependency-aware module groupings:

```typescript
const modules = await use_mcp_tool("codedb-myproject", "codedb_modules", {
  min_size: 3,  // minimum files per module
  max_size: 50  // split large modules
});
// Returns: module groups based on dependency graph + label propagation
```

**Build visual atlas** (requires Node.js):

```bash
cd codedb-mcp/skills/code-module-atlas
node scripts/build-module-atlas.mjs myproject
cd assets/viewer
npm install
npm run dev -- --port 5174
```

Opens interactive 3D viewer showing:
- Star nodes for each source file
- Module boundaries and labels
- Dependency edges
- File details on selection

## Configuration Patterns

### Language-Specific Indexing

```toml
[languages]
rust = { enabled = true }
typescript = { enabled = true, extensions = [".ts", ".tsx"] }
python = { enabled = true, extensions = [".py"] }
csharp = { enabled = true, extensions = [".cs"] }
cpp = { enabled = true, extensions = [".cpp", ".hpp", ".cc", ".h"] }
```

### Performance Tuning

```toml
[performance]
# Parallel indexing threads
threads = 8

# Memory limits
max_memory_mb = 2048

# Cache generations (for rolling updates)
cache_version = 23
```

### Vector Search Configuration

```toml
[vector]
# Model2Vec model from HuggingFace
model = "minishlab/potion-code-16M"

# Cache location (optional, defaults to HuggingFace cache)
cache_dir = "/path/to/model/cache"

# Lazy loading (default: true)
lazy_load = true
```

Vector embeddings are built **lazily** on first semantic search, not during initial indexing.

### Monorepo Configuration

For large monorepos, create separate configs per sub-project:

```bash
project-root/
  .codedb-mcp/codedb-mcp.toml          # Root project
  backend/.codedb-mcp/codedb-mcp.toml  # Backend service
  frontend/.codedb-mcp/codedb-mcp.toml # Frontend app
```

Each can run as separate MCP server instances.

## Common Workflows

### Workflow 1: Understanding a New Feature Area

```typescript
// Step 1: Get high-level context without file dumps
const context = await use_mcp_tool("codedb-project", "codedb_context", {
  query: "how does user authentication work?",
  max_tokens: 2500
});

// Step 2: Find specific entry points
const entry = await use_mcp_tool("codedb-project", "codedb_symbol", {
  symbol: "AuthController|authenticate"
});

// Step 3: Check dependencies
const deps = await use_mcp_tool("codedb-project", "codedb_deps", {
  path: entry[0].path,
  direction: "depends_on"
});

// Step 4: Read focused files
const code = await use_mcp_tool("codedb-project", "codedb_read", {
  path: entry[0].path
});
```

### Workflow 2: Impact Analysis for Refactoring

```typescript
// Step 1: Find the symbol definition
const defs = await use_mcp_tool("codedb-project", "codedb_symbol", {
  symbol: "legacy_payment_process"
});

// Step 2: Find all callers
const callers = await use_mcp_tool("codedb-project", "codedb_callers", {
  path: defs[0].path,
  line: defs[0].line,
  symbol: "legacy_payment_process"
});

// Step 3: Check reverse dependencies
const dependents = await use_mcp_tool("codedb-project", "codedb_deps", {
  path: defs[0].path,
  direction: "imported_by"
});

// Step 4: Bundle outline reads for all affected files
const outlines = await use_mcp_tool("codedb-project", "codedb_bundle", {
  calls: callers.map(c => ({
    tool: "codedb_outline",
    args: {path: c.caller_path}
  }))
});
```

### Workflow 3: Exploring Code Architecture

```typescript
// Step 1: Semantic search for business domain
const modules = await use_mcp_tool("codedb-project", "codedb_search", {
  query: "payment processing workflow",
  mode: "semantic",
  limit: 30
});

// Step 2: Get focused context
const context = await use_mcp_tool("codedb-project", "codedb_context", {
  query: "payment processing workflow",
  focus_paths: ["src/payments/**"],
  max_tokens: 4000
});

// Step 3: Analyze module structure
const deps_analysis = await use_mcp_tool("codedb-project", "codedb_modules", {
  min_size: 5
});
```

### Workflow 4: Incremental Index Updates

```bash
# Daily development routine
git pull
/path/to/codedb-mcp --project-root . index  # Incremental: 0.5-1.5s

# After major branch merge
/path/to/codedb-mcp --project-root . index --force-rebuild  # Full: 10-15s
```

**Incremental performance (1,000 files changed):**
- Add: 1.508s
- Modify: 1.544s
- Delete: 0.504s

## Token Optimization

### Codex Token Observer

Monitor and optimize MCP tool token usage:

```bash
cd codedb-mcp/skills/codedb-mcp
node scripts/codex-observe.mjs --project myproject --since 24h --top 12
```

**Reports:**
- Model token counts per session
- Tool output token estimates
- High-output call patterns
- Missed `codedb_bundle` opportunities
- Non-codedb source lookups that could use codedb tools

**Example findings (Unity C# project):**
- 43% token reduction vs shell-based lookup
- 38% faster execution
- 254k tokens saved across 3 feature analysis sessions

### Best Practices for Token Efficiency

**DO:**
- Start with `codedb_context` for exploration (1.5-2k tokens)
- Use `codedb_explore` with explicit `max_chars` budgets
- Bundle related queries with `codedb_bundle`
- Use `codedb_symbol` + `codedb_callers` instead of grepping
- Scope searches with `focus_paths` or file patterns

**DON'T:**
- Call `codedb_read` on many files without first using `codedb_context`
- Use unbounded `codedb_text_search` on large codebases
- Repeatedly call `codedb_outline` in loops (use `codedb_bundle` instead)
- Dump full file contents when you only need specific symbols

### Token Benchmark Comparison

| Approach | World Map Logic | Hero Attributes | Alliance Rally | Total |
|----------|-----------------|-----------------|----------------|-------|
| codedb-mcp enabled | 92,639 tokens | 114,436 tokens | 128,865 tokens | 335,940 tokens |
| Shell-based lookup | 231,810 tokens | 173,576 tokens | 185,448 tokens | 590,834 tokens |
| **Savings** | **60.0%** | **34.1%** | **30.5%** | **43.1%** |

## Troubleshooting

### Index Not Building

**Symptoms:** `codedb_index` fails or hangs

**Solutions:**
1. Check file permissions on `.codedb-mcp/` directory
2. Verify include/exclude patterns in config:
   ```bash
   /path/to/codedb-mcp --project-root . status
   ```
3. Check for corrupted cache:
   ```bash
   rm -rf .codedb-mcp/*.bin
   /path/to/codedb-mcp --project-root . index --force-rebuild
   ```
4. Enable debug logging:
   ```bash
   RUST_LOG=debug /path/to/codedb-mcp --project-root . index
   ```

### Slow Search Performance

**Symptoms:** `codedb_text_search` or `codedb_search` taking seconds

**Causes:**
- First query triggers lazy sidecar load (expected: 15-30ms)
- Regex search on very large files
- Vector model not cached

**Solutions:**
1. Warm up lazy sidecars:
   ```typescript
   // First call loads word index (~100ms)
   await use_mcp_tool("codedb-project", "codedb_word", {word: "test"});
   
   // First semantic search loads embeddings (~1-2s)
   await use_mcp_tool("codedb-project", "codedb_search", {
     query: "warmup",
     mode: "semantic"
   });
   ```

2. Scope searches to reduce candidate files:
   ```typescript
   await use_mcp_tool("codedb-project", "codedb_text_search", {
     query: "pattern",
     scope: ["src/core/**"]
   });
   ```

3. Use `max_tokens` or `limit` parameters to cap output

### Vector Search Not Working

**Symptoms:** Semantic search returns no results or errors

**Solutions:**
1. Ensure Model2Vec model is downloaded:
   ```bash
   # Check HuggingFace cache
   ls ~/.cache/huggingface/hub/models--minishlab--potion-code-16M/
   ```

2. Configure explicit cache location if needed:
   ```toml
   [vector]
   model = "minishlab/potion-code-16M"
   cache_dir = "/alternative/cache/path"
   ```

3. Trigger embedding build manually:
   ```typescript
   const results = await use_mcp_tool("codedb-project", "codedb_search", {
     query: "any query",
     mode: "semantic"
   });
   // First call builds embeddings (1-2s), subsequent calls are fast
   ```

### Stale Results

**Symptoms:** Search/symbol results don't reflect recent changes

**Solutions:**
1. Check index freshness:
   ```typescript
   const status = await use_mcp_tool("codedb-project", "codedb_status", {});
   // Check "last_indexed" timestamp
   ```

2. Run incremental update:
   ```bash
   /path/to/codedb-mcp --project-root . index
   ```

3. If incremental fails, force rebuild:
   ```bash
   /path/to/codedb-mcp --project-root . index --force-rebuild
   ```

### High Memory Usage

**Symptoms:** codedb-mcp process using excessive RAM

**Solutions:**
1. Adjust thread count in config:
   ```toml
   [performance]
   threads = 4  # reduce from default 8
   ```

2. Limit scope of indexed files:
   ```toml
   [scan]
   exclude = ["vendor/**", "third_party/**", "*.generated.*"]
   ```

3. Disable vector search if not needed:
   ```toml
   [vector]
   enabled = false
   ```

### MCP Server Won't Start

**Symptoms:** Agent can't connect to codedb MCP server

**Solutions:**
1. Test server manually:
   ```bash
   /path/to/codedb-mcp --project-root /path/to/project serve
   # Should start and wait for stdio MCP protocol
   ```

2. Check MCP config syntax:
   ```json
   {
     "mcpServers": {
       "codedb-project": {
         "command": "/absolute/path/to/codedb-mcp",
         "args": ["--project-root", "/absolute/path/to/project", "serve"]
       }
     }
   }
   ```

3. Verify project has index:
   ```bash
   ls /path/to/project/.codedb-mcp/
   # Should contain *.bin files and codedb-mcp.toml
   ```

4. Check server logs in agent output

## DeepWiki Documentation Generation

Generate repository documentation automatically:

```bash
cd codedb-mcp/skills/deepwiki

# Generate full documentation
node scripts/generate-deepwiki.mjs myproject

# Output: myproject-deepwiki/
#   index.md                  # Overview
#   modules/                  # Module-focused pages
#   architecture/             # Architecture diagrams
#   flows/                    # Feature flows
```

**DeepWiki uses:**
- MCP evidence from `codedb_modules`, `codedb_deps`, `codedb_context`
- Agent reasoning to write business-focused explanations
- Cited source files with line numbers
- Dependency-aware structure

**Configure generation:**

```typescript
// In agent context
const wiki = await generate_deepwiki({
  project: "myproject",
  focus_areas: ["payment processing", "user management"],
  max_modules: 20,
  include_risks: true
});
```

## Integration Examples

### With Claude Code

```json
// In Claude Desktop MCP config
{
  "mcpServers": {
    "codedb-backend": {
      "command": "/usr/local/bin/codedb-mcp",
      "args": ["--project-root", "/home/user/projects/backend", "serve"]
    }
  }
}
```

Then in Claude:
- "Search for authentication logic in the backend"
- "Show me what depends on UserService"
- "Find all callers of validate_token"

### With Cursor

Add to Cursor MCP settings, then use in composer:

```
@codedb-project how does the payment processing workflow handle refunds?
```

The skill will use `codedb_context` to gather relevant code without dumping files.

### With Codex

```bash
# Enable codedb-mcp skill and MCP server
codex config add-mcp codedb-project /path/to/codedb-mcp --project-root /path/to/project

# Use in conversation
codex chat "analyze the alliance rally join flow in u3dclient"
```

Codex will automatically use `codedb_context`, `codedb_search`, `codedb_deps` tools to answer efficiently.

## Performance Optimization

### Index Optimization

```toml
# Optimize for large codebases
[performance]
threads = 16                    # Use more CPU for faster indexing
max_memory_mb = 4096           # Allow more memory for large graphs

[scan]
exclude = [
  "node_modules/**",
  "vendor/**",
  "*.generated.*",
  "*.pb.*",                    # Generated protobuf
  "dist/**",
  "build/**"
]

# Skip large non-source files
max_file_size_kb = 1024
```

### Query Optimization

```typescript
// Prefer narrow scopes
const results = await use_mcp_tool("codedb-project", "codedb_context", {
  query: "payment logic",
  focus_paths: ["src/payments/**", "src/billing/**"],
  max_tokens: 2000
});

// Bundle related queries
const data = await use_mcp_tool("codedb-project", "codedb_bundle", {
  calls: [
    {tool: "codedb_symbol", args: {symbol: "PaymentProcessor"}},
    {tool: "codedb_context", args: {query: "payment flow", max_tokens: 1500}},
    {tool: "codedb_deps", args: {path: "src/payments/processor.rs"}}
  ]
});
```

### Cache Management

```bash
# Check cache size
du -sh /path/to/project/.codedb-mcp/

# Clean old generations (keep only current)
/path/to/codedb-mcp --project-root . cleanup

# Full rebuild if cache corrupted
rm -rf /path/to/project/.codedb-mcp/*.bin
/path/to/codedb-mcp --project-root . index --force-rebuild
```

## Advanced Rust API (for tool development)

If building custom tools that integrate with codedb:

```rust
use codedb_mcp::{CodeDb, IndexConfig, SearchMode};
