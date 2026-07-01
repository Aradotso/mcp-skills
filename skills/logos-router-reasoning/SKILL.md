---
name: logos-router-reasoning
description: Use Logos Router for distributed semantic reasoning with zero-drift consensus and adaptive depth control across local AI nodes
triggers:
  - set up logos router for distributed reasoning
  - configure reasoning mesh with strict write discipline
  - create adaptive depth reasoning workflow
  - implement zero-drift consensus protocol
  - route AI queries across reasoning nodes
  - build multi-step reasoning pipeline with logos
  - configure semantic caching for reasoning
  - troubleshoot logos router mesh configuration
---

# Logos Router Reasoning Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Logos Router is a distributed semantic reasoning gateway that fragments AI reasoning across local nodes with zero-drift consensus guarantees. Instead of funneling all reasoning through a single model, it distributes subproblems across a mesh of reasoning units coordinated through a strict write discipline protocol. It eliminates reasoning "drift" through cross-validation and maintains causal audit trails for every routing decision.

**Key capabilities:**
- Adaptive reasoning depth (1-7 steps based on problem complexity)
- Multilingual semantic routing (16 languages)
- Strict Write Discipline (SWD) for zero-drift guarantees
- 24/7 reasoning continuity with graceful degradation
- Local infrastructure with optional mesh bridging

## Installation

### Prerequisites

```bash
# Python 3.11+ required
python --version

# At least one inference engine required:
# - VLLM
# - Ollama
# - llama.cpp
# - TensorRT-LLM
```

### Install Logos Router

```bash
# Clone the repository
git clone https://github.com/rak7777/mythic-mcp-proxy.git
cd mythic-mcp-proxy

# Install dependencies
pip install -r requirements.txt

# Install logos router
pip install -e .
```

### Verify Installation

```python
from logos import Router, __version__

print(f"Logos Router version: {__version__}")
```

## Configuration

### Basic Mesh Configuration

Create a `mesh_config.yaml` file:

```yaml
mesh:
  name: development-lattice
  consensus_threshold: 0.95  # Confidence required before write
  max_concurrent_threads: 8
  
  nodes:
    - name: primary-reasoner
      type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 7
      gpu_memory: 16GB
      
    - name: verifier-node
      type: local
      engine: ollama
      model: llama3.1-8b
      max_thinking_depth: 5
      role: verification
      
  languages:
    - en
    - es
    - zh
    - fr
    
  cache:
    semantic_cache_enabled: true
    cache_ttl: 3600
    max_cache_size_mb: 2048
    
  monitoring:
    audit_trail: true
    log_reasoning_steps: true
    drift_detection: true
```

### Advanced Consensus Configuration

```yaml
mesh:
  name: production-lattice
  consensus_threshold: 0.97
  escalation_threshold: 0.85
  
  nodes:
    - name: fast-reasoner
      type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 4
      priority: high
      
    - name: deep-reasoner
      type: local
      engine: vllm
      model: opus-full-4.8
      max_thinking_depth: 7
      priority: escalation
      
    - name: specialist-math
      type: local
      engine: ollama
      model: deepseek-math
      max_thinking_depth: 6
      specialty: mathematical
      
  strict_write_discipline:
    enabled: true
    min_peer_confirmations: 2
    timeout_seconds: 30
    fallback_mode: graceful_degradation
```

## Core API Usage

### Basic Query Routing

```python
from logos import Router

# Initialize router with config
router = Router(config="mesh_config.yaml")

# Simple query with adaptive depth
response = router.query(
    prompt="Explain the halting problem and its implications for AI safety",
    depth="adaptive",
    language="en"
)

print(response.text)
print(f"Reasoning depth: {response.depth}")
print(f"Nodes consulted: {response.nodes_involved}")
print(f"Consensus score: {response.consensus_score}")
```

### Explicit Depth Control

```python
# Force shallow reasoning (fast)
quick_response = router.query(
    prompt="What is 2 + 2?",
    depth=1,
    language="en"
)

# Force deep reasoning (thorough)
deep_response = router.query(
    prompt="Analyze the ethical implications of AGI alignment strategies",
    depth=7,
    language="en",
    require_consensus=True
)

# Let router decide
adaptive_response = router.query(
    prompt="How does TCP congestion control work?",
    depth="adaptive"
)
```

### Multilingual Reasoning

```python
# Query in Spanish, get response in Spanish
spanish_response = router.query(
    prompt="¿Cuáles son las implicaciones de la computación cuántica?",
    language="es",
    depth="adaptive"
)

# Query in English, get response in Mandarin
cross_lingual = router.query(
    prompt="Explain quantum entanglement",
    language="en",
    response_language="zh"
)

# Auto-detect input language
auto_response = router.query(
    prompt="Comment fonctionne l'apprentissage par renforcement?",
    language="auto"
)
```

## Multi-Step Workflows

### Workflow API

```python
from logos import Router

router = Router(config="mesh_config.yaml")

# Create a workflow for complex reasoning
workflow = router.create_workflow(
    name="code_review_analysis",
    description="Analyze code changes for correctness and style"
)

# Add sequential steps
workflow.add_step(
    name="extract_changes",
    prompt="Extract all modified functions from: {input_file}",
    depth=3
)

workflow.add_step(
    name="analyze_logic",
    prompt="Verify logic correctness of: {extract_changes.output}",
    depth=5,
    require_consensus=True
)

workflow.add_step(
    name="check_style",
    prompt="Check code style violations in: {extract_changes.output}",
    depth=2
)

workflow.add_step(
    name="synthesize_report",
    prompt="Create summary report from: {analyze_logic.output} and {check_style.output}",
    depth=4
)

# Execute workflow
result = workflow.execute(input_file="src/main.py")
print(result.final_output)
print(f"Total reasoning depth: {result.total_depth}")
print(f"Steps completed: {result.steps_completed}")
```

### Document Analysis Workflow

```python
# Analyze research paper with verification
task = router.create_workflow("research_analysis")

task.extract_claims(
    document="paper.pdf",
    depth=4
)

task.verify_claims(
    consensus=True,
    min_confidence=0.95,
    depth=6
)

task.cross_reference(
    databases=["arxiv", "pubmed"],
    depth=5
)

task.generate_summary(
    style="academic",
    language="en",
    depth=4
)

result = task.execute()
print(result.summary)
print(f"Claims verified: {result.verified_claims}")
print(f"Contradictions found: {result.contradictions}")
```

## Strict Write Discipline Protocol

### Understanding SWD

```python
from logos import Router, StrictWriteConfig

# Configure strict write discipline
swd_config = StrictWriteConfig(
    enabled=True,
    min_peer_confirmations=2,
    consensus_threshold=0.95,
    timeout_seconds=30,
    escalation_on_failure=True
)

router = Router(
    config="mesh_config.yaml",
    swd_config=swd_config
)

# Query with SWD enabled
response = router.query(
    prompt="Design a distributed consensus algorithm",
    depth="adaptive",
    strict_write=True
)

# Inspect verification trail
for step in response.reasoning_trail:
    print(f"Step {step.index}: {step.text}")
    print(f"  Verified by nodes: {step.verifying_nodes}")
    print(f"  Consensus score: {step.consensus_score}")
    print(f"  Escalated: {step.escalated}")
```

### Handling Consensus Failures

```python
from logos.exceptions import ConsensusFailure

try:
    response = router.query(
        prompt="Complex ambiguous query",
        depth=7,
        strict_write=True,
        require_consensus=True
    )
except ConsensusFailure as e:
    print(f"Consensus failed at step: {e.failed_step}")
    print(f"Divergent outputs: {e.divergent_outputs}")
    print(f"Confidence scores: {e.confidence_scores}")
    
    # Retry with higher depth
    response = router.query(
        prompt=e.original_prompt,
        depth=7,
        escalate=True
    )
```

## Semantic Caching

### Enable and Configure Caching

```python
from logos import Router, CacheConfig

cache_config = CacheConfig(
    enabled=True,
    ttl=3600,  # 1 hour
    max_size_mb=2048,
    similarity_threshold=0.92
)

router = Router(
    config="mesh_config.yaml",
    cache_config=cache_config
)

# First query (cache miss)
response1 = router.query(
    prompt="What is machine learning?",
    depth="adaptive"
)
print(f"Cache hit: {response1.cache_hit}")  # False

# Similar query (cache hit)
response2 = router.query(
    prompt="Explain machine learning concepts",
    depth="adaptive"
)
print(f"Cache hit: {response2.cache_hit}")  # True
print(f"Latency reduction: {response2.latency_ms}ms vs {response1.latency_ms}ms")
```

### Cache Management

```python
# Clear specific cache entries
router.cache.invalidate(pattern="machine learning")

# Get cache statistics
stats = router.cache.get_stats()
print(f"Hit rate: {stats.hit_rate}")
print(f"Total entries: {stats.entry_count}")
print(f"Memory usage: {stats.memory_mb}MB")

# Flush entire cache
router.cache.flush()
```

## Monitoring and Audit Trails

### Inspect Reasoning Chains

```python
response = router.query(
    prompt="Prove that P != NP is undecidable in current systems",
    depth=7,
    audit=True
)

# Access full reasoning DAG
reasoning_dag = response.reasoning_dag

for node in reasoning_dag.nodes:
    print(f"Node: {node.id}")
    print(f"  Depth: {node.depth}")
    print(f"  Handler: {node.handling_node}")
    print(f"  Verified by: {node.verifiers}")
    print(f"  Children: {node.children}")
    print(f"  Timestamp: {node.timestamp}")
```

### Drift Detection

```python
from logos import DriftMonitor

monitor = DriftMonitor(router)

# Run drift analysis
drift_report = monitor.analyze_drift(
    test_queries=["What is recursion?", "Explain loops"],
    iterations=100,
    baseline_config="mesh_config.yaml"
)

print(f"Mean drift: {drift_report.mean_drift}")
print(f"Max drift: {drift_report.max_drift}")
print(f"Drift variance: {drift_report.variance}")

# Drift should be < 0.01% with SWD enabled
assert drift_report.mean_drift < 0.0001
```

## Real-Time Streaming

### WebSocket Streaming

```python
from logos import Router
import asyncio

router = Router(config="mesh_config.yaml")

async def stream_reasoning():
    async for chunk in router.query_stream(
        prompt="Explain quantum computing step by step",
        depth=7,
        stream_reasoning=True
    ):
        print(f"[{chunk.node_id}] Depth {chunk.depth}: {chunk.text}")
        print(f"  Confidence: {chunk.confidence}")

asyncio.run(stream_reasoning())
```

### REST API Streaming

```python
import requests

response = requests.post(
    "http://localhost:8080/v1/query/stream",
    json={
        "prompt": "Analyze this code for security issues",
        "depth": "adaptive",
        "language": "en",
        "stream": True
    },
    stream=True
)

for line in response.iter_lines():
    if line:
        chunk = json.loads(line)
        print(f"Step: {chunk['reasoning_step']}")
        print(f"Text: {chunk['text']}")
```

## CLI Usage

### Start Router Server

```bash
# Start with default config
logos serve --config mesh_config.yaml --port 8080

# Start with specific nodes
logos serve --config mesh_config.yaml --nodes primary-reasoner,verifier-node

# Enable debug logging
logos serve --config mesh_config.yaml --log-level debug --audit-trail
```

### Query from CLI

```bash
# Simple query
logos query "What is the halting problem?" --depth adaptive

# Multilingual query
logos query "¿Qué es la computación cuántica?" --language es --depth 5

# With consensus requirement
logos query "Prove the Riemann hypothesis" --depth 7 --require-consensus

# Output as JSON
logos query "Explain TCP/IP" --format json --output result.json
```

### Manage Mesh

```bash
# List active nodes
logos mesh list

# Add node dynamically
logos mesh add --name backup-node --engine ollama --model llama3.1-8b

# Remove node
logos mesh remove --name backup-node

# Get mesh status
logos mesh status --verbose
```

### Cache Management

```bash
# View cache stats
logos cache stats

# Clear cache
logos cache clear

# Preload common queries
logos cache preload --queries queries.txt
```

## Common Patterns

### Code Review Assistant

```python
from logos import Router

router = Router(config="mesh_config.yaml")

def review_code(file_path: str) -> dict:
    """Comprehensive code review with consensus verification."""
    
    with open(file_path, 'r') as f:
        code = f.read()
    
    # Multi-step review workflow
    workflow = router.create_workflow("code_review")
    
    # Step 1: Syntax and logic
    workflow.add_step(
        name="analyze_logic",
        prompt=f"Review this code for logic errors:\n\n{code}",
        depth=5,
        require_consensus=True
    )
    
    # Step 2: Security analysis
    workflow.add_step(
        name="security_scan",
        prompt=f"Identify security vulnerabilities:\n\n{code}",
        depth=6,
        require_consensus=True
    )
    
    # Step 3: Performance issues
    workflow.add_step(
        name="performance_check",
        prompt=f"Find performance bottlenecks:\n\n{code}",
        depth=4
    )
    
    # Step 4: Best practices
    workflow.add_step(
        name="style_check",
        prompt=f"Check adherence to best practices:\n\n{code}",
        depth=3
    )
    
    result = workflow.execute()
    
    return {
        "logic_issues": result.analyze_logic.output,
        "security_issues": result.security_scan.output,
        "performance_issues": result.performance_check.output,
        "style_issues": result.style_check.output,
        "consensus_scores": result.consensus_scores
    }

# Use it
review = review_code("src/app.py")
print(f"Found {len(review['security_issues'])} security issues")
```

### Research Assistant

```python
def analyze_research_paper(pdf_path: str, target_lang: str = "en") -> dict:
    """Analyze research paper with multi-lingual support."""
    
    router = Router(config="mesh_config.yaml")
    
    workflow = router.create_workflow("research_analysis")
    
    # Extract and translate claims
    workflow.add_step(
        name="extract_claims",
        prompt=f"Extract all scientific claims from: {pdf_path}",
        depth=5,
        language="auto"
    )
    
    # Verify against known facts
    workflow.add_step(
        name="fact_check",
        prompt="Verify these claims: {extract_claims.output}",
        depth=7,
        require_consensus=True
    )
    
    # Identify novel contributions
    workflow.add_step(
        name="identify_novelty",
        prompt="What novel contributions are in: {extract_claims.output}?",
        depth=6
    )
    
    # Generate summary
    workflow.add_step(
        name="summarize",
        prompt="Summarize findings from all previous steps",
        depth=4,
        language=target_lang
    )
    
    result = workflow.execute()
    
    return {
        "claims": result.extract_claims.output,
        "verified_claims": result.fact_check.output,
        "novel_contributions": result.identify_novelty.output,
        "summary": result.summarize.output,
        "reasoning_depth": result.total_depth
    }
```

### Autonomous Agent Integration

```python
from logos import Router

class AutonomousAgent:
    def __init__(self, config_path: str):
        self.router = Router(config=config_path)
        self.memory = []
    
    def think(self, observation: str, depth: str = "adaptive") -> str:
        """Process observation and decide action."""
        
        # Build context from memory
        context = "\n".join(self.memory[-5:])  # Last 5 steps
        
        prompt = f"""
        Previous context:
        {context}
        
        Current observation:
        {observation}
        
        What should I do next?
        """
        
        response = self.router.query(
            prompt=prompt,
            depth=depth,
            require_consensus=True
        )
        
        # Store in memory with reasoning trail
        self.memory.append({
            "observation": observation,
            "action": response.text,
            "reasoning": response.reasoning_trail,
            "confidence": response.consensus_score
        })
        
        return response.text
    
    def reflect(self) -> dict:
        """Analyze recent decision quality."""
        
        recent_decisions = self.memory[-10:]
        
        reflection = self.router.query(
            prompt=f"Analyze these decisions for quality:\n{recent_decisions}",
            depth=7,
            require_consensus=True
        )
        
        return {
            "analysis": reflection.text,
            "avg_confidence": sum(d["confidence"] for d in recent_decisions) / len(recent_decisions),
            "reasoning_depth": reflection.depth
        }

# Use the agent
agent = AutonomousAgent("mesh_config.yaml")
action = agent.think("User requested file deletion")
print(f"Agent decided: {action}")

# Periodic reflection
if len(agent.memory) >= 10:
    reflection = agent.reflect()
    print(f"Decision quality: {reflection['avg_confidence']}")
```

## Troubleshooting

### Node Connection Failures

```python
from logos import Router
from logos.exceptions import NodeConnectionError

router = Router(config="mesh_config.yaml")

try:
    response = router.query("Test query", depth=3)
except NodeConnectionError as e:
    print(f"Failed to connect to node: {e.node_name}")
    print(f"Error: {e.error_message}")
    
    # Fallback to available nodes
    available_nodes = router.mesh.get_healthy_nodes()
    print(f"Available nodes: {available_nodes}")
    
    # Retry with reduced node requirements
    response = router.query(
        "Test query",
        depth=3,
        min_nodes=1  # Reduce from default 2
    )
```

### Consensus Timeout Issues

```python
from logos import Router, TimeoutConfig

# Increase timeout for complex queries
timeout_config = TimeoutConfig(
    query_timeout=60,  # 60 seconds
    consensus_timeout=45,
    node_response_timeout=30
)

router = Router(
    config="mesh_config.yaml",
    timeout_config=timeout_config
)

response = router.query(
    prompt="Complex multi-step reasoning task",
    depth=7,
    require_consensus=True
)
```

### Memory Issues

```bash
# Check node memory usage
logos mesh stats --memory

# Reduce concurrent threads
logos config set mesh.max_concurrent_threads 4

# Enable memory monitoring
logos serve --config mesh_config.yaml --memory-limit 8GB --auto-gc
```

### Low Consensus Scores

```python
# Debug consensus failures
response = router.query(
    prompt="Ambiguous question",
    depth=7,
    debug_consensus=True
)

print(f"Consensus score: {response.consensus_score}")

for step in response.reasoning_trail:
    if step.consensus_score < 0.9:
        print(f"Low consensus at step {step.index}:")
        print(f"  Node outputs: {step.node_outputs}")
        print(f"  Divergence: {step.divergence_analysis}")
```

### Drift Detection

```python
from logos import Router, DriftMonitor

router = Router(config="mesh_config.yaml")
monitor = DriftMonitor(router)

# Monitor for drift in production
def check_drift_periodically():
    baseline_queries = [
        "What is 2+2?",
        "Define recursion",
        "Explain HTTP"
    ]
    
    report = monitor.analyze_drift(
        test_queries=baseline_queries,
        iterations=50
    )
    
    if report.mean_drift > 0.01:  # 1% threshold
        print("WARNING: Drift detected!")
        print(f"Mean drift: {report.mean_drift}")
        print("Consider recalibrating mesh nodes")
        
        # Auto-recalibrate
        router.mesh.recalibrate()

# Run periodically
import schedule
schedule.every(6).hours.do(check_drift_periodically)
```

### Performance Optimization

```yaml
# Optimized config for throughput
mesh:
  name: high-throughput-lattice
  consensus_threshold: 0.92  # Slightly lower for speed
  max_concurrent_threads: 16
  
  nodes:
    - name: fast-node-1
      type: local
      engine: vllm
      model: opus-mini-4.8
      max_thinking_depth: 4  # Cap depth
      batch_size: 8
      
  cache:
    semantic_cache_enabled: true
    cache_ttl: 7200
    aggressive_caching: true
    
  optimizations:
    early_stopping: true
    speculative_execution: true
    parallel_verification: true
```

```python
# Use optimized config
router = Router(config="optimized_config.yaml")

# Batch queries for efficiency
queries = [
    "Query 1",
    "Query 2", 
    "Query 3"
]

responses = router.query_batch(
    queries,
    depth=3,
    parallel=True,
    max_workers=8
)

for i, response in enumerate(responses):
    print(f"Response {i}: {response.text}")
```

## Environment Variables

```bash
# Core settings
export LOGOS_CONFIG_PATH="/path/to/mesh_config.yaml"
export LOGOS_LOG_LEVEL="info"  # debug, info, warning, error

# Node configuration
export LOGOS_NODE_GPU_MEMORY="16GB"
export LOGOS_NODE_MAX_DEPTH="7"

# Cache settings
export LOGOS_CACHE_ENABLED="true"
export LOGOS_CACHE_TTL="3600"
export LOGOS_CACHE_SIZE_MB="2048"

# Consensus settings
export LOGOS_CONSENSUS_THRESHOLD="0.95"
export LOGOS_MIN_PEER_CONFIRMATIONS="2"

# API server
export LOGOS_API_PORT="8080"
export LOGOS_API_HOST="0.0.0.0"
export LOGOS_API_KEY="${LOGOS_API_KEY}"  # For authentication
```

## Best Practices

1. **Start with adaptive depth** - Let the router decide complexity
2. **Enable consensus for critical tasks** - Use `require_consensus=True` for high-stakes reasoning
3. **Monitor drift regularly** - Set up periodic drift checks
4. **Use workflows for multi-step tasks** - Better than chaining individual queries
5. **Enable semantic caching** - Significant performance gains for repeated patterns
6. **Configure appropriate timeouts** - Match timeouts to query complexity
7. **Maintain diverse node pool** - Mix of fast and deep reasoners
8. **Audit reasoning trails** - Review trails for important decisions
9. **Set up graceful degradation** - Ensure system works even with node failures
10. **Use language auto-detection** - More flexible than hardcoding languages
