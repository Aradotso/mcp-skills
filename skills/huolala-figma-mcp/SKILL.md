---
name: huolala-figma-mcp
description: MCP service that converts Figma designs into high-fidelity UI code with layout detection, asset slicing, and platform-specific output.
triggers:
  - convert figma design to code
  - set up figma to code mcp service
  - extract ui from figma design
  - generate html from figma file
  - configure huolala figma mcp
  - process figma design with mcp
  - export figma to react or vue
  - troubleshoot figma mcp service
---

# Huolala Figma MCP Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**Huolala Figma MCP** is an open-source MCP service that automatically converts Figma designs into high-fidelity UI code. Built on FastMCP, it exposes MCP tools to AI clients (Cursor, Claude, Qorder, etc.) and returns a ZIP package containing `index.html`, sliced images, fonts, and DSL data that can be further converted to React, Vue, Swift, Kotlin, React Native, and more.

Key features:
- **High fidelity**: Cleans redundant layers, recognizes red-dot badges, system bars, home bars
- **Advanced layout**: Distributed alignment, center alignment, linear layout, overlapping layout, list detection, auto text sizing
- **Extensible**: DSL processor pipeline with priority registry; custom Skills for platform-specific workflows
- **Visual fidelity**: Optional VLM multimodal support for complex UI recognition

## Installation

### Prerequisites
- Python 3.11 or higher
- Figma Personal Access Token ([get one here](https://www.figma.com/settings))

### Setup

```bash
# Clone the repository
git clone https://github.com/HuolalaTech/huolala-figma-mcp.git
cd huolala-figma-mcp

# Create virtual environment
python3.11 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -e .

# Configure Figma token
cp .env.example .env
# Edit .env and set MDAP_FIGMA_TOKEN=your_figma_token_here
```

### Start the MCP Service

```bash
python -m mdap_u2c --port=10001
```

The service will start on `http://localhost:10001`.

### Configure MCP Client

Add to your MCP client configuration (e.g., Cursor's MCP settings):

```json
{
  "mcpServers": {
    "ui2code-local": {
      "url": "http://localhost:10001/mcp",
      "transport": "http"
    }
  }
}
```

## Core MCP Tool

### `figma_to_code_package`

Converts a Figma design to a downloadable ZIP package with HTML, assets, and DSL.

**Parameters:**

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `figma_url` | string | Full Figma URL with `/file/{key}/` or `/design/{key}/` and `?node-id=` | Yes |
| `image_scale` | array | Image scale factors (1-4), e.g. `[3, 2]` for @3x and @2x | No (default: `[2]`) |
| `target_platform` | string | Target platform: `h5`, `vue`, `react`, `react native`, `ios`, `android` | No (default: `h5`) |

**Example usage in Python:**

```python
from mdap_u2c.services.ui2code_service import UI2CodeService

service = UI2CodeService()

result = service.figma_to_code_package(
    figma_url="https://www.figma.com/design/abc123/MyDesign?node-id=1-234",
    image_scale=[3, 2],
    target_platform="react"
)

print(f"Download ZIP: {result['zip_url']}")
print(f"Metadata: {result['metadata']}")
```

**Response structure:**

```json
{
  "zip_url": "http://localhost:10001/exports/abc123_1-234.zip",
  "metadata": {
    "file_key": "abc123",
    "node_id": "1-234",
    "image_scales": [3, 2],
    "target_platform": "react",
    "export_time": "2026-06-17T12:34:56Z"
  },
  "message": "zip_url can be downloaded directly"
}
```

**ZIP package contents:**

- `index.html` — Rule-based HTML for LLM secondary conversion
- `images/` — Sliced icons and image assets
- `fonts/` — Font files used in design
- `dsl.json` — Intermediate DSL data structure
- `design.png` — Original design screenshot
- `vlm_result.json` — (Optional) Multimodal LLM output if VLM is enabled

## Configuration

### Environment Variables

```bash
# Required: Figma API access
MDAP_FIGMA_TOKEN=figd_your_figma_personal_access_token

# Optional: VLM multimodal support (disabled by default)
MDAP_VLM_PROVIDER=openai
MDAP_VLM_API_KEY=sk-your_openai_api_key
MDAP_VLM_MODEL=gpt-4o
MDAP_VLM_BASE_URL=https://api.openai.com/v1

# Optional: Service port
MDAP_PORT=10001
```

### DSL Processor Pipeline

The DSL processing pipeline is configurable via processor priority. Edit `src/mdap_u2c/dsl_processors/` to enable/disable processors:

```python
from mdap_u2c.dsl_processors.registry import ProcessorRegistry

# Example: Disable VLM processor
ProcessorRegistry.unregister("vlm_processor")

# Example: Add custom processor
@ProcessorRegistry.register(priority=150)
class CustomProcessor:
    def process(self, dsl_data, context):
        # Custom processing logic
        return dsl_data
```

## Using Prompt Templates

Huolala Figma MCP includes two built-in prompt templates for structured workflows:

### 1. `get_figma_property`

Extract Figma design properties (text, styles, etc.) for partial UI updates.

**Template location:** `assets/prompts/get_figma_property.md`

**Usage in Cursor/Claude:**

```
/get_figma_property
```

Then provide:
- Figma URL
- Properties to extract (e.g., "all text content", "color palette", "spacing values")

### 2. `ui2code_with_skills`

Full UI-to-code workflow with platform-specific Skills.

**Template location:** `assets/prompts/ui2code_with_skills.md`

**Usage in Cursor/Claude:**

```
/ui2code_with_skills
```

Then provide:
- Figma URL
- Target language (React, Vue, Swift, etc.)
- Additional instructions

**Example workflow:**

1. Agent calls `figma_to_code_package` with your Figma URL
2. Downloads and extracts ZIP package
3. Loads platform-specific Skill from `assets/prompts/skills/`
4. Converts HTML to target framework using DSL and design context

## Code Examples

### Programmatic Usage

```python
from mdap_u2c.services.ui2code_service import UI2CodeService
from mdap_u2c.config import Config
import requests
import zipfile
import io

# Initialize service
config = Config()
service = UI2CodeService()

# Convert Figma design
figma_url = "https://www.figma.com/design/abc123/MyApp?node-id=42-1337"

result = service.figma_to_code_package(
    figma_url=figma_url,
    image_scale=[3, 2],
    target_platform="vue"
)

# Download and extract ZIP
response = requests.get(result["zip_url"])
with zipfile.ZipFile(io.BytesIO(response.content)) as z:
    z.extractall("./output")
    
    # Read DSL
    with z.open("dsl.json") as f:
        dsl_data = json.load(f)
        print(f"DSL root node: {dsl_data['type']}")
    
    # Read HTML
    with z.open("index.html") as f:
        html_content = f.read().decode("utf-8")
        print(f"HTML length: {len(html_content)} chars")
```

### Direct Figma API Integration

```python
from mdap_u2c.figma.client import FigmaClient
from mdap_u2c.figma.dsl_converter import FigmaDSLConverter

# Initialize Figma client
client = FigmaClient(token=config.figma_token)

# Fetch design
file_key = "abc123"
node_id = "42-1337"

file_data = client.get_file(file_key, node_ids=[node_id])
node_data = file_data["document"]["children"][0]

# Convert to DSL
converter = FigmaDSLConverter()
dsl_node = converter.convert_node(node_data)

print(f"Converted node: {dsl_node.type}, {dsl_node.width}x{dsl_node.height}")
```

### Custom DSL Processor

```python
from mdap_u2c.dsl_processors.registry import ProcessorRegistry
from mdap_u2c.dsl_processors.base import BaseProcessor

@ProcessorRegistry.register(priority=200)
class CustomBrandingProcessor(BaseProcessor):
    """Add custom branding to all text nodes."""
    
    def process(self, dsl_data, context):
        def add_branding(node):
            if node.get("type") == "text":
                text = node.get("text", "")
                node["text"] = f"[Brand] {text}"
            
            for child in node.get("children", []):
                add_branding(child)
        
        add_branding(dsl_data)
        return dsl_data
```

### Export Service Usage

```python
from mdap_u2c.services.export_service import ExportService

export_service = ExportService()

# Export DSL to ZIP
zip_path = export_service.export_to_zip(
    dsl_data=dsl_node,
    file_key="abc123",
    node_id="42-1337",
    image_scale=[3, 2]
)

print(f"Exported to: {zip_path}")
```

## Common Patterns

### Pattern 1: Batch Processing Multiple Designs

```python
from mdap_u2c.services.ui2code_service import UI2CodeService

service = UI2CodeService()

figma_urls = [
    "https://www.figma.com/design/abc/Page1?node-id=1-1",
    "https://www.figma.com/design/abc/Page2?node-id=2-1",
    "https://www.figma.com/design/abc/Page3?node-id=3-1",
]

for url in figma_urls:
    result = service.figma_to_code_package(
        figma_url=url,
        image_scale=[2],
        target_platform="react"
    )
    print(f"Processed: {result['zip_url']}")
```

### Pattern 2: Custom Platform Conversion

```python
# After getting ZIP with HTML and DSL:
import json
from pathlib import Path

# Load DSL
dsl_path = Path("output/dsl.json")
with open(dsl_path) as f:
    dsl = json.load(f)

# Load HTML as base
html_path = Path("output/index.html")
with open(html_path) as f:
    html_base = f.read()

# Use LLM to convert (pseudo-code for agent workflow)
prompt = f"""
Convert this HTML to SwiftUI.

Base HTML:
{html_base}

DSL structure:
{json.dumps(dsl, indent=2)}

Use DSL for accurate positioning and styles.
"""

# Agent generates SwiftUI code using both HTML and DSL context
```

### Pattern 3: Enable VLM for List Detection

```python
# Set environment variables before starting service
import os

os.environ["MDAP_VLM_PROVIDER"] = "openai"
os.environ["MDAP_VLM_API_KEY"] = "sk-your_key"
os.environ["MDAP_VLM_MODEL"] = "gpt-4o"

# Then start service
# python -m mdap_u2c --port=10001

# VLM will now detect and merge repeating list components
```

## Automated Testing

Run batch comparison tests to validate HTML output quality:

```bash
# Install test dependencies
pip install ".[ui2code-test]"
playwright install chromium

# Run automated tests
python tests/ui2code_auto_test.py
```

Configure test URLs in `tests/test_url_list.txt`:

```
https://www.figma.com/design/abc/Test1?node-id=1-1
https://www.figma.com/design/abc/Test2?node-id=2-1
```

Test output includes:
- Screenshots of generated HTML
- Visual similarity scores
- Detailed comparison reports

## Troubleshooting

### Issue: "Figma token invalid"

**Cause:** `MDAP_FIGMA_TOKEN` is missing or expired.

**Solution:**
1. Get a new token from [Figma Settings](https://www.figma.com/settings)
2. Update `.env`: `MDAP_FIGMA_TOKEN=figd_new_token`
3. Restart service: `python -m mdap_u2c --port=10001`

### Issue: "Node not found in Figma file"

**Cause:** Incorrect `node-id` in Figma URL or node was deleted.

**Solution:**
1. Open Figma design in browser
2. Right-click the frame/component → Copy link
3. Use the copied URL (contains correct `node-id`)

### Issue: ZIP package missing images

**Cause:** Image scale configuration or export failure.

**Solution:**
```python
# Try different image scales
result = service.figma_to_code_package(
    figma_url=url,
    image_scale=[3, 2, 1],  # Try multiple scales
    target_platform="h5"
)
```

### Issue: VLM not working

**Cause:** VLM environment variables not configured.

**Solution:**
```bash
# Check all VLM vars are set
echo $MDAP_VLM_PROVIDER
echo $MDAP_VLM_API_KEY
echo $MDAP_VLM_MODEL
echo $MDAP_VLM_BASE_URL

# If missing, add to .env and restart
```

### Issue: Layout detection incorrect

**Cause:** Complex overlapping elements or custom DSL processor issues.

**Solution:**
1. Check `dsl.json` in exported ZIP for structure issues
2. Review processor logs: `python -m mdap_u2c --port=10001 --debug`
3. Disable problematic processors in `src/mdap_u2c/dsl_processors/registry.py`
4. Adjust Figma design to reduce layer complexity

### Issue: Service fails to start

**Cause:** Port conflict or missing dependencies.

**Solution:**
```bash
# Use different port
python -m mdap_u2c --port=10002

# Reinstall dependencies
pip install -e . --force-reinstall

# Check Python version
python --version  # Must be 3.11+
```

## Project Structure Reference

```
huolala-figma-mcp/
├── src/mdap_u2c/
│   ├── figma/              # Figma API client and DSL conversion
│   ├── dsl_processors/     # DSL processing pipeline
│   │   ├── registry.py     # Processor registration
│   │   ├── layout.py       # Layout calculation
│   │   ├── vlm.py          # VLM integration
│   │   └── ...
│   ├── services/           # Business orchestration
│   │   ├── ui2code_service.py
│   │   └── export_service.py
│   ├── server/             # MCP/HTTP service
│   └── config/             # Configuration
├── assets/
│   ├── prompts/            # MCP prompt templates
│   │   ├── get_figma_property.md
│   │   ├── ui2code_with_skills.md
│   │   └── skills/         # Platform-specific conversion prompts
│   ├── fonts/              # Font assets
│   └── cv_templates/       # Computer vision templates
├── tests/
│   ├── ui2code_auto_test.py
│   └── test_url_list.txt
└── pyproject.toml
```

---

For full documentation, see the [GitHub repository](https://github.com/HuolalaTech/huolala-figma-mcp).
