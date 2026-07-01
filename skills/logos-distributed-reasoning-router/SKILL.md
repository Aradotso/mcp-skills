---
name: logos-distributed-reasoning-router
description: Deploy and use Logos Router for zero-drift distributed AI reasoning with adaptive Claude Opus-style thinking and multilingual semantic routing
triggers:
  - how do I set up Logos Router for distributed reasoning
  - configure a reasoning mesh with zero drift consensus
  - use Logos Router for multi-step AI workflows
  - implement strict write discipline with Logos
  - create adaptive reasoning nodes with Logos Router
  - troubleshoot Logos Router mesh consensus
  - set up multilingual semantic routing with Logos
  - deploy local reasoning infrastructure with zero drift
---

# Logos Distributed Reasoning Router

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Logos Router is a distributed semantic reasoning gateway that eliminates AI reasoning "drift" by fragmenting inference across local nodes with zero-drift consensus guarantees. Instead of funneling all reasoning through a single model, it distributes subproblems across a mesh of reasoning units that coordinate through a Strict Write Discipline (SWD) protocol. Every reasoning step is verified by multiple nodes before being committed, creating an immutable DAG of cognition with causal audit trails.

**Key Features:**
- **Zero-drift consensus**: Cross-validation prevents hallucination chains
- **Adaptive reasoning depth**: Automatically escalates complexity for hard problems
- **Multilingual support**: 16 languages with universal semantic representation
- **24/7 continuity**: Mesh resilience with transparent failover
- **Local sovereignty**: All reasoning runs on your infrastructure
- **Causal audit trails**: Complete lineage of every routing decision

## Installation

### Prerequisites

- Python 3.11 or later
- At least one local inference engine (VLLM, Ollama, llama.cpp, etc.)
- Network access between mesh nodes (localhost works for single-machine)

### Install from Source

```bash
# Clone the repository
git clone https://github.com/rak7777/mythic-mcp-proxy.git
cd mythic-mcp-proxy

# Install dependencies
pip install -r requirements.txt

# Install the router package
pip install -e .
```

### Verify Installation

```python
from logos import Router

print(Router.version())
```

## Configuration

### Basic Mesh Configuration

Create a YAML configuration file defining your reasoning mesh topology:

```yaml
# config/basic_mesh.yaml
mesh:
  name: local-reasoning-mesh
  consensus_threshold: 0.95  # 95% confidence required before commit
  nodes:
    - type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      host: localhost
      port: 8000
    - type: local
      engine: ollama
      model: llama3-70b
      max_thinking_depth: 5
      host: localhost
      port: 11434
  languages:
    - en
    - zh
    - es
    - fr
    - de
```

### Advanced Configuration with Multiple Nodes

```yaml
# config/advanced_mesh.yaml
mesh:
  name: distributed-thought-lattice
  consensus_threshold: 0.95
  max_concurrent_queries: 50
  semantic_cache_size: 1024  # MB
  
  nodes:
    - name: primary-reasoner
      type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      capabilities:
        - deep_reasoning
        - code_analysis
        - mathematical_proof
      
    - name: verification-node
      type: local
      engine: ollama
      model: mixtral-8x7b
      max_thinking_depth: 5
      capabilities:
        - fact_checking
        - logical_consistency
      
    - name: synthesis-node
      type: local
      engine: llama.cpp
      model: llama3-70b
      max_thinking_depth: 6
      capabilities:
        - summarization
        - harmonization

  reasoning:
    strict_write_discipline: true
    verification_peer_count: 2
    escalation_threshold: 0.85
    graceful_degradation: true
    
  languages:
    - en
    - zh
    - es
    - ar
    - hi
    - fr
    - de
    - ja
    - ko
    - pt
    - ru

  storage:
    grail_layer_path: ./reasoning_states
    max_history_days: 90
    compression: true
```

## Core API Usage

### Basic Query Routing

```python
from logos import Router

# Initialize router with configuration
router = Router(config="config/basic_mesh.yaml")

# Single query with adaptive reasoning
response = router.query(
    prompt="Explain the halting problem and its implications for AI safety",
    depth="adaptive",  # Can be "adaptive", 1-7, or "minimal"
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
from logos import Router, Workflow

router = Router(config="config/advanced_mesh.yaml")

# Create a workflow for complex multi-step reasoning
workflow = router.create_workflow(
    name="research_paper_analysis",
    consensus=True  # Enable strict consensus at each step
)

# Define workflow steps
workflow.add_step(
    "extract_claims",
    prompt="Extract all factual claims from this document",
    input_file="papers/ai_safety_paper.pdf",
    depth=5
)

workflow.add_step(
    "verify_claims",
    prompt="Verify each claim against known literature",
    depends_on="extract_claims",
    depth=7,
    require_consensus=True
)

workflow.add_step(
    "generate_summary",
    prompt="Synthesize findings into an academic summary",
    depends_on="verify_claims",
    depth=6,
    style="academic"
)

# Execute workflow (blocks until complete)
result = workflow.execute()

print(f"Workflow completed in {result.duration_seconds}s")
print(f"Total reasoning steps: {result.total_steps}")
print(f"Consensus failures: {result.consensus_failures}")
print(result.final_output)
```

### Streaming Reasoning Steps

```python
from logos import Router

router = Router(config="config/basic_mesh.yaml")

# Stream reasoning steps in real-time
for step in router.query_stream(
    prompt="Design a distributed consensus algorithm for Byzantine fault tolerance",
    depth="adaptive"
):
    print(f"[Node {step.node_id}] Depth {step.current_depth}: {step.text}")
    print(f"  Confidence: {step.confidence:.2%}")
    
    if step.requires_escalation:
        print(f"  ⚠️  Escalating to higher reasoning depth")
    
    if step.consensus_reached:
        print(f"  ✓ Consensus reached ({step.consensus_score:.2%})")
```

### Multilingual Reasoning

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

# Query in Chinese, get response in Chinese
response = router.query(
    prompt="解释量子纠缠如何影响信息理论",
    language="zh",
    depth=6
)

print(response.text)  # Response in Chinese

# Query in Spanish, request response in English
response = router.query(
    prompt="¿Cómo funcionan las redes neuronales convolucionales?",
    input_language="es",
    output_language="en",
    depth=5
)

print(response.text)  # Response in English
```

## Strict Write Discipline (SWD)

### Understanding SWD Protocol

The Strict Write Discipline protocol ensures zero-drift reasoning:

1. **Proposal**: Node generates candidate reasoning step
2. **Broadcast**: Candidate sent to peer nodes for verification
3. **Verification**: Peers check logical consistency and factual accuracy
4. **Consensus**: Step committed if confidence exceeds threshold
5. **Escalation**: Failed consensus triggers higher-depth arbitration

### Inspecting SWD Audit Trails

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

response = router.query(
    prompt="Prove that the set of real numbers is uncountable",
    depth="adaptive"
)

# Examine complete reasoning trail
for i, step in enumerate(response.audit_trail):
    print(f"\n=== Step {i+1} ===")
    print(f"Node: {step.node_id}")
    print(f"Proposal: {step.proposal}")
    print(f"Peer verifications: {step.peer_count}")
    print(f"Consensus score: {step.consensus_score:.2%}")
    print(f"Committed: {step.committed}")
    
    if step.escalated:
        print(f"⚠️  Escalated to depth {step.escalation_depth}")
        print(f"Arbiter: {step.arbiter_node}")
```

### Configuring SWD Parameters

```python
from logos import Router, SWDConfig

swd_config = SWDConfig(
    consensus_threshold=0.97,  # Require 97% consensus
    verification_peer_count=3,  # Use 3 peers for verification
    escalation_threshold=0.80,  # Escalate if below 80%
    max_escalation_depth=9,     # Maximum depth for escalation
    timeout_seconds=30          # Timeout for peer verification
)

router = Router(
    config="config/basic_mesh.yaml",
    swd_config=swd_config
)
```

## CLI Usage

### Basic Commands

```bash
# Start the reasoning mesh
logos start --config config/basic_mesh.yaml

# Query the mesh from CLI
logos query "Explain the Church-Turing thesis" --depth adaptive

# Query with specific language
logos query "Qu'est-ce que l'intelligence artificielle?" --language fr

# Run a workflow from file
logos workflow run workflows/paper_analysis.yaml

# Check mesh status
logos status

# View reasoning audit trail
logos audit --query-id abc123-def456

# Stop the mesh
logos stop
```

### Advanced CLI Options

```bash
# Query with custom consensus threshold
logos query "Design a consensus algorithm" \
  --depth 7 \
  --consensus-threshold 0.98 \
  --peers 4

# Stream reasoning steps to console
logos query "Prove the Pythagorean theorem" \
  --stream \
  --show-consensus

# Export reasoning trail to JSON
logos audit --query-id abc123 --format json > trail.json

# Benchmark mesh performance
logos benchmark --queries 100 --depth adaptive

# Add a new node to running mesh
logos node add \
  --engine ollama \
  --model llama3-70b \
  --host localhost \
  --port 11434
```

## REST API Integration

### Starting API Server

```python
from logos import Router, APIServer

router = Router(config="config/advanced_mesh.yaml")

# Start REST API server
server = APIServer(router, host="0.0.0.0", port=8080)
server.start()
```

### API Endpoints

```python
import requests
import json

# Submit a query
response = requests.post(
    "http://localhost:8080/query",
    json={
        "prompt": "Explain gradient descent optimization",
        "depth": "adaptive",
        "language": "en",
        "stream": False
    }
)

result = response.json()
print(result["text"])
print(f"Consensus: {result['consensus_score']}")

# Get query status
status = requests.get(f"http://localhost:8080/query/{result['query_id']}")
print(status.json())

# Stream reasoning steps via WebSocket
import websocket

ws = websocket.create_connection("ws://localhost:8080/query/stream")
ws.send(json.dumps({
    "prompt": "Design a distributed database",
    "depth": 7
}))

while True:
    step = json.loads(ws.recv())
    if step["type"] == "complete":
        break
    print(f"Step: {step['text']} (confidence: {step['confidence']})")

ws.close()
```

## Common Patterns

### Pattern: Code Review with Consensus

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

def review_code_with_consensus(code_file_path):
    workflow = router.create_workflow(
        name="code_review",
        consensus=True
    )
    
    workflow.add_step(
        "analyze_structure",
        prompt=f"Analyze code structure and architecture in {code_file_path}",
        depth=5
    )
    
    workflow.add_step(
        "identify_issues",
        prompt="Identify potential bugs, security issues, and code smells",
        depends_on="analyze_structure",
        depth=7,
        require_consensus=True
    )
    
    workflow.add_step(
        "suggest_improvements",
        prompt="Suggest specific improvements with code examples",
        depends_on="identify_issues",
        depth=6
    )
    
    result = workflow.execute()
    return result

# Use the pattern
review = review_code_with_consensus("src/main.py")
print(review.final_output)
```

### Pattern: Multi-Language Documentation Generation

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

def generate_multilingual_docs(source_code, target_languages):
    base_response = router.query(
        prompt=f"Generate comprehensive API documentation for this code:\n\n{source_code}",
        language="en",
        depth=6
    )
    
    docs = {"en": base_response.text}
    
    for lang in target_languages:
        translated = router.query(
            prompt=f"Translate this technical documentation, preserving all code examples:\n\n{base_response.text}",
            input_language="en",
            output_language=lang,
            depth=4
        )
        docs[lang] = translated.text
    
    return docs

# Generate docs in multiple languages
docs = generate_multilingual_docs(
    source_code=open("api.py").read(),
    target_languages=["zh", "es", "fr", "de", "ja"]
)

for lang, content in docs.items():
    with open(f"docs/api_{lang}.md", "w") as f:
        f.write(content)
```

### Pattern: Resilient Long-Running Analysis

```python
from logos import Router
import time

router = Router(config="config/advanced_mesh.yaml")

def analyze_large_dataset_with_checkpoints(dataset_path):
    workflow = router.create_workflow(
        name="dataset_analysis",
        checkpoint_interval=10  # Checkpoint every 10 steps
    )
    
    # Load checkpoint if exists
    if workflow.has_checkpoint():
        print("Resuming from checkpoint...")
        workflow.load_checkpoint()
    
    workflow.add_step(
        "load_data",
        prompt=f"Load and validate dataset structure from {dataset_path}",
        depth=3
    )
    
    workflow.add_step(
        "statistical_analysis",
        prompt="Perform comprehensive statistical analysis",
        depends_on="load_data",
        depth=6,
        timeout_minutes=30
    )
    
    workflow.add_step(
        "pattern_detection",
        prompt="Detect patterns, anomalies, and correlations",
        depends_on="statistical_analysis",
        depth=7,
        timeout_minutes=45
    )
    
    workflow.add_step(
        "generate_report",
        prompt="Generate executive summary and detailed findings",
        depends_on="pattern_detection",
        depth=5
    )
    
    try:
        result = workflow.execute()
        return result
    except Exception as e:
        print(f"Workflow interrupted: {e}")
        workflow.save_checkpoint()
        raise

# Execute with automatic recovery
result = analyze_large_dataset_with_checkpoints("data/large_dataset.csv")
```

### Pattern: Adaptive Reasoning Depth

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

def adaptive_query_with_fallback(prompt, max_attempts=3):
    """Try adaptive reasoning, fall back to lower depth if consensus fails"""
    
    depths = ["adaptive", 7, 5, 3]
    
    for attempt, depth in enumerate(depths[:max_attempts]):
        try:
            response = router.query(
                prompt=prompt,
                depth=depth,
                timeout_seconds=60
            )
            
            if response.consensus_score >= 0.90:
                return response
            
            print(f"Attempt {attempt+1}: Consensus {response.consensus_score:.2%}, trying lower depth...")
            
        except Exception as e:
            print(f"Attempt {attempt+1} failed: {e}")
            continue
    
    # Final attempt with minimal depth
    return router.query(prompt=prompt, depth=1)

# Use adaptive fallback
response = adaptive_query_with_fallback(
    "Explain the philosophical implications of Gödel's incompleteness theorems"
)
```

## Troubleshooting

### Issue: Low Consensus Scores

**Symptoms**: Queries frequently fail to reach consensus threshold

**Solutions**:

```python
from logos import Router

# 1. Lower consensus threshold temporarily
router = Router(
    config="config/basic_mesh.yaml",
    consensus_threshold=0.85  # Down from 0.95
)

# 2. Increase verification peer count
router.configure_swd(verification_peer_count=4)

# 3. Check node health
status = router.get_mesh_status()
for node in status.nodes:
    print(f"{node.name}: {node.health_score:.2%}")
    if node.health_score < 0.80:
        print(f"  ⚠️  Low health - check {node.name} logs")

# 4. Inspect specific consensus failure
response = router.query(prompt="...", depth=7)
if response.consensus_score < 0.95:
    for step in response.audit_trail:
        if not step.committed:
            print(f"Failed step: {step.proposal}")
            print(f"Peer disagreements: {step.disagreements}")
```

### Issue: High Latency

**Symptoms**: Queries take longer than expected

**Solutions**:

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

# 1. Profile query performance
response = router.query(
    prompt="...",
    depth="adaptive",
    profile=True
)

print(f"Total time: {response.timing.total_seconds}s")
print(f"Consensus overhead: {response.timing.consensus_seconds}s")
print(f"Node processing: {response.timing.processing_seconds}s")
print(f"Network latency: {response.timing.network_seconds}s")

# 2. Enable semantic caching
router.enable_semantic_cache(size_mb=2048)

# 3. Reduce reasoning depth for simple queries
if len(prompt.split()) < 20:
    response = router.query(prompt=prompt, depth=3)
else:
    response = router.query(prompt=prompt, depth="adaptive")

# 4. Load balance across nodes
router.rebalance_mesh()
```

### Issue: Node Disconnection

**Symptoms**: Nodes drop out of mesh during operation

**Solutions**:

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

# 1. Enable automatic failover
router.enable_auto_failover(
    health_check_interval=10,  # seconds
    failure_threshold=3         # failed checks before removal
)

# 2. Monitor node health
def monitor_mesh():
    while True:
        status = router.get_mesh_status()
        
        for node in status.nodes:
            if not node.is_healthy:
                print(f"⚠️  {node.name} unhealthy: {node.error}")
                
                # Attempt reconnection
                router.reconnect_node(node.name)
        
        time.sleep(30)

# 3. Check network connectivity
router.ping_all_nodes()

# 4. Review node logs
logs = router.get_node_logs(node_name="primary-reasoner", last_n=100)
for log in logs:
    if log.level == "ERROR":
        print(f"{log.timestamp}: {log.message}")
```

### Issue: Memory Exhaustion

**Symptoms**: Nodes run out of memory during complex reasoning

**Solutions**:

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

# 1. Enable graceful degradation
router.enable_graceful_degradation(
    memory_threshold_percent=85,
    auto_reduce_depth=True
)

# 2. Limit concurrent queries
router.set_max_concurrent_queries(10)

# 3. Configure memory limits per node
router.configure_node(
    node_name="primary-reasoner",
    max_memory_gb=12,
    max_context_tokens=32000
)

# 4. Clean up old reasoning states
router.cleanup_grail_layer(older_than_days=7)

# 5. Monitor memory usage
status = router.get_mesh_status()
for node in status.nodes:
    print(f"{node.name}: {node.memory_used_gb:.1f}GB / {node.memory_total_gb:.1f}GB")
    if node.memory_percent > 90:
        router.restart_node(node.name, graceful=True)
```

### Issue: Audit Trail Storage Growth

**Symptoms**: Grail layer storage grows too large

**Solutions**:

```python
from logos import Router

router = Router(config="config/advanced_mesh.yaml")

# 1. Enable compression
router.configure_grail_layer(
    compression=True,
    compression_level=9
)

# 2. Set retention policy
router.set_audit_retention(
    max_days=30,
    max_size_gb=50,
    archive_old_trails=True,
    archive_path="./archives"
)

# 3. Prune low-value trails
router.prune_audit_trails(
    min_consensus_score=0.80,
    min_depth=3
)

# 4. Export and delete old trails
old_trails = router.export_audit_trails(
    older_than_days=90,
    output_path="./exports/trails_2026.json"
)
router.delete_exported_trails(old_trails)
```

## Environment Variables

```bash
# Configuration
export LOGOS_CONFIG_PATH=/path/to/config.yaml
export LOGOS_MESH_NAME=my-reasoning-mesh

# Performance
export LOGOS_MAX_CONCURRENT_QUERIES=50
export LOGOS_SEMANTIC_CACHE_SIZE_MB=2048

# Consensus
export LOGOS_CONSENSUS_THRESHOLD=0.95
export LOGOS_VERIFICATION_PEER_COUNT=2

# Storage
export LOGOS_GRAIL_LAYER_PATH=/data/reasoning_states
export LOGOS_AUDIT_RETENTION_DAYS=90

# API
export LOGOS_API_HOST=0.0.0.0
export LOGOS_API_PORT=8080
export LOGOS_API_AUTH_TOKEN=${LOGOS_AUTH_TOKEN}

# Logging
export LOGOS_LOG_LEVEL=INFO
export LOGOS_LOG_PATH=/var/log/logos
```

## References

- GitHub: https://github.com/rak7777/mythic-mcp-proxy
- Documentation: See README.md in repository
- Reasoning Profiles: `/profiles` directory in source
- Example Workflows: `/examples` directory in source
