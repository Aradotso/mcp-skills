---
name: logos-router-reasoning-mesh
description: Deploy and operate Logos Router, a distributed semantic reasoning gateway with zero-drift consensus for AI agents
triggers:
  - set up logos router for distributed reasoning
  - configure a reasoning mesh with multiple nodes
  - implement strict write discipline protocol
  - route AI queries across reasoning nodes
  - create adaptive multilingual reasoning workflows
  - debug zero-drift consensus failures
  - deploy local reasoning infrastructure
  - build semantic reasoning mesh
---

# Logos Router: Distributed Reasoning Mesh

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Logos Router is a distributed semantic reasoning gateway that fragments AI reasoning across local nodes with zero-drift consensus guarantees. Unlike traditional single-model inference, it implements a Strict Write Discipline (SWD) protocol where every reasoning step is verified through consensus before committing. This eliminates hallucination chains and maintains architectural fidelity across complex multi-step reasoning tasks.

## Architecture Layers

- **Grail Layer**: Content-addressed immutable storage of reasoning states (DAG of cognition)
- **Chiron Layer**: Semantic routing that assigns subproblems to capable nodes based on manifests and load
- **Orpheus Layer**: Synthesis engine that weaves fragmented outputs into coherent responses

## Installation

### Prerequisites

```bash
# System requirements
python --version  # 3.11 or later required
# At least one inference engine: vllm, Ollama, llama.cpp, etc.
# Network access between mesh nodes (localhost sufficient for single-machine)
```

### Clone and Setup

```bash
git clone https://github.com/rak7777/mythic-mcp-proxy.git
cd mythic-mcp-proxy
pip install -r requirements.txt
```

### Basic Configuration

Create a mesh configuration file `my_mesh.yaml`:

```yaml
mesh:
  name: my-thought-lattice
  consensus_threshold: 0.95  # confidence required before write
  drift_tolerance: 0.003     # maximum acceptable divergence
  nodes:
    - type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      host: localhost
      port: 8001
    - type: local
      engine: ollama
      model: llama3
      max_thinking_depth: 5
      host: localhost
      port: 11434
  languages:
    - en
    - zh
    - es
    - ar
    - hi
    - fr
  caching:
    semantic_cache_enabled: true
    cache_ttl_hours: 168
  verification:
    min_peer_verifiers: 2
    escalation_depth: 9
```

## Core API Usage

### Single Query Reasoning

```python
from logos import Router

# Initialize router with configuration
router = Router(config="my_mesh.yaml")

# Simple query with adaptive depth
response = router.query(
    prompt="Explain the relationship between recursion and self-reference in formal systems.",
    depth="adaptive",  # or fixed: 3, 5, 7
    language="en"
)

print(response.text)
print(f"Reasoning depth used: {response.depth}")
print(f"Nodes consulted: {response.nodes_involved}")
print(f"Consensus score: {response.consensus_score}")
print(f"Verification steps: {len(response.audit_trail)}")
```

### Multi-Step Workflow

```python
from logos import Router

router = Router(config="my_mesh.yaml")

# Create a workflow for document analysis
task = router.create_workflow("analyze_this_paper.pdf")

# Chain reasoning steps
task.extract_claims()
task.verify_claims(consensus=True)
task.cross_reference(sources=["arxiv.org", "scholar.google.com"])
task.generate_summary(style="academic")

# Execute with automatic state management
result = task.execute()

print(result.summary)
print(f"Claims verified: {result.verified_claims_count}")
print(f"Total reasoning time: {result.elapsed_seconds}s")
```

### Multilingual Reasoning

```python
from logos import Router

router = Router(config="my_mesh.yaml")

# Query in one language, respond in another
response = router.query(
    prompt="Explain quantum entanglement",
    input_language="en",
    output_language="zh",
    depth=5
)

print(response.text)  # Response in Mandarin

# Language auto-detection
response = router.query(
    prompt="¿Cómo funciona la fotosíntesis?",
    depth="adaptive"  # Automatically detects Spanish
)
```

### Accessing the Audit Trail

```python
from logos import Router

router = Router(config="my_mesh.yaml")

response = router.query(
    prompt="Design a distributed caching system",
    depth=7,
    return_audit=True
)

# Inspect reasoning steps
for step in response.audit_trail:
    print(f"Step {step.index}: {step.action}")
    print(f"  Node: {step.node_id}")
    print(f"  Confidence: {step.confidence}")
    print(f"  Verifiers: {step.verifying_nodes}")
    print(f"  Timestamp: {step.timestamp}")
    print(f"  Content hash: {step.content_hash}")
```

## Strict Write Discipline (SWD) Protocol

The SWD protocol ensures zero-drift reasoning:

```python
from logos import Router, SWDConfig

# Configure custom SWD parameters
swd_config = SWDConfig(
    min_verifiers=3,
    consensus_threshold=0.97,
    escalation_enabled=True,
    escalation_depth=9,
    timeout_seconds=30
)

router = Router(
    config="my_mesh.yaml",
    swd_config=swd_config
)

# Query with custom verification
response = router.query(
    prompt="Prove the halting problem is undecidable",
    depth=7,
    swd_strict=True  # Enforces strictest verification
)

# Check if consensus was achieved
if response.consensus_achieved:
    print(f"Verified by {len(response.verifiers)} nodes")
    print(f"Final confidence: {response.consensus_score}")
else:
    print(f"Consensus failed after {response.verification_attempts} attempts")
    print(f"Escalated to depth {response.final_depth}")
```

## Node Management

### Adding Nodes Dynamically

```python
from logos import Router, NodeConfig

router = Router(config="my_mesh.yaml")

# Add a new reasoning node at runtime
new_node = NodeConfig(
    type="remote",
    engine="vllm",
    model="mistral-7b",
    host="192.168.1.100",
    port=8002,
    max_thinking_depth=5,
    capabilities=["fast_inference", "math_reasoning"]
)

router.add_node(new_node)

# Verify node is available
status = router.get_node_status(new_node.id)
print(f"Node {new_node.id} status: {status.state}")
```

### Monitoring Node Health

```python
from logos import Router

router = Router(config="my_mesh.yaml")

# Get mesh status
mesh_status = router.get_mesh_status()

for node in mesh_status.nodes:
    print(f"Node: {node.id}")
    print(f"  Load: {node.current_load}%")
    print(f"  Queries processed: {node.queries_processed}")
    print(f"  Avg response time: {node.avg_response_ms}ms")
    print(f"  Drift rate: {node.drift_rate}")
    print(f"  Uptime: {node.uptime_hours}h")
```

## Semantic Caching

```python
from logos import Router

router = Router(config="my_mesh.yaml")

# First query - full reasoning
response1 = router.query(
    prompt="What are the implications of Gödel's incompleteness theorems?",
    depth=6
)
print(f"Query 1 time: {response1.elapsed_seconds}s")

# Second similar query - semantic cache hit
response2 = router.query(
    prompt="Explain the consequences of Gödel's incompleteness results",
    depth=6
)
print(f"Query 2 time: {response2.elapsed_seconds}s")  # Much faster
print(f"Cache hit: {response2.cache_hit}")
print(f"Semantic similarity: {response2.cache_similarity_score}")

# Clear cache if needed
router.clear_semantic_cache()
```

## Real-Time Reasoning Streams

```python
from logos import Router
import asyncio

router = Router(config="my_mesh.yaml")

# Stream reasoning steps as they occur
async def stream_reasoning():
    async for step in router.stream_query(
        prompt="Design a consensus algorithm for distributed systems",
        depth=7
    ):
        print(f"[{step.node_id}] {step.content}")
        print(f"  Confidence: {step.confidence}")
        if step.verification_pending:
            print(f"  Awaiting verification from {step.pending_verifiers}")

asyncio.run(stream_reasoning())
```

## REST API Server

```python
from logos import Router, create_api_server

router = Router(config="my_mesh.yaml")

# Create REST API server
app = create_api_server(router)

# Run server
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

Query via REST:

```bash
# Single query
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain quantum computing",
    "depth": "adaptive",
    "language": "en"
  }'

# Get mesh status
curl http://localhost:8000/status

# Get audit trail for a query
curl http://localhost:8000/audit/<query_id>
```

## WebSocket Interface

```python
import asyncio
import websockets
import json

async def interactive_reasoning():
    uri = "ws://localhost:8000/ws"
    async with websockets.connect(uri) as websocket:
        # Send query
        await websocket.send(json.dumps({
            "prompt": "Design a load balancer",
            "depth": 6,
            "stream": True
        }))
        
        # Receive streaming steps
        async for message in websocket:
            step = json.loads(message)
            print(f"Step: {step['content']}")
            print(f"Node: {step['node_id']}")
            print(f"Confidence: {step['confidence']}")

asyncio.run(interactive_reasoning())
```

## Custom Reasoning Profiles

Create a reasoning profile in `profiles/custom.json`:

```json
{
  "name": "code-review",
  "description": "Specialized profile for code review reasoning",
  "default_depth": 5,
  "verification_required": true,
  "capabilities": [
    "static_analysis",
    "security_audit",
    "performance_analysis"
  ],
  "thinking_patterns": [
    "decompose_into_modules",
    "identify_edge_cases",
    "verify_error_handling",
    "check_resource_leaks"
  ],
  "synthesis_strategy": "structured_critique"
}
```

Use the profile:

```python
from logos import Router

router = Router(config="my_mesh.yaml")

response = router.query(
    prompt="Review this Python function for security issues",
    profile="code-review",
    context={"code": open("function.py").read()}
)

print(response.critique)
```

## Configuration Patterns

### High-Throughput Setup

```yaml
mesh:
  name: high-throughput-mesh
  consensus_threshold: 0.90  # Slightly relaxed for speed
  nodes:
    - type: local
      engine: vllm
      model: mistral-7b
      max_thinking_depth: 4  # Shallower for faster processing
      batch_size: 32
    - type: local
      engine: vllm
      model: mistral-7b
      max_thinking_depth: 4
      batch_size: 32
    - type: local
      engine: vllm
      model: mistral-7b
      max_thinking_depth: 4
      batch_size: 32
  caching:
    semantic_cache_enabled: true
    aggressive_caching: true
  verification:
    min_peer_verifiers: 1  # Minimum for speed
    parallel_verification: true
```

### High-Fidelity Setup

```yaml
mesh:
  name: research-grade-mesh
  consensus_threshold: 0.98  # Very strict
  drift_tolerance: 0.001
  nodes:
    - type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 9
    - type: local
      engine: vllm
      model: claude-3-opus
      max_thinking_depth: 9
  verification:
    min_peer_verifiers: 3
    cross_model_verification: true
    escalation_enabled: true
    escalation_depth: 12
  audit:
    full_trail: true
    persist_to_disk: true
```

## Troubleshooting

### Consensus Failures

```python
from logos import Router, ConsensusError

router = Router(config="my_mesh.yaml")

try:
    response = router.query(
        prompt="Complex ambiguous query",
        depth=7
    )
except ConsensusError as e:
    print(f"Consensus failed: {e.message}")
    print(f"Conflicting nodes: {e.conflicting_nodes}")
    print(f"Divergence score: {e.divergence_score}")
    
    # Retry with escalation
    response = router.query(
        prompt="Complex ambiguous query",
        depth=9,
        force_escalation=True
    )
```

### Node Connectivity Issues

```python
from logos import Router

router = Router(config="my_mesh.yaml")

# Check individual node health
for node_id in router.list_nodes():
    try:
        health = router.ping_node(node_id, timeout=5)
        print(f"Node {node_id}: {health.status}")
    except Exception as e:
        print(f"Node {node_id} unreachable: {e}")
        router.mark_node_unavailable(node_id)

# Router automatically reroutes around failed nodes
```

### High Latency Debugging

```python
from logos import Router

router = Router(config="my_mesh.yaml")

# Enable detailed timing
response = router.query(
    prompt="Test query",
    depth=5,
    profile_timing=True
)

# Inspect timing breakdown
print(f"Total time: {response.elapsed_seconds}s")
print(f"Routing time: {response.timing.routing_ms}ms")
print(f"Inference time: {response.timing.inference_ms}ms")
print(f"Verification time: {response.timing.verification_ms}ms")
print(f"Synthesis time: {response.timing.synthesis_ms}ms")

# Identify bottleneck nodes
for node_timing in response.timing.per_node:
    print(f"Node {node_timing.node_id}: {node_timing.elapsed_ms}ms")
```

### Drift Detection

```python
from logos import Router

router = Router(config="my_mesh.yaml")

# Run drift analysis
drift_report = router.analyze_drift(
    queries=100,
    reference_depth=7
)

print(f"Average drift: {drift_report.avg_drift}")
print(f"Max drift: {drift_report.max_drift}")
print(f"Drift violations: {drift_report.violations_count}")

# Nodes contributing most to drift
for node_drift in drift_report.per_node:
    print(f"Node {node_drift.node_id}: {node_drift.drift_contribution}")
```

## Environment Variables

Configure Logos Router using environment variables:

```bash
# Mesh configuration
export LOGOS_MESH_NAME="production-mesh"
export LOGOS_CONFIG_PATH="/etc/logos/mesh.yaml"

# Consensus settings
export LOGOS_CONSENSUS_THRESHOLD="0.95"
export LOGOS_MIN_VERIFIERS="2"
export LOGOS_DRIFT_TOLERANCE="0.003"

# Storage
export LOGOS_GRAIL_PATH="/var/lib/logos/grail"
export LOGOS_CACHE_PATH="/var/cache/logos"

# API settings
export LOGOS_API_HOST="0.0.0.0"
export LOGOS_API_PORT="8000"
export LOGOS_API_KEY_REQUIRED="false"

# Logging
export LOGOS_LOG_LEVEL="INFO"
export LOGOS_LOG_PATH="/var/log/logos"

# Node discovery
export LOGOS_NODE_DISCOVERY="mdns"  # or "static", "consul"
export LOGOS_NODE_ANNOUNCE_INTERVAL="30"
```

## Best Practices

1. **Start with adaptive depth**: Let the router determine complexity
2. **Monitor drift metrics**: Keep drift below 0.005 for production use
3. **Use semantic caching**: Dramatically improves throughput for similar queries
4. **Enable audit trails**: Essential for debugging consensus failures
5. **Graceful degradation**: Configure fallback strategies when consensus fails
6. **Node diversity**: Mix model types for better cross-verification
7. **Language consistency**: Keep input and output languages consistent unless translation needed
8. **Profile tuning**: Create custom profiles for specialized reasoning tasks

## CI/CD Integration Example

```python
# ci_reasoning_check.py
from logos import Router
import sys

router = Router(config="ci_mesh.yaml")

# Analyze code changes
response = router.query(
    prompt=f"Review this code change for correctness and maintainability",
    context={
        "diff": open("changes.diff").read(),
        "language": "python"
    },
    profile="code-review",
    depth=6
)

# Fail CI if critical issues found
if response.metadata.get("critical_issues", 0) > 0:
    print(f"Critical issues found: {response.critique}")
    sys.exit(1)

print("Code review passed")
sys.exit(0)
```
