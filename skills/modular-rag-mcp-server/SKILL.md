---
name: modular-rag-mcp-server
description: Expert in deploying and customizing a modular RAG system with MCP protocol for AI assistants
triggers:
  - how do I set up the modular RAG MCP server
  - help me configure the RAG knowledge hub
  - integrate RAG with Claude Desktop using MCP
  - troubleshoot hybrid search and reranking
  - add documents to the RAG system
  - evaluate RAG performance with Ragas
  - customize RAG components like embeddings or reranker
  - run the RAG dashboard
---

# Modular RAG MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Expert skill for deploying, configuring, and extending the Modular RAG MCP Server — a pluggable, observable RAG (Retrieval-Augmented Generation) system that exposes tools via Model Context Protocol for AI assistants like Claude Desktop and GitHub Copilot.

## What This Project Does

The Modular RAG MCP Server is a complete RAG pipeline featuring:

- **Ingestion Pipeline**: PDF → Markdown → Chunking → Embedding → Vector Store (with multimodal image captioning)
- **Hybrid Search**: Dense vectors (semantic) + Sparse BM25 (exact match) + RRF fusion + optional reranking
- **MCP Protocol**: Standard MCP server exposing `query_knowledge_hub`, `list_collections`, `get_document_summary` tools
- **Dashboard**: Streamlit-based management UI with 6 pages (overview, data browser, ingestion tracking, query tracking, evaluation)
- **Evaluation Framework**: Ragas + custom metrics for regression testing
- **Full Observability**: White-box tracing of ingestion and query pipelines

**Key Architecture**: Every core component (LLM, Embedding, Reranker, Splitter, VectorStore, Evaluator) is pluggable via abstract interfaces. Switch backends through configuration without code changes.

## Installation

### Prerequisites

- Python 3.9+
- VS Code with GitHub Copilot or Claude Desktop
- API keys for your chosen providers (OpenAI, Anthropic, Cohere, etc.)

### Quick Setup with Setup Skill

The project includes a **Setup Skill** that automates the entire configuration:

```bash
# Clone the repository
git clone https://github.com/jerry-ai-dev/MODULAR-RAG-MCP-SERVER.git
cd MODULAR-RAG-MCP-SERVER

# In VS Code with Copilot/Claude, type in chat:
setup
```

The Setup Skill will:
1. Ask you to select providers (OpenAI, Anthropic, Cohere, etc.)
2. Configure API keys
3. Install dependencies
4. Generate configuration files
5. Launch the dashboard

### Manual Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Copy and configure environment variables
cp .env.example .env
# Edit .env with your API keys
```

## Configuration

### Main Configuration File (`src/core/config.py`)

The system uses a centralized configuration approach. Key settings:

```python
from src.core.config import get_config

config = get_config()

# Access configuration
llm_provider = config.llm.provider  # "openai", "anthropic", etc.
embedding_provider = config.embedding.provider
vector_store_type = config.vector_store.type  # "qdrant", "chroma", etc.
```

### Environment Variables

Create `.env` file with required keys:

```bash
# LLM Provider
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Embedding Provider
COHERE_API_KEY=your_cohere_key_here

# Reranker (optional)
JINA_API_KEY=your_jina_key_here

# Vector Store (if using cloud)
QDRANT_URL=your_qdrant_url
QDRANT_API_KEY=your_qdrant_key
```

### Provider Configuration

Edit `src/core/config.py` to set default providers:

```python
class LLMConfig:
    provider: str = "openai"  # or "anthropic", "cohere"
    model: str = "gpt-4"
    temperature: float = 0.7
    max_tokens: int = 2048

class EmbeddingConfig:
    provider: str = "openai"  # or "cohere", "huggingface"
    model: str = "text-embedding-3-small"
    dimension: int = 1536

class RerankerConfig:
    enabled: bool = True
    provider: str = "cohere"  # or "jina", "cross-encoder"
    model: str = "rerank-english-v3.0"
    top_k: int = 5
```

## Key Components and API

### 1. Ingestion Pipeline

Ingest documents into the knowledge base:

```python
from src.ingestion.pipeline import IngestionPipeline
from src.core.config import get_config

config = get_config()
pipeline = IngestionPipeline(config)

# Ingest a PDF document
result = pipeline.ingest_document(
    file_path="path/to/document.pdf",
    collection_name="my_collection",
    metadata={"source": "internal_docs", "version": "1.0"}
)

print(f"Ingested {result['chunks_created']} chunks")
print(f"Ingestion ID: {result['ingestion_id']}")
```

### 2. Hybrid Search and Query

Query the knowledge base with hybrid search:

```python
from src.retrieval.hybrid_search import HybridSearchRetriever
from src.core.config import get_config

config = get_config()
retriever = HybridSearchRetriever(config)

# Perform hybrid search
results = retriever.retrieve(
    query="How does the authentication system work?",
    collection_name="my_collection",
    top_k=10,  # Initial retrieval
    rerank_top_k=5  # After reranking
)

for idx, result in enumerate(results):
    print(f"{idx+1}. Score: {result.score:.4f}")
    print(f"   Text: {result.text[:100]}...")
    print(f"   Metadata: {result.metadata}")
```

### 3. MCP Server Integration

The MCP server exposes tools for AI assistants. Start the server:

```bash
# Run MCP server (usually configured in Claude Desktop config)
python src/mcp/server.py
```

Configure in Claude Desktop (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "rag-knowledge-hub": {
      "command": "python",
      "args": ["/path/to/project/src/mcp/server.py"],
      "env": {
        "PYTHONPATH": "/path/to/project"
      }
    }
  }
}
```

**Available MCP Tools:**

1. **query_knowledge_hub**: Query the RAG system
   ```python
   # When Claude calls this tool:
   {
     "query": "What are the deployment requirements?",
     "collection_name": "my_collection",
     "top_k": 5
   }
   ```

2. **list_collections**: List all available collections
   ```python
   # Returns: ["collection1", "collection2", ...]
   ```

3. **get_document_summary**: Get summary of a specific document
   ```python
   {
     "document_id": "doc_123",
     "collection_name": "my_collection"
   }
   ```

### 4. Dashboard

Launch the Streamlit dashboard:

```bash
streamlit run src/dashboard/app.py
```

Dashboard pages:
- **Overview**: System status, collection stats, recent activity
- **Data Browser**: Browse and search ingested documents
- **Ingestion Management**: Upload new documents, view ingestion history
- **Ingestion Tracking**: Monitor ingestion pipeline steps
- **Query Tracking**: Analyze query performance and results
- **Evaluation Panel**: Run evaluations with Ragas metrics

### 5. Evaluation with Ragas

Evaluate RAG performance:

```python
from src.evaluation.evaluator import RAGEvaluator
from src.core.config import get_config

config = get_config()
evaluator = RAGEvaluator(config)

# Prepare test dataset
test_cases = [
    {
        "query": "What is the API rate limit?",
        "expected_answer": "The API rate limit is 1000 requests per hour.",
        "ground_truth_context": ["Rate limits are set to 1000 req/hour..."]
    },
    # ... more test cases
]

# Run evaluation
results = evaluator.evaluate(
    test_cases=test_cases,
    collection_name="my_collection",
    metrics=["faithfulness", "answer_relevancy", "context_precision"]
)

print(f"Average Faithfulness: {results['faithfulness']:.3f}")
print(f"Average Answer Relevancy: {results['answer_relevancy']:.3f}")
```

## Common Patterns

### Switching Embedding Providers

To switch from OpenAI to Cohere embeddings:

```python
# In src/core/config.py
class EmbeddingConfig:
    provider: str = "cohere"  # Changed from "openai"
    model: str = "embed-english-v3.0"
    dimension: int = 1024  # Cohere dimension
```

Or programmatically:

```python
from src.core.config import get_config

config = get_config()
config.embedding.provider = "cohere"
config.embedding.model = "embed-english-v3.0"
config.embedding.dimension = 1024
```

### Adding Custom Chunking Strategy

Implement a custom text splitter:

```python
from src.ingestion.splitters.base import BaseSplitter
from typing import List

class CustomSplitter(BaseSplitter):
    def __init__(self, chunk_size: int = 500, overlap: int = 50):
        self.chunk_size = chunk_size
        self.overlap = overlap
    
    def split(self, text: str, metadata: dict = None) -> List[dict]:
        chunks = []
        start = 0
        while start < len(text):
            end = start + self.chunk_size
            chunk_text = text[start:end]
            chunks.append({
                "text": chunk_text,
                "metadata": {
                    **(metadata or {}),
                    "chunk_index": len(chunks),
                    "start_char": start
                }
            })
            start += self.chunk_size - self.overlap
        return chunks

# Register and use
from src.ingestion.pipeline import IngestionPipeline

pipeline = IngestionPipeline(config)
pipeline.splitter = CustomSplitter(chunk_size=300, overlap=30)
```

### Implementing Custom Reranker

```python
from src.retrieval.rerankers.base import BaseReranker
from typing import List

class CustomReranker(BaseReranker):
    def rerank(self, query: str, documents: List[dict], top_k: int = 5) -> List[dict]:
        # Custom reranking logic
        scored_docs = []
        for doc in documents:
            # Example: simple keyword matching score
            score = sum(1 for word in query.lower().split() 
                       if word in doc['text'].lower())
            scored_docs.append({**doc, 'rerank_score': score})
        
        # Sort by score and return top_k
        scored_docs.sort(key=lambda x: x['rerank_score'], reverse=True)
        return scored_docs[:top_k]

# Use in retriever
from src.retrieval.hybrid_search import HybridSearchRetriever

retriever = HybridSearchRetriever(config)
retriever.reranker = CustomReranker()
```

### Multimodal Image Processing

The system supports image captioning in PDFs:

```python
from src.ingestion.pipeline import IngestionPipeline

pipeline = IngestionPipeline(config)

# Enable image captioning
result = pipeline.ingest_document(
    file_path="document_with_images.pdf",
    collection_name="multimodal_docs",
    enable_image_captioning=True,  # Vision LLM generates descriptions
    metadata={"type": "technical_manual"}
)

# Images are converted to text descriptions and embedded with surrounding text
```

### Batch Ingestion

Ingest multiple documents:

```python
import os
from pathlib import Path

pipeline = IngestionPipeline(config)
docs_dir = Path("./documents")

results = []
for pdf_file in docs_dir.glob("*.pdf"):
    try:
        result = pipeline.ingest_document(
            file_path=str(pdf_file),
            collection_name="batch_collection",
            metadata={"filename": pdf_file.name}
        )
        results.append(result)
        print(f"✓ Ingested {pdf_file.name}")
    except Exception as e:
        print(f"✗ Failed {pdf_file.name}: {e}")

print(f"Total successful: {len(results)}")
```

## Troubleshooting

### MCP Server Not Connecting

**Issue**: Claude Desktop cannot connect to MCP server

**Solution**:
1. Check Claude Desktop config path (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`)
2. Ensure Python path and project path are absolute
3. Verify environment variables are set in config:
   ```json
   {
     "mcpServers": {
       "rag-knowledge-hub": {
         "command": "/usr/bin/python3",
         "args": ["/absolute/path/to/project/src/mcp/server.py"],
         "env": {
           "PYTHONPATH": "/absolute/path/to/project",
           "OPENAI_API_KEY": "sk-..."
         }
       }
     }
   }
   ```
4. Restart Claude Desktop completely

### Poor Retrieval Results

**Issue**: Query returns irrelevant documents

**Solutions**:
1. **Check chunking strategy**: Smaller chunks for precise retrieval, larger for more context
   ```python
   config.ingestion.chunk_size = 300  # Reduce for precision
   config.ingestion.chunk_overlap = 50
   ```

2. **Enable reranking**: Use cross-encoder or LLM reranker
   ```python
   config.reranker.enabled = True
   config.reranker.provider = "cohere"
   config.reranker.top_k = 5
   ```

3. **Adjust hybrid search weights**:
   ```python
   from src.retrieval.hybrid_search import HybridSearchRetriever
   
   retriever = HybridSearchRetriever(config)
   retriever.dense_weight = 0.7  # Semantic search
   retriever.sparse_weight = 0.3  # BM25 exact match
   ```

4. **Use evaluation to iterate**:
   ```python
   # Create golden test set
   evaluator = RAGEvaluator(config)
   results = evaluator.evaluate(test_cases, collection_name="my_collection")
   # Adjust parameters based on metrics
   ```

### Vector Store Connection Issues

**Issue**: Cannot connect to Qdrant/Chroma

**Solution**:
1. For Qdrant Cloud:
   ```bash
   # .env
   QDRANT_URL=https://your-cluster.qdrant.io
   QDRANT_API_KEY=your_api_key
   ```

2. For local Qdrant:
   ```bash
   # Start Qdrant with Docker
   docker run -p 6333:6333 qdrant/qdrant
   
   # In config
   QDRANT_URL=http://localhost:6333
   ```

3. For Chroma (local):
   ```python
   # config.py
   class VectorStoreConfig:
       type: str = "chroma"
       persist_directory: str = "./chroma_db"
   ```

### Out of Memory During Ingestion

**Issue**: Large PDFs cause OOM errors

**Solutions**:
1. Process in batches:
   ```python
   # Increase chunk size, reduce batch size
   config.ingestion.chunk_size = 800
   config.ingestion.batch_size = 10  # Embed 10 chunks at a time
   ```

2. Use streaming for large documents:
   ```python
   pipeline = IngestionPipeline(config)
   pipeline.process_streaming(
       file_path="large_document.pdf",
       collection_name="large_docs"
   )
   ```

### API Rate Limits

**Issue**: Hitting provider rate limits

**Solutions**:
1. Implement retry with exponential backoff:
   ```python
   config.llm.max_retries = 5
   config.llm.retry_delay = 2.0  # seconds
   ```

2. Use batch embedding APIs:
   ```python
   # OpenAI allows batching up to 2048 texts
   config.embedding.batch_size = 100
   ```

3. Switch to providers with higher limits (e.g., Cohere for embeddings)

## Advanced Usage

### Custom RAG Pipeline

Build a custom RAG pipeline with specific components:

```python
from src.core.config import get_config
from src.retrieval.hybrid_search import HybridSearchRetriever
from src.generation.generator import Generator
from src.evaluation.evaluator import RAGEvaluator

config = get_config()

# Custom retriever configuration
retriever = HybridSearchRetriever(config)
retriever.dense_weight = 0.6
retriever.sparse_weight = 0.4

# Custom generator
generator = Generator(config)
generator.system_prompt = "You are a helpful technical assistant..."

# Run custom RAG
def custom_rag_query(query: str, collection: str):
    # Retrieve
    contexts = retriever.retrieve(query, collection, top_k=5)
    
    # Generate
    response = generator.generate(
        query=query,
        contexts=[c.text for c in contexts],
        metadata=[c.metadata for c in contexts]
    )
    
    # Evaluate (optional)
    evaluator = RAGEvaluator(config)
    metrics = evaluator.evaluate_single(
        query=query,
        response=response,
        contexts=[c.text for c in contexts]
    )
    
    return {
        "response": response,
        "contexts": contexts,
        "metrics": metrics
    }

result = custom_rag_query("What are the system requirements?", "docs")
print(result["response"])
```

### Integrating with Your Own Application

Use the RAG system as a library:

```python
from src.rag_system import RAGSystem
from src.core.config import get_config

# Initialize
config = get_config()
rag = RAGSystem(config)

# In your FastAPI/Flask app
@app.post("/ask")
async def ask_question(query: str, collection: str = "default"):
    result = rag.query(
        query=query,
        collection_name=collection,
        top_k=5
    )
    return {
        "answer": result["response"],
        "sources": result["contexts"],
        "confidence": result["metrics"]["answer_relevancy"]
    }
```

## Branch Strategy

- **`main`**: Clean, production-ready code (1 commit with latest complete code)
- **`dev`**: Full commit history showing development progression
- **`clean-start`**: Skeleton with Skills and DEV_SPEC, zero progress (for learning from scratch)

Choose branch based on your needs:
- Quick deployment → `main`
- Understanding the build process → `dev`
- Learning by building yourself → `clean-start`

## Additional Resources

- **DEV_SPEC.md**: Complete architecture design and task breakdown
- **Resume Writer Skill**: Generate customized resume descriptions for this project
- **QA Tester Skill**: Automated testing across unit/integration/E2E layers
- **Package Skill**: Clean and package project for distribution

Use these skills in VS Code by typing the skill name in Copilot/Claude chat.
