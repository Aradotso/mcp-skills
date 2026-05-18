---
name: mcp-documentation-server
description: MCP server providing local-first document management with AI-powered semantic search, hybrid vector search, and intelligent chunking using Orama and Gemini
triggers:
  - set up documentation search server
  - add semantic search to my documents
  - configure MCP documentation server
  - search documents with embeddings
  - upload and process documentation files
  - create AI-powered knowledge base
  - index documents with vector search
  - manage local document database
---

# mcp-documentation-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

MCP Documentation Server provides local-first document management with semantic search capabilities. It uses an embedded Orama vector database for hybrid full-text and vector search, intelligent parent-child chunking for better context, and optional Google Gemini AI integration for advanced document analysis.

## Installation

### Quick Start with MCP Client

Add to your MCP client configuration (e.g., Claude Desktop `~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "documentation": {
      "command": "npx",
      "args": ["-y", "@andrea9293/mcp-documentation-server"]
    }
  }
}
```

### With Environment Variables

```json
{
  "mcpServers": {
    "documentation": {
      "command": "npx",
      "args": ["-y", "@andrea9293/mcp-documentation-server"],
      "env": {
        "MCP_BASE_DIR": "/path/to/workspace",
        "GEMINI_API_KEY": "your-gemini-api-key",
        "MCP_EMBEDDING_MODEL": "Xenova/paraphrase-multilingual-mpnet-base-v2",
        "START_WEB_UI": "true",
        "WEB_PORT": "3080"
      }
    }
  }
}
```

### Development Installation

```bash
git clone https://github.com/andrea9293/mcp-documentation-server.git
cd mcp-documentation-server
npm install
npm run build
```

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_BASE_DIR` | `~/.mcp-documentation-server` | Base directory for data storage |
| `MCP_EMBEDDING_MODEL` | `Xenova/all-MiniLM-L6-v2` | Embedding model (384 dims) |
| `GEMINI_API_KEY` | — | Google Gemini API key for AI search |
| `MCP_CACHE_ENABLED` | `true` | Enable LRU embedding cache |
| `START_WEB_UI` | `true` | Start built-in web interface |
| `WEB_PORT` | `3080` | Web UI port |
| `MCP_STREAMING_ENABLED` | `true` | Stream large files |
| `MCP_STREAM_CHUNK_SIZE` | `65536` | Streaming buffer (64KB) |
| `MCP_STREAM_FILE_SIZE_LIMIT` | `10485760` | Streaming threshold (10MB) |

### Embedding Models

**Fast (default):**
- `Xenova/all-MiniLM-L6-v2` — 384 dimensions, ~80MB

**High Quality (recommended):**
- `Xenova/paraphrase-multilingual-mpnet-base-v2` — 768 dimensions, ~420MB, multilingual

⚠️ **Changing models requires re-indexing all documents** (embeddings are incompatible).

### Storage Structure

```
~/.mcp-documentation-server/
├── data/
│   ├── orama-chunks.msp      # Vector DB (child chunks + embeddings)
│   ├── orama-docs.msp        # Document DB (full content + metadata)
│   ├── orama-parents.msp     # Parent chunks DB (context sections)
│   ├── migration-complete.flag
│   └── *.md                  # Markdown document copies
└── uploads/                  # Drop files here for processing
```

## MCP Tools

### Document Management

#### add_document

Add a new document to the knowledge base.

```typescript
// Tool call
{
  "title": "API Reference",
  "content": "# Authentication\nUse Bearer tokens...",
  "metadata": {
    "category": "api",
    "version": "2.0",
    "author": "team"
  }
}
```

Response includes document ID, chunk count, and timing stats.

#### list_documents

List all documents with metadata and previews.

```typescript
// Tool call (no parameters required)

// Returns array of documents:
[
  {
    "id": "doc_abc123",
    "title": "API Reference",
    "preview": "# Authentication\nUse Bearer...",
    "metadata": { "category": "api" },
    "created": "2025-01-15T10:30:00Z",
    "updated": "2025-01-15T10:30:00Z"
  }
]
```

#### get_document

Retrieve full document content by ID.

```typescript
// Tool call
{
  "id": "doc_abc123"
}

// Returns complete document with metadata and full content
```

#### delete_document

Remove a document and all associated data.

```typescript
// Tool call
{
  "id": "doc_abc123"
}

// Deletes document, chunks, embeddings, and backup files
```

### File Processing

#### process_uploads

Process all files from the uploads folder.

```typescript
// Tool call (no parameters)
// Processes .txt, .md, .pdf files from ~/.mcp-documentation-server/uploads/

// Returns:
{
  "processed": 3,
  "failed": 0,
  "results": [
    {
      "filename": "guide.md",
      "success": true,
      "documentId": "doc_xyz789",
      "chunks": 12
    }
  ]
}
```

#### get_uploads_path

Get the absolute path to the uploads folder.

```typescript
// Tool call (no parameters)
// Returns: "/Users/username/.mcp-documentation-server/uploads"
```

#### list_uploads_files

List files in the uploads folder.

```typescript
// Tool call (no parameters)

// Returns:
[
  {
    "name": "api-guide.md",
    "size": 45678,
    "sizeFormatted": "44.6 KB",
    "extension": ".md"
  }
]
```

#### get_ui_url

Get the Web UI URL.

```typescript
// Tool call (no parameters)
// Returns: "http://localhost:3080"
```

### Search Tools

#### search_documents

Semantic vector search within a specific document.

```typescript
// Tool call
{
  "documentId": "doc_abc123",
  "query": "authentication methods",
  "limit": 5
}

// Returns:
{
  "results": [
    {
      "content": "Bearer token authentication is used...",
      "parentContent": "# Authentication\nBearer token authentication...",
      "score": 0.92,
      "documentId": "doc_abc123",
      "documentTitle": "API Reference"
    }
  ],
  "total": 5
}
```

#### search_all_documents

Hybrid full-text + vector search across all documents.

```typescript
// Tool call
{
  "query": "rate limiting configuration",
  "limit": 10
}

// Returns deduplicated results sorted by relevance
// Includes both exact text matches and semantic similarity
```

#### get_context_window

Fetch surrounding chunks for richer context.

```typescript
// Tool call
{
  "documentId": "doc_abc123",
  "chunkIndex": 5,
  "windowSize": 2
}

// Returns chunks [3, 4, 5, 6, 7] with the target chunk at index 5
// Useful for giving LLMs broader context around a search result
```

#### search_documents_with_ai

AI-powered search using Google Gemini (requires `GEMINI_API_KEY`).

```typescript
// Tool call
{
  "documentIds": ["doc_abc123", "doc_xyz789"],
  "query": "How do I implement rate limiting with Redis?",
  "conversationHistory": [
    {
      "role": "user",
      "content": "What caching options are available?"
    },
    {
      "role": "assistant",
      "content": "Redis and Memcached are supported..."
    }
  ]
}

// Returns AI-generated answer with context and sources
```

## Common Patterns

### Setting Up a Knowledge Base

```typescript
// 1. Add documents programmatically
const apiDoc = await mcp.call("add_document", {
  title: "REST API Guide",
  content: "# REST API\n\n## Endpoints...",
  metadata: { type: "api", version: "1.0" }
});

// 2. Or drop files in uploads folder
const uploadsPath = await mcp.call("get_uploads_path");
// Copy files to uploadsPath, then:
const result = await mcp.call("process_uploads");

// 3. Verify documents
const docs = await mcp.call("list_documents");
console.log(`${docs.length} documents indexed`);
```

### Semantic Search Workflow

```typescript
// Search across all documents
const results = await mcp.call("search_all_documents", {
  query: "database connection pooling",
  limit: 5
});

// Get more context for top result
if (results.results.length > 0) {
  const topResult = results.results[0];
  const context = await mcp.call("get_context_window", {
    documentId: topResult.documentId,
    chunkIndex: topResult.chunkIndex,
    windowSize: 3
  });
  
  // context.chunks contains surrounding sections
}
```

### AI-Assisted Research

```typescript
// Use Gemini for complex queries
const answer = await mcp.call("search_documents_with_ai", {
  documentIds: ["doc_1", "doc_2"],
  query: "Compare authentication approaches and recommend best practices",
  conversationHistory: []
});

// answer contains:
// - AI-generated response
// - Source chunks with citations
// - Confidence scores
```

### Batch Document Processing

```typescript
// Process multiple files efficiently
const files = await mcp.call("list_uploads_files");
console.log(`Found ${files.length} files to process`);

const result = await mcp.call("process_uploads");

result.results.forEach(item => {
  if (item.success) {
    console.log(`✓ ${item.filename}: ${item.chunks} chunks`);
  } else {
    console.error(`✗ ${item.filename}: ${item.error}`);
  }
});
```

### Managing Document Lifecycle

```typescript
// List and filter documents
const allDocs = await mcp.call("list_documents");
const apiDocs = allDocs.filter(doc => 
  doc.metadata?.category === "api"
);

// Update a document (delete + re-add)
await mcp.call("delete_document", { id: "doc_old" });
await mcp.call("add_document", {
  title: "Updated API Docs",
  content: updatedContent,
  metadata: { category: "api", version: "2.0" }
});
```

## Web UI

Access the built-in web interface at `http://localhost:3080` (default).

Features:
- Dashboard with document statistics
- Upload drag & drop interface
- Visual search across all documents
- Document browser with delete/view
- AI search interface (if Gemini key configured)
- Context window explorer

Disable with `START_WEB_UI=false` or change port with `WEB_PORT`.

## Troubleshooting

### Embeddings Not Generated

**Symptom:** Search returns no results or poor results.

**Solution:**
```bash
# Check if model is downloaded
ls ~/.cache/huggingface/

# Force re-download by clearing cache
rm -rf ~/.cache/huggingface/
# Restart server to re-download
```

### Model Change Not Taking Effect

**Symptom:** Changed `MCP_EMBEDDING_MODEL` but search quality unchanged.

**Solution:**
```bash
# Delete existing database
rm ~/.mcp-documentation-server/data/orama-*.msp

# Re-add all documents
# Server will recreate database with new model dimensions
```

### Large File Processing Fails

**Symptom:** File upload times out or fails.

**Solution:**
```json
{
  "env": {
    "MCP_STREAMING_ENABLED": "true",
    "MCP_STREAM_FILE_SIZE_LIMIT": "5242880"
  }
}
```

Reduces threshold to 5MB to trigger streaming mode earlier.

### AI Search Not Available

**Symptom:** `search_documents_with_ai` tool missing.

**Solution:**
- Verify `GEMINI_API_KEY` is set in configuration
- Get API key from https://aistudio.google.com/app/apikey
- Restart MCP server after adding key

### Port Already in Use

**Symptom:** Web UI fails to start with `EADDRINUSE`.

**Solution:**
```json
{
  "env": {
    "WEB_PORT": "3081"
  }
}
```

Or disable Web UI: `"START_WEB_UI": "false"`

### Memory Issues with Large Documents

**Symptom:** Server crashes with out-of-memory error.

**Solution:**
- Enable streaming: `MCP_STREAMING_ENABLED=true`
- Reduce chunk size: Process files individually
- Use more efficient model: `Xenova/all-MiniLM-L6-v2` (384 dims)

### Migration from Legacy JSON

**Symptom:** Old documents not appearing.

**Solution:**
The server automatically migrates legacy JSON documents on first startup. Check:
```bash
ls ~/.mcp-documentation-server/data/migration-complete.flag
```

If flag exists but documents missing, check logs for migration errors.

## Development

Run in development mode:
```bash
npm run dev       # MCP server with hot reload
npm run inspect   # FastMCP web UI for testing tools
npm run web       # Web UI only (development)
```

Build and run production:
```bash
npm run build
npm start
```

## Architecture

- **FastMCP** — MCP server framework with stdio transport
- **Orama** — Embedded vector database (3 instances: docs, chunks, parents)
- **Transformers.js** — Local embedding generation (@xenova/transformers)
- **Parent-Child Chunking** — Large context parents + small precise children
- **LRU Cache** — In-memory embedding cache for performance
- **Streaming Reader** — Handle large files without memory bloat

Data flow:
1. Document → IntelligentChunker → Parent chunks + Child chunks
2. Child chunks → EmbeddingProvider → Vectors (384/768 dims)
3. Vectors + chunks → OramaStore → Persisted to disk
4. Query → Hybrid search (text + vector) → Deduplicated by parent → Results with context
