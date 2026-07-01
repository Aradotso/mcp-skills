---
name: logos-distributed-reasoning-router
description: Deploy and configure Logos Router for distributed semantic reasoning with zero-drift consensus across local AI inference nodes
triggers:
  - set up logos router for distributed reasoning
  - configure logos router mesh with multiple nodes
  - implement zero-drift consensus reasoning
  - create adaptive reasoning workflow with logos
  - troubleshoot logos router configuration
  - deploy local AI reasoning mesh
  - use strict write discipline protocol
  - route semantic queries across reasoning nodes
---

# Logos Distributed Reasoning Router

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Logos Router is a distributed semantic reasoning gateway that fragments AI reasoning across local nodes with zero-drift consensus guarantees. Instead of funneling all inference through a single model, it distributes subproblems across a mesh of reasoning units that coordinate through a Strict Write Discipline protocol. This eliminates reasoning drift and maintains fidelity across complex multi-step cognitive tasks.

**Key Capabilities:**
- **Zero-drift consensus**: Cross-validation of every reasoning step before committing
- **Adaptive reasoning depth**: Dynamic escalation/reduction of cognitive complexity
- **Multilingual routing**: 16 languages with universal intermediate representation
- **24/7 continuity**: Transparent failover across mesh nodes
- **Causal audit trails**: Full traceability of routing decisions

## Installation

### Prerequisites

```bash
# Requires Python 3.11+
python --version

# At least one local inference engine (VLLM, Ollama, llama.cpp)
# Network connectivity between mesh nodes
```

### Install from GitHub

```bash
# Clone the repository
git clone https://github.com/rak7777/mythic-mcp-proxy.git
cd mythic-mcp-proxy

# Install dependencies
pip install -r requirements.txt

# Verify installation
python -m logos --version
```

### Docker Installation

```bash
# Build container
docker build -t logos-router .

# Run with mounted config
docker run -v $(pwd)/config:/config -p 8080:8080 logos-router
```

## Configuration

### Basic Mesh Configuration

Create `mesh_config.yaml`:

```yaml
mesh:
  name: my-reasoning-lattice
  consensus_threshold: 0.95  # Confidence required before write commit
  nodes:
    - type: local
      engine: vllm
      model: llama-3-70b
      endpoint: http://localhost:8000
      max_thinking_depth: 7
    - type: local
      engine: ollama
      model: mistral
      endpoint: http://localhost:11434
      max_thinking_depth: 5
  
  # Strict Write Discipline settings
  write_discipline:
    enabled: true
    peer_verification_count: 2  # Minimum peers for consensus
    timeout_seconds: 30
    escalation_depth: 9  # Max depth for arbitration
  
  # Language support
  languages:
    - en
    - es
    - fr
    - zh
    - ar
  
  # Semantic caching
  cache:
    enabled: true
    type: semantic
    max_entries: 10000
    ttl_hours: 24
```

### Node-Specific Configuration

For heterogeneous mesh with different capabilities:

```yaml
mesh:
  name: mixed-capability-mesh
  nodes:
    - id: fast-node
      type: local
      engine: vllm
      model: llama-3-8b
      max_thinking_depth: 3
      priority: speed
      
    - id: deep-node
      type: local
      engine: vllm
      model: llama-3-70b
      max_thinking_depth: 9
      priority: accuracy
      
    - id: specialized-node
      type: local
      engine: ollama
      model: codellama
      max_thinking_depth: 7
      domains:
        - code_reasoning
        - mathematical_proof
```

## Python API Usage

### Basic Query Routing

```python
from logos import Router

# Initialize router with config
router = Router(config="mesh_config.yaml")

# Simple query with adaptive depth
response = router.query(
    prompt="Explain the relationship between recursion and self-reference in formal systems.",
    depth="adaptive",  # Can also be integer 1-9
    language="en"
)

print(response.text)
print(f"Reasoning depth: {response.depth}")
print(f"Nodes involved: {response.nodes_involved}")
print(f"Consensus score: {response.consensus_score}")
```

### Multi-Step Workflow

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Create workflow for complex task
workflow = router.create_workflow(
    name="paper_analysis",
    input_file="research_paper.pdf"
)

# Define reasoning steps
workflow.add_step("extract_claims", depth=5)
workflow.add_step("verify_claims", consensus=True, depth=7)
workflow.add_step("identify_gaps", depth=6)
workflow.add_step("generate_summary", style="academic", depth=4)

# Execute with audit trail
result = workflow.execute()

# Access results
print(result.summary)
print(f"Total reasoning time: {result.elapsed_seconds}s")

# Inspect audit trail
for step in result.audit_trail:
    print(f"{step.name}: {step.confidence} ({step.nodes_consulted} nodes)")
```

### Explicit Node Selection

```python
# Route to specific node type
response = router.query(
    prompt="Refactor this Python function for better performance",
    node_selector=lambda n: "code" in n.domains,
    depth=5
)

# Round-robin across all nodes
response = router.query(
    prompt="Translate this paragraph to Spanish",
    routing_strategy="round_robin",
    depth=3
)

# Load-balanced routing
response = router.query(
    prompt="Analyze sentiment in this text",
    routing_strategy="least_loaded",
    depth=4
)
```

### Streaming Reasoning Steps

```python
# Stream reasoning process in real-time
for step in router.query_stream(
    prompt="Prove the Pythagorean theorem using geometric reasoning",
    depth=7
):
    print(f"[{step.node_id}] Depth {step.depth}: {step.fragment}")
    if step.requires_consensus:
        print(f"  Waiting for {step.pending_verifications} peer verifications...")
    if step.committed:
        print(f"  ✓ Committed with {step.consensus_score} confidence")
```

## CLI Usage

### Basic Commands

```bash
# Start router daemon
logos serve --config mesh_config.yaml --port 8080

# Single query from CLI
logos query "Explain quantum entanglement" --depth adaptive --lang en

# Query with specific node
logos query "Debug this code" --node specialized-node --depth 6

# Load workflow from file
logos workflow run paper_analysis.yaml
```

### Mesh Management

```bash
# List active nodes
logos mesh list

# Check node health
logos mesh health --verbose

# Add node to running mesh
logos mesh add-node \
  --type local \
  --engine ollama \
  --model mixtral \
  --endpoint http://192.168.1.100:11434

# Remove node gracefully
logos mesh remove-node fast-node --drain-timeout 300
```

### Audit and Debugging

```bash
# View reasoning trail for query
logos audit show <query_id>

# Export audit trail
logos audit export <query_id> --format json > audit.json

# Check consensus statistics
logos stats consensus --last 1000

# Measure drift across nodes
logos benchmark drift --queries 1000 --output drift_report.html
```

## REST API

### Start API Server

```python
from logos import Router
from logos.api import create_app

router = Router(config="mesh_config.yaml")
app = create_app(router)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

### API Endpoints

```bash
# Query endpoint
curl -X POST http://localhost:8080/v1/query \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Analyze this dataset for anomalies",
    "depth": "adaptive",
    "language": "en",
    "context": {"dataset_size": 10000}
  }'

# Workflow execution
curl -X POST http://localhost:8080/v1/workflow \
  -H "Content-Type: application/json" \
  -d '{
    "name": "code_review",
    "steps": [
      {"action": "analyze_complexity", "depth": 5},
      {"action": "find_bugs", "depth": 7, "consensus": true},
      {"action": "suggest_improvements", "depth": 6}
    ],
    "input": {"repository": "https://github.com/user/repo"}
  }'

# Health check
curl http://localhost:8080/health

# Mesh status
curl http://localhost:8080/v1/mesh/status
```

## Advanced Patterns

### Custom Reasoning Profiles

```python
from logos import Router, ReasoningProfile

# Define custom profile
profile = ReasoningProfile(
    name="mathematical_proof",
    base_depth=7,
    escalation_triggers=[
        "proof by contradiction",
        "induction step",
        "lemma verification"
    ],
    verification_strictness=0.98,
    allowed_domains=["mathematics", "logic"]
)

router = Router(config="mesh_config.yaml")
router.register_profile(profile)

# Use custom profile
response = router.query(
    prompt="Prove that √2 is irrational",
    profile="mathematical_proof"
)
```

### Semantic Caching

```python
from logos import Router
from logos.cache import SemanticCache

# Configure semantic cache
cache = SemanticCache(
    embedding_model="sentence-transformers/all-MiniLM-L6-v2",
    similarity_threshold=0.92,
    max_entries=50000
)

router = Router(config="mesh_config.yaml", cache=cache)

# First query - full reasoning
response1 = router.query("What is machine learning?")
print(f"Cache hit: {response1.from_cache}")  # False

# Similar query - uses cached reasoning
response2 = router.query("Explain machine learning concepts")
print(f"Cache hit: {response2.from_cache}")  # True
print(f"Original query: {response2.cache_source}")
```

### Graceful Degradation

```python
from logos import Router, DegradationPolicy

# Define degradation behavior
policy = DegradationPolicy(
    max_queue_depth=100,
    fallback_depth=3,
    notify_on_degradation=True
)

router = Router(
    config="mesh_config.yaml",
    degradation_policy=policy
)

# Query under load
response = router.query(
    prompt="Complex reasoning task",
    depth=8,
    allow_degradation=True
)

if response.degraded:
    print(f"Degraded from depth {response.requested_depth} to {response.actual_depth}")
    print(f"Reason: {response.degradation_reason}")
```

### Cross-Language Reasoning

```python
# Input in Spanish, output in English
response = router.query(
    prompt="¿Cuáles son las implicaciones éticas de la inteligencia artificial?",
    input_language="es",
    output_language="en",
    depth=6
)

# Multilingual workflow
workflow = router.create_workflow("translate_and_analyze")
workflow.add_step("translate", from_lang="zh", to_lang="en")
workflow.add_step("extract_entities", language="en")
workflow.add_step("sentiment_analysis", language="en")
result = workflow.execute(input_text="中文输入文本...")
```

## Environment Variables

```bash
# Required
export LOGOS_CONFIG_PATH="/path/to/mesh_config.yaml"

# Optional
export LOGOS_LOG_LEVEL="INFO"  # DEBUG, INFO, WARNING, ERROR
export LOGOS_CACHE_DIR="/var/logos/cache"
export LOGOS_AUDIT_DIR="/var/logos/audit"
export LOGOS_MAX_CONCURRENT_QUERIES="50"
export LOGOS_CONSENSUS_TIMEOUT="30"  # seconds
export LOGOS_METRICS_ENABLED="true"

# Backend-specific
export VLLM_ENDPOINT="http://localhost:8000"
export OLLAMA_ENDPOINT="http://localhost:11434"
```

## Troubleshooting

### Node Connectivity Issues

```python
from logos import Router
from logos.diagnostics import check_node_connectivity

router = Router(config="mesh_config.yaml")

# Test all nodes
results = check_node_connectivity(router.mesh)
for node_id, status in results.items():
    if not status.healthy:
        print(f"Node {node_id} unreachable: {status.error}")
        print(f"  Last seen: {status.last_heartbeat}")
        print(f"  Recommendation: {status.recommended_action}")
```

### Consensus Failures

```bash
# Check consensus statistics
logos stats consensus --show-failures

# Lower threshold temporarily
logos config set mesh.consensus_threshold 0.90

# Increase timeout for slow nodes
logos config set mesh.write_discipline.timeout_seconds 60
```

### High Latency Debugging

```python
from logos import Router
from logos.profiling import enable_profiling

router = Router(config="mesh_config.yaml")
enable_profiling(router, detail_level="verbose")

response = router.query("Test query", depth=5)

# View profiling data
profile = response.profiling_data
print(f"Node selection: {profile.routing_time_ms}ms")
print(f"Reasoning time: {profile.inference_time_ms}ms")
print(f"Consensus time: {profile.consensus_time_ms}ms")
print(f"Synthesis time: {profile.synthesis_time_ms}ms")

# Identify bottleneck
bottleneck = profile.identify_bottleneck()
print(f"Bottleneck: {bottleneck.stage} ({bottleneck.percentage}% of total time)")
```

### Memory Pressure

```yaml
# mesh_config.yaml - reduce memory usage
mesh:
  nodes:
    - type: local
      engine: vllm
      model: llama-3-8b  # Use smaller model
      max_thinking_depth: 5  # Reduce max depth
      max_context_length: 4096  # Limit context
  
  cache:
    max_entries: 1000  # Reduce cache size
    
  write_discipline:
    peer_verification_count: 1  # Reduce verification peers
```

### Drift Measurement

```python
from logos import Router
from logos.evaluation import measure_drift

router = Router(config="mesh_config.yaml")

# Run drift benchmark
drift_report = measure_drift(
    router=router,
    test_queries="benchmark_queries.json",
    num_runs=1000,
    ground_truth="expected_outputs.json"
)

print(f"Average drift: {drift_report.mean_drift}%")
print(f"Max drift: {drift_report.max_drift}%")
print(f"Queries exceeding threshold: {drift_report.failures}/{drift_report.total}")

# Export detailed report
drift_report.save("drift_analysis.html")
```

## Integration Examples

### CI/CD Code Review

```python
# .github/workflows/logos-review.py
from logos import Router
import sys

router = Router(config="mesh_config.yaml")

workflow = router.create_workflow("code_review")
workflow.add_step("analyze_diff", depth=5)
workflow.add_step("find_security_issues", depth=7, consensus=True)
workflow.add_step("check_performance", depth=6)
workflow.add_step("verify_tests", depth=5)

result = workflow.execute(
    input_file=sys.argv[1]  # Git diff file
)

if result.critical_issues:
    print("❌ Critical issues found:")
    for issue in result.critical_issues:
        print(f"  - {issue.description} (confidence: {issue.confidence})")
    sys.exit(1)
else:
    print("✓ Code review passed")
    sys.exit(0)
```

### Real-Time Streaming Interface

```python
from logos import Router
import asyncio

router = Router(config="mesh_config.yaml")

async def stream_reasoning(prompt: str):
    async for event in router.query_stream_async(prompt, depth="adaptive"):
        if event.type == "reasoning_step":
            print(f"[Step {event.depth}] {event.text}")
        elif event.type == "consensus_check":
            print(f"[Consensus] {event.verifications}/{event.required} verified")
        elif event.type == "complete":
            print(f"\n[Final] {event.text}")
            print(f"Total nodes: {event.nodes_involved}")

asyncio.run(stream_reasoning("Explain quantum computing"))
```

This skill provides comprehensive coverage of Logos Router's distributed reasoning capabilities, focusing on practical usage patterns for AI coding agents assisting developers with deployment, configuration, and integration.
