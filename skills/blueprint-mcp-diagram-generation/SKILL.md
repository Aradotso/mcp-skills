---
name: blueprint-mcp-diagram-generation
description: Generate architecture diagrams, flowcharts, and system visualizations using Nano Banana Pro through Arcade MCP
triggers:
  - create an architecture diagram
  - generate a system diagram
  - visualize this codebase structure
  - draw a flowchart for this process
  - make a sequence diagram
  - show me a data flow diagram
  - create a visual architecture overview
  - diagram the system design
---

# Blueprint MCP Diagram Generation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Blueprint MCP enables AI agents to generate professional diagrams for understanding codebases, system architecture, data flows, and processes. It uses Nano Banana Pro's diagram generation capabilities through the Arcade MCP ecosystem.

## What It Does

Blueprint MCP provides three core tools for diagram generation:
- **start_diagram_job**: Initiates diagram generation, returns a job ID
- **check_job_status**: Polls job completion status
- **download_diagram**: Retrieves the generated diagram as base64-encoded PNG

Works seamlessly with other Arcade MCP servers (GitHub, HubSpot, Google Drive, Slack) to extract data and visualize it as diagrams.

## Installation

### Prerequisites

1. **Arcade Account**: Sign up at https://arcade.dev
2. **Google AI Studio API Key**: Get from https://aistudio.google.com/

### Setup Steps

```bash
# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install Arcade CLI
pip install arcade-mcp

# Login to Arcade
arcade-mcp login

# Store Google API key as secret
arcade-mcp secret set GOOGLE_API_KEY="your_api_key_here"

# Clone and deploy Blueprint MCP
git clone https://github.com/ArcadeAI/blueprint-mcp.git
cd blueprint-mcp
arcade-mcp deploy
```

### Configure Gateway

1. Go to https://api.arcade.dev/dashboard
2. Navigate to "Gateways" → "Create Gateway"
3. Add your deployed `architect_mcp` server
4. Copy the gateway URL

### Add to Your IDE

**Cursor/Claude Desktop:**
```json
{
  "mcpServers": {
    "arcade-gateway": {
      "url": "https://api.arcade.dev/gateway/YOUR_GATEWAY_ID"
    }
  }
}
```

## Core Workflow

Blueprint MCP uses an asynchronous job-based workflow:

```
1. start_diagram_job → returns job_id
2. wait ~30 seconds (Nano Banana Pro generates)
3. check_job_status → returns "Complete" when ready
4. download_diagram → returns base64 PNG data
5. decode and save to workspace
```

## Tool Reference

### start_diagram_job

Initiates diagram generation with a detailed prompt.

**Parameters:**
- `prompt` (string, required): Detailed description of the diagram to generate

**Returns:**
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "pending"
}
```

**Example:**
```python
# Agent initiates diagram generation
job = start_diagram_job(
    prompt="Create architecture diagram showing microservices: API Gateway connecting to Auth Service, Order Service, and Payment Service. Auth Service connects to User DB. Order Service connects to Order DB and Inventory Service. Payment Service connects to Payment DB and External Payment Provider. Use technical whiteboard style, muted colors, 16:9."
)
# Returns: {"job_id": "abc-123", "status": "pending"}
```

### check_job_status

Polls the status of a diagram generation job.

**Parameters:**
- `job_id` (string, required): Job ID from start_diagram_job

**Returns:**
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "Complete"
}
```

**Possible statuses:** `"pending"`, `"processing"`, `"Complete"`, `"failed"`

**Example:**
```python
# Check job status after waiting
status = check_job_status(job_id="abc-123")
# Returns: {"job_id": "abc-123", "status": "Complete"}
```

### download_diagram

Downloads the completed diagram as base64-encoded PNG.

**Parameters:**
- `job_id` (string, required): Job ID from start_diagram_job

**Returns:**
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "image_base64": "iVBORw0KGgoAAAANSUhEUgAA..."
}
```

**Example:**
```python
# Download completed diagram
result = download_diagram(job_id="abc-123")
base64_data = result["image_base64"]

# Decode and save (agent handles this)
import base64
image_data = base64.b64decode(base64_data)
with open("architecture_diagram.png", "wb") as f:
    f.write(image_data)
```

## Effective Prompt Engineering

### Architecture Diagrams

**Good prompt structure:**
```
Create [diagram type] with [number] layers/sections:
- LAYER 1: [Component names and purpose]
- LAYER 2: [Component names and purpose]
- Show [specific connections] with labeled arrows
- Use [style preference: technical/whiteboard/UML]
- Colors: [muted/vibrant/monochrome]
- Aspect ratio: 16:9 or 4:3
```

**Example:**
```python
prompt = """Create microservices architecture diagram with 3 layers:
LAYER 1: Client Applications (Web App, Mobile App, Admin Dashboard)
LAYER 2: API Gateway with rate limiting, routing to 4 microservices
LAYER 3: Services (Auth Service → User DB, Product Service → Product DB, 
Order Service → Order DB + Message Queue, Payment Service → Payment Provider)
Show HTTP/REST connections with labeled arrows (JWT tokens, API calls, webhooks)
Use technical whiteboard style, muted blue/gray colors, monospace fonts, 16:9"""
```

### Sequence Diagrams

**Pattern:**
```
Create sequence diagram showing [flow name]:
1. [Actor] → [System]: [action/message]
2. [System] → [Component]: [action/message]
3. [Component] → [External]: [action/message]
Return flow showing responses
Include error handling path for [specific error]
```

**Example:**
```python
prompt = """Create sequence diagram for OAuth 2.0 login flow:
1. User → Frontend: Click "Login with Google"
2. Frontend → Auth Service: Initiate OAuth
3. Auth Service → Google: Authorization request (redirect)
4. Google → User: Login consent screen
5. User → Google: Approve consent
6. Google → Auth Service: Authorization code (callback)
7. Auth Service → Google: Exchange code for tokens
8. Google → Auth Service: Access token + Refresh token
9. Auth Service → Frontend: JWT session token
10. Frontend → User: Logged in dashboard
Show error path for token validation failure
Technical style, clear labels, 16:9"""
```

### Data Flow Diagrams

**Pattern:**
```
Create data flow diagram for [process]:
Sources: [list data sources]
Transformations: [list processing steps]
Destinations: [list outputs/storage]
Show data format at each step (JSON, CSV, Parquet, etc.)
Include data volume estimates if relevant
```

**Example:**
```python
prompt = """Create ETL pipeline data flow diagram:
SOURCES: Customer DB (PostgreSQL), Event Stream (Kafka), External API (REST)
TRANSFORMATIONS: 
- Ingestion Layer: Data validation, deduplication
- Processing Layer: PII masking, aggregation, enrichment
- Analytics Layer: Metrics calculation, ML feature engineering
DESTINATIONS: Data Warehouse (Snowflake), Analytics DB (ClickHouse), S3 Archive
Show data formats (JSON→Parquet→Delta), batch sizes, error handling paths
Technical style, muted colors, include retry logic, 16:9"""
```

### Flowcharts

**Pattern:**
```
Create flowchart for [process name]:
Start: [initial state]
Decision points: [list conditions with yes/no paths]
Actions: [list processing steps]
End states: [success/failure outcomes]
Include [specific edge cases or error handling]
```

**Example:**
```python
prompt = """Create payment processing flowchart:
START: Customer submits order
DECISION 1: Payment method? (Credit Card / PayPal / Crypto)
- Credit Card → Validate card → Decision: Valid?
  - Yes → Charge card → Decision: Successful?
    - Yes → Create order → Send confirmation → END (Success)
    - No → Retry (max 3) → Decision: Retry successful?
      - Yes → Create order → END (Success)
      - No → Refund holds → END (Payment Failed)
  - No → Show error → END (Invalid Card)
- PayPal → Redirect to PayPal → Similar flow
- Crypto → Generate wallet address → Wait for confirmation
Include timeout handling (15 min), fraud check step before charging
Clear yes/no labels, muted colors, 16:9"""
```

## Real-World Patterns

### Pattern 1: Analyze Codebase and Diagram

```python
# Agent workflow combining file reading + diagram generation

# Step 1: Analyze code structure
files = list_directory("src/services/")
# Returns: ["auth.py", "orders.py", "payments.py"]

# Step 2: Read key files
auth_code = read_file("src/services/auth.py")
order_code = read_file("src/services/orders.py")

# Step 3: Generate diagram based on code analysis
job = start_diagram_job(
    prompt=f"""Create architecture diagram of the services layer:
    - AuthService: {extract_classes_and_methods(auth_code)}
    - OrderService: {extract_classes_and_methods(order_code)}
    Show dependencies, database connections, and API endpoints
    Use class diagram style with methods and properties listed
    Technical whiteboard, monospace fonts, 16:9"""
)

# Step 4: Wait and download
time.sleep(30)
status = check_job_status(job_id=job["job_id"])
if status["status"] == "Complete":
    diagram = download_diagram(job_id=job["job_id"])
    save_base64_image(diagram["image_base64"], "services_architecture.png")
```

### Pattern 2: Multi-Tool Integration (HubSpot + Diagram)

```python
# Requires HubSpot MCP server in same Arcade gateway

# Step 1: Fetch data from HubSpot
deal = hubspot_get_deal(deal_id="12345")
# Returns: {"name": "Acme Corp Enterprise", "stage": "Proposal", ...}

# Step 2: Create solution diagram
job = start_diagram_job(
    prompt=f"""Create proposed solution architecture for {deal['name']}:
    CLIENT ENVIRONMENT: {deal['description']}
    PROPOSED SOLUTION:
    - Multi-region deployment (US-East, EU-West)
    - Load balancers → Application tier (3 instances)
    - Microservices: Auth, Data Processing, Analytics, Reporting
    - Databases: Primary PostgreSQL (replicated), Redis cache, S3 storage
    - Integration points: {deal['integration_requirements']}
    Show data flow, security zones (DMZ, internal, data), backup strategy
    Professional style suitable for client presentation, 16:9"""
)

# Step 3: Process and attach to deal
time.sleep(30)
status = check_job_status(job_id=job["job_id"])
if status["status"] == "Complete":
    diagram = download_diagram(job_id=job["job_id"])
    save_base64_image(diagram["image_base64"], "acme_solution_architecture.png")
    # Upload to HubSpot, Google Drive, etc.
```

### Pattern 3: Documentation Pipeline

```python
# Step 1: Read design doc
design_doc = read_file("docs/system_design.md")

# Step 2: Extract architectural decisions
sections = parse_markdown(design_doc)
# Returns: {"components": [...], "data_flow": [...], "deployment": [...]}

# Step 3: Generate multiple diagram types
diagrams_to_generate = [
    ("architecture", "high-level component architecture"),
    ("deployment", "production deployment topology"),
    ("data_flow", "data pipeline from ingestion to analytics")
]

for diagram_type, description in diagrams_to_generate:
    job = start_diagram_job(
        prompt=f"""Based on this design doc, create {description} diagram:
        {sections[diagram_type]}
        Use technical whiteboard style, muted colors, clear labels, 16:9"""
    )
    # Store job_id for later processing
    jobs.append((diagram_type, job["job_id"]))

# Step 4: Batch download all diagrams
time.sleep(35)
for diagram_type, job_id in jobs:
    status = check_job_status(job_id=job_id)
    if status["status"] == "Complete":
        result = download_diagram(job_id=job_id)
        save_base64_image(
            result["image_base64"], 
            f"docs/images/{diagram_type}_diagram.png"
        )
```

## Configuration

### Timing Considerations

- **Generation time**: ~30 seconds per diagram
- **Recommended wait**: 30-35 seconds before first status check
- **Polling interval**: Check every 5 seconds if not complete
- **Timeout**: Consider job failed after 2 minutes

### Style Preferences

Common style keywords for prompts:
- **Technical whiteboard**: Clean, professional, engineering-focused
- **UML standard**: Formal UML notation (class, sequence, component diagrams)
- **Hand-drawn**: Sketch-like appearance
- **Minimalist**: Simple boxes and arrows
- **Detailed**: Include properties, methods, data types

### Color Schemes

- **Muted**: `gray, light blue, purple, orange` (professional)
- **Vibrant**: `bright blue, green, red, yellow` (presentations)
- **Monochrome**: `black, white, gray` (printable)
- **Brand colors**: Specify hex codes if needed

## Troubleshooting

### Job Never Completes

**Symptom**: `check_job_status` returns `"pending"` indefinitely

**Solutions:**
```python
# Add timeout logic
import time
max_wait = 120  # 2 minutes
start_time = time.time()

while time.time() - start_time < max_wait:
    status = check_job_status(job_id=job_id)
    if status["status"] == "Complete":
        break
    elif status["status"] == "failed":
        print(f"Job failed: {status}")
        break
    time.sleep(5)
else:
    print("Job timeout - consider retrying")
```

### Diagram Quality Issues

**Problem**: Diagram is unclear or missing components

**Solution**: Improve prompt specificity:
```python
# Too vague
prompt = "Create architecture diagram for my app"

# Better - specific components and connections
prompt = """Create 3-tier architecture diagram:
PRESENTATION: React frontend (port 3000)
APPLICATION: Node.js API (port 8080) with Express, JWT auth
DATA: MongoDB (port 27017) + Redis cache (port 6379)
Show: HTTP requests, WebSocket connections, database queries
Include: Load balancer, CDN for static assets, backup database
Style: Technical whiteboard, boxes with labels, arrows with protocols
Aspect: 16:9, muted blue/gray colors"""
```

### Base64 Decoding Errors

**Problem**: Image data corruption when saving

**Solution:**
```python
import base64

# Ensure proper decoding
try:
    # Remove data URL prefix if present
    base64_data = result["image_base64"]
    if base64_data.startswith("data:image"):
        base64_data = base64_data.split(",")[1]
    
    # Decode and save
    image_bytes = base64.b64decode(base64_data)
    with open("diagram.png", "wb") as f:
        f.write(image_bytes)
except Exception as e:
    print(f"Decoding error: {e}")
```

### Authentication Issues

**Problem**: `GOOGLE_API_KEY` not found or invalid

**Solution:**
```bash
# Verify secret is stored
arcade-mcp secret list

# Re-set if missing
arcade-mcp secret set GOOGLE_API_KEY="your_new_key"

# Redeploy server
arcade-mcp deploy
```

## Best Practices

1. **Be specific**: Include component names, connection types, labels
2. **Specify style**: Technical whiteboard, UML, minimalist, etc.
3. **Set aspect ratio**: `16:9` for presentations, `4:3` for documents
4. **Use layers/sections**: Structure complex diagrams hierarchically
5. **Label connections**: HTTP, REST, WebSocket, database queries, etc.
6. **Include error paths**: Show retry logic, fallbacks, error handling
7. **Wait appropriately**: 30+ seconds before first status check
8. **Save intermediate results**: Store job IDs for batch processing
9. **Combine tools**: Use with GitHub, Google Drive, HubSpot for context
10. **Iterate prompts**: Refine based on output quality

## Integration Examples

### With GitHub MCP

```python
# Fetch repository structure
repo_files = github_list_files(repo="myorg/myapp", path="src/")

# Generate architecture from actual code
job = start_diagram_job(
    prompt=f"""Analyze repository structure and create architecture diagram:
    Files: {repo_files}
    Show module dependencies, entry points, and data flow
    Technical style, 16:9"""
)
```

### With Google Drive MCP

```python
# Read system design doc from Drive
doc_content = gdrive_read_file(file_id="doc_123")

# Generate diagrams from documentation
job = start_diagram_job(
    prompt=f"""Create architecture diagram from design doc:
    {doc_content}
    Extract components, connections, and deployment info
    Professional style for stakeholder review, 16:9"""
)
```

This skill enables AI agents to transform code analysis, documentation, and system data into visual diagrams that clarify architecture and facilitate understanding.
