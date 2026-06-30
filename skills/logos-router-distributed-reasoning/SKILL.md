---
name: logos-router-distributed-reasoning
description: Deploy and configure Logos Router for distributed semantic reasoning with zero-drift consensus and adaptive thinking depth
triggers:
  - set up logos router for distributed AI reasoning
  - configure a reasoning mesh with multiple nodes
  - implement strict write discipline protocol
  - create adaptive depth reasoning workflows
  - configure multilingual semantic routing
  - debug consensus threshold issues in logos router
  - optimize reasoning node topology
  - integrate logos router with local inference engines
---

# Logos Router — Distributed Reasoning Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Logos Router is a distributed semantic reasoning gateway that fragments AI reasoning across local nodes with zero-drift consensus guarantees. Unlike traditional single-model inference pipelines, it distributes subproblems across a mesh of reasoning units that coordinate through a disciplined write protocol. The system maintains reasoning fidelity through cross-validation, preventing the drift common in long-context AI interactions.

**Key differentiators:**
- **Zero-drift consensus**: Strict Write Discipline (SWD) protocol validates every reasoning step
- **Adaptive depth**: Dynamically adjusts cognitive complexity based on problem difficulty
- **Multilingual support**: 16 languages with unified reasoning representation
- **Causal audit trails**: Every decision is traceable and verifiable
- **Local-first**: All reasoning runs on your infrastructure

## Installation

### Prerequisites

```bash
# Requires Python 3.11+
python --version

# Requires a local inference engine (VLLM, Ollama, llama.cpp, etc.)
# Example with Ollama:
curl -fsSL https://ollama.com/install.sh | sh
ollama pull opus-mini-4.8
```

### Install Logos Router

```bash
# Clone the repository
git clone https://github.com/rak7777/mythic-mcp-proxy.git
cd mythic-mcp-proxy

# Install dependencies
pip install -r requirements.txt

# Verify installation
python -m logos.router --version
```

## Configuration

### Basic Mesh Configuration

Create a `mesh_config.yaml` file to define your reasoning topology:

```yaml
mesh:
  name: local-reasoning-mesh
  consensus_threshold: 0.95  # Confidence required before write (0.0-1.0)
  
  nodes:
    - name: primary-node
      type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      memory_limit: 8GB
      
    - name: verification-node
      type: local
      engine: ollama
      model: llama-3.3-70b
      max_thinking_depth: 5
      memory_limit: 4GB
  
  languages:
    - en
    - zh
    - es
    - ar
    - hi
    - fr
  
  grail:
    storage_path: ./reasoning_store
    retention_days: 30
    
  chiron:
    routing_strategy: semantic_similarity
    load_balance: true
    
  orpheus:
    synthesis_mode: harmonized
    detect_contradictions: true
```

### Environment Configuration

```bash
# Create .env file for sensitive configuration
cat > .env << EOF
LOGOS_MESH_CONFIG=./mesh_config.yaml
LOGOS_LOG_LEVEL=INFO
LOGOS_API_PORT=8080
LOGOS_WEBSOCKET_PORT=8081

# Inference engine endpoints
VLLM_ENDPOINT=http://localhost:8000
OLLAMA_ENDPOINT=http://localhost:11434

# Optional: Cross-mesh bridging
LOGOS_ENABLE_FEDERATION=false
LOGOS_FEDERATION_KEY=${FEDERATION_KEY}
EOF
```

## Core Usage Patterns

### Single-Query Reasoning

```python
from logos import Router

# Initialize router with configuration
router = Router(config="mesh_config.yaml")

# Simple query with adaptive depth
response = router.query(
    prompt="Explain the relationship between recursion and self-reference in formal systems.",
    depth="adaptive",  # or specify 1-7
    language="en"
)

print(response.text)
print(f"Reasoning depth used: {response.depth}")
print(f"Nodes consulted: {response.nodes_involved}")
print(f"Consensus score: {response.consensus_score}")

# Access audit trail
for step in response.reasoning_trail:
    print(f"Step {step.index}: {step.action}")
    print(f"  Node: {step.node_id}")
    print(f"  Confidence: {step.confidence}")
    print(f"  Verified by: {step.verifiers}")
```

### Multi-Step Workflow

```python
from logos import Router, Workflow

router = Router(config="mesh_config.yaml")

# Create a multi-step reasoning workflow
workflow = router.create_workflow(
    name="code_analysis",
    description="Analyze code for security and performance issues"
)

# Define workflow steps
workflow.add_step(
    name="extract_structure",
    prompt="Analyze the code structure and identify key components",
    depth=3
)

workflow.add_step(
    name="security_review",
    prompt="Identify potential security vulnerabilities",
    depth=5,
    requires_consensus=True
)

workflow.add_step(
    name="performance_analysis",
    prompt="Analyze performance characteristics and bottlenecks",
    depth=4
)

workflow.add_step(
    name="generate_report",
    prompt="Synthesize findings into a comprehensive report",
    depth=6,
    synthesis_mode="harmonized"
)

# Execute workflow with input
with open("target_code.py", "r") as f:
    code_content = f.read()

result = workflow.execute(input_data={"code": code_content})

# Access workflow results
for step_name, step_result in result.steps.items():
    print(f"\n{step_name}:")
    print(step_result.text)
    print(f"Consensus: {step_result.consensus_score}")
```

### Document Analysis Workflow

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Create document analysis task
task = router.create_workflow("analyze_research_paper.pdf")

# Chain analysis steps
task.extract_claims()  # Depth 4
task.verify_claims(consensus=True)  # Depth 6, requires 0.95 consensus
task.cross_reference(sources=["./references/"])  # Depth 5
task.generate_summary(style="academic", length=500)  # Depth 4

# Execute with progress tracking
result = task.execute(progress_callback=lambda step, progress: 
    print(f"{step}: {progress * 100:.1f}%")
)

print(result.summary)
print(f"Claims verified: {len(result.verified_claims)}")
print(f"Total reasoning time: {result.execution_time}s")
```

### Multilingual Reasoning

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Query in Chinese, get response in English
response = router.query(
    prompt="解释量子纠缠的基本原理",  # Chinese input
    depth="adaptive",
    source_language="zh",
    target_language="en"  # Output in English
)

print(response.text)  # English explanation

# Or keep same language
response_zh = router.query(
    prompt="解释量子纠缠的基本原理",
    depth=5,
    language="zh"  # Input and output in Chinese
)
```

### Streaming Reasoning Steps

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Stream reasoning steps in real-time
for step in router.query_stream(
    prompt="Design a distributed consensus algorithm",
    depth=7
):
    print(f"Node {step.node_id}: {step.text}")
    print(f"  Confidence: {step.confidence:.3f}")
    print(f"  Verification: {step.verification_status}")
    
    # Access intermediate reasoning
    if step.type == "deliberation":
        print(f"  Considering: {step.alternatives}")
```

## Advanced Configuration

### Custom Reasoning Profile

Create a custom reasoning profile for specialized domains:

```yaml
# profiles/scientific_reasoning.yaml
profile:
  name: scientific-reasoning
  max_depth: 7
  
  heuristics:
    - name: hypothesis_generation
      trigger: "problem requires speculation"
      depth_escalation: +2
      
    - name: empirical_verification
      trigger: "factual claim detected"
      requires_consensus: true
      min_verifiers: 3
      
    - name: mathematical_rigor
      trigger: "quantitative reasoning"
      precision_threshold: 0.99
  
  synthesis:
    style: academic
    citation_format: apa
    contradiction_policy: flag_and_escalate
```

Load custom profile:

```python
from logos import Router

router = Router(
    config="mesh_config.yaml",
    profile="profiles/scientific_reasoning.yaml"
)

response = router.query(
    prompt="Design an experiment to test quantum entanglement",
    use_profile=True
)
```

### Node Capability Manifests

Define specialized node capabilities:

```yaml
# nodes/math_specialist.yaml
node:
  name: math-reasoning-node
  type: local
  engine: vllm
  model: minerva-540b
  
  capabilities:
    - mathematical_reasoning
    - symbolic_manipulation
    - proof_verification
    
  specializations:
    - domain: calculus
      confidence_boost: 0.15
    - domain: linear_algebra
      confidence_boost: 0.12
      
  resource_limits:
    max_concurrent_queries: 4
    memory_limit: 16GB
    gpu_memory: 24GB
```

Register specialized node:

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Register specialized node
router.register_node("nodes/math_specialist.yaml")

# Query will route to specialized node for math problems
response = router.query(
    prompt="Prove that the square root of 2 is irrational",
    prefer_capabilities=["mathematical_reasoning", "proof_verification"]
)
```

### Graceful Degradation Configuration

```yaml
mesh:
  name: production-mesh
  
  degradation:
    enabled: true
    
    triggers:
      - metric: memory_usage
        threshold: 90%
        action: reduce_depth
        
      - metric: latency
        threshold: 15s
        action: reduce_concurrency
        
      - metric: consensus_failures
        threshold: 3
        action: escalate_and_notify
    
    depth_reduction:
      strategy: proportional  # or "aggressive", "conservative"
      min_depth: 3
      notify_user: true
```

## CLI Usage

### Start Router Server

```bash
# Start with default config
python -m logos.router serve

# Start with custom config
python -m logos.router serve --config mesh_config.yaml --port 8080

# Start with verbose logging
python -m logos.router serve --log-level DEBUG

# Start with specific node topology
python -m logos.router serve --nodes 4 --engines vllm,ollama
```

### Query from CLI

```bash
# Single query
logos-query "Explain quantum entanglement" --depth adaptive

# With language specification
logos-query "¿Qué es la mecánica cuántica?" --language es

# Stream reasoning steps
logos-query "Design a consensus algorithm" --stream --depth 7

# Save reasoning trail
logos-query "Analyze this code" --input code.py --output analysis.json --trail
```

### Mesh Management

```bash
# Check mesh status
logos-mesh status

# Add node to running mesh
logos-mesh add-node --config node_config.yaml

# Remove node
logos-mesh remove-node --id node-123

# View reasoning statistics
logos-mesh stats --timeframe 24h

# Export reasoning trail
logos-mesh export-trail --query-id abc123 --format json
```

## REST API Integration

### Start API Server

```python
from logos import Router
from logos.api import create_app

router = Router(config="mesh_config.yaml")
app = create_app(router)

# Run with uvicorn
import uvicorn
uvicorn.run(app, host="0.0.0.0", port=8080)
```

### API Endpoints

```python
import requests
import json

# Query endpoint
response = requests.post(
    "http://localhost:8080/v1/query",
    json={
        "prompt": "Explain distributed consensus algorithms",
        "depth": "adaptive",
        "language": "en",
        "stream": False
    }
)

result = response.json()
print(result["text"])
print(result["consensus_score"])

# Workflow endpoint
workflow_response = requests.post(
    "http://localhost:8080/v1/workflow",
    json={
        "name": "code_review",
        "steps": [
            {
                "name": "analyze_structure",
                "prompt": "Analyze code structure",
                "depth": 3
            },
            {
                "name": "security_check",
                "prompt": "Check for security issues",
                "depth": 5,
                "requires_consensus": True
            }
        ],
        "input_data": {
            "code": "def factorial(n): return 1 if n == 0 else n * factorial(n-1)"
        }
    }
)

# Stream endpoint (SSE)
import sseclient

response = requests.post(
    "http://localhost:8080/v1/query/stream",
    json={"prompt": "Design a distributed system", "depth": 7},
    stream=True
)

client = sseclient.SSEClient(response)
for event in client.events():
    step_data = json.loads(event.data)
    print(f"Step: {step_data['text']}")
```

## WebSocket Real-Time Streaming

```python
import asyncio
import websockets
import json

async def stream_reasoning():
    uri = "ws://localhost:8081/v1/stream"
    
    async with websockets.connect(uri) as websocket:
        # Send query
        await websocket.send(json.dumps({
            "prompt": "Explain category theory",
            "depth": 6,
            "language": "en"
        }))
        
        # Receive reasoning steps
        async for message in websocket:
            step = json.loads(message)
            
            if step["type"] == "reasoning_step":
                print(f"Node {step['node_id']}: {step['text']}")
                print(f"Confidence: {step['confidence']:.3f}")
                
            elif step["type"] == "consensus_check":
                print(f"Consensus: {step['score']:.3f}")
                
            elif step["type"] == "final_response":
                print(f"\nFinal: {step['text']}")
                break

asyncio.run(stream_reasoning())
```

## Troubleshooting

### Low Consensus Scores

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Enable diagnostic mode
router.enable_diagnostics()

response = router.query(
    prompt="Your query here",
    depth=5
)

# Check consensus details
diagnostics = router.get_diagnostics()
for step in diagnostics.consensus_failures:
    print(f"Step {step.index} failed consensus:")
    print(f"  Primary node score: {step.primary_score}")
    print(f"  Verifier scores: {step.verifier_scores}")
    print(f"  Reason: {step.failure_reason}")
    
# Adjust consensus threshold if needed
router.set_consensus_threshold(0.90)  # Lower from 0.95
```

### High Latency Issues

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Profile query performance
response = router.query(
    prompt="Your query",
    depth=5,
    profile=True
)

# Analyze bottlenecks
print(f"Total time: {response.metrics.total_time}s")
print(f"Node selection: {response.metrics.routing_time}s")
print(f"Inference: {response.metrics.inference_time}s")
print(f"Consensus: {response.metrics.consensus_time}s")
print(f"Synthesis: {response.metrics.synthesis_time}s")

# Optimize by reducing depth or adding nodes
if response.metrics.inference_time > 10.0:
    print("Consider adding more inference nodes")
```

### Memory Pressure

```yaml
# mesh_config.yaml - Enable aggressive memory management
mesh:
  nodes:
    - name: constrained-node
      memory_limit: 4GB
      swap_to_disk: true
      cache_strategy: aggressive_eviction
      
  grail:
    compression: true
    prune_old_trails: true
    max_trail_size: 100MB
```

### Node Failures

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Enable automatic failover
router.configure_resilience(
    auto_failover=True,
    health_check_interval=10,  # seconds
    max_retries=3
)

# Monitor node health
for node_id, health in router.get_node_health().items():
    if health.status != "healthy":
        print(f"Node {node_id}: {health.status}")
        print(f"  Last seen: {health.last_seen}")
        print(f"  Error: {health.error_message}")
```

### Debugging Reasoning Trails

```python
from logos import Router

router = Router(config="mesh_config.yaml")

response = router.query(
    prompt="Complex reasoning query",
    depth=7,
    save_trail=True
)

# Export full reasoning trail for inspection
trail = response.export_trail(format="json")

with open("reasoning_trail.json", "w") as f:
    json.dump(trail, f, indent=2)

# Visualize reasoning DAG
from logos.visualization import plot_reasoning_dag

plot_reasoning_dag(
    trail,
    output="reasoning_graph.svg",
    show_consensus=True,
    show_alternatives=True
)
```

## Best Practices

1. **Start with adaptive depth**: Let the router determine optimal reasoning complexity
2. **Use consensus for critical decisions**: Enable `requires_consensus=True` for high-stakes reasoning
3. **Monitor consensus scores**: Scores below 0.90 indicate potential reasoning issues
4. **Profile before scaling**: Use diagnostic mode to identify bottlenecks before adding nodes
5. **Cache semantic patterns**: Enable semantic caching for repeated query patterns
6. **Version reasoning profiles**: Track profile changes alongside code changes
7. **Audit critical trails**: Export and review reasoning trails for important decisions

## Performance Optimization

```python
from logos import Router

# Production-optimized configuration
router = Router(
    config="mesh_config.yaml",
    cache_enabled=True,
    cache_ttl=3600,  # 1 hour
    parallel_verification=True,
    max_concurrent_queries=10
)

# Batch queries for efficiency
queries = [
    "Query 1",
    "Query 2",
    "Query 3"
]

responses = router.batch_query(
    prompts=queries,
    depth=4,
    max_parallel=3
)

for i, response in enumerate(responses):
    print(f"Query {i+1}: {response.text}")
```
