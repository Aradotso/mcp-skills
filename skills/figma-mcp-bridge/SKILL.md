---
name: figma-mcp-bridge
description: Bypass Figma API limits by streaming live document data from Figma plugin to AI tools via MCP server for design-to-code workflows
triggers:
  - connect to figma design file
  - get figma document structure
  - extract design tokens from figma
  - export figma components to code
  - read figma styles and variables
  - query figma selection and nodes
  - create frames and text in figma
  - apply animations to figma nodes
---

# Figma MCP Bridge

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Figma MCP Bridge is a plugin + MCP server combo that streams live Figma document data to AI tools without hitting Figma's restrictive API rate limits (6 requests/month for free users). It supports **multiple simultaneous Figma files**, read operations (get nodes, styles, variables, screenshots), and an opt-in set of **write tools** for safe agent-driven edits.

### Key Capabilities

- **Read Tools**: Document trees, selection, nodes by ID, styles, variables, design context, metadata
- **Screenshot Export**: PNG/SVG/JPG/PDF, base64-encoded or saved to local filesystem
- **Write Tools**: Create/edit frames, text, shapes, images; set fills, strokes, effects, auto-layout; duplicate, reparent, group, delete nodes
- **Animation Tools** (beta): Apply/remove animation styles, keyframe tracks, timeline control
- **Multi-File Support**: Connect to multiple Figma files simultaneously, query by `fileKey`

## Installation

### 1. Add MCP Server to AI Tool

Add to your AI tool's MCP configuration (Cursor, Claude Desktop, Windsurf, etc.):

```json
{
  "mcpServers": {
    "figma-bridge": {
      "command": "npx",
      "args": ["-y", "@gethopp/figma-mcp-bridge"]
    }
  }
}
```

For **local development**:

```json
{
  "mcpServers": {
    "figma-bridge": {
      "command": "node",
      "args": ["/absolute/path/to/figma-mcp-bridge/server/dist/index.js"]
    }
  }
}
```

### 2. Install Figma Plugin

1. Download `plugin/` folder from [latest release](https://github.com/gethopp/figma-mcp-bridge/releases)
2. In Figma: `Plugins > Development > Import plugin from manifest`
3. Select `manifest.json` from the downloaded `plugin/` folder

### 3. Connect Plugin to File

1. Open your Figma file
2. Run the imported plugin: `Plugins > Development > Figma MCP Bridge`
3. Plugin UI shows "Connected" when WebSocket establishes

For **multi-file workflows**, repeat step 3 in each Figma file. The bridge maintains all connections simultaneously.

## Core Tools

### File & Metadata

#### List Connected Files

```typescript
// When multiple Figma files are connected
const files = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "list_files",
  arguments: {}
});
// Returns: [{ fileKey: "abc123", name: "Design System" }, ...]
```

#### Get File Metadata

```typescript
const metadata = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_metadata",
  arguments: {
    fileKey: "abc123" // Optional if only one file connected
  }
});
// Returns: { name: "Mobile App", pages: [...], currentPage: {...} }
```

### Reading Document Structure

#### Get Full Document Tree

```typescript
const doc = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_document",
  arguments: {
    fileKey: "abc123" // Optional
  }
});
// Returns complete node tree of current page
```

#### Get Design Context (Depth-Limited Tree)

```typescript
// Optimized for AI understanding, limits depth to reduce token usage
const context = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_design_context",
  arguments: {
    maxDepth: 3, // Default: 3
    fileKey: "abc123"
  }
});
```

#### Get Current Selection

```typescript
const selection = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_selection",
  arguments: { fileKey: "abc123" }
});
// Returns array of selected nodes with full properties
```

#### Get Specific Node by ID

```typescript
// Node IDs use colon format: "4029:12345"
const node = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_node",
  arguments: {
    nodeId: "4029:12345",
    fileKey: "abc123"
  }
});
```

### Design Tokens & Styles

#### Get All Styles

```typescript
const styles = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_styles",
  arguments: { fileKey: "abc123" }
});
// Returns: { paintStyles: [...], textStyles: [...], effectStyles: [...], gridStyles: [...] }
```

#### Get Variable Definitions (Design Tokens)

```typescript
const variables = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_variable_defs",
  arguments: { fileKey: "abc123" }
});
// Returns all variable collections, modes, and values
```

### Screenshots & Export

#### Get Screenshot (Base64)

```typescript
const screenshot = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_screenshot",
  arguments: {
    nodeIds: ["4029:12345", "4029:67890"],
    format: "PNG", // PNG | SVG | JPG | PDF
    scale: 2, // Export scale multiplier
    fileKey: "abc123"
  }
});
// Returns base64-encoded image data
```

#### Save Screenshots to Filesystem

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "save_screenshots",
  arguments: {
    nodeIds: ["4029:12345"],
    outputDir: "/path/to/output", // Absolute or relative to MCP server working dir
    format: "PNG",
    scale: 2,
    fileKey: "abc123"
  }
});
// Saves files as: output/node-4029-12345.png
```

## Write Tools (Design Editor Only)

**Important**: Write tools only work when plugin is opened in Figma's **design editor**. Dev Mode is read-only and returns clear errors.

### Creating Nodes

#### Create Frame

```typescript
const frame = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "create_frame",
  arguments: {
    name: "Card Container",
    x: 100,
    y: 200,
    width: 320,
    height: 480,
    parentId: "4029:100", // Optional, defaults to current page
    fileKey: "abc123"
  }
});
// Returns: { id: "4029:12345", message: "Frame created" }
```

#### Create Text

```typescript
const text = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "create_text",
  arguments: {
    content: "Hello World",
    x: 150,
    y: 250,
    fontFamily: "Inter", // Optional, defaults to Inter Regular
    fontStyle: "Bold",
    fontSize: 24,
    parentId: "4029:100",
    fileKey: "abc123"
  }
});
```

#### Create Shape

```typescript
// Rectangle, Ellipse, or Line
const shape = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "create_shape",
  arguments: {
    type: "RECTANGLE", // RECTANGLE | ELLIPSE | LINE
    name: "Background",
    x: 0,
    y: 0,
    width: 320,
    height: 480,
    parentId: "4029:100",
    fileKey: "abc123"
  }
});
```

#### Create Image

```typescript
// From local path, URL, or data URI
const image = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "create_image",
  arguments: {
    source: "/path/to/image.png", // Local path (relative to MCP server), URL, or data:image/...
    name: "Hero Image",
    x: 0,
    y: 0,
    width: 800,
    height: 600,
    parentId: "4029:100",
    fileKey: "abc123"
  }
});
```

### Modifying Nodes

#### Set Node Properties

```typescript
// Patch common properties: name, position, size, visibility, opacity, corner radius
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_node_properties",
  arguments: {
    nodeId: "4029:12345",
    name: "Updated Frame",
    x: 200,
    y: 300,
    width: 400,
    height: 600,
    visible: true,
    opacity: 0.9,
    cornerRadius: 8,
    fileKey: "abc123"
  }
});
```

#### Set Text Content

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_text_content",
  arguments: {
    nodeId: "4029:12345",
    content: "New text content",
    fileKey: "abc123"
  }
});
// Automatically loads required fonts
```

#### Set Text Properties

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_text_properties",
  arguments: {
    nodeId: "4029:12345",
    fontFamily: "Roboto",
    fontStyle: "Medium",
    fontSize: 18,
    textAlignHorizontal: "CENTER", // LEFT | CENTER | RIGHT | JUSTIFIED
    textAlignVertical: "TOP", // TOP | CENTER | BOTTOM
    textAutoResize: "WIDTH_AND_HEIGHT", // NONE | WIDTH_AND_HEIGHT | HEIGHT
    color: { r: 0.2, g: 0.3, b: 0.4, a: 1.0 },
    x: 100,
    y: 200,
    width: 300,
    height: 100,
    fileKey: "abc123"
  }
});
```

#### Set Solid Fill

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_solid_fill",
  arguments: {
    nodeId: "4029:12345",
    color: { r: 0.1, g: 0.5, b: 0.9, a: 1.0 },
    paintType: "fills", // "fills" | "strokes"
    fileKey: "abc123"
  }
});
```

#### Set Gradient Fill

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_gradient_fill",
  arguments: {
    nodeId: "4029:12345",
    gradientType: "GRADIENT_LINEAR", // GRADIENT_LINEAR | GRADIENT_RADIAL | GRADIENT_ANGULAR | GRADIENT_DIAMOND
    stops: [
      { position: 0, color: { r: 1, g: 0, b: 0, a: 1 } },
      { position: 1, color: { r: 0, g: 0, b: 1, a: 1 } }
    ],
    paintType: "fills",
    fileKey: "abc123"
  }
});
```

#### Set Effects (Shadows, Blurs)

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_effects",
  arguments: {
    nodeId: "4029:12345",
    effects: [
      {
        type: "DROP_SHADOW",
        color: { r: 0, g: 0, b: 0, a: 0.25 },
        offset: { x: 0, y: 4 },
        radius: 8,
        visible: true
      },
      {
        type: "LAYER_BLUR",
        radius: 4,
        visible: true
      }
    ],
    fileKey: "abc123"
  }
});
// Effect types: DROP_SHADOW | INNER_SHADOW | LAYER_BLUR | BACKGROUND_BLUR
```

#### Set Stroke Properties

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_stroke_properties",
  arguments: {
    nodeId: "4029:12345",
    strokeWeight: 2,
    strokeAlign: "INSIDE", // CENTER | INSIDE | OUTSIDE
    dashPattern: [5, 3], // Array of dash/gap lengths, [] for solid
    strokeCap: "ROUND", // NONE | ROUND | SQUARE | ARROW_LINES | ARROW_EQUILATERAL
    strokeJoin: "ROUND", // MITER | BEVEL | ROUND
    fileKey: "abc123"
  }
});
```

#### Set Auto Layout

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_auto_layout",
  arguments: {
    nodeId: "4029:12345",
    layoutMode: "VERTICAL", // NONE | HORIZONTAL | VERTICAL
    primaryAxisAlignItems: "CENTER", // MIN | CENTER | MAX | SPACE_BETWEEN
    counterAxisAlignItems: "CENTER", // MIN | CENTER | MAX
    paddingLeft: 16,
    paddingRight: 16,
    paddingTop: 12,
    paddingBottom: 12,
    itemSpacing: 8,
    primaryAxisSizingMode: "AUTO", // FIXED | AUTO
    counterAxisSizingMode: "FIXED",
    layoutWrap: "NO_WRAP", // NO_WRAP | WRAP
    fileKey: "abc123"
  }
});
```

### Node Operations

#### Duplicate Nodes

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "duplicate_nodes",
  arguments: {
    nodeIds: ["4029:12345", "4029:67890"],
    fileKey: "abc123"
  }
});
// Returns: { duplicatedIds: ["4029:11111", "4029:22222"] }
```

#### Reparent Nodes

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "reparent_nodes",
  arguments: {
    nodeIds: ["4029:12345"],
    newParentId: "4029:100",
    index: 0, // Optional insertion index
    fileKey: "abc123"
  }
});
```

#### Group Nodes

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "group_nodes",
  arguments: {
    nodeIds: ["4029:12345", "4029:67890"], // Must share same parent
    name: "Card Group",
    fileKey: "abc123"
  }
});
// Returns: { groupId: "4029:99999" }
```

#### Ungroup Node

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "ungroup_node",
  arguments: {
    nodeId: "4029:12345", // Must be GROUP or FRAME
    fileKey: "abc123"
  }
});
// Children move up to parent, group is deleted
```

#### Delete Nodes

```typescript
// Requires explicit confirmation
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "delete_nodes",
  arguments: {
    nodeIds: ["4029:12345"],
    confirm: true, // REQUIRED
    fileKey: "abc123"
  }
});
```

### Selection & Viewport

#### Set Selection

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "set_selection",
  arguments: {
    nodeIds: ["4029:12345", "4029:67890"],
    fileKey: "abc123"
  }
});
// Works in both design editor and Dev Mode
```

#### Scroll and Zoom into View

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "scroll_and_zoom_into_view",
  arguments: {
    nodeIds: ["4029:12345"],
    fileKey: "abc123"
  }
});
// Frames viewport around specified nodes
```

## Animation Tools (Beta)

#### Get Motion Styles

```typescript
const styles = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_motion_styles",
  arguments: { fileKey: "abc123" }
});
// Returns list of available animation presets
```

#### Apply Animation Style

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "apply_animation_style",
  arguments: {
    nodeId: "4029:12345",
    styleId: "motion-style-id",
    fileKey: "abc123"
  }
});
```

#### Apply Manual Keyframe Track

```typescript
const result = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "apply_manual_keyframe_track",
  arguments: {
    nodeId: "4029:12345",
    property: "x", // x, y, width, height, opacity, rotation, etc.
    keyframes: [
      { time: 0, value: 0 },
      { time: 1, value: 100 }
    ],
    fileKey: "abc123"
  }
});
```

## Common Patterns

### Extract Design System Tokens

```typescript
// Get all styles and variables for design system documentation
const [styles, variables] = await Promise.all([
  use_mcp_tool({
    server_name: "figma-bridge",
    tool_name: "get_styles",
    arguments: { fileKey: "design-system-key" }
  }),
  use_mcp_tool({
    server_name: "figma-bridge",
    tool_name: "get_variable_defs",
    arguments: { fileKey: "design-system-key" }
  })
]);

// Convert to CSS custom properties
const cssVars = variables.collections.flatMap(collection =>
  collection.modes.flatMap(mode =>
    Object.entries(mode.values).map(([name, value]) =>
      `--${name}: ${value};`
    )
  )
);
```

### Build Component from Selection

```typescript
// 1. Get currently selected nodes
const selection = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_selection",
  arguments: {}
});

// 2. Get detailed node data
const nodeDetails = await Promise.all(
  selection.map(node =>
    use_mcp_tool({
      server_name: "figma-bridge",
      tool_name: "get_node",
      arguments: { nodeId: node.id }
    })
  )
);

// 3. Generate React component code from node structure
// (Your code generation logic here)
```

### Create Slide Deck from Template

```typescript
// 1. Create title slide
const titleSlide = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "create_frame",
  arguments: {
    name: "Slide 1",
    x: 0,
    y: 0,
    width: 1920,
    height: 1080
  }
});

// 2. Add title text
await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "create_text",
  arguments: {
    content: "Presentation Title",
    parentId: titleSlide.id,
    x: 960,
    y: 400,
    fontFamily: "Inter",
    fontStyle: "Bold",
    fontSize: 72
  }
});

// 3. Duplicate slide for content pages
const duplicated = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "duplicate_nodes",
  arguments: { nodeIds: [titleSlide.id] }
});

// 4. Update duplicated slide content
// ...
```

### Export All Components as SVG

```typescript
// 1. Get document tree
const doc = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_document",
  arguments: {}
});

// 2. Find all component nodes
const components = findNodesByType(doc, "COMPONENT");

// 3. Export each as SVG
for (const component of components) {
  await use_mcp_tool({
    server_name: "figma-bridge",
    tool_name: "save_screenshots",
    arguments: {
      nodeIds: [component.id],
      outputDir: "./components",
      format: "SVG"
    }
  });
}
```

### Multi-File Design Token Sync

```typescript
// 1. List all connected files
const files = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "list_files",
  arguments: {}
});

// 2. Get variables from source design system
const sourceVars = await use_mcp_tool({
  server_name: "figma-bridge",
  tool_name: "get_variable_defs",
  arguments: { fileKey: files[0].fileKey } // Design system file
});

// 3. For each project file, validate variable usage
for (const file of files.slice(1)) {
  const projectDoc = await use_mcp_tool({
    server_name: "figma-bridge",
    tool_name: "get_document",
    arguments: { fileKey: file.fileKey }
  });
  // Check for inconsistent variable references
  // ...
}
```

## Troubleshooting

### Plugin Shows "Disconnected"

1. Ensure MCP server is running (check AI tool logs)
2. Restart the Figma plugin
3. Check WebSocket port 1994 is not blocked by firewall
4. For local dev, verify `server/dist/index.js` path is correct

### "Cannot edit in Dev Mode" Error

Write tools only work in Figma's design editor. Switch from Dev Mode to design view:
- Click "Design" tab at top of Figma window
- Or close Dev Mode sidebar

### Font Loading Errors

When creating/editing text nodes, the plugin must load fonts. Ensure:
- Font family/style names are exact (case-sensitive)
- Fonts are available in your Figma account
- For new text nodes without font specified, `Inter Regular` is used as default

### Multi-File `fileKey` Issues

- Use `list_files` to see all connected files and their keys
- If `fileKey` is omitted and multiple files are connected, tools may fail
- File key format: short alphanumeric string (e.g., `"abc123XYZ"`), not full URL

### Screenshot Export Fails

- Ensure nodes exist and are visible
- For `save_screenshots`, check output directory permissions
- Large exports may timeout — reduce scale or node count
- SVG export only works for vector-compatible nodes

### Node ID Format

Node IDs must use **colon format**: `"4029:12345"`, not `4029-12345` or other variants. Copy IDs directly from:
- `get_selection` results
- `get_document` tree
- Figma plugin Dev Mode inspector

### Leader Election Issues

If multiple MCP server instances are running (e.g., multiple AI tools):
- Only one becomes "leader" and accepts WebSocket connections
- Others become "followers" and proxy requests to leader via HTTP
- If leader crashes, a follower automatically promotes itself
- Check logs for "Elected as leader" or "Running as follower" messages

### Local Development Build Issues

```bash
# Server build
cd server
npm install
npm run build

# Plugin build
cd plugin
bun install
bun run build

# If bun not installed
npm install -g bun
```

Ensure `server/dist/index.js` exists after build before configuring MCP client.
