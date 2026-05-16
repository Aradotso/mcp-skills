---
name: jcodemunch-mcp-code-retrieval
description: Token-efficient GitHub source code exploration via tree-sitter AST parsing and structured retrieval
triggers:
  - install jcodemunch for code navigation
  - set up structured code retrieval
  - index my codebase with jcodemunch
  - find function implementations efficiently
  - reduce token usage for code reading
  - search code symbols with tree-sitter
  - explore repository structure efficiently
  - get code context with token budget
---

# jCodeMunch MCP - Structured Code Retrieval

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

jCodeMunch is an MCP server that indexes codebases using tree-sitter AST parsing and enables structured retrieval of code symbols (functions, classes, methods, constants) with byte-level precision. It cuts code-reading token usage by 95%+ by letting agents retrieve exact implementations instead of reading entire files.

## Core Concept

Traditional approach: open files → scan thousands of lines → repeat (token incinerator).

jCodeMunch approach: index once → query cheaply → retrieve exact symbols (95%+ token savings).

## Installation

### Quick Install (VS Code, Cursor, Claude Code)

**VS Code:**
```bash
# One-click via badge or manual:
uvx jcodemunch-mcp
```

**Cursor:**
Use the one-click install badge or add to MCP settings:
```json
{
  "mcpServers": {
    "jcodemunch": {
      "command": "uvx",
      "args": ["jcodemunch-mcp"]
    }
  }
}
```

**Claude Code (CLI):**
```bash
# Install via uvx
uvx jcodemunch-mcp

# Or use the jcm CLI helper
jcm install claude-code
```

### Universal Install (Any MCP-compatible client)

Add to your MCP configuration file:

```json
{
  "mcpServers": {
    "jcodemunch": {
      "command": "uvx",
      "args": ["jcodemunch-mcp"]
    }
  }
}
```

### CLI Installation

```bash
# Install the jcm CLI
pip install jcodemunch-mcp

# Or use uvx
uvx jcodemunch-mcp
```

## Configuration

Create `.jcodemunch/config.jsonc` in your project root:

```jsonc
{
  // Index settings
  "index_path": ".jcodemunch/index",
  "excluded_patterns": [
    "node_modules/**",
    "venv/**",
    ".git/**",
    "*.pyc",
    "__pycache__/**"
  ],
  
  // Token budget defaults
  "default_token_budget": 8000,
  "max_token_budget": 16000,
  
  // Compact format (MUNCH) - saves 45.5% tokens on average
  "compact_format_enabled": true,
  "compact_format_threshold": 0.15, // Use compact if ≥15% savings
  
  // Semantic search (optional, requires sentence-transformers)
  "semantic_search_enabled": false,
  "embedding_model": "sentence-transformers/all-MiniLM-L6-v2",
  
  // Tool configuration
  "disabled_tools": [],  // Disable specific tools if needed
  
  // Language support
  "supported_languages": [
    "python", "javascript", "typescript", "go", "rust",
    "java", "c", "cpp", "c_sharp", "ruby", "php"
  ]
}
```

## Core Tools & Usage

### 1. Indexing

**Index a repository:**
```python
# Via MCP tool call
index_repository(
    path="/path/to/repo",
    force_reindex=False  # Set True to rebuild from scratch
)
```

**Check index status:**
```python
get_index_info()
# Returns: stats, file count, symbol count, last indexed time
```

### 2. Symbol Search

**Find symbols by name (BM25 + fuzzy matching):**
```python
find_symbols(
    query="get_user",
    limit=10,
    file_pattern="*.py",  # Optional filter
    format="auto"  # auto|compact|json
)
```

**Find implementations (multi-source resolution):**
```python
find_implementations(
    symbol="UserService.authenticate",
    include_lsp=True,  # LSP dispatch
    include_hierarchy=True,  # Class hierarchy
    include_duck_typed=True,  # Duck-typed matches
    include_decorators=True  # Decorator handlers
)
```

### 3. Context Retrieval

**Get exact function/class implementation:**
```python
get_symbol_content(
    identifier="UserService.authenticate",
    include_docstring=True,
    include_decorators=True,
    format="auto"
)
```

**Get context bundle with token budget:**
```python
get_ranked_context(
    query="authentication logic",
    token_budget=4000,
    include_imports=True,
    include_references=True,
    format="auto"
)
# Returns: ranked symbols within budget, total tokens used
```

**Task-aware context orchestration:**
```python
assemble_task_context(
    task="fix bug in user authentication where tokens expire too early",
    token_budget=8000,
    session_id="fix-auth-bug-001"  # Optional session tracking
)
# Auto-classifies intent (bug_fix, feature, refactor, etc.)
# Extracts anchor symbols from task description
# Runs appropriate tool sequence under budget
```

### 4. Structural Queries

**Find references to a symbol:**
```python
find_references(
    identifier="get_user",
    include_calls=True,
    include_imports=True,
    format="auto"
)
```

**Find who imports a module/symbol:**
```python
find_importers(
    target="services.auth",
    format="auto"
)
```

**Get blast radius (impact analysis):**
```python
get_blast_radius(
    identifier="User.email",
    max_depth=3,
    include_source=True,  # Include source snippets
    format="auto"
)
```

**Class hierarchy:**
```python
get_class_hierarchy(
    class_name="BaseModel",
    direction="both",  # up|down|both
    format="auto"
)
```

### 5. Code Quality & Refactoring

**Find dead code:**
```python
find_dead_code(
    scope="all",  # all|file|module
    include_private=True,
    format="auto"
)
```

**Find untested symbols:**
```python
get_untested_symbols(
    scope="all",
    min_complexity=5,  # Focus on complex code
    format="auto"
)
```

**Find similar/duplicate code:**
```python
find_similar_symbols(
    threshold=0.8,  # Similarity threshold (0-1)
    min_cluster_size=2,
    use_semantic=True,  # Requires semantic search enabled
    use_structural=True,
    use_behavioral=True,  # Callee Jaccard
    format="auto"
)
# Returns: clusters with canonical pick + consolidation verdict
```

**Check if safe to delete:**
```python
check_delete_safe(
    identifier="legacy_auth_handler",
    format="auto"
)
# Returns: composite verdict + ranked blockers + recommended action
```

### 6. Architectural Analysis

**Get symbol importance (PageRank):**
```python
get_symbol_importance(
    limit=20,
    scope="all",  # all|file|module
    format="auto"
)
```

**Repository overview map (cold-start orientation):**
```python
get_repo_map(
    token_budget=4000,
    signature_only=True,  # Just signatures, not full implementations
    format="auto"
)
# Returns: PageRank-ranked symbol overview within budget
```

**Dependency cycles:**
```python
get_dependency_cycles(
    format="auto"
)
```

**Hotspot detection (complexity × churn):**
```python
get_hotspots(
    min_complexity=10,
    days_back=90,
    format="auto"
)
```

### 7. Multi-Repo Operations

**Cross-repo API contracts:**
```python
get_group_contracts(
    repos=["/path/to/repo1", "/path/to/repo2"],
    min_shared=2,  # Minimum repos sharing symbol
    format="auto"
)
# Returns: ranked contracts classified as:
# - de_facto_api (stable, high usage)
# - leaky_internal (unintended exposure)
# - dead_contract (no runtime hits)
# - version_skew (inconsistent across repos)
```

### 8. Git Integration

**Get changed symbols from git diff:**
```python
get_changed_symbols(
    base_ref="main",
    head_ref="feature-branch",
    format="auto"
)
```

## Real-World Workflows

### Workflow 1: Fix a Bug

```python
# 1. Get task-oriented context
assemble_task_context(
    task="fix NullPointerException in payment processing",
    token_budget=6000,
    session_id="fix-payment-bug"
)

# 2. Find related code
find_symbols(
    query="payment process",
    limit=5
)

# 3. Get implementation + references
get_symbol_content(identifier="PaymentService.process")
find_references(identifier="PaymentService.process")

# 4. Check blast radius before fix
get_blast_radius(
    identifier="PaymentService.process",
    max_depth=2,
    include_source=True
)
```

### Workflow 2: Refactor Dead Code

```python
# 1. Find dead code
dead_symbols = find_dead_code(scope="all", include_private=True)

# 2. For each dead symbol, verify safety
check_delete_safe(identifier="legacy_user_handler")

# 3. Get blast radius to confirm
get_blast_radius(identifier="legacy_user_handler", max_depth=1)

# 4. Find similar code that might be consolidated
find_similar_symbols(threshold=0.85, min_cluster_size=2)
```

### Workflow 3: Understand New Codebase

```python
# 1. Get high-level overview
get_repo_map(token_budget=3000, signature_only=True)

# 2. Find most important symbols
get_symbol_importance(limit=15)

# 3. Explore class hierarchies
get_class_hierarchy(class_name="BaseController", direction="down")

# 4. Get architectural overview
get_dependency_cycles()
get_hotspots(min_complexity=8, days_back=30)
```

### Workflow 4: Feature Implementation

```python
# 1. Find where similar features are implemented
find_symbols(query="user authentication", limit=10)

# 2. Get ranked context for the task
assemble_task_context(
    task="add OAuth2 authentication alongside existing password auth",
    token_budget=8000
)

# 3. Find all authentication-related symbols
get_ranked_context(
    query="authentication oauth password",
    token_budget=5000,
    include_imports=True
)

# 4. Check who will be affected
find_importers(target="auth.handlers")
```

## Compact Format (MUNCH)

All tools accept `format` parameter:

- `auto` - Use compact if ≥15% savings, else JSON (default)
- `compact` - Always use compact format (45.5% avg token savings)
- `json` - Always use JSON (backwards compatible)

Example savings on `get_blast_radius`:
- JSON: 3,850 tokens
- Compact: 700 tokens (5.5× reduction)

## Advanced Features

### Semantic Search (Optional)

Enable in config:
```jsonc
{
  "semantic_search_enabled": true,
  "embedding_model": "sentence-transformers/all-MiniLM-L6-v2"
}
```

Install dependencies:
```bash
pip install sentence-transformers torch
```

Use hybrid search:
```python
find_symbols(
    query="handles user authentication with tokens",
    use_semantic=True,  # Combines BM25 + embeddings
    limit=10
)
```

### Custom Context Providers

Add dbt or Git context:
```python
# dbt models as context
get_dbt_context(
    model_name="fct_orders",
    include_upstream=True,
    include_downstream=True
)

# Git context
get_changed_symbols(base_ref="main", head_ref="HEAD")
```

### Session Management

Track multi-turn context:
```python
plan_turn(
    session_id="feature-impl-001",
    task="implement OAuth2",
    budget=10000
)

assemble_task_context(
    task="add Google OAuth provider",
    session_id="feature-impl-001",
    token_budget=6000
)
```

## Language Support

Fully supported via tree-sitter:
- Python, JavaScript, TypeScript, Go, Rust
- Java, C, C++, C#, Ruby, PHP
- And more (see `LANGUAGE_SUPPORT.md`)

## Troubleshooting

### Index not found
```python
# Force reindex
index_repository(path=".", force_reindex=True)
```

### Semantic search not working
```bash
# Install required dependencies
pip install sentence-transformers torch
```

### Too many results
```python
# Use stricter filters
find_symbols(
    query="handler",
    file_pattern="**/controllers/*.py",
    limit=5
)
```

### Token budget exceeded
```python
# Use smaller budget or signature-only mode
get_repo_map(token_budget=2000, signature_only=True)
```

### Performance issues
```jsonc
// Add to config.jsonc
{
  "excluded_patterns": [
    "node_modules/**",
    "venv/**",
    "dist/**",
    "build/**",
    ".git/**"
  ]
}
```

## CLI Commands (jcm)

```bash
# Install client-specific config
jcm install claude-code
jcm install cursor
jcm install windsurf

# Index current directory
jcm index .

# Search symbols
jcm search "UserService"

# Get symbol info
jcm info "UserService.authenticate"

# Check config
jcm config --validate
```

## Best Practices

1. **Index early**: Run `index_repository()` before starting work
2. **Use token budgets**: Always specify budget for context retrieval
3. **Enable compact format**: Set `format="auto"` for 45%+ token savings
4. **Filter aggressively**: Use `file_pattern` and `scope` to narrow results
5. **Use task context**: `assemble_task_context()` auto-orchestrates the right tools
6. **Check blast radius**: Before refactoring, verify impact with `get_blast_radius()`
7. **Exclude build artifacts**: Add node_modules, dist, venv to `excluded_patterns`
8. **Use semantic search**: For natural language queries, enable embeddings

## Commercial Use

- **Free**: Non-commercial use
- **Builder ($79)**: 1 developer
- **Studio ($349)**: Up to 5 developers
- **Platform ($1,999)**: Organization-wide

See license details at https://j.gravelle.us/jCodeMunch/descriptions.php
