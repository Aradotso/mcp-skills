---
name: comfyui-mcp-agent
description: Expert skill for driving ComfyUI (image/video/audio generation) via the comfyui-mcp MCP server — workflow authoring, model management, natural language graph editing, and agent-native control plane.
triggers:
  - "generate an image with ComfyUI"
  - "create a ComfyUI workflow for video generation"
  - "install a custom node pack in ComfyUI"
  - "debug why my ComfyUI workflow failed"
  - "edit the ComfyUI graph to change the prompt"
  - "download and configure a Flux model"
  - "convert this workflow to API format"
  - "list available ComfyUI models and custom nodes"
---

# comfyui-mcp-agent

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Expert skill for **comfyui-mcp**, an agent-native control plane for [ComfyUI](https://github.com/comfyanonymous/ComfyUI). This MCP server exposes **108 tools** for image/video/audio generation, workflow authoring, model management, custom node installation, and **natural language graph editing**. It ships with **32 AI skills** (Flux, WAN, LTX 2.3, Qwen, Ideogram 4, Krea2), **13 installer packs**, and Claude Code plugin integration.

**comfyui-mcp** is local-first but not local-only: one config targets local installs, LAN, VPS, or Comfy Cloud. It authors and edits graphs node-by-node, runs and iterates on workflows, manages models and custom nodes, and ships model-specific expertise (samplers, CFG, resolutions, curated URLs) so agents get it right without trial and error.

## Installation

### As MCP Server (Claude Desktop / Claude Code)

Add to your Claude config (`~/.claude/settings.json`):

```json
{
  "mcpServers": {
    "comfyui": {
      "command": "npx",
      "args": ["-y", "comfyui-mcp"],
      "env": {
        "CIVITAI_API_TOKEN": "",
        "COMFYUI_URL": "",
        "COMFYUI_API_KEY": ""
      }
    }
  }
}
```

**Environment variables** (all optional):

- `CIVITAI_API_TOKEN` — for downloading gated models from Civitai
- `COMFYUI_URL` — override auto-detected ComfyUI endpoint (default: `http://127.0.0.1:8188`)
- `COMFYUI_API_KEY` — for Comfy Cloud mode (cloud.comfy.org)
- `COMFYUI_MCP_HTTP_TOKEN` — auth token for remote HTTP transport (`--tunnel` or `--http`)

### As Claude Code Plugin

```bash
# In Claude Code
/plugin marketplace add artokun/comfyui-mcp
/plugin install comfy
```

### Remote / Hosted Connector (HTTP Transport)

Run as an authenticated, publicly-reachable server:

```bash
npx -y comfyui-mcp@latest --tunnel
```

This forces HTTP transport, generates an auth token, opens a cloudflared tunnel, and prints a `https://…/mcp` URL + token for Claude Desktop Custom Connectors.

### Panel Agent (Autonomous Sidebar in ComfyUI)

Install the **[ComfyUI Agent Panel](https://github.com/artokun/comfyui-mcp-panel)** via ComfyUI-Manager:

```bash
# Search "comfyui-agent-panel" in ComfyUI-Manager
# Or manual install:
cd custom_nodes
git clone https://github.com/artokun/comfyui-mcp-panel.git
cd comfyui-mcp-panel
npm install
```

Then start the orchestrator from your local machine:

```bash
npx -y comfyui-mcp@latest connect
```

## Core MCP Tools

### Workflow Execution

```typescript
// Generate an image from a prompt
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "generate_image",
  arguments: {
    prompt: "a sunset over mountains, cinematic lighting",
    negative_prompt: "blurry, low quality",
    width: 1024,
    height: 1024,
    cfg: 7.0,
    steps: 20,
    seed: -1,
    checkpoint: "flux1-dev-fp8.safetensors"
  }
});

// Enqueue a custom workflow (API format)
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "enqueue_workflow",
  arguments: {
    workflow: {
      "1": {
        "class_type": "CheckpointLoaderSimple",
        "inputs": { "ckpt_name": "flux1-dev-fp8.safetensors" }
      },
      "2": {
        "class_type": "CLIPTextEncode",
        "inputs": { "text": "a photo of a cat", "clip": ["1", 1] }
      }
      // ...
    }
  }
});

// Get job status
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "get_job_status",
  arguments: { prompt_id: "abc-123" }
});

// Cancel running job
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "interrupt_execution",
  arguments: {}
});
```

### Graph Editing (Natural Language)

```typescript
// Add a node to the current workflow
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "add_node",
  arguments: {
    node_type: "KSampler",
    params: {
      seed: 42,
      steps: 30,
      cfg: 8.0,
      sampler_name: "euler_ancestral",
      scheduler: "normal"
    }
  }
});

// Update existing node
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "update_node",
  arguments: {
    node_id: "5",
    updates: { "cfg": 9.0, "steps": 25 }
  }
});

// Delete node
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "delete_node",
  arguments: { node_id: "12" }
});

// Connect two nodes
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "connect_nodes",
  arguments: {
    source_node: "3",
    source_output: 0,
    target_node: "7",
    target_input: "image"
  }
});

// Get current workflow state
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "get_workflow",
  arguments: {}
});
```

### Model Management

```typescript
// List available checkpoints
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "list_checkpoints",
  arguments: {}
});

// Download a model from URL
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "download_model",
  arguments: {
    url: "https://huggingface.co/black-forest-labs/FLUX.1-dev/resolve/main/flux1-dev.safetensors",
    filename: "flux1-dev.safetensors",
    type: "checkpoints"
  }
});

// Download from Civitai (requires CIVITAI_API_TOKEN)
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "download_civitai_model",
  arguments: {
    model_id: "43331",
    version_id: "47274",
    filename: "majicmixRealistic_v7.safetensors",
    type: "checkpoints"
  }
});

// List LoRAs
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "list_loras",
  arguments: {}
});

// List VAEs
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "list_vaes",
  arguments: {}
});
```

### Custom Node Management

```typescript
// List installed custom nodes
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "list_custom_nodes",
  arguments: {}
});

// Install custom node pack
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "install_custom_nodes",
  arguments: {
    repo_url: "https://github.com/ltdrdata/ComfyUI-Manager.git"
  }
});

// Update custom node
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "update_custom_nodes",
  arguments: {
    node_name: "ComfyUI-Manager"
  }
});

// Uninstall custom node
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "uninstall_custom_nodes",
  arguments: {
    node_name: "ComfyUI-Impact-Pack"
  }
});

// Get node info from object_info
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "get_node_info",
  arguments: {
    node_type: "KSampler"
  }
});
```

### System Management

```typescript
// Get system stats (VRAM, RAM, disk)
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "get_system_stats",
  arguments: {}
});

// Restart ComfyUI
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "restart_comfyui",
  arguments: {}
});

// Stop ComfyUI
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "stop_comfyui",
  arguments: {}
});

// Get queue status
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "get_queue",
  arguments: {}
});

// Clear queue
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "clear_queue",
  arguments: {}
});
```

### Workflow Conversion & Visualization

```typescript
// Convert UI format to API format
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "convert_workflow",
  arguments: {
    workflow: { /* UI format workflow JSON */ },
    target_format: "api"
  }
});

// Visualize workflow as Mermaid diagram
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "visualize_workflow",
  arguments: {
    workflow: { /* workflow JSON */ }
  }
});

// Compare two workflows
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "compare_workflows",
  arguments: {
    workflow_a: { /* first workflow */ },
    workflow_b: { /* second workflow */ }
  }
});
```

## Claude Code Slash Commands

When installed as a plugin, comfyui-mcp adds these slash commands:

### `/comfy:gen <prompt>`

Generate an image from a text description. Auto-selects checkpoint, builds workflow, returns image.

```
/comfy:gen a portrait of a woman in cyberpunk style, neon lights
```

### `/comfy:viz <workflow>`

Visualize a workflow as a Mermaid diagram with nodes grouped by category.

```
/comfy:viz
```

### `/comfy:node-skill <pack>`

Generate a Claude skill for a custom node pack from Registry ID or GitHub URL.

```
/comfy:node-skill https://github.com/ltdrdata/ComfyUI-Impact-Pack
```

### `/comfy:debug [prompt_id]`

Diagnose why a workflow failed — reads history, logs, traces root cause, suggests fixes.

```
/comfy:debug abc-123
```

### `/comfy:batch <prompt, params>`

Parameter sweep generation across cfg, sampler, steps, seed, etc.

```
/comfy:batch "a forest scene" cfg=[6,7,8,9] steps=[20,30]
```

### `/comfy:convert <file>`

Convert between UI format and API format workflows.

```
/comfy:convert workflow.json
```

### `/comfy:install <pack>`

Install a custom node pack — git clone, pip install, optional restart.

```
/comfy:install ComfyUI-Manager
```

### `/comfy:gallery [filter]`

Browse generated outputs with metadata — filter by date, count, or filename.

```
/comfy:gallery today
```

### `/comfy:compare <a vs b>`

Diff two workflows side by side — shows added/removed nodes and changed parameters.

```
/comfy:compare workflow_v1.json workflow_v2.json
```

### `/comfy:recipe <name> <prompt>`

Multi-step recipes: `portrait`, `hires-fix`, `style-transfer`, `product-shot`.

```
/comfy:recipe portrait "a woman with curly hair"
```

## Model-Specific Patterns

### Flux.1 (Dev / Schnell)

```typescript
// Flux requires specific settings
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "generate_image",
  arguments: {
    prompt: "cinematic photo of a sunset",
    checkpoint: "flux1-dev-fp8.safetensors",
    width: 1024,
    height: 1024,
    cfg: 1.0,  // Flux works best at CFG 1.0
    steps: 20,  // Dev: 20-30, Schnell: 4-8
    sampler_name: "euler",
    scheduler: "simple"
  }
});
```

### WAN (Video Generation)

```typescript
// WAN requires ComfyUI-WAN-Animate custom nodes
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "install_custom_nodes",
  arguments: {
    repo_url: "https://github.com/artokun/ComfyUI-WAN-Animate.git"
  }
});

// Generate video with WAN
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "enqueue_workflow",
  arguments: {
    workflow: {
      // WAN workflow nodes...
      // See packs/wan-animate.json for full example
    }
  }
});
```

### LTX Video 2.3

```typescript
// Download LTX model
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "download_model",
  arguments: {
    url: "https://huggingface.co/Lightricks/LTX-Video/resolve/main/ltx-video-2.3-fp8.safetensors",
    filename: "ltx-video-2.3-fp8.safetensors",
    type: "checkpoints"
  }
});

// Install ComfyUI-LTXVideo
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "install_custom_nodes",
  arguments: {
    repo_url: "https://github.com/Lightricks/ComfyUI-LTXVideo.git"
  }
});
```

### Qwen Image / Image Edit

```typescript
// Install Qwen nodes
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "install_custom_nodes",
  arguments: {
    repo_url: "https://github.com/Qwen2-VL/ComfyUI-Qwen2VL.git"
  }
});

// Download Qwen model
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "download_model",
  arguments: {
    url: "https://huggingface.co/Qwen/Qwen2-VL-7B-Instruct/resolve/main/model.safetensors",
    filename: "qwen2-vl-7b.safetensors",
    type: "checkpoints"
  }
});
```

## Installer Packs

The project ships **13 one-command installer packs** in `packs/`:

- `anima.json` — ANIMA anime generation
- `ideogram4.json` — Ideogram 4 text-to-image
- `ltx-2.3.json` — LTX Video 2.3
- `ernie.json` — ERNIE (Baidu) models
- `wan-animate.json` — WAN video animation
- `wan-longer-videos.json` — WAN extended video
- `wan-transparent.json` — WAN transparent generation
- `qwen-image.json` — Qwen image generation
- `qwen-image-edit.json` — Qwen image editing
- `z-image-turbo.json` — Z-Image turbo mode
- `z-image-base.json` — Z-Image base
- `z-image-xy-plot.json` — Z-Image XY plot
- `artokun-flow-wan-animate.json` — WAN replace/animate flow

Each pack is a manifest of custom nodes + model URLs + workflow. Apply with:

```typescript
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "apply_manifest",
  arguments: {
    manifest: require("./packs/flux-dev.json")
  }
});
```

Or use the generated installers:

```bash
# Windows
install-windows.bat flux-dev

# Linux / RunPod
bash install-runpod.sh flux-dev
```

## Real-Time Progress Monitoring

Instead of polling `get_job_status`, use the WebSocket progress monitor:

```bash
# Start in background
node monitor-progress.mjs <prompt_id> &

# Reports step progress:
# "step 5/14 (36%)"
# "Completed: output_001.png"
```

The monitor connects to ComfyUI's WebSocket (`/ws`) and emits real-time updates. On completion, it prints output filenames; on error, it reports the failing node.

## Troubleshooting

### Common Errors

**OOM (Out of Memory)**

```typescript
// Check VRAM before execution
const stats = use_mcp_tool({
  server_name: "comfyui",
  tool_name: "get_system_stats"
});

if (stats.vram_free_gb < 1.0) {
  console.warn("Low VRAM — consider using fp8 checkpoint or smaller resolution");
}
```

**Missing Custom Nodes**

```typescript
// Error: "Cannot find node type 'XYZ'"
// Solution: Install the custom node pack
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "install_custom_nodes",
  arguments: {
    repo_url: "https://github.com/author/ComfyUI-XYZ.git"
  }
});

// Restart ComfyUI
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "restart_comfyui"
});
```

**Model Not Found**

```typescript
// List available checkpoints
const checkpoints = use_mcp_tool({
  server_name: "comfyui",
  tool_name: "list_checkpoints"
});

// Download if missing
use_mcp_tool({
  server_name: "comfyui",
  tool_name: "download_model",
  arguments: {
    url: "https://...",
    filename: "model.safetensors",
    type: "checkpoints"
  }
});
```

**Black Images / NaN Tensors**

- **Flux/SD3**: Use CFG 1.0-3.5 (not 7-9)
- **SDXL Lightning**: Use 4-8 steps (not 20-30)
- **FP16 on CPU**: Switch to FP32 or use `--force-fp16` launch arg

**dtype Mismatch**

```
RuntimeError: expected dtype Float but got Half
```

Solution: Use matching precision checkpoint (fp8 vs fp16) or force cast in workflow.

### Launch Flags

Pass performance flags via environment or CLI:

```bash
# Low VRAM mode
python main.py --lowvram

# Force FP16
python main.py --force-fp16

# CPU offload
python main.py --normalvram

# Disable xformers
python main.py --disable-xformers
```

## Resources

- **Documentation**: https://comfyui-mcp.artokun.io/docs
- **GitHub**: https://github.com/artokun/comfyui-mcp
- **Discord**: https://discord.gg/cW9arBhzCu
- **ComfyUI**: https://github.com/comfyanonymous/ComfyUI
- **ComfyUI Registry**: https://registry.comfy.org
- **Agent Panel**: https://github.com/artokun/comfyui-mcp-panel

## Key Principles for Agents

1. **Always check VRAM before heavy workflows** — use `get_system_stats` and warn if < 1GB free
2. **Use model-specific settings** — Flux at CFG 1.0, SDXL Lightning at 4-8 steps, etc.
3. **Auto-install missing custom nodes** — if a workflow needs a node, install it first
4. **Prefer curated model URLs from skills** — the model-registry skill has verified download links
5. **Monitor progress via WebSocket** — don't poll `get_job_status` in a loop
6. **Validate workflows before enqueueing** — use `get_node_info` to check inputs/outputs
7. **Restart after custom node installs** — many nodes require a restart to register
8. **Use packs for complex setups** — don't manually build multi-node workflows when a pack exists
9. **Check object_info for node params** — ComfyUI's schema is dynamic; always verify allowed values
10. **Fail gracefully with debug commands** — if a workflow errors, run `/comfy:debug` to diagnose
