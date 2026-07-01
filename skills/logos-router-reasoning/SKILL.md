---
name: logos-router-reasoning
description: Deploy and configure Logos Router for distributed semantic reasoning with zero-drift consensus across local AI nodes
triggers:
  - set up logos router for distributed reasoning
  - configure zero-drift consensus protocol
  - create a reasoning mesh with multiple nodes
  - implement strict write discipline for AI
  - route queries across semantic reasoning nodes
  - build adaptive reasoning workflows with logos
  - debug logos router consensus failures
  - configure multilingual semantic routing
---

# Logos Router Reasoning Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Logos Router is a distributed semantic reasoning gateway that fragments AI reasoning across local nodes with zero-drift consensus guarantees. It implements a Strict Write Discipline (SWD) protocol to eliminate hallucination chains and maintain reasoning fidelity across multi-step inference tasks. The router coordinates reasoning through three layers: Grail (persistence), Chiron (routing), and Orpheus (synthesis).

**Key features:**
- Zero-drift consensus protocol for reasoning fidelity
- Adaptive reasoning depth (1-7 steps) based on problem complexity
- 16-language semantic routing with universal intermediate representation
- Causal audit trails for every routing decision
- 24/7 reasoning continuity with automatic failover

## Installation

### Prerequisites

- Python 3.11 or later
- Local inference engine (VLLM, Ollama, llama.cpp, etc.)
- Network connectivity between mesh nodes (localhost OK for single-machine)

### Setup

```bash
# Clone the repository
git clone https://github.com/rak7777/mythic-mcp-proxy.git
cd mythic-mcp-proxy

# Install dependencies
pip install -r requirements.txt

# Initialize configuration
cp config.example.yaml config.yaml
```

## Configuration

### Basic Mesh Configuration

Create a `config.yaml` file defining your reasoning mesh:

```yaml
mesh:
  name: my-reasoning-lattice
  consensus_threshold: 0.95  # confidence required before write
  max_concurrent_queries: 10
  
  nodes:
    - id: node-primary
      type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      host: localhost
      port: 8001
      
    - id: node-secondary
      type: local
      engine: ollama
      model: llama3-70b
      max_thinking_depth: 5
      host: localhost
      port: 8002
  
  languages:
    - en
    - zh
    - es
    - ar
    - hi
    - fr

# Strict Write Discipline settings
swd:
  enabled: true
  peer_verification_count: 2
  escalation_depth_threshold: 5
  
# Semantic caching
cache:
  enabled: true
  similarity_threshold: 0.92
  max_size_gb: 8
```

### Multi-Node Configuration

For distributed deployment across multiple machines:

```yaml
mesh:
  name: distributed-lattice
  consensus_threshold: 0.95
  
  nodes:
    - id: gpu-node-1
      type: remote
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      host: 192.168.1.100
      port: 8001
      capabilities:
        - deep-reasoning
        - multilingual
        
    - id: gpu-node-2
      type: remote
      engine: vllm
      model: mistral-large-2
      max_thinking_depth: 6
      host: 192.168.1.101
      port: 8001
      capabilities:
        - fast-inference
        - code-generation
        
    - id: cpu-node-fallback
      type: remote
      engine: llama-cpp
      model: phi-3-mini
      max_thinking_depth: 3
      host: 192.168.1.102
      port: 8001
      capabilities:
        - fallback
        - low-resource
```

## Core Usage

### Single-Query Reasoning

```python
from logos import Router

# Initialize router with configuration
router = Router(config="config.yaml")

# Simple query with adaptive depth
response = router.query(
    prompt="Explain the halting problem and its implications for AI safety.",
    depth="adaptive",
    language="en"
)

print(response.text)
print(f"Reasoning depth: {response.depth}")
print(f"Nodes consulted: {response.nodes_involved}")
print(f"Consensus score: {response.consensus_score}")

# Access reasoning trail
for step in response.trail:
    print(f"Step {step.index}: {step.summary}")
    print(f"  Node: {step.node_id}")
    print(f"  Verification: {step.verification_score}")
```

### Explicit Reasoning Depth Control

```python
# Force shallow reasoning (fast)
quick_response = router.query(
    prompt="What is 2+2?",
    depth=1,
    language="en"
)

# Force deep reasoning (thorough)
deep_response = router.query(
    prompt="Analyze the P vs NP problem and recent approaches.",
    depth=7,
    language="en"
)

# Adaptive with constraints
constrained = router.query(
    prompt="Summarize this research paper.",
    depth="adaptive",
    max_depth=4,
    timeout=30  # seconds
)
```

### Multilingual Routing

```python
# Query in one language, respond in another
response = router.query(
    prompt="解释量子纠缠的基本原理",  # Chinese
    language="zh",
    output_language="en"
)

# Process multilingual context
response = router.query(
    prompt="Compare these documents",
    context=[
        {"text": "English document...", "language": "en"},
        {"text": "Document en español...", "language": "es"},
        {"text": "文档中文...", "language": "zh"}
    ],
    language="auto",
    output_language="en"
)
```

### Multi-Step Workflows

```python
# Create workflow for complex task
task = router.create_workflow("analyze_codebase")

# Define workflow steps
task.add_step(
    "extract_functions",
    prompt="List all functions and their purposes",
    depth=3
)

task.add_step(
    "identify_patterns",
    prompt="Identify architectural patterns",
    depth=5,
    depends_on=["extract_functions"]
)

task.add_step(
    "generate_report",
    prompt="Generate refactoring recommendations",
    depth=4,
    depends_on=["identify_patterns"]
)

# Execute with consensus verification
result = task.execute(consensus=True)

# Access intermediate results
for step_name, step_result in result.steps.items():
    print(f"{step_name}: {step_result.summary}")
    print(f"  Nodes: {step_result.nodes_involved}")
```

### Document Analysis Workflow

```python
# Analyze document with reasoning
task = router.create_workflow("paper_analysis")

# Load document
task.load_document("research_paper.pdf")

# Extract claims
claims = task.extract_claims(min_confidence=0.8)

# Verify each claim with consensus
verification = task.verify_claims(
    claims=claims,
    consensus=True,
    verification_sources=["local_knowledge", "reasoning"]
)

# Generate summary
summary = task.generate_summary(
    style="academic",
    length="medium",
    include_citations=True
)

# Execute workflow
result = task.execute()

print(result.summary.text)
print(f"Claims extracted: {len(result.claims)}")
print(f"Claims verified: {result.verification.verified_count}")
```

## Advanced Usage

### Consensus Verification

```python
# Enable strict consensus for critical queries
response = router.query(
    prompt="Review this security-critical code change",
    depth=6,
    consensus_mode="strict",  # requires higher threshold
    min_peer_verifications=3
)

# Check consensus details
if response.consensus_passed:
    print("Consensus achieved:")
    for verification in response.verifications:
        print(f"  Node {verification.node_id}: {verification.score}")
else:
    print("Consensus failed - escalating")
    print(f"Reason: {response.consensus_failure_reason}")
```

### Causal Audit Trails

```python
# Query with full audit trail
response = router.query(
    prompt="Should we refactor this module?",
    depth="adaptive",
    audit_trail=True
)

# Inspect reasoning chain
print("Reasoning Trail:")
for i, step in enumerate(response.trail):
    print(f"\nStep {i+1}:")
    print(f"  Node: {step.node_id}")
    print(f"  Thought: {step.thought}")
    print(f"  Confidence: {step.confidence}")
    print(f"  Verified by: {step.verified_by}")
    print(f"  Causal parent: {step.parent_hash}")
    print(f"  Hash: {step.hash}")
```

### Custom Reasoning Profiles

```python
# Load custom reasoning profile
router = Router(
    config="config.yaml",
    reasoning_profile="profiles/code_review.json"
)

# Create custom profile programmatically
custom_profile = {
    "name": "security-audit",
    "max_depth": 7,
    "verification_weight": 0.9,
    "heuristics": {
        "prioritize": ["security", "correctness", "edge_cases"],
        "verification_frequency": "every_step",
        "escalation_triggers": ["security_concern", "ambiguity"]
    }
}

router.load_profile(custom_profile)
```

### Sandboxed Reasoning Zones

```python
# Create isolated reasoning zone
zone = router.create_zone("sensitive_analysis")

# Execute queries in isolation
response1 = zone.query("Analyze proprietary algorithm A")
response2 = zone.query("Analyze proprietary algorithm B")

# Responses don't cross-contaminate
# Zone can be destroyed to clear memory
zone.destroy()
```

### Streaming Reasoning Steps

```python
# Stream reasoning steps as they occur
for step in router.query_stream(
    prompt="Explain quantum computing",
    depth="adaptive"
):
    print(f"Step {step.index}: {step.text}")
    print(f"  Node: {step.node_id}")
    print(f"  Confidence: {step.confidence}")
    
    if step.is_final:
        print("\nFinal response:")
        print(step.synthesized_text)
```

## REST API Usage

### Starting the API Server

```bash
# Start router API server
python -m logos.server --config config.yaml --port 8000

# With specific host binding
python -m logos.server --config config.yaml --host 0.0.0.0 --port 8000
```

### API Endpoints

```python
import requests

# Query endpoint
response = requests.post("http://localhost:8000/query", json={
    "prompt": "Explain recursion",
    "depth": "adaptive",
    "language": "en",
    "consensus": True
})

result = response.json()
print(result["text"])
print(f"Depth used: {result['depth']}")
print(f"Nodes: {result['nodes_involved']}")

# Workflow endpoint
workflow = requests.post("http://localhost:8000/workflow", json={
    "name": "code_analysis",
    "steps": [
        {
            "id": "extract",
            "prompt": "Extract all functions",
            "depth": 3
        },
        {
            "id": "analyze",
            "prompt": "Analyze architecture",
            "depth": 5,
            "depends_on": ["extract"]
        }
    ]
})

workflow_id = workflow.json()["workflow_id"]

# Execute workflow
execution = requests.post(
    f"http://localhost:8000/workflow/{workflow_id}/execute",
    json={"consensus": True}
)

# Poll for results
status = requests.get(f"http://localhost:8000/workflow/{workflow_id}/status")
```

## WebSocket Streaming

```python
import asyncio
import websockets
import json

async def stream_reasoning():
    uri = "ws://localhost:8000/stream"
    
    async with websockets.connect(uri) as websocket:
        # Send query
        await websocket.send(json.dumps({
            "prompt": "Explain neural networks",
            "depth": "adaptive",
            "language": "en"
        }))
        
        # Receive reasoning steps
        async for message in websocket:
            step = json.loads(message)
            
            if step["type"] == "reasoning_step":
                print(f"Step {step['index']}: {step['text']}")
                print(f"  Confidence: {step['confidence']}")
            
            elif step["type"] == "final":
                print(f"\nFinal: {step['text']}")
                break
            
            elif step["type"] == "error":
                print(f"Error: {step['message']}")
                break

asyncio.run(stream_reasoning())
```

## Monitoring and Debugging

### Mesh Status

```python
# Check mesh health
status = router.mesh_status()

print(f"Mesh: {status.name}")
print(f"Active nodes: {status.active_nodes}/{status.total_nodes}")

for node in status.nodes:
    print(f"\nNode {node.id}:")
    print(f"  Status: {node.status}")
    print(f"  Load: {node.current_load}/{node.max_load}")
    print(f"  Queries processed: {node.queries_processed}")
    print(f"  Average latency: {node.avg_latency_ms}ms")
```

### Reasoning Metrics

```python
# Get reasoning metrics
metrics = router.get_metrics()

print(f"Total queries: {metrics.total_queries}")
print(f"Average depth: {metrics.avg_depth}")
print(f"Consensus pass rate: {metrics.consensus_pass_rate}")
print(f"Drift rate: {metrics.drift_rate}")

# Per-node metrics
for node_id, node_metrics in metrics.nodes.items():
    print(f"\nNode {node_id}:")
    print(f"  Queries: {node_metrics.query_count}")
    print(f"  Success rate: {node_metrics.success_rate}")
    print(f"  Avg latency: {node_metrics.avg_latency}")
```

### Debug Mode

```python
# Enable debug logging
router = Router(config="config.yaml", debug=True)

# Query with verbose output
response = router.query(
    prompt="Test query",
    depth="adaptive",
    debug=True
)

# Access debug information
print("Debug info:")
print(f"  Routing decisions: {response.debug.routing_decisions}")
print(f"  Node selection: {response.debug.node_selection_reasoning}")
print(f"  Consensus process: {response.debug.consensus_log}")
```

## Common Patterns

### High-Throughput Batch Processing

```python
# Process multiple queries concurrently
queries = [
    "Summarize document A",
    "Summarize document B",
    "Summarize document C"
]

# Batch processing with load balancing
responses = router.batch_query(
    queries,
    depth=3,
    max_concurrent=5,
    load_balance=True
)

for i, response in enumerate(responses):
    print(f"Query {i+1}: {response.text[:100]}...")
```

### Fallback and Graceful Degradation

```python
# Query with fallback options
try:
    response = router.query(
        prompt="Complex analysis",
        depth=7,
        timeout=60,
        fallback_depth=4  # reduce depth if timeout
    )
except router.ConsensusFailure as e:
    print(f"Consensus failed: {e}")
    # Fall back to single-node inference
    response = router.query(
        prompt="Complex analysis",
        depth=4,
        consensus=False,
        force_node="node-primary"
    )
```

### Reasoning with External Context

```python
# Provide external context for grounded reasoning
response = router.query(
    prompt="Analyze this API design",
    context={
        "code": open("api.py").read(),
        "docs": open("api_docs.md").read(),
        "requirements": ["security", "performance", "maintainability"]
    },
    depth="adaptive",
    consensus=True
)
```

## Troubleshooting

### Consensus Failures

```python
# Diagnose consensus issues
response = router.query(
    prompt="Test query",
    depth=5,
    consensus=True,
    debug=True
)

if not response.consensus_passed:
    print("Consensus failure diagnosis:")
    print(f"  Threshold: {response.consensus_threshold}")
    print(f"  Achieved: {response.consensus_score}")
    print(f"  Failing nodes: {response.failing_nodes}")
    
    # Check node disagreements
    for disagreement in response.disagreements:
        print(f"\n  Node {disagreement.node_id}:")
        print(f"    Score: {disagreement.score}")
        print(f"    Reason: {disagreement.reason}")
```

### Node Connectivity Issues

```bash
# Test node connectivity
python -m logos.test_mesh --config config.yaml

# Ping specific node
python -m logos.ping_node --node-id node-primary --config config.yaml
```

### Performance Optimization

```python
# Profile query performance
with router.profile() as profiler:
    response = router.query(
        prompt="Test query",
        depth="adaptive"
    )

print(profiler.report())
# Shows: routing time, node selection time, inference time, synthesis time

# Optimize cache settings
router.configure_cache(
    similarity_threshold=0.95,  # higher = fewer hits but more accurate
    max_size_gb=16
)
```

### Memory Management

```python
# Monitor memory usage
memory = router.get_memory_usage()

print(f"Grail layer: {memory.grail_gb}GB")
print(f"Active reasoning: {memory.active_reasoning_gb}GB")
print(f"Cache: {memory.cache_gb}GB")

# Clear cache if needed
router.clear_cache(keep_frequent=True)

# Garbage collect reasoning trails older than 24h
router.gc_trails(max_age_hours=24)
```

## Environment Variables

```bash
# Node configuration
export LOGOS_NODE_ID=node-primary
export LOGOS_ENGINE=vllm
export LOGOS_MODEL_PATH=/models/opus-mini-4.8
export LOGOS_MAX_DEPTH=7

# Mesh configuration
export LOGOS_MESH_NAME=production-lattice
export LOGOS_CONSENSUS_THRESHOLD=0.95

# API server
export LOGOS_API_HOST=0.0.0.0
export LOGOS_API_PORT=8000

# Performance tuning
export LOGOS_MAX_CONCURRENT_QUERIES=10
export LOGOS_CACHE_SIZE_GB=16
export LOGOS_GC_INTERVAL_HOURS=24
```

## Integration Examples

### CI/CD Code Review

```python
# Automated code review in CI pipeline
from logos import Router

router = Router(config="ci_config.yaml")

# Review pull request
diff = open("pr_diff.patch").read()

review = router.query(
    prompt=f"Review this code change:\n\n{diff}",
    depth=6,
    consensus=True,
    reasoning_profile="code_review"
)

# Fail build if critical issues found
if any(issue.severity == "critical" for issue in review.issues):
    exit(1)

# Post review comment
print(review.text)
```

### Document Search with Semantic Reasoning

```python
# Build semantic index
from logos import Router

router = Router(config="config.yaml")

documents = load_documents("./docs")

# Create reasoning-enhanced index
for doc in documents:
    router.index_document(
        doc.content,
        metadata=doc.metadata,
        reasoning_depth=3  # extract semantic features
    )

# Query with reasoning
results = router.semantic_search(
    query="How do we handle authentication?",
    top_k=5,
    reasoning_depth=4,  # reason about relevance
    synthesize=True  # synthesize answer from results
)

print(results.synthesized_answer)
```

This skill covers the essential operations for deploying and using Logos Router's distributed reasoning capabilities with zero-drift consensus guarantees.
