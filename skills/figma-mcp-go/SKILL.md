---
name: figma-mcp-go
description: Figma MCP server with full read/write access via plugin bridge — no API tokens, no rate limits, text to designs and designs to code
triggers:
  - how do I control Figma from my AI editor
  - automate Figma design creation without API limits
  - connect Figma to Claude or Cursor
  - create Figma components programmatically
  - export Figma designs to code
  - read and modify Figma files with MCP
  - set up figma-mcp-go plugin
  - update Figma text nodes automatically
---

# figma-mcp-go

> Skill by [ara.so](https://ara.so) — MCP Skills collection

An open-source MCP server that provides full read/write access to Figma files through a plugin bridge, bypassing REST API rate limits. Built in Go, it enables AI tools like Claude, Cursor, and GitHub Copilot to automate design workflows, create components, modify styles, manage variables, and export designs to code.

## What It Does

- **No API Token Required**: Works through a Figma plugin, avoiding REST API limits (6 calls/month on free plans)
- **73 Tools**: Complete CRUD operations for frames, components, text, styles, variables, prototypes, and more
- **Bidirectional**: Read design context and write changes back to live Figma files
- **Export**: Screenshot nodes, generate PDFs, export design tokens as JSON/CSS
- **Built-in Strategies**: Includes MCP prompts for design best practices

## Installation

### 1. Configure MCP Server

**Claude Code CLI:**
```bash
claude mcp add -s project figma-mcp-go -- npx -y @vkhanhqui/figma-mcp-go@latest
```

**Codex CLI:**
```bash
codex mcp add figma-mcp-go -- npx -y @vkhanhqui/figma-mcp-go@latest
```

**Cursor / VS Code / GitHub Copilot (`.vscode/mcp.json`):**
```json
{
  "servers": {
    "figma-mcp-go": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@vkhanhqui/figma-mcp-go"]
    }
  }
}
```

**Claude Desktop (`.mcp.json`):**
```json
{
  "mcpServers": {
    "figma-mcp-go": {
      "command": "npx",
      "args": ["-y", "@vkhanhqui/figma-mcp-go"]
    }
  }
}
```

### 2. Install Figma Plugin

1. Download [plugin.zip](https://github.com/vkhanhqui/figma-mcp-go/releases)
2. Open Figma Desktop → **Plugins** → **Development** → **Import plugin from manifest**
3. Select `manifest.json` from the extracted ZIP
4. Run the plugin inside any Figma file (it must stay open while using MCP tools)

## Core Workflows

### Reading Design Context

**Get current page structure:**
```typescript
// Use get_design_context for depth-limited tree with detail control
const context = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "get_design_context",
  arguments: {
    rootNodeId: "0:1",  // page ID or specific frame
    maxDepth: 3,
    detail: "compact"   // minimal | compact | full
  }
});
```

**Get metadata and page list:**
```typescript
const metadata = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "get_metadata",
  arguments: {}
});

const pages = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "get_pages",
  arguments: {}
});
```

**Search for nodes:**
```typescript
const results = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "search_nodes",
  arguments: {
    rootNodeId: "0:1",
    nameSubstring: "Button",
    types: ["COMPONENT", "INSTANCE"]
  }
});
```

### Creating Design Elements

**Create a frame with auto-layout:**
```typescript
const frame = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_frame",
  arguments: {
    name: "Hero Section",
    x: 0,
    y: 0,
    width: 1440,
    height: 600,
    fills: "#0066FF",
    layoutMode: "VERTICAL",
    primaryAxisAlignItems: "CENTER",
    counterAxisAlignItems: "CENTER",
    itemSpacing: 24,
    paddingTop: 48,
    paddingRight: 48,
    paddingBottom: 48,
    paddingLeft: 48
  }
});
```

**Create text node:**
```typescript
const text = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_text",
  arguments: {
    characters: "Welcome to Our Platform",
    fontFamily: "Inter",
    fontStyle: "Bold",
    fontSize: 48,
    x: 100,
    y: 100,
    parentId: frame.id  // optional, parent frame
  }
});
```

**Import image from base64:**
```typescript
const image = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "import_image",
  arguments: {
    base64Data: "iVBORw0KGgoAAAANSUhEUg...",  // without data:image prefix
    x: 0,
    y: 0,
    width: 800,
    height: 600,
    name: "Hero Image",
    parentId: frame.id
  }
});
```

### Modifying Existing Nodes

**Update text content:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "set_text",
  arguments: {
    nodeId: "123:456",
    characters: "Updated heading text"
  }
});
```

**Set fills and strokes:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "set_fills",
  arguments: {
    nodeIds: ["123:456", "123:457"],
    color: "#FF5733"
  }
});

await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "set_strokes",
  arguments: {
    nodeId: "123:456",
    color: "#000000",
    weight: 2
  }
});
```

**Batch find and replace text:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "find_replace_text",
  arguments: {
    rootNodeId: "0:1",  // search entire page
    findText: "Company Name",
    replaceText: "Acme Corp",
    useRegex: false,
    caseSensitive: false
  }
});
```

**Resize and reposition:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "resize_nodes",
  arguments: {
    nodeIds: ["123:456"],
    width: 1200,
    height: 800
  }
});

await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "move_nodes",
  arguments: {
    nodeIds: ["123:456"],
    x: 200,
    y: 300
  }
});
```

### Working with Components

**Create component from frame:**
```typescript
const component = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_component",
  arguments: {
    nodeId: "123:456",  // must be a FRAME
    name: "Button/Primary"
  }
});
```

**Swap component instance:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "swap_component",
  arguments: {
    instanceId: "123:789",
    newComponentId: "456:123"
  }
});
```

**Detach instance:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "detach_instance",
  arguments: {
    nodeIds: ["123:789"]
  }
});
```

### Styles and Variables

**Create paint style:**
```typescript
const style = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_paint_style",
  arguments: {
    name: "Colors/Primary/500",
    color: "#0066FF"
  }
});
```

**Create text style:**
```typescript
const textStyle = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_text_style",
  arguments: {
    name: "Heading/H1",
    fontFamily: "Inter",
    fontStyle: "Bold",
    fontSize: 48,
    lineHeight: 56,
    letterSpacing: -0.5
  }
});
```

**Apply style to node:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "apply_style_to_node",
  arguments: {
    nodeId: "123:456",
    styleId: style.id,
    styleType: "FILL"  // FILL | STROKE | TEXT | EFFECT | GRID
  }
});
```

**Create variable collection:**
```typescript
const collection = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_variable_collection",
  arguments: {
    name: "Spacing",
    initialModeName: "Default"
  }
});
```

**Create and bind variable:**
```typescript
const variable = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_variable",
  arguments: {
    collectionId: collection.id,
    name: "spacing-md",
    resolvedType: "FLOAT"
  }
});

await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "set_variable_value",
  arguments: {
    variableId: variable.id,
    modeId: collection.defaultModeId,
    value: 16
  }
});

await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "bind_variable_to_node",
  arguments: {
    nodeId: "123:456",
    variableId: variable.id,
    field: "itemSpacing"  // fillColor | strokeColor | width | height | opacity | rotation | visible | cornerRadius | itemSpacing | paddingLeft | etc.
  }
});
```

### Prototyping

**Set prototype reactions:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "set_reactions",
  arguments: {
    nodeId: "123:456",
    reactions: [
      {
        trigger: { type: "ON_CLICK" },
        actions: [
          {
            type: "NODE",
            destinationId: "123:789",
            navigation: "NAVIGATE",
            transition: {
              type: "DISSOLVE",
              duration: 0.3,
              easing: { type: "EASE_OUT" }
            }
          }
        ]
      }
    ],
    mode: "replace"  // replace | append
  }
});
```

**Get reactions:**
```typescript
const reactions = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "get_reactions",
  arguments: {
    nodeId: "123:456"
  }
});
```

### Exporting

**Take screenshot:**
```typescript
const screenshot = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "get_screenshot",
  arguments: {
    nodeId: "123:456",
    format: "PNG",
    scale: 2
  }
});
// Returns base64-encoded image
```

**Save screenshots to disk:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "save_screenshots",
  arguments: {
    nodeIds: ["123:456", "123:789"],
    outputDir: "/path/to/exports",
    format: "PNG",
    scale: 2,
    filenameTemplate: "{nodeName}_{nodeId}"
  }
});
```

**Export frames to PDF:**
```typescript
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "export_frames_to_pdf",
  arguments: {
    frameIds: ["123:1", "123:2", "123:3"],
    outputPath: "/path/to/output.pdf",
    scale: 2
  }
});
```

**Export design tokens:**
```typescript
const tokens = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "export_tokens",
  arguments: {
    format: "json"  // json | css
  }
});
// Returns design tokens (variables + paint styles)
```

## Common Patterns

### Design System Setup

```typescript
// 1. Create color variables
const colorCollection = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_variable_collection",
  arguments: { name: "Colors", initialModeName: "Light" }
});

await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "add_variable_mode",
  arguments: { collectionId: colorCollection.id, modeName: "Dark" }
});

const primaryVar = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_variable",
  arguments: {
    collectionId: colorCollection.id,
    name: "primary",
    resolvedType: "COLOR"
  }
});

// Set values for both modes
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "set_variable_value",
  arguments: {
    variableId: primaryVar.id,
    modeId: colorCollection.modes[0].id,  // Light
    value: "#0066FF"
  }
});

await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "set_variable_value",
  arguments: {
    variableId: primaryVar.id,
    modeId: colorCollection.modes[1].id,  // Dark
    value: "#66B3FF"
  }
});

// 2. Create component with bound variable
const buttonFrame = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_frame",
  arguments: {
    name: "Button",
    width: 200,
    height: 48,
    layoutMode: "HORIZONTAL",
    primaryAxisAlignItems: "CENTER",
    counterAxisAlignItems: "CENTER"
  }
});

await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "bind_variable_to_node",
  arguments: {
    nodeId: buttonFrame.id,
    variableId: primaryVar.id,
    field: "fillColor"
  }
});

const component = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_component",
  arguments: { nodeId: buttonFrame.id, name: "Button/Primary" }
});
```

### Bulk Content Updates

```typescript
// Find all text nodes in a page
const textNodes = await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "scan_text_nodes",
  arguments: { rootNodeId: "0:1" }
});

// Update placeholder text
for (const node of textNodes) {
  if (node.characters.includes("Lorem ipsum")) {
    await use_mcp_tool({
      server_name: "figma-mcp-go",
      tool_name: "set_text",
      arguments: {
        nodeId: node.id,
        characters: node.characters.replace("Lorem ipsum", "Real content")
      }
    });
  }
}

// Or use find_replace_text for efficiency
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "find_replace_text",
  arguments: {
    rootNodeId: "0:1",
    findText: "Lorem ipsum",
    replaceText: "Real content",
    useRegex: false
  }
});
```

### Component Organization

```typescript
// Group related frames
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "group_nodes",
  arguments: {
    nodeIds: ["123:1", "123:2", "123:3"],
    groupName: "Button Variants"
  }
});

// Create sections for organization
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "create_section",
  arguments: {
    name: "Components",
    x: 0,
    y: 0,
    width: 2000,
    height: 1000
  }
});

// Batch rename nodes
await use_mcp_tool({
  server_name: "figma-mcp-go",
  tool_name: "batch_rename_nodes",
  arguments: {
    rootNodeId: "0:1",
    findPattern: "Frame",
    replaceText: "Card",
    useRegex: false
  }
});
```

## Built-in MCP Prompts

Access strategic guidance through MCP prompts:

- `read_design_strategy`: Best practices for reading Figma designs efficiently
- `design_strategy`: Best practices for creating and modifying designs
- `text_replacement_strategy`: Chunked approach for replacing text across large designs
- `annotation_conversion_strategy`: Convert manual annotations to native Figma annotations
- `swap_overrides_instances`: Transfer overrides between component instances
- `reaction_to_connector_strategy`: Map prototype reactions into flow diagrams

## Troubleshooting

**Plugin not connecting:**
- Ensure Figma Desktop (not web) is running
- Verify the plugin is active in the current file
- Restart the plugin if idle for extended periods

**Tools returning errors:**
- Check node IDs are valid using `get_document` or `search_nodes`
- Ensure node types match tool requirements (e.g., `create_component` requires FRAME)
- Verify parent nodes exist before reparenting

**Font not loading:**
- Text creation automatically loads fonts, but custom fonts must be installed on your system
- Use `get_fonts` to see available fonts in the current file

**Export failures:**
- Ensure output directories exist and have write permissions
- For `save_screenshots` and `export_frames_to_pdf`, paths are relative to server working directory

**Performance with large files:**
- Use `maxDepth` in `get_design_context` to limit tree traversal
- Use `detail: "minimal"` to reduce payload size
- Query specific nodes with `get_node` or `search_nodes` instead of full page trees
