---
name: logos-router-reasoning-mesh
description: Deploy and configure Logos Router, a distributed semantic reasoning gateway with zero-drift consensus for AI agents across local inference nodes.
triggers:
  - set up logos router for distributed reasoning
  - configure a reasoning mesh with multiple nodes
  - implement zero-drift consensus protocol
  - route AI queries across semantic reasoning nodes
  - create adaptive claude opus reasoning workflows
  - debug logos router mesh configuration
  - add multilingual support to reasoning pipeline
  - implement strict write discipline protocol
---

# Logos Router Reasoning Mesh

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Logos Router is a distributed semantic reasoning gateway that eliminates cognitive drift by routing AI queries across a mesh of local inference nodes with consensus-based verification. Unlike traditional single-model inference, it fragments reasoning across multiple nodes with a Strict Write Discipline protocol that validates every reasoning step before output.

## Core Concepts

**Zero-Drift Promise**: Every reasoning step is cross-validated by multiple nodes before being committed, preventing the subtle architectural divergences that occur in traditional AI assistants.

**Three-Layer Architecture**:
- **Grail Layer**: Content-addressed persistence for immutable reasoning trails
- **Chiron Layer**: Semantic routing and node assignment
- **Orpheus Layer**: Multi-voice synthesis and conflict resolution

**Strict Write Discipline (SWD)**: A consensus protocol requiring 0.95+ confidence across peer nodes before emitting tokens.

## Installation

### Prerequisites

```bash
# Requires Python 3.11+
python --version

# Install from source
git clone https://github.com/rak7777/mythic-mcp-proxy.git
cd mythic-mcp-proxy
pip install -e .
```

### With Local Inference Engine

```bash
# With VLLM
pip install logos-router[vllm]

# With Ollama
pip install logos-router[ollama]

# With llama.cpp
pip install logos-router[llamacpp]
```

## Configuration

### Basic Mesh Configuration

Create `mesh_config.yaml`:

```yaml
mesh:
  name: dev-reasoning-lattice
  consensus_threshold: 0.95
  max_reasoning_depth: 7
  
  nodes:
    - type: local
      engine: vllm
      model: opus-mini-4.8
      endpoint: http://localhost:8000
      max_thinking_depth: 7
      capabilities:
        - logical_reasoning
        - code_analysis
        - multi_step_planning
    
    - type: local
      engine: ollama
      model: llama3.1:70b
      endpoint: http://localhost:11434
      max_thinking_depth: 5
      capabilities:
        - factual_verification
        - summarization

  languages:
    - en
    - es
    - fr
    - zh

  persistence:
    backend: filesystem
    path: ./reasoning_trail
    
  routing:
    strategy: semantic_similarity
    load_balancing: true
    fallback_depth: 3
```

### Environment Variables

```bash
export LOGOS_CONFIG_PATH=./mesh_config.yaml
export LOGOS_PERSISTENCE_DIR=./reasoning_trail
export LOGOS_LOG_LEVEL=INFO
export LOGOS_CONSENSUS_THRESHOLD=0.95
export LOGOS_MAX_NODES=8
```

## Basic Usage

### Single Query Reasoning

```python
from logos import Router

# Initialize router with config
router = Router(config="mesh_config.yaml")

# Simple adaptive reasoning query
response = router.query(
    prompt="Explain the halting problem and its implications for program verification.",
    depth="adaptive",  # or: "shallow", "medium", "deep", or integer 1-7
    language="en"
)

print(response.text)
print(f"Reasoning depth: {response.depth}")
print(f"Nodes used: {response.nodes_involved}")
print(f"Consensus score: {response.consensus_score}")

# Access reasoning trail
for step in response.reasoning_trail:
    print(f"Step {step.index}: {step.conclusion}")
    print(f"  Verified by: {step.verifying_nodes}")
    print(f"  Confidence: {step.confidence}")
```

### Multi-Language Reasoning

```python
# Query in Spanish, get response in English
response = router.query(
    prompt="¿Cuál es la diferencia entre recursión y iteración?",
    source_language="es",
    target_language="en",
    depth=5
)

# Query in English, respond in Mandarin
response = router.query(
    prompt="What are the benefits of functional programming?",
    source_language="en",
    target_language="zh",
    depth="adaptive"
)
```

## Advanced Workflows

### Multi-Step Reasoning Workflow

```python
from logos import Router, Workflow

router = Router(config="mesh_config.yaml")

# Create a workflow for complex analysis
workflow = router.create_workflow(
    name="code_review_analysis",
    consensus_required=True
)

# Define reasoning steps
workflow.add_step(
    "extract_structure",
    prompt="Analyze the architecture of this codebase: {code_path}",
    depth=6,
    timeout=30
)

workflow.add_step(
    "identify_issues",
    prompt="Based on the structure, identify potential issues",
    depends_on=["extract_structure"],
    depth=7
)

workflow.add_step(
    "suggest_refactors",
    prompt="Suggest refactoring approaches for identified issues",
    depends_on=["identify_issues"],
    depth=5,
    parallel=True  # Can run with other steps
)

workflow.add_step(
    "synthesize_report",
    prompt="Create a comprehensive review report",
    depends_on=["extract_structure", "identify_issues", "suggest_refactors"],
    depth=4
)

# Execute workflow
result = workflow.execute(
    variables={"code_path": "./src"},
    consensus_threshold=0.97
)

# Access results
for step_name, output in result.steps.items():
    print(f"\n=== {step_name} ===")
    print(output.text)
    print(f"Consensus: {output.consensus_score}")
```

### Streaming Reasoning Steps

```python
import asyncio
from logos import AsyncRouter

async def stream_reasoning():
    router = AsyncRouter(config="mesh_config.yaml")
    
    async for step in router.query_stream(
        prompt="Design a distributed caching system for a social network",
        depth=7
    ):
        print(f"[Node {step.node_id}] {step.fragment}")
        print(f"  Status: {step.status}")
        print(f"  Confidence: {step.confidence}")
        
        if step.status == "consensus_pending":
            print(f"  Awaiting verification from {step.pending_nodes}")
        elif step.status == "committed":
            print(f"  ✓ Verified and committed")

asyncio.run(stream_reasoning())
```

## Code Analysis Patterns

### Repository Analysis

```python
from logos import Router
from pathlib import Path

router = Router(config="mesh_config.yaml")

# Analyze entire codebase
def analyze_repository(repo_path: str):
    workflow = router.create_workflow("repo_analysis")
    
    # Step 1: Extract file structure
    workflow.add_step(
        "map_structure",
        prompt=f"List all source files in {repo_path} and categorize by purpose",
        depth=3
    )
    
    # Step 2: Analyze dependencies
    workflow.add_step(
        "analyze_deps",
        prompt="Identify dependencies between modules",
        depends_on=["map_structure"],
        depth=5
    )
    
    # Step 3: Find code smells
    workflow.add_step(
        "detect_issues",
        prompt="Identify code smells, anti-patterns, and security concerns",
        depends_on=["map_structure"],
        depth=6,
        parallel=True
    )
    
    # Step 4: Architecture recommendations
    workflow.add_step(
        "recommend",
        prompt="Provide architectural improvement recommendations",
        depends_on=["analyze_deps", "detect_issues"],
        depth=7
    )
    
    return workflow.execute()

result = analyze_repository("./my_project")
```

### Function-Level Reasoning

```python
def explain_function(code: str, context: str = ""):
    response = router.query(
        prompt=f"""
Analyze this function in detail:

```
{code}
```

Context: {context}

Provide:
1. Purpose and behavior
2. Time/space complexity
3. Edge cases and potential bugs
4. Suggested improvements
""",
        depth=6,
        consensus_threshold=0.96
    )
    
    return {
        "explanation": response.text,
        "confidence": response.consensus_score,
        "reasoning_steps": len(response.reasoning_trail)
    }

# Usage
code = """
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
"""

analysis = explain_function(code, "Part of a dynamic programming tutorial")
print(analysis["explanation"])
```

## Configuration Patterns

### Multi-Node Heterogeneous Mesh

```yaml
mesh:
  name: production-reasoning-mesh
  consensus_threshold: 0.97
  
  nodes:
    # Fast local model for quick reasoning
    - type: local
      name: quick-node-1
      engine: ollama
      model: llama3.1:8b
      max_thinking_depth: 3
      capabilities:
        - simple_queries
        - factual_lookup
      priority: high
      
    # Medium depth reasoning
    - type: local
      name: medium-node-1
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 5
      capabilities:
        - code_analysis
        - logical_reasoning
      priority: medium
      
    # Deep reasoning node
    - type: local
      name: deep-node-1
      engine: vllm
      model: opus-4.8
      max_thinking_depth: 7
      capabilities:
        - complex_reasoning
        - multi_step_planning
        - research_synthesis
      priority: low
      gpu_memory_gb: 40
      
  routing:
    strategy: capability_aware
    load_balancing: true
    auto_escalation: true  # Escalate to deeper nodes if consensus fails
    max_retries: 3
```

### Custom Reasoning Profiles

```python
from logos import Router, ReasoningProfile

# Define custom profile
profile = ReasoningProfile(
    name="security-audit",
    default_depth=7,
    consensus_threshold=0.98,
    required_capabilities=["code_analysis", "security_reasoning"],
    verification_steps=[
        "check_input_validation",
        "check_auth_implementation",
        "check_data_exposure",
        "check_crypto_usage"
    ],
    synthesis_strategy="conservative"  # Prefer false positives over false negatives
)

router = Router(config="mesh_config.yaml")
router.register_profile(profile)

# Use profile
response = router.query(
    prompt="Audit this authentication handler for security issues: {code}",
    profile="security-audit",
    variables={"code": auth_code}
)
```

## Monitoring and Debugging

### Inspect Reasoning Trail

```python
from logos import Router

router = Router(config="mesh_config.yaml")
response = router.query(
    prompt="Design a rate limiting algorithm",
    depth=6,
    trace=True  # Enable detailed tracing
)

# Examine the reasoning DAG
trail = response.reasoning_trail

print("=== Reasoning Trail ===")
for step in trail:
    print(f"\nStep {step.index} (Node: {step.node_id})")
    print(f"Prompt fragment: {step.prompt_fragment}")
    print(f"Conclusion: {step.conclusion}")
    print(f"Confidence: {step.confidence}")
    print(f"Verifying nodes: {step.verifying_nodes}")
    print(f"Consensus score: {step.consensus_score}")
    
    if step.conflicts:
        print(f"⚠ Conflicts detected:")
        for conflict in step.conflicts:
            print(f"  - {conflict.node_id}: {conflict.alternative_conclusion}")
            print(f"    Arbitration: {conflict.resolution}")
```

### Performance Metrics

```python
# Get mesh statistics
stats = router.get_mesh_stats()

print(f"Active nodes: {stats.active_nodes}")
print(f"Total queries: {stats.total_queries}")
print(f"Average latency: {stats.avg_latency_ms}ms")
print(f"Average depth: {stats.avg_depth}")
print(f"Consensus failures: {stats.consensus_failures}")
print(f"Auto-escalations: {stats.auto_escalations}")

# Per-node statistics
for node_id, node_stats in stats.nodes.items():
    print(f"\nNode {node_id}:")
    print(f"  Queries handled: {node_stats.queries_handled}")
    print(f"  Avg latency: {node_stats.avg_latency_ms}ms")
    print(f"  Success rate: {node_stats.success_rate}%")
    print(f"  Current load: {node_stats.current_load}%")
```

### Enable Debug Logging

```python
import logging
from logos import Router

# Configure detailed logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('logos_debug.log'),
        logging.StreamHandler()
    ]
)

router = Router(config="mesh_config.yaml", log_level="DEBUG")

# Logs will show:
# - Node selection decisions
# - Consensus calculations
# - Verification attempts
# - Escalation triggers
```

## Troubleshooting

### Consensus Failures

```python
from logos import Router
from logos.exceptions import ConsensusFailure

router = Router(config="mesh_config.yaml")

try:
    response = router.query(
        prompt="Your query here",
        depth=6,
        consensus_threshold=0.95,
        max_consensus_attempts=5
    )
except ConsensusFailure as e:
    print(f"Consensus failed after {e.attempts} attempts")
    print(f"Best consensus score achieved: {e.best_score}")
    print(f"Conflicting conclusions:")
    for node_id, conclusion in e.conflicting_conclusions.items():
        print(f"  {node_id}: {conclusion}")
    
    # Retry with lower threshold or different nodes
    response = router.query(
        prompt="Your query here",
        depth=5,
        consensus_threshold=0.90,
        exclude_nodes=e.problematic_nodes
    )
```

### Node Unavailability

```yaml
# Configure fallback behavior in mesh_config.yaml
mesh:
  fallback:
    enabled: true
    min_nodes: 2  # Minimum nodes required for consensus
    graceful_degradation: true
    reduced_depth_on_failure: true
    
  health_check:
    interval_seconds: 30
    timeout_seconds: 5
    auto_remove_unhealthy: true
    retry_after_seconds: 300
```

```python
# Check mesh health before critical operations
if router.mesh_healthy(min_nodes=3):
    response = router.query(prompt="...", depth=7)
else:
    print("Insufficient nodes available")
    # Use fallback single-node inference
    response = router.query_single_node(prompt="...", node_id="quick-node-1")
```

### Memory Issues

```python
# Configure memory limits per node
from logos import Router, NodeConfig

node_config = NodeConfig(
    max_concurrent_queries=5,
    max_reasoning_memory_mb=8192,
    context_window_tokens=16000,
    auto_garbage_collect=True
)

router = Router(
    config="mesh_config.yaml",
    node_defaults=node_config
)

# Force cleanup between large queries
response1 = router.query(prompt="Large query 1", depth=7)
router.cleanup_node_memory()
response2 = router.query(prompt="Large query 2", depth=7)
```

### Low Consensus Scores

```python
# Investigate low consensus
response = router.query(
    prompt="Your query",
    depth=6,
    return_alternatives=True  # Get all node conclusions
)

if response.consensus_score < 0.90:
    print("Low consensus detected:")
    for alt in response.alternatives:
        print(f"\nNode {alt.node_id} (confidence: {alt.confidence}):")
        print(alt.conclusion)
    
    # Try with more specific prompt or different capabilities
    response = router.query(
        prompt="More specific query",
        depth=5,
        required_capabilities=["factual_verification"]
    )
```

## REST API Usage

```python
# Start API server
from logos import Router

router = Router(config="mesh_config.yaml")
app = router.create_api()

# Run server
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8080)
```

```bash
# Query via REST
curl -X POST http://localhost:8080/query \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain bubble sort complexity",
    "depth": 5,
    "language": "en",
    "consensus_threshold": 0.95
  }'

# Get mesh status
curl http://localhost:8080/mesh/status

# Get reasoning trail
curl http://localhost:8080/query/{query_id}/trail
```

## Best Practices

1. **Start with lower consensus thresholds (0.90) during development**, increase to 0.95+ for production
2. **Use adaptive depth** for most queries, reserve fixed high depth (6-7) for critical reasoning
3. **Configure at least 2-3 nodes** for meaningful consensus validation
4. **Enable persistence** to replay reasoning trails during debugging
5. **Monitor consensus failure rates** - sustained failures indicate misconfigured nodes or incompatible models
6. **Use workflow abstractions** for multi-step reasoning instead of chaining queries manually
7. **Set appropriate timeouts** - deep reasoning (depth 7) can take 10-30s per step
8. **Leverage capability-aware routing** - tag nodes with their strengths for better assignments
