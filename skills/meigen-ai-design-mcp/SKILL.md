---
name: meigen-ai-design-mcp
description: MCP server for AI image & video generation with 9 models (GPT Image 2, Nanobanana 2, Flux 2, Midjourney V8.1, Veo 3.1, local ComfyUI), 1,446 curated prompts, and parallel batch orchestration
triggers:
  - generate images with AI
  - create product photos or marketing visuals
  - search design inspiration prompts
  - set up AI image generation
  - generate videos from images
  - use ComfyUI workflows
  - enhance prompts for better images
  - batch generate image variations
---

# meigen-ai-design-mcp

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

MeiGen AI Design MCP is an open-source MCP server that transforms AI coding tools (Claude Code, Cursor, Windsurf, Roo Code, OpenClaw, etc.) into professional design assistants. It provides unified access to 9 leading image and video generation models, a curated library of 1,446 trending prompts, and parallel sub-agent orchestration for batch generation.

## What It Does

- **Multi-provider image generation**: MeiGen Cloud (9 models), OpenAI-compatible APIs, or local ComfyUI
- **Video generation**: Seedance 2.0, Happyhorse 1.0, Veo 3.1 via MeiGen platform
- **Curated prompt library**: 1,446 trending prompts from [nanobanana-trending-prompts](https://github.com/jau123/nanobanana-trending-prompts)
- **Smart prompt enhancement**: Transform brief ideas into professional image prompts
- **Parallel batch generation**: Sub-agents for efficient multi-variant creation
- **ComfyUI workflow management**: Import, modify, and run custom workflows
- **Standalone CLI mode**: Use without an MCP host for scripts and CI/CD

## Installation

### Claude Code (Recommended)

```bash
# Via MeiGen marketplace
/plugin marketplace add jau123/MeiGen-AI-Design-MCP
/plugin install meigen@meigen-marketplace

# Or via wshobson/agents (30k+ stars)
/plugin marketplace add wshobson/agents
/plugin install meigen-ai-design@claude-code-workflows
```

**Restart Claude Code** after installation.

### Other MCP-Compatible Hosts

Add to your MCP config (`.mcp.json`, `claude_desktop_config.json`, etc.):

```json
{
  "mcpServers": {
    "meigen": {
      "command": "npx",
      "args": ["-y", "meigen@1.3.1"],
      "env": {
        "MEIGEN_API_TOKEN": "${MEIGEN_API_TOKEN}"
      }
    }
  }
}
```

### Cursor / VS Code / Windsurf / Roo Code

```bash
npx meigen init cursor      # Cursor
npx meigen init vscode      # VS Code
npx meigen init windsurf    # Windsurf
npx meigen init roo         # Roo Code
```

### OpenClaw

```bash
# Full plugin (commands + skills + MCP)
openclaw bundles install clawhub:meigen-ai-design

# Or skill only
npx clawhub@latest install creative-toolkit
```

### Hermes Agent

Add to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  meigen:
    command: "npx"
    args: ["-y", "meigen@1.3.1"]
    env:
      MEIGEN_API_TOKEN: "${MEIGEN_API_TOKEN}"
    timeout: 600          # Video generation can take 5-10 min
    connect_timeout: 120  # First npx download needs time
```

## Configuration

### Initial Setup

Run the setup wizard:

```bash
/meigen:setup
```

The wizard walks through:
1. **Provider selection**: ComfyUI (local), MeiGen Cloud, or OpenAI-compatible API
2. **Credentials**: API tokens, URLs, or keys
3. **Restart**: One final restart to activate

### Provider Options

#### Option 1: Local ComfyUI (Free)

Run on your own GPU with full control:

```json
{
  "comfyuiUrl": "http://localhost:8188",
  "comfyuiDefaultWorkflow": "txt2img"
}
```

**Environment variables:**
```bash
COMFYUI_URL=http://localhost:8188
COMFYUI_DEFAULT_WORKFLOW=txt2img
```

#### Option 2: MeiGen Cloud

Get your API token at [meigen.ai](https://www.meigen.ai) → Settings → API Keys:

```json
{
  "meigenApiToken": "meigen_sk_..."
}
```

**Environment variable:**
```bash
MEIGEN_API_TOKEN=meigen_sk_...
```

#### Option 3: OpenAI-Compatible API

Bring your own key and endpoint:

```json
{
  "openaiCompatibleApiKey": "sk-...",
  "openaiCompatibleEndpoint": "https://api.openai.com/v1"
}
```

**Environment variables:**
```bash
OPENAI_API_KEY=sk-...
OPENAI_API_BASE=https://api.openai.com/v1
```

## MCP Tools

### search_gallery (Free)

Search 1,446 curated trending prompts with visual previews.

**TypeScript usage:**
```typescript
const result = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "search_gallery",
  arguments: {
    keywords: "product photography luxury",
    limit: 10
  }
});
```

**Natural language prompt:**
> "Search the gallery for luxury product photography inspiration"

### get_inspiration (Free)

Get full prompt, all images, and metadata for a specific gallery entry.

**TypeScript usage:**
```typescript
const inspiration = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "get_inspiration",
  arguments: {
    id: "nanobanana_12345"
  }
});
```

### enhance_prompt (Free)

Transform brief ideas into professional image prompts with style awareness.

**TypeScript usage:**
```typescript
const enhanced = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "enhance_prompt",
  arguments: {
    userPrompt: "cat in kitchen",
    style: "photorealistic",
    additionalContext: "sunlit, warm tones"
  }
});
```

**Natural language prompt:**
> "Enhance this prompt: a cat in a kitchen"

### list_models (Free)

List available models across all configured providers.

**TypeScript usage:**
```typescript
const models = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "list_models",
  arguments: {}
});
```

### generate_image (Requires API Key)

Generate an image with automatic provider routing, reference image upload, and aspect ratio handling.

**TypeScript usage:**
```typescript
const image = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "generate_image",
  arguments: {
    prompt: "a minimalist tech logo, geometric shapes, blue gradient",
    model: "midjourney-v8.1",
    aspectRatio: "1:1",
    referenceImagePath: "/Users/dev/Desktop/sketch.jpg"  // Optional
  }
});
```

**Natural language prompt:**
> "Generate a minimalist tech logo in 1:1 ratio using Midjourney V8.1"

**With reference image:**
> "Generate a product shot based on this bottle.jpg in my Downloads folder"

### generate_video (Requires API Key)

Generate video via MeiGen platform (Seedance 2.0, Happyhorse 1.0, or Veo 3.1).

**TypeScript usage:**
```typescript
const video = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "generate_video",
  arguments: {
    prompt: "Camera slowly zooms into the perfume bottle, bokeh background",
    model: "seedance-2.0-fast",
    firstFrameImagePath: "/Users/dev/Pictures/bottle.jpg",  // Optional
    duration: 5
  }
});
```

**Natural language prompt:**
> "Animate this product photo into a 5-second video with a slow zoom"

### comfyui_workflow (Free)

Manage ComfyUI workflow templates.

**TypeScript usage:**
```typescript
// List workflows
const workflows = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "comfyui_workflow",
  arguments: {
    action: "list"
  }
});

// View a workflow
const workflow = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "comfyui_workflow",
  arguments: {
    action: "view",
    name: "txt2img"
  }
});

// Import a workflow
await use_mcp_tool({
  server_name: "meigen",
  tool_name: "comfyui_workflow",
  arguments: {
    action: "import",
    name: "custom-flux",
    workflow: JSON.stringify(workflowJson)
  }
});
```

### manage_preferences (Free)

Store user preferences (style, aspect ratio, model, favorite prompts).

**TypeScript usage:**
```typescript
await use_mcp_tool({
  server_name: "meigen",
  tool_name: "manage_preferences",
  arguments: {
    action: "set",
    key: "preferredStyle",
    value: "minimalist editorial"
  }
});

const prefs = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "manage_preferences",
  arguments: {
    action: "get"
  }
});
```

## Slash Commands

Quick shortcuts for common tasks:

| Command | Description |
|---------|-------------|
| `/meigen:gen <prompt>` | Skip conversation, generate image immediately |
| `/meigen:find <keywords>` | Search prompt library |
| `/meigen:models` | Browse and switch models |
| `/meigen:setup` | Run provider configuration wizard |

**Example:**
```bash
/meigen:gen a calico cat in a sunlit kitchen, photorealistic
/meigen:find luxury product photography
/meigen:models
```

## Standalone CLI Mode

Use MeiGen without an MCP host (for shell scripts, CI/CD, etc.):

```bash
# Set token once
export MEIGEN_API_TOKEN=meigen_sk_...

# Basic generation
npx meigen gen --prompt "a calico cat in a sunlit kitchen"

# Specific model + aspect ratio
npx meigen gen -p "tech logo" -m midjourney-v8.1 -r 1:1

# With reference image
npx meigen gen -p "product hero shot" --ref ~/Desktop/bottle.jpg

# Submit only (no polling, good for CI)
npx meigen gen -p "..." --no-wait

# Machine-readable output (pipe to jq)
npx meigen gen -p "..." --json | jq -r '.imageUrls[0]'
```

**Full CLI flags:**
```bash
npx meigen gen --help

Options:
  -p, --prompt <text>        Image prompt (required)
  -m, --model <name>         Model name (default: nanobanana-2)
  -r, --ratio <ratio>        Aspect ratio (e.g. 16:9, 1:1, 3:4)
  --ref <path>               Reference image path
  --no-wait                  Submit only, don't poll for result
  --json                     Machine-readable JSON output
  -h, --help                 Display help
```

**Output:**
- Images saved to `~/Pictures/meigen/` (override with `MEIGEN_OUTPUT_DIR`)
- Videos saved to `~/Movies/meigen/`

## Common Patterns

### Parallel Batch Generation

Generate multiple variations in parallel using sub-agents:

**Natural language prompt:**
> "Create 4 product display images for this perfume, one of which should feature a model"

The AI will:
1. Upload the reference image
2. Craft 4 distinct prompts (luxury still life, model campaign, botanical, minimalist)
3. Spawn 4 `image-generator` sub-agents in parallel
4. Return all 4 images in ~2 minutes

### Style-Aware Prompt Enhancement

**Natural language prompt:**
> "Enhance this for a cinematic feel: a coffee cup on a desk"

**Enhanced result:**
```
A pristine white ceramic coffee cup on a dark walnut desk, steam rising 
gracefully, shot with shallow depth of field, warm morning light streaming 
through venetian blinds creating dramatic shadows, cinematic color grading, 
shot on Arri Alexa, 35mm lens, f/1.4
```

### ComfyUI Workflow Import

Import any ComfyUI API-format workflow:

```typescript
// Read workflow from file
const workflowJson = JSON.parse(
  fs.readFileSync('path/to/workflow_api.json', 'utf8')
);

// Import into MeiGen
await use_mcp_tool({
  server_name: "meigen",
  tool_name: "comfyui_workflow",
  arguments: {
    action: "import",
    name: "flux-lora-custom",
    workflow: JSON.stringify(workflowJson)
  }
});

// Use it
await use_mcp_tool({
  server_name: "meigen",
  tool_name: "generate_image",
  arguments: {
    prompt: "cyberpunk street scene",
    model: "comfyui",
    workflow: "flux-lora-custom"
  }
});
```

### Video from Still Image

Animate a product photo into a 5-second video:

```typescript
const video = await use_mcp_tool({
  server_name: "meigen",
  tool_name: "generate_video",
  arguments: {
    prompt: "Slow camera push towards product, bokeh background, cinematic lighting",
    model: "seedance-2.0-fast",
    firstFrameImagePath: "/Users/dev/Desktop/product.jpg",
    duration: 5
  }
});
```

**Models:**
- `seedance-2.0-fast`: 5s, 1280×720, ~30s generation
- `seedance-2.0-pro`: 5s, 1280×720, higher quality, ~2min
- `happyhorse-1.0`: 6s, 1360×768, realistic motion
- `veo-3.1`: 8s+, 1280×720, ultra-realistic, ~5-10min

### Multi-Model Comparison

Generate the same prompt with different models:

**Natural language prompt:**
> "Generate this prompt with Nanobanana 2, Midjourney V8.1, and Flux 2 Klein: 
> a serene Japanese garden at dawn"

The AI will parallelize across 3 sub-agents and present all results.

## Troubleshooting

### "No provider configured"

**Symptom:** `generate_image` fails with "No provider configured"

**Solution:**
```bash
/meigen:setup
# Or set environment variables manually:
export MEIGEN_API_TOKEN=meigen_sk_...
```

### ComfyUI connection refused

**Symptom:** `ECONNREFUSED 127.0.0.1:8188`

**Solution:**
1. Ensure ComfyUI is running: `http://localhost:8188`
2. Check firewall isn't blocking the port
3. Verify `COMFYUI_URL` matches your actual URL

### Reference image upload fails

**Symptom:** Local image path not found

**Solution:**
- Use absolute paths: `/Users/username/Desktop/image.jpg`
- Or relative to project root: `./assets/image.jpg`
- MeiGen auto-compresses images >5MB before upload

### Video generation timeout

**Symptom:** MCP tool times out before video finishes

**Solution:**
- For Hermes Agent, set `timeout: 600` in config
- For other hosts, increase MCP server timeout (default 120s → 600s)
- Veo 3.1 can take 5-10 minutes for high-quality 8s videos

### "Model not available"

**Symptom:** Requested model not found

**Solution:**
```bash
/meigen:models
# Or use tool:
use_mcp_tool({
  server_name: "meigen",
  tool_name: "list_models",
  arguments: {}
})
```

Available models depend on your provider:
- **MeiGen Cloud**: `nanobanana-2`, `gpt-image-2`, `seedream-5.0`, `midjourney-v8.1`, `flux-2-klein`, `seedance-2.0-fast`, `seedance-2.0-pro`, `happyhorse-1.0`, `veo-3.1`
- **ComfyUI**: Uses local workflows, model specified in workflow JSON
- **OpenAI-compatible**: `dall-e-3`, `dall-e-2`, or custom models at your endpoint

### Prompt library search returns no results

**Symptom:** `search_gallery` finds nothing

**Solution:**
- Try broader keywords: "product" instead of "luxury minimalist product photography"
- Use style terms: "cinematic", "minimalist", "vintage", "3D render"
- Browse without keywords (empty search) to see all 1,446 prompts

## Environment Variables Reference

| Variable | Description | Required |
|----------|-------------|----------|
| `MEIGEN_API_TOKEN` | MeiGen Cloud API token (get at meigen.ai) | For MeiGen provider |
| `COMFYUI_URL` | ComfyUI server URL | For ComfyUI provider |
| `COMFYUI_DEFAULT_WORKFLOW` | Default workflow name | Optional (default: `txt2img`) |
| `OPENAI_API_KEY` | OpenAI or compatible API key | For OpenAI provider |
| `OPENAI_API_BASE` | API endpoint URL | Optional (default: OpenAI endpoint) |
| `MEIGEN_OUTPUT_DIR` | Override image output directory | Optional (default: `~/Pictures/meigen`) |

## Resources

- **GitHub**: [jau123/MeiGen-AI-Design-MCP](https://github.com/jau123/MeiGen-AI-Design-MCP)
- **Homepage**: [meigen.ai](https://www.meigen.ai)
- **Prompt Library**: [nanobanana-trending-prompts](https://github.com/jau123/nanobanana-trending-prompts)
- **Discord**: [Join community](https://discord.gg/uX6rnersUx)
- **License**: MIT
