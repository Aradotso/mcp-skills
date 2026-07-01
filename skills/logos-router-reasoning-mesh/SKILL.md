---
name: logos-router-reasoning-mesh
description: Deploy and configure Logos Router, a distributed semantic reasoning gateway with zero-drift consensus for AI agents
triggers:
  - set up logos router for distributed reasoning
  - configure reasoning mesh with consensus protocol
  - implement strict write discipline for AI inference
  - create adaptive reasoning workflow with logos
  - deploy multilingual semantic routing gateway
  - set up zero-drift local reasoning protocol
  - configure logos router nodes and mesh topology
  - implement distributed AI reasoning with consensus
---

# Logos Router Reasoning Mesh

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Logos Router is a distributed semantic reasoning gateway that fragments AI reasoning across local nodes with zero-drift consensus guarantees. Unlike monolithic inference pipelines, it distributes subproblems across a mesh of reasoning units that coordinate through a strict write protocol, ensuring every reasoning step is verified before commitment.

## Installation

### Prerequisites

- Python 3.11 or later
- At least one reasoning-capable inference engine (VLLM, Ollama, llama.cpp)
- Network access between mesh nodes (localhost sufficient for single-machine)

### Basic Setup

```bash
# Clone the repository
git clone https://github.com/rak7777/mythic-mcp-proxy.git
cd mythic-mcp-proxy

# Install dependencies
pip install -r requirements.txt

# Verify installation
python -m logos --version
```

### With VLLM Backend

```bash
# Install VLLM
pip install vllm

# Set up environment
export LOGOS_ENGINE=vllm
export LOGOS_MODEL_PATH=/path/to/model
```

### With Ollama Backend

```bash
# Install Ollama separately, then:
export LOGOS_ENGINE=ollama
export OLLAMA_HOST=localhost:11434
```

## Core Configuration

### Basic Mesh Configuration

Create a `mesh.yaml` configuration file:

```yaml
mesh:
  name: local-reasoning-lattice
  consensus_threshold: 0.95
  max_concurrent_queries: 10
  
  nodes:
    - id: primary-node
      type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      memory_limit_gb: 12
      
    - id: secondary-node
      type: local
      engine: ollama
      model: llama3.1:70b
      max_thinking_depth: 5
      memory_limit_gb: 8
  
  languages:
    - en
    - es
    - fr
    - zh
    - ar
    
  write_protocol:
    strict_discipline: true
    min_peer_verification: 2
    escalation_threshold: 0.85
    
  caching:
    semantic_cache_enabled: true
    max_cache_size_gb: 4
    ttl_hours: 24
```

### Environment Variables

```bash
# Required
export LOGOS_CONFIG_PATH=./mesh.yaml
export LOGOS_STORAGE_PATH=./reasoning_store

# Optional
export LOGOS_LOG_LEVEL=INFO
export LOGOS_CONSENSUS_TIMEOUT=30
export LOGOS_MAX_REASONING_DEPTH=7
```

## Basic Usage

### Single Query Reasoning

```python
from logos import Router

# Initialize router with config
router = Router(config="mesh.yaml")

# Simple query
response = router.query(
    prompt="Explain the halting problem in computer science",
    depth="adaptive",
    language="en"
)

print(response.text)
print(f"Reasoning depth: {response.depth}")
print(f"Nodes involved: {response.nodes_involved}")
print(f"Consensus score: {response.consensus_score}")
```

### With Explicit Depth Control

```python
# Force shallow reasoning for simple queries
quick_response = router.query(
    prompt="What is 2+2?",
    depth=1,
    language="en"
)

# Force deep reasoning for complex problems
deep_response = router.query(
    prompt="Analyze the computational complexity of this sorting algorithm",
    depth=7,
    language="en",
    require_consensus=True
)
```

### Multilingual Reasoning

```python
# Query in one language, respond in another
response = router.query(
    prompt="Explica la paradoja del mentiroso",
    input_language="es",
    output_language="en",
    depth="adaptive"
)

# Automatic language detection
auto_response = router.query(
    prompt="递归和自我指涉的关系是什么？",
    depth="adaptive"
)
```

## Advanced Workflows

### Multi-Step Reasoning Workflow

```python
from logos import Router, Workflow

router = Router(config="mesh.yaml")

# Create workflow
workflow = router.create_workflow(
    name="research_analysis",
    context_files=["paper.pdf", "notes.txt"]
)

# Define steps
workflow.add_step(
    name="extract_claims",
    operation="extract",
    target="main_thesis"
)

workflow.add_step(
    name="verify_claims",
    operation="verify",
    consensus=True,
    min_confidence=0.95
)

workflow.add_step(
    name="generate_summary",
    operation="synthesize",
    style="academic",
    max_length=500
)

# Execute with full audit trail
result = workflow.execute()

print(result.summary)
print(f"Total reasoning steps: {result.total_steps}")
print(f"Nodes used: {result.nodes_used}")

# Access audit trail
for step in result.audit_trail:
    print(f"{step.timestamp}: {step.operation} (confidence: {step.confidence})")
```

### Consensus-Based Reasoning

```python
# Require multiple nodes to agree before accepting output
consensus_response = router.query_with_consensus(
    prompt="Is this code vulnerable to SQL injection?",
    code="""
    def get_user(username):
        query = f"SELECT * FROM users WHERE name = '{username}'"
        return db.execute(query)
    """,
    min_nodes=3,
    threshold=0.95,
    depth="adaptive"
)

if consensus_response.consensus_reached:
    print(f"Consensus: {consensus_response.text}")
    print(f"Agreement: {consensus_response.agreement_score:.2%}")
else:
    print(f"No consensus. Conflicting views:")
    for view in consensus_response.conflicting_views:
        print(f"  - Node {view.node_id}: {view.reasoning}")
```

### Streaming Reasoning Steps

```python
# Stream reasoning process in real-time
for step in router.query_stream(
    prompt="Design a distributed cache with eventual consistency",
    depth=7,
    language="en"
):
    print(f"[{step.node_id}] Depth {step.depth}: {step.reasoning_fragment}")
    
    if step.verification_required:
        print(f"  → Awaiting consensus from {step.pending_peers}")
    
    if step.consensus_reached:
        print(f"  ✓ Consensus: {step.consensus_score:.2%}")
```

## Configuration Patterns

### High-Fidelity Research Mode

```yaml
mesh:
  name: research-mode
  consensus_threshold: 0.98
  
  nodes:
    - type: local
      engine: vllm
      model: opus-4.8-research
      max_thinking_depth: 10
      memory_limit_gb: 24
  
  write_protocol:
    strict_discipline: true
    min_peer_verification: 3
    escalation_threshold: 0.90
    require_causal_chain: true
    
  verification:
    fact_checking: true
    citation_required: true
    logic_validation: true
```

### Fast Response Mode

```yaml
mesh:
  name: fast-mode
  consensus_threshold: 0.85
  
  nodes:
    - type: local
      engine: ollama
      model: llama3.1:8b
      max_thinking_depth: 3
      memory_limit_gb: 4
  
  write_protocol:
    strict_discipline: false
    min_peer_verification: 1
    
  caching:
    semantic_cache_enabled: true
    aggressive_caching: true
```

### Multi-Node Production Setup

```yaml
mesh:
  name: production-mesh
  consensus_threshold: 0.95
  
  nodes:
    - id: node-gpu-1
      type: remote
      endpoint: http://inference-1.local:8080
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      
    - id: node-gpu-2
      type: remote
      endpoint: http://inference-2.local:8080
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      
    - id: node-cpu-fallback
      type: local
      engine: llama.cpp
      model: llama3.1:8b-q4
      max_thinking_depth: 3
      
  load_balancing:
    strategy: semantic_similarity
    prefer_local: true
    
  resilience:
    auto_failover: true
    health_check_interval: 30
    retry_failed_nodes: true
```

## CLI Commands

### Starting the Router

```bash
# Start with default config
logos start

# Start with custom config
logos start --config custom-mesh.yaml

# Start with specific nodes only
logos start --nodes node-1,node-2

# Start in debug mode
logos start --debug --log-level DEBUG
```

### Query from CLI

```bash
# Simple query
logos query "Explain quantum entanglement"

# With options
logos query "Analyze this code" \
  --depth 7 \
  --language en \
  --consensus \
  --output result.json

# Stream reasoning steps
logos query "Design a microservice architecture" \
  --stream \
  --show-reasoning
```

### Mesh Management

```bash
# Check mesh status
logos status

# Add node dynamically
logos node add --id new-node --endpoint http://node-3:8080

# Remove node
logos node remove --id node-2

# View audit trail
logos audit --query-id abc123

# Clear semantic cache
logos cache clear

# Export reasoning graph
logos export --query-id abc123 --format graphviz
```

## API Integration

### REST API

```python
import requests

# Query endpoint
response = requests.post(
    "http://localhost:8000/v1/query",
    json={
        "prompt": "Explain the CAP theorem",
        "depth": "adaptive",
        "language": "en",
        "consensus": True
    },
    headers={"Authorization": f"Bearer {os.getenv('LOGOS_API_KEY')}"}
)

result = response.json()
print(result["text"])
print(f"Reasoning path: {result['reasoning_path']}")
```

### WebSocket Streaming

```python
import asyncio
import websockets
import json

async def stream_reasoning():
    uri = f"ws://localhost:8000/v1/stream?token={os.getenv('LOGOS_API_KEY')}"
    
    async with websockets.connect(uri) as websocket:
        await websocket.send(json.dumps({
            "prompt": "Design a distributed consensus algorithm",
            "depth": 7,
            "language": "en"
        }))
        
        async for message in websocket:
            step = json.loads(message)
            print(f"Step {step['depth']}: {step['fragment']}")
            
            if step.get("consensus_reached"):
                print(f"Consensus: {step['consensus_score']}")

asyncio.run(stream_reasoning())
```

## Reasoning Profiles

### Custom Reasoning Profile

Create `profiles/custom-profile.json`:

```json
{
  "name": "technical-analysis",
  "description": "Deep technical analysis with code focus",
  "parameters": {
    "max_depth": 8,
    "verification_strategy": "peer-review",
    "thinking_style": "analytical",
    "output_format": "structured"
  },
  "stages": [
    {
      "name": "comprehension",
      "operations": ["parse", "identify_components"],
      "min_confidence": 0.9
    },
    {
      "name": "analysis",
      "operations": ["evaluate_logic", "check_edge_cases"],
      "min_confidence": 0.95
    },
    {
      "name": "synthesis",
      "operations": ["generate_recommendations", "prioritize"],
      "min_confidence": 0.92
    }
  ],
  "validation_rules": [
    "code_syntax_valid",
    "logic_sound",
    "best_practices_followed"
  ]
}
```

Use the profile:

```python
router = Router(config="mesh.yaml")
response = router.query(
    prompt="Review this authentication implementation",
    profile="technical-analysis",
    context={"code": auth_code}
)
```

## Monitoring and Debugging

### Enable Detailed Logging

```python
import logging
from logos import Router

# Configure logging
logging.basicConfig(level=logging.DEBUG)
logos_logger = logging.getLogger("logos")
logos_logger.setLevel(logging.DEBUG)

router = Router(config="mesh.yaml", debug=True)

# Query with full trace
response = router.query(
    prompt="Explain Byzantine fault tolerance",
    depth="adaptive",
    trace=True
)

# Access reasoning trace
for trace_entry in response.trace:
    print(f"{trace_entry.timestamp}: {trace_entry.node_id}")
    print(f"  Operation: {trace_entry.operation}")
    print(f"  Input: {trace_entry.input}")
    print(f"  Output: {trace_entry.output}")
    print(f"  Consensus: {trace_entry.consensus_score}")
```

### Health Checks

```python
# Check mesh health
health = router.health_check()

print(f"Mesh status: {health.status}")
print(f"Active nodes: {health.active_nodes}/{health.total_nodes}")
print(f"Avg response time: {health.avg_response_time}ms")
print(f"Cache hit rate: {health.cache_hit_rate:.1%}")

for node in health.node_status:
    print(f"{node.id}: {node.status} (load: {node.load:.1%})")
```

## Troubleshooting

### Consensus Failures

```python
# Handle consensus failures gracefully
try:
    response = router.query(
        prompt="Controversial query",
        depth="adaptive",
        consensus=True,
        timeout=60
    )
except ConsensusTimeoutError as e:
    print(f"Consensus not reached in {e.timeout}s")
    print(f"Partial results from {len(e.partial_results)} nodes")
    
    # Use partial results or retry with lower threshold
    response = router.query(
        prompt="Controversial query",
        depth="adaptive",
        consensus=True,
        consensus_threshold=0.85
    )
```

### Node Failures

```bash
# Check failing nodes
logos node diagnose --id node-1

# View node logs
logos logs --node node-1 --tail 100

# Restart specific node
logos node restart --id node-1

# Force node removal if unresponsive
logos node remove --id node-1 --force
```

### Memory Issues

```yaml
# Reduce memory usage in config
mesh:
  nodes:
    - type: local
      max_thinking_depth: 4  # Reduce from 7
      memory_limit_gb: 6     # Lower limit
      
  caching:
    max_cache_size_gb: 2     # Reduce cache
    aggressive_eviction: true
```

### Performance Tuning

```python
# Profile query performance
response = router.query(
    prompt="Complex reasoning task",
    depth="adaptive",
    profile_performance=True
)

print(f"Total time: {response.performance.total_time}s")
print(f"Time by stage:")
print(f"  Routing: {response.performance.routing_time}s")
print(f"  Reasoning: {response.performance.reasoning_time}s")
print(f"  Consensus: {response.performance.consensus_time}s")
print(f"  Synthesis: {response.performance.synthesis_time}s")
```

## Common Patterns

### Code Review Workflow

```python
def review_code_with_consensus(code: str, language: str = "python"):
    router = Router(config="mesh.yaml")
    
    response = router.query_with_consensus(
        prompt=f"Review this {language} code for security, performance, and best practices",
        context={"code": code, "language": language},
        min_nodes=3,
        threshold=0.95,
        depth=6
    )
    
    return {
        "issues": response.get_structured_field("issues"),
        "recommendations": response.get_structured_field("recommendations"),
        "severity": response.get_structured_field("severity"),
        "consensus": response.consensus_score
    }
```

### Document Analysis Pipeline

```python
async def analyze_document(file_path: str):
    router = Router(config="mesh.yaml")
    workflow = router.create_workflow("doc_analysis")
    
    # Extract content
    workflow.add_step("extract", operation="parse", target=file_path)
    
    # Identify key claims
    workflow.add_step("identify_claims", operation="extract_structured")
    
    # Verify each claim
    workflow.add_step("verify", operation="verify", consensus=True)
    
    # Generate summary
    workflow.add_step("summarize", operation="synthesize", max_length=300)
    
    result = await workflow.execute_async()
    return result
```

### Multi-Language Support

```python
def query_in_user_language(prompt: str, user_lang: str = None):
    router = Router(config="mesh.yaml")
    
    # Auto-detect or use specified language
    response = router.query(
        prompt=prompt,
        input_language=user_lang or "auto",
        output_language=user_lang or "auto",
        depth="adaptive"
    )
    
    return {
        "answer": response.text,
        "detected_language": response.detected_language,
        "reasoning_depth": response.depth
    }
```

This skill enables AI coding agents to effectively deploy, configure, and utilize Logos Router for distributed reasoning tasks with zero-drift consensus guarantees.
