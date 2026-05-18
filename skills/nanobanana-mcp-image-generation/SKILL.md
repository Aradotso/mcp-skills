---
name: nanobanana-mcp-image-generation
description: AI image generation with Google Gemini via MCP - smart model selection, 4K output, aspect ratios, and templates
triggers:
  - generate an image with gemini
  - create images using nanobanana
  - use gemini for image generation
  - generate 4k images with ai
  - create product photos with gemini
  - setup nanobanana mcp server
  - generate images with aspect ratio control
  - use gemini image generation templates
---

# Nano Banana MCP Image Generation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Nano Banana is a production-ready MCP server that provides AI-powered image generation through Google's Gemini models. It features intelligent automatic model selection between three tiers (Flash, NB2, and Pro), 4K output, aspect ratio control, smart templates, and file management.

**Key Features:**
- 🍌 **Gemini 3.1 Flash Image (NB2)**: Default model — 4K resolution at Flash speed with Google Search grounding
- 🏆 **Gemini 3 Pro Image**: Maximum reasoning depth for complex compositions
- ⚡ **Gemini 2.5 Flash Image**: Legacy high-speed model for rapid prototyping
- 🤖 **Smart Auto Selection**: Automatically routes to the best model based on your prompt
- 📐 **Aspect Ratio Control**: 1:1, 16:9, 9:16, 21:9, and more
- 📋 **Smart Templates**: Pre-built prompts for photography, design, and editing
- 📁 **File Management**: Upload and reference images via Gemini Files API

## Installation

### Prerequisites

1. **Google Gemini API Key**: Get one free at https://makersuite.google.com/app/apikey
2. Set environment variable: `GEMINI_API_KEY=your-api-key-here`

### MCP Client Configuration

#### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "nanobanana": {
      "command": "uvx",
      "args": ["nanobanana-mcp-server@latest"],
      "env": {
        "GEMINI_API_KEY": "your-gemini-api-key-here"
      }
    }
  }
}
```

#### Cursor

Add to Cursor's MCP configuration:

```json
{
  "mcpServers": {
    "nanobanana": {
      "command": "uvx",
      "args": ["nanobanana-mcp-server@latest"],
      "env": {
        "GEMINI_API_KEY": "your-gemini-api-key-here"
      }
    }
  }
}
```

#### Codex (OpenAI)

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.nanobanana]
command = "uvx"
args = ["nanobanana-mcp-server@latest"]

[mcp_servers.nanobanana.env]
GEMINI_API_KEY = "your-gemini-api-key-here"
```

#### Continue.dev

Add to `config.json`:

```json
{
  "mcpServers": [
    {
      "name": "nanobanana",
      "command": "uvx",
      "args": ["nanobanana-mcp-server@latest"],
      "env": {
        "GEMINI_API_KEY": "your-gemini-api-key-here"
      }
    }
  ]
}
```

### Vertex AI Authentication (Google Cloud)

For production deployments on Google Cloud, use Application Default Credentials:

```json
{
  "mcpServers": {
    "nanobanana": {
      "command": "uvx",
      "args": ["nanobanana-mcp-server@latest"],
      "env": {
        "NANOBANANA_AUTH_METHOD": "vertex_ai",
        "GCP_PROJECT_ID": "your-project-id",
        "GCP_REGION": "global"
      }
    }
  }
}
```

**Prerequisites for Vertex AI:**
- Enable Vertex AI API: `gcloud services enable aiplatform.googleapis.com`
- Grant IAM role: `roles/aiplatform.user`

## Core MCP Tools

### 1. `generate_image` - Generate Images from Text

The primary tool for creating images from text prompts.

**Parameters:**
- `prompt` (required): Text description of the image to generate
- `model_tier` (optional): `"auto"` (default), `"nb2"`, `"pro"`, or `"flash"`
- `resolution` (optional): `"4k"` (3840px, default for pro/nb2) or `"1k"` (1024px)
- `aspect_ratio` (optional): `"1:1"`, `"16:9"`, `"9:16"`, `"21:9"`, `"4:3"`, `"3:4"`, etc.
- `n` (optional): Number of images to generate (1-4, default 1)
- `thinking_level` (optional): `"LOW"` or `"HIGH"` (Pro model only)
- `enable_grounding` (optional): Enable Google Search grounding (boolean)
- `output_path` (optional): Custom save path for generated images
- `negative_prompt` (optional): Elements to avoid in the image
- `use_template` (optional): Template ID (e.g., `"product_photo"`, `"portrait"`)
- `seed` (optional): Integer for reproducible generation

**Basic Usage:**

```python
# Simple image generation (auto selects NB2 by default)
generate_image(
    prompt="A serene mountain landscape at sunset with a lake reflection"
)

# Generate with specific aspect ratio
generate_image(
    prompt="Modern minimalist product photo of a coffee mug",
    aspect_ratio="4:3"
)

# Generate multiple variations
generate_image(
    prompt="Abstract geometric pattern in blue and gold",
    n=3
)
```

**Advanced Usage:**

```python
# High-quality 4K generation with NB2 (default)
generate_image(
    prompt="Professional product photography of a luxury watch on marble surface",
    model_tier="nb2",
    resolution="4k",
    aspect_ratio="16:9",
    enable_grounding=True
)

# Maximum quality with Pro model
generate_image(
    prompt="Cinematic scene: three characters in a tense standoff at dusk, dramatic lighting",
    model_tier="pro",
    resolution="4k",
    thinking_level="HIGH",
    enable_grounding=True,
    negative_prompt="blurry, low quality, distorted faces"
)

# Fast generation with Flash model
generate_image(
    prompt="Simple icon design for a mobile app",
    model_tier="flash",
    n=4
)

# Custom output path
generate_image(
    prompt="Logo design for tech startup",
    output_path="/path/to/output/logo.png",
    aspect_ratio="1:1"
)

# Reproducible generation with seed
generate_image(
    prompt="Fantasy castle in the clouds",
    seed=42,
    aspect_ratio="16:9"
)
```

### 2. `edit_image` - Edit Existing Images

Edit or modify existing images with text prompts.

**Parameters:**
- `image_path` (required): Path to the input image file
- `prompt` (required): Description of the desired edit
- `model_tier` (optional): `"auto"`, `"nb2"`, `"pro"`, or `"flash"`
- `mask_path` (optional): Path to a mask image (white = edit region)
- `resolution` (optional): `"4k"` or `"1k"`
- `output_path` (optional): Custom save path

**Usage Examples:**

```python
# Basic image editing
edit_image(
    image_path="/path/to/photo.jpg",
    prompt="Add a sunset sky in the background"
)

# Masked editing (inpainting)
edit_image(
    image_path="/path/to/portrait.jpg",
    prompt="Change the shirt color to blue",
    mask_path="/path/to/mask.png",
    model_tier="pro"
)

# High-quality editing with Pro model
edit_image(
    image_path="/path/to/landscape.jpg",
    prompt="Add dramatic storm clouds and rain",
    model_tier="pro",
    resolution="4k",
    output_path="/path/to/edited.jpg"
)
```

### 3. `upload_file` - Upload Images for Reference

Upload images to Gemini Files API for use in generation or editing.

**Parameters:**
- `file_path` (required): Path to the image file to upload
- `display_name` (optional): Friendly name for the file

**Usage:**

```python
# Upload a reference image
upload_file(
    file_path="/path/to/reference.jpg",
    display_name="product_reference"
)

# Upload multiple references
upload_file(file_path="/path/to/style1.jpg", display_name="style_example_1")
upload_file(file_path="/path/to/style2.jpg", display_name="style_example_2")
```

**Then reference in generation:**

```python
generate_image(
    prompt="Create a similar product photo in the same style",
    # The uploaded files are automatically available to the model context
    model_tier="pro"
)
```

## Model Selection Guide

### 🍌 Nano Banana 2 (NB2) - Default Model

**When to use:**
- Default choice for most use cases
- Production-ready 4K output needed
- Text rendering in images
- Subject consistency (multiple characters/objects)
- Real-world accuracy (with grounding enabled)

**Specs:**
- Speed: ~2-4 seconds
- Resolution: Up to 4K (3840px)
- Special: Google Search grounding, subject consistency, text rendering

```python
generate_image(
    prompt="Instagram post with text overlay: 'Summer Sale 50% Off'",
    model_tier="nb2",
    resolution="4k",
    aspect_ratio="1:1",
    enable_grounding=True
)
```

### 🏆 Pro Model - Maximum Quality

**When to use:**
- Complex narrative scenes
- Maximum reasoning required
- Multiple characters with intricate interactions
- When prompt contains: "4K", "professional", "production", "cinematic"

**Specs:**
- Speed: ~5-8 seconds
- Resolution: Up to 4K (3840px)
- Special: Advanced reasoning, configurable thinking levels

```python
generate_image(
    prompt="Cinematic establishing shot: a dystopian city with three distinct districts visible, neon lights reflecting on wet streets, flying vehicles in the distance",
    model_tier="pro",
    resolution="4k",
    thinking_level="HIGH",
    enable_grounding=True
)
```

### ⚡ Flash Model - High Speed

**When to use:**
- High-volume generation (batches)
- Quick drafts and iterations
- 1024px resolution is sufficient
- When prompt contains: "quick", "draft", "sketch"

**Specs:**
- Speed: ~2-3 seconds
- Resolution: Up to 1024px
- Special: Fastest generation

```python
generate_image(
    prompt="Quick sketch of a user interface mockup",
    model_tier="flash",
    n=4  # Generate 4 variations quickly
)
```

### 🤖 Auto Mode (Recommended)

Let the system intelligently select the best model:

```python
# Auto routes to NB2 by default
generate_image(prompt="A cat sitting on a windowsill")

# Auto routes to Pro for quality keywords
generate_image(prompt="Professional 4K product photography of a watch")

# Auto routes to NB2 for speed keywords (NB2 is fast enough)
generate_image(prompt="Quick product thumbnail", n=3)
```

## Smart Templates

Access pre-built prompt templates via `use_template` parameter.

### Available Templates

**Photography Templates:**
- `product_photo`: Clean product photography on white background
- `portrait`: Professional portrait photography with natural lighting
- `landscape`: Scenic landscape photography with depth
- `food`: Appetizing food photography with natural lighting

**Design Templates:**
- `logo`: Modern, minimalist logo design
- `illustration`: Digital illustration with vibrant colors
- `abstract`: Abstract art with geometric patterns
- `icon`: Simple, clean icon design

**Editing Templates:**
- `background_replace`: Replace image background while keeping subject
- `style_transfer`: Apply artistic style to existing image
- `enhance`: Enhance image quality and details

### Using Templates

```python
# Product photography template
generate_image(
    prompt="wireless headphones",
    use_template="product_photo",
    aspect_ratio="1:1"
)

# Portrait template
generate_image(
    prompt="female executive, confident expression",
    use_template="portrait",
    resolution="4k"
)

# Logo design template
generate_image(
    prompt="tech startup focusing on AI, modern and clean",
    use_template="logo",
    aspect_ratio="1:1"
)

# Background replacement template
edit_image(
    image_path="/path/to/portrait.jpg",
    prompt="office environment with natural light",
    use_template="background_replace",
    model_tier="pro"
)
```

## Common Patterns

### Pattern 1: Social Media Content Generation

```python
# Instagram post (square)
generate_image(
    prompt="Motivational quote background: 'Dream Big' in elegant typography, pastel gradient",
    model_tier="nb2",
    aspect_ratio="1:1",
    resolution="4k"
)

# YouTube thumbnail (16:9)
generate_image(
    prompt="Eye-catching thumbnail: person with shocked expression, bright colors, text 'You Won't Believe This!'",
    model_tier="nb2",
    aspect_ratio="16:9",
    enable_grounding=True
)

# Story/Reel (9:16)
generate_image(
    prompt="Vertical video background: animated gradient with floating geometric shapes",
    aspect_ratio="9:16",
    resolution="4k"
)
```

### Pattern 2: Product Photography Workflow

```python
# Step 1: Generate base product image
generate_image(
    prompt="Modern wireless earbuds on marble surface, studio lighting, professional product photography",
    model_tier="nb2",
    resolution="4k",
    aspect_ratio="4:3",
    output_path="/products/earbuds_base.jpg"
)

# Step 2: Generate variations with different angles
generate_image(
    prompt="Same wireless earbuds, 45-degree angle, spotlight from top right",
    model_tier="nb2",
    resolution="4k",
    n=3
)

# Step 3: Edit to add lifestyle context
edit_image(
    image_path="/products/earbuds_base.jpg",
    prompt="Add lifestyle context: gym environment in blurred background",
    model_tier="pro"
)
```

### Pattern 3: Iterative Design Refinement

```python
# Start with fast iterations
generate_image(
    prompt="Logo concept for coffee shop, minimalist",
    model_tier="flash",
    n=4,
    aspect_ratio="1:1"
)

# Refine selected concept with NB2
generate_image(
    prompt="Coffee shop logo: simplified coffee cup icon with steam forming 'C' letter, earth tones",
    model_tier="nb2",
    resolution="4k",
    seed=12345  # Use seed from preferred flash result
)

# Final high-quality version with Pro
generate_image(
    prompt="Final coffee shop logo: refined coffee cup with elegant steam detail, warm brown and cream colors, professional branding quality",
    model_tier="pro",
    resolution="4k",
    aspect_ratio="1:1"
)
```

### Pattern 4: Multi-Image Campaign

```python
# Generate consistent style across multiple images
base_prompt = "e-commerce product photography, white background, soft shadows, professional lighting"

# Image 1: Hero shot
generate_image(
    prompt=f"{base_prompt}, luxury watch close-up, central composition",
    model_tier="nb2",
    resolution="4k",
    aspect_ratio="16:9",
    seed=100
)

# Image 2: Detail shot
generate_image(
    prompt=f"{base_prompt}, same luxury watch, focus on crown and dial details",
    model_tier="nb2",
    resolution="4k",
    seed=100  # Same seed for consistency
)

# Image 3: Lifestyle context
generate_image(
    prompt=f"{base_prompt}, same luxury watch on wrist, business attire visible",
    model_tier="pro",  # Pro for complex composition
    resolution="4k",
    seed=100
)
```

### Pattern 5: Image Editing Pipeline

```python
# Upload original image
upload_file(
    file_path="/raw/photo.jpg",
    display_name="original_photo"
)

# Edit 1: Color correction
edit_image(
    image_path="/raw/photo.jpg",
    prompt="Enhance colors, increase saturation slightly, professional color grading",
    output_path="/edited/step1_color.jpg"
)

# Edit 2: Background replacement
edit_image(
    image_path="/edited/step1_color.jpg",
    prompt="Replace background with modern office interior, maintain subject lighting",
    use_template="background_replace",
    model_tier="pro",
    output_path="/edited/step2_background.jpg"
)

# Edit 3: Final touches
edit_image(
    image_path="/edited/step2_background.jpg",
    prompt="Add subtle vignette, enhance sharpness, professional portrait finish",
    model_tier="pro",
    resolution="4k",
    output_path="/edited/final.jpg"
)
```

## Aspect Ratio Quick Reference

| Ratio | Use Case | Example |
|-------|----------|---------|
| `1:1` | Instagram posts, profile pics | Social media content |
| `4:3` | Classic photography | Product shots, portraits |
| `3:4` | Portrait orientation | Magazine covers |
| `16:9` | Widescreen, YouTube | Thumbnails, presentations |
| `9:16` | Mobile portrait | Phone wallpapers, Stories |
| `21:9` | Ultra-wide cinematic | Movie-style scenes |
| `2:3` | Photo print standard | Traditional prints |
| `3:2` | DSLR camera standard | Professional photography |

## Troubleshooting

### Issue: "API Key not found"

**Solution:**
```bash
# Verify environment variable is set
echo $GEMINI_API_KEY

# For Claude Desktop, check config file has the key
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Issue: "Model not available in region"

**Solution:**
- NB2 and Pro models require `GCP_REGION=global`
- Legacy Flash model uses `us-central1`

```json
{
  "env": {
    "NANOBANANA_AUTH_METHOD": "vertex_ai",
    "GCP_PROJECT_ID": "your-project",
    "GCP_REGION": "global"
  }
}
```

### Issue: Low-quality results

**Solution:**
```python
# Use Pro model with high thinking level
generate_image(
    prompt="[your prompt] - photorealistic, high detail, professional quality",
    model_tier="pro",
    resolution="4k",
    thinking_level="HIGH",
    negative_prompt="blurry, low quality, distorted, artificial"
)
```

### Issue: Inconsistent style across generations

**Solution:**
```python
# Use seed for reproducibility
generate_image(
    prompt="Your prompt here",
    seed=42,  # Same seed = similar style
    model_tier="nb2"
)

# Or enable grounding for factual consistency
generate_image(
    prompt="Your prompt here",
    enable_grounding=True,
    model_tier="nb2"
)
```

### Issue: File upload fails

**Solution:**
```python
# Check file exists and is readable
import os
assert os.path.isfile("/path/to/image.jpg"), "File not found"

# Supported formats: JPEG, PNG, GIF, WebP
# Max size: 20MB per file

upload_file(
    file_path="/path/to/image.jpg",
    display_name="my_reference"
)
```

### Issue: Generation timeout

**Solution:**
- Pro model can take 5-8 seconds
- Use NB2 for faster results (2-4 seconds)
- Use Flash for maximum speed (2-3 seconds)

```python
# If Pro times out, try NB2
generate_image(
    prompt="Your complex prompt",
    model_tier="nb2",  # Faster than Pro
    resolution="4k"    # Still 4K quality
)
```

## Advanced Configuration

### Custom Output Directory

```python
import os

# Set custom output path for all generations
OUTPUT_DIR = "/path/to/my/images"
os.makedirs(OUTPUT_DIR, exist_ok=True)

generate_image(
    prompt="Landscape photo",
    output_path=f"{OUTPUT_DIR}/landscape_001.jpg"
)
```

### Batch Processing with Error Handling

```python
prompts = [
    "Modern office interior",
    "Coastal sunset landscape",
    "Abstract geometric pattern"
]

for i, prompt in enumerate(prompts):
    try:
        generate_image(
            prompt=prompt,
            output_path=f"/batch/image_{i:03d}.jpg",
            model_tier="nb2",
            resolution="4k"
        )
    except Exception as e:
        print(f"Failed to generate image {i}: {e}")
        continue
```

### Environment Variables Reference

```bash
# Required (choose one authentication method)
GEMINI_API_KEY=your-api-key-here

# OR for Vertex AI
NANOBANANA_AUTH_METHOD=vertex_ai
GCP_PROJECT_ID=your-project-id
GCP_REGION=global

# Optional
NANOBANANA_LOG_LEVEL=INFO  # DEBUG, INFO, WARNING, ERROR
NANOBANANA_OUTPUT_DIR=/custom/output/path
```

## Best Practices

1. **Use Auto Mode by default** - Let the system select the best model
2. **Specify aspect ratios** - Better results than cropping after generation
3. **Enable grounding for realism** - Use `enable_grounding=True` for real-world accuracy
4. **Use templates** - Save time with pre-optimized prompts
5. **Iterate from Flash to Pro** - Start fast, refine with quality
6. **Set seeds for consistency** - Reproducible results across generations
7. **Use negative prompts** - Explicitly exclude unwanted elements
8. **Leverage Pro's thinking** - Set `thinking_level="HIGH"` for complex scenes
9. **Batch similar tasks** - Generate variations with `n` parameter
10. **Upload references** - Use `upload_file` for style/subject consistency

## Resources

- **API Key**: https://makersuite.google.com/app/apikey
- **MCP Registry**: https://registry.modelcontextprotocol.io/?q=nanobanana
- **GitHub**: https://github.com/zhongweili/nanobanana-mcp-server
- **Gemini Models**: https://ai.google.dev/models/gemini
