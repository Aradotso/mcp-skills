---
name: excalidraw-mcp-server
description: Build and deploy MCP servers that stream interactive Excalidraw diagrams with smooth viewport control
triggers:
  - create an excalidraw diagram in my MCP server
  - add excalidraw visualization to this tool
  - stream a hand-drawn diagram using MCP
  - integrate excalidraw with model context protocol
  - build an interactive diagram server
  - render excalidraw in my AI app
  - create MCP app with diagrams
  - add visual whiteboard to my MCP tool
---

# Excalidraw MCP Server Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill teaches AI agents how to build MCP servers that return interactive Excalidraw diagrams using the [MCP Apps extension](https://modelcontextprotocol.io/docs/extensions/apps). Excalidraw MCP enables streaming hand-drawn style diagrams directly into AI chat interfaces like Claude Desktop, ChatGPT, VS Code, and Goose.

## What It Does

The Excalidraw MCP server provides:
- **Interactive diagram rendering**: Returns HTML apps that render Excalidraw canvases
- **Streaming updates**: Supports progressive diagram building with viewport control
- **Fullscreen editing**: Users can click to edit diagrams in a modal
- **Remote & local deployment**: Run as Vercel serverless function or local Node.js process

## Installation

### Remote Server (Recommended)

Use the hosted version at `https://mcp.excalidraw.com`:

```json
{
  "mcpServers": {
    "excalidraw": {
      "url": "https://mcp.excalidraw.com"
    }
  }
}
```

### Local Installation

**From source:**

```bash
git clone https://github.com/excalidraw/excalidraw-mcp.git
cd excalidraw-mcp-app
pnpm install && pnpm run build
```

**Configure in Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "excalidraw": {
      "command": "node",
      "args": ["/path/to/excalidraw-mcp-app/dist/index.js", "--stdio"]
    }
  }
}
```

**From release:**

1. Download `excalidraw-mcp-app.mcpb` from [releases](https://github.com/excalidraw/excalidraw-mcp/releases)
2. Double-click to install in Claude Desktop

## Core Concepts

### MCP Apps Extension

MCP Apps is an official extension that lets servers return interactive HTML instead of just text. The response format:

```typescript
{
  content: [
    {
      type: "resource",
      resource: {
        uri: "excalidraw://diagram",
        mimeType: "text/html",
        text: "<html>...</html>"
      }
    }
  ]
}
```

### Excalidraw Scene Format

Excalidraw diagrams are JSON with `elements` and `appState`:

```typescript
interface ExcalidrawScene {
  type: "excalidraw";
  version: 2;
  source: string;
  elements: ExcalidrawElement[];
  appState: {
    viewBackgroundColor: string;
    currentItemFontFamily?: number;
    // ... viewport state
  };
  files?: Record<string, BinaryFileData>;
}
```

## Building an Excalidraw MCP Server

### Basic Server Structure

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  {
    name: "excalidraw-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "draw_diagram",
      description: "Create an interactive Excalidraw diagram",
      inputSchema: {
        type: "object",
        properties: {
          elements: {
            type: "array",
            description: "Excalidraw elements (rectangles, arrows, text, etc.)",
          },
          description: {
            type: "string",
            description: "What the diagram shows",
          },
        },
        required: ["elements"],
      },
    },
  ],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "draw_diagram") {
    const { elements, description } = request.params.arguments as any;
    
    const html = generateExcalidrawHTML(elements, description);
    
    return {
      content: [
        {
          type: "resource",
          resource: {
            uri: `excalidraw://diagram-${Date.now()}`,
            mimeType: "text/html",
            text: html,
          },
        },
      ],
    };
  }
  
  throw new Error("Unknown tool");
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Generating the HTML Response

```typescript
function generateExcalidrawHTML(
  elements: any[],
  description?: string
): string {
  const scene = {
    type: "excalidraw",
    version: 2,
    source: "mcp-server",
    elements: elements,
    appState: {
      viewBackgroundColor: "#ffffff",
      currentItemFontFamily: 1,
    },
    files: {},
  };
  
  const sceneJSON = JSON.stringify(scene);
  
  return `<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>${description || "Excalidraw Diagram"}</title>
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@excalidraw/excalidraw/dist/excalidraw.production.min.js"></script>
  <style>
    body { margin: 0; padding: 0; overflow: hidden; }
    #app { width: 100vw; height: 100vh; }
  </style>
</head>
<body>
  <div id="app"></div>
  <script>
    const { Excalidraw } = ExcalidrawLib;
    const scene = ${sceneJSON};
    
    const App = () => {
      return React.createElement(Excalidraw, {
        initialData: scene,
        viewModeEnabled: true,
        zenModeEnabled: false,
        gridModeEnabled: false,
      });
    };
    
    ReactDOM.render(
      React.createElement(App),
      document.getElementById('app')
    );
  </script>
</body>
</html>`;
}
```

### Creating Excalidraw Elements

**Rectangle:**

```typescript
{
  id: "rect-1",
  type: "rectangle",
  x: 100,
  y: 100,
  width: 200,
  height: 150,
  strokeColor: "#1e1e1e",
  backgroundColor: "#ffc9c9",
  fillStyle: "hachure",
  strokeWidth: 2,
  roughness: 1,
  opacity: 100,
  angle: 0,
  roundness: { type: 3 },
  seed: 12345,
  version: 1,
  versionNonce: 1,
  isDeleted: false,
  groupIds: [],
  boundElements: null,
}
```

**Text:**

```typescript
{
  id: "text-1",
  type: "text",
  x: 150,
  y: 150,
  width: 100,
  height: 25,
  text: "Hello World",
  fontSize: 20,
  fontFamily: 1, // 1=Virgil, 2=Helvetica, 3=Cascadia
  textAlign: "center",
  verticalAlign: "middle",
  strokeColor: "#1e1e1e",
  backgroundColor: "transparent",
  fillStyle: "hachure",
  strokeWidth: 2,
  roughness: 1,
  opacity: 100,
  angle: 0,
  seed: 12346,
  version: 1,
  versionNonce: 1,
  isDeleted: false,
  groupIds: [],
  containerId: null,
  originalText: "Hello World",
}
```

**Arrow:**

```typescript
{
  id: "arrow-1",
  type: "arrow",
  x: 300,
  y: 175,
  width: 150,
  height: 0,
  points: [[0, 0], [150, 0]],
  strokeColor: "#1e1e1e",
  backgroundColor: "transparent",
  fillStyle: "hachure",
  strokeWidth: 2,
  roughness: 1,
  opacity: 100,
  angle: 0,
  startBinding: { elementId: "rect-1", focus: 0, gap: 1 },
  endBinding: { elementId: "rect-2", focus: 0, gap: 1 },
  startArrowhead: null,
  endArrowhead: "arrow",
  seed: 12347,
  version: 1,
  versionNonce: 1,
  isDeleted: false,
  groupIds: [],
}
```

**Ellipse:**

```typescript
{
  id: "ellipse-1",
  type: "ellipse",
  x: 500,
  y: 100,
  width: 120,
  height: 120,
  strokeColor: "#2f9e44",
  backgroundColor: "#d3f9d8",
  fillStyle: "solid",
  strokeWidth: 2,
  roughness: 1,
  opacity: 100,
  angle: 0,
  seed: 12348,
  version: 1,
  versionNonce: 1,
  isDeleted: false,
  groupIds: [],
  boundElements: null,
}
```

## Advanced Patterns

### Architecture Diagram Generator

```typescript
function createArchitectureDiagram(
  components: { name: string; type: "user" | "server" | "database" }[]
): any[] {
  const elements: any[] = [];
  let xOffset = 100;
  
  components.forEach((comp, i) => {
    const id = `comp-${i}`;
    const y = comp.type === "user" ? 100 : comp.type === "server" ? 300 : 500;
    
    // Component box
    elements.push({
      id,
      type: "rectangle",
      x: xOffset,
      y,
      width: 150,
      height: 80,
      strokeColor: "#1e1e1e",
      backgroundColor: comp.type === "database" ? "#ffd43b" : "#a5d8ff",
      fillStyle: "hachure",
      strokeWidth: 2,
      roughness: 1,
      opacity: 100,
      angle: 0,
      roundness: { type: 3 },
      seed: Math.random() * 10000,
      version: 1,
      versionNonce: 1,
      isDeleted: false,
      groupIds: [],
      boundElements: null,
    });
    
    // Label
    elements.push({
      id: `text-${i}`,
      type: "text",
      x: xOffset + 10,
      y: y + 30,
      width: 130,
      height: 20,
      text: comp.name,
      fontSize: 16,
      fontFamily: 1,
      textAlign: "center",
      verticalAlign: "middle",
      strokeColor: "#1e1e1e",
      backgroundColor: "transparent",
      fillStyle: "hachure",
      strokeWidth: 2,
      roughness: 1,
      opacity: 100,
      angle: 0,
      seed: Math.random() * 10000,
      version: 1,
      versionNonce: 1,
      isDeleted: false,
      groupIds: [],
      containerId: id,
      originalText: comp.name,
    });
    
    // Arrow to next component
    if (i < components.length - 1) {
      elements.push({
        id: `arrow-${i}`,
        type: "arrow",
        x: xOffset + 75,
        y: y + 80,
        width: 0,
        height: 120,
        points: [[0, 0], [0, 120]],
        strokeColor: "#1e1e1e",
        backgroundColor: "transparent",
        fillStyle: "hachure",
        strokeWidth: 2,
        roughness: 1,
        opacity: 100,
        angle: 0,
        startBinding: { elementId: id, focus: 0, gap: 1 },
        endBinding: null,
        startArrowhead: null,
        endArrowhead: "arrow",
        seed: Math.random() * 10000,
        version: 1,
        versionNonce: 1,
        isDeleted: false,
        groupIds: [],
      });
    }
    
    xOffset += 250;
  });
  
  return elements;
}
```

### Streaming Diagrams with Updates

For progressive rendering, send multiple tool responses:

```typescript
async function streamDiagram(
  initialElements: any[],
  updates: any[][]
): Promise<void> {
  // Send initial state
  await sendToolResponse({
    content: [{
      type: "resource",
      resource: {
        uri: "excalidraw://stream-1",
        mimeType: "text/html",
        text: generateExcalidrawHTML(initialElements),
      },
    }],
  });
  
  // Stream updates
  for (const update of updates) {
    await new Promise(resolve => setTimeout(resolve, 500));
    
    const allElements = [...initialElements, ...update];
    await sendToolResponse({
      content: [{
        type: "resource",
        resource: {
          uri: "excalidraw://stream-1", // Same URI to update
          mimeType: "text/html",
          text: generateExcalidrawHTML(allElements),
        },
      }],
    });
  }
}
```

### Viewport Control

```typescript
function generateExcalidrawHTMLWithViewport(
  elements: any[],
  viewport: { x: number; y: number; zoom: number }
): string {
  const scene = {
    type: "excalidraw",
    version: 2,
    source: "mcp-server",
    elements,
    appState: {
      viewBackgroundColor: "#ffffff",
      scrollX: viewport.x,
      scrollY: viewport.y,
      zoom: { value: viewport.zoom },
    },
    files: {},
  };
  
  // ... rest of HTML generation
}
```

## Deployment

### Vercel Serverless Function

Create `api/mcp.ts`:

```typescript
import type { VercelRequest, VercelResponse } from "@vercel/node";
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";

const server = new Server(
  {
    name: "excalidraw-server",
    version: "1.0.0",
  },
  {
    capabilities: { tools: {} },
  }
);

// ... set up handlers (same as above)

export default async function handler(
  req: VercelRequest,
  res: VercelResponse
) {
  const transport = new SSEServerTransport("/message", res);
  await server.connect(transport);
  await transport.handleRequest(req);
}
```

**Deploy:**

```bash
vercel --prod
```

Your server will be at `https://your-project.vercel.app/api/mcp`.

### Docker Container

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile
COPY . .
RUN pnpm run build
CMD ["node", "dist/index.js", "--stdio"]
```

```bash
docker build -t excalidraw-mcp .
docker run -i excalidraw-mcp
```

## Troubleshooting

### Elements Not Rendering

**Problem:** Diagram appears blank.

**Solution:** Ensure all required element properties are present:

```typescript
{
  id: string,           // Must be unique
  type: string,         // "rectangle", "ellipse", "arrow", "text", etc.
  x: number,            // Position
  y: number,
  width: number,        // Dimensions
  height: number,
  seed: number,         // For roughness randomization
  version: number,      // Version tracking
  versionNonce: number, // Change detection
  isDeleted: boolean,   // Soft delete flag
  groupIds: string[],   // Group membership
}
```

### Arrow Bindings Not Working

**Problem:** Arrows don't connect to shapes.

**Solution:** Ensure `startBinding` and `endBinding` reference valid element IDs:

```typescript
{
  type: "arrow",
  startBinding: {
    elementId: "rect-1",  // Must match existing element
    focus: 0,              // -1 to 1, position on edge
    gap: 1,                // Distance from edge
  },
  endBinding: {
    elementId: "rect-2",
    focus: 0,
    gap: 1,
  },
}
```

### Text Inside Shapes Not Centered

**Problem:** Text doesn't appear inside rectangles.

**Solution:** Set `containerId` to bind text to container:

```typescript
{
  type: "text",
  containerId: "rect-1",  // ID of containing rectangle
  textAlign: "center",
  verticalAlign: "middle",
}
```

### Streaming Updates Not Appearing

**Problem:** Updated diagrams don't replace previous ones.

**Solution:** Use the same `uri` for all updates:

```typescript
const diagramURI = `excalidraw://diagram-${sessionId}`;

// All updates must use this same URI
resource: {
  uri: diagramURI,
  mimeType: "text/html",
  text: generateExcalidrawHTML(updatedElements),
}
```

### CORS Errors in Vercel Deployment

**Problem:** Client can't connect to Vercel MCP endpoint.

**Solution:** Add CORS headers in `vercel.json`:

```json
{
  "headers": [
    {
      "source": "/api/mcp",
      "headers": [
        { "key": "Access-Control-Allow-Origin", "value": "*" },
        { "key": "Access-Control-Allow-Methods", "value": "GET,POST,OPTIONS" },
        { "key": "Access-Control-Allow-Headers", "value": "Content-Type" }
      ]
    }
  ]
}
```

## Best Practices

1. **Generate unique IDs**: Use `crypto.randomUUID()` or timestamps
2. **Set reasonable viewport**: Calculate bounds from element positions
3. **Use consistent colors**: Define a palette for visual coherence
4. **Add descriptions**: Include metadata for accessibility
5. **Validate elements**: Check required properties before rendering
6. **Handle errors gracefully**: Return text fallbacks if diagram fails
7. **Test locally first**: Use `--stdio` mode before deploying remotely

## Resources

- [MCP Apps Documentation](https://modelcontextprotocol.io/docs/extensions/apps)
- [Excalidraw Element Spec](https://github.com/excalidraw/excalidraw/blob/master/src/element/types.ts)
- [Example Server](https://github.com/excalidraw/excalidraw-mcp)
- [MCP SDK](https://github.com/modelcontextprotocol/typescript-sdk)
