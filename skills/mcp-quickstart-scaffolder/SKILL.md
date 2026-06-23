---
name: mcp-quickstart-scaffolder
description: Scaffold, test, and deploy MCP servers in TypeScript or Python with example tools, OpenAPI import, and Cloudflare Workers deployment
triggers:
  - create a new MCP server
  - scaffold an MCP server from OpenAPI
  - generate MCP tools from an API
  - turn a curl command into an MCP tool
  - deploy an MCP server to Cloudflare Workers
  - set up a testable MCP server quickly
  - build an MCP server with example tools
  - convert an API to an MCP server
---

# mcp-quickstart-scaffolder

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`mcp-quickstart` is a scaffolding tool that generates production-ready Model Context Protocol (MCP) servers in 30 seconds. It creates servers with example tools, resources, prompts, tests, and the MCP Inspector pre-configured. Supports TypeScript and Python, stdio and HTTP transports, and can automatically generate MCP tools from OpenAPI specs or curl commands. Also supports one-command deployment to Cloudflare Workers.

## Installation & Usage

### Quick start (interactive)

```bash
npm create mcp-quickstart@latest
```

### Non-interactive scaffolding

```bash
# TypeScript stdio server
npm create mcp-quickstart@latest my-server -- --lang ts -y

# Python stdio server
npx mcp-quickstart weather-server --lang python -y

# HTTP transport
npm create mcp-quickstart@latest http-server -- --lang ts --transport http -y
```

### Generate from OpenAPI

```bash
# From a URL
npx mcp-quickstart petstore-mcp --from-openapi https://petstore3.swagger.io/api/v3/openapi.json

# From a local file
npx mcp-quickstart api-server --from-openapi ./openapi.yaml --lang ts -y
```

### Generate from curl

```bash
# From a curl command string
npx mcp-quickstart search-tool --from-curl "curl https://api.example.com/v1/search?q=test -H 'Authorization: Bearer TOKEN'"

# From a file containing curl
npx mcp-quickstart my-tool --from-curl curl-command.txt
```

### Deploy to Cloudflare Workers

```bash
# Scaffold with Cloudflare transport
npx mcp-quickstart edge-server --transport cloudflare -y
cd edge-server
npm install
npx wrangler login
npm run deploy
```

### OpenAPI → Cloudflare Workers (end-to-end)

```bash
# Generate API-backed remote MCP server and deploy
npx mcp-quickstart api-edge \
  --from-openapi https://petstore3.swagger.io/api/v3/openapi.json \
  --transport cloudflare -y
cd api-edge
npm install
npx wrangler secret put API_AUTH_VALUE  # if needed
npm run deploy
```

## CLI Options

```
npm create mcp-quickstart@latest [name] [options]

Options:
  --from-openapi <path|url>  Generate one MCP tool per API operation from OpenAPI spec
  --from-curl <cmd|file>     Turn a single curl command into an MCP tool
  --lang <ts|python>         Language (default: prompt)
  --transport <stdio|http|cloudflare>  Transport/deploy target (default: prompt)
  --examples <bool>          Include example primitives (default: true)
  --yes, -y                  Accept defaults, skip prompts
  --help, -h                 Show help
```

## Generated Project Structure

### TypeScript stdio

```
my-server/
├── src/
│   ├── index.ts       # MCP server setup, tool/resource/prompt registration
│   ├── tools.ts       # Pure business logic (testable)
│   └── types.ts       # TypeScript types
├── __tests__/
│   └── tools.test.ts  # Unit tests (passing out of the box)
├── .env.example       # Environment variable template
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

### OpenAPI-generated

```
petstore-mcp/
├── src/
│   ├── index.ts       # One registerTool(...) per API operation
│   ├── http.ts        # Generic HTTP client with auth
│   └── types.ts       # Generated zod schemas from OpenAPI
├── .env.example       # API_BASE_URL, API_AUTH_HEADER, API_AUTH_VALUE
└── package.json
```

## Development Workflow

### After scaffolding

```bash
cd my-server
npm install              # or: uv sync (Python)
npm run dev              # Start server on stdio
npm run inspect          # Open MCP Inspector
npm test                 # Run tests
```

### TypeScript build

```bash
npm run build            # Compile to dist/
node dist/index.js       # Run built server
```

### Python development

```bash
uv sync                  # Install dependencies
uv run my-server         # Run server
uv run pytest            # Run tests
```

## Example Tool Implementation (TypeScript)

Generated `src/tools.ts`:

```typescript
export interface TextStatsInput {
  text: string;
}

export interface TextStatsOutput {
  characters: number;
  words: number;
  lines: number;
  sentences: number;
}

export function calculateTextStats(input: TextStatsInput): TextStatsOutput {
  const { text } = input;
  
  const characters = text.length;
  const words = text.trim() === '' ? 0 : text.trim().split(/\s+/).length;
  const lines = text.split('\n').length;
  const sentences = text.split(/[.!?]+/).filter(s => s.trim().length > 0).length;

  return { characters, words, lines, sentences };
}
```

Generated `src/index.ts` (tool registration):

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { calculateTextStats } from "./tools.js";

const server = new Server(
  {
    name: "my-server",
    version: "0.1.0",
  },
  {
    capabilities: {
      tools: {},
      resources: {},
      prompts: {},
    },
  }
);

// Tool registration
server.setRequestHandler("tools/list", async () => ({
  tools: [
    {
      name: "text_stats",
      description: "Calculate character, word, line, and sentence counts",
      inputSchema: {
        type: "object",
        properties: {
          text: { type: "string", description: "Text to analyze" },
        },
        required: ["text"],
      },
    },
  ],
}));

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "text_stats") {
    const validated = z.object({ text: z.string() }).parse(request.params.arguments);
    const result = calculateTextStats(validated);
    return {
      content: [{ type: "text", text: JSON.stringify(result, null, 2) }],
    };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("MCP server running on stdio");
}

main().catch(console.error);
```

## Example Resource Implementation

```typescript
server.setRequestHandler("resources/list", async () => ({
  resources: [
    {
      uri: "greeting://{name}",
      name: "Personalized greeting",
      description: "A dynamic greeting resource",
      mimeType: "text/plain",
    },
  ],
}));

server.setRequestHandler("resources/read", async (request) => {
  const uri = request.params.uri;
  if (uri.startsWith("greeting://")) {
    const name = uri.replace("greeting://", "");
    return {
      contents: [
        {
          uri,
          mimeType: "text/plain",
          text: `Hello, ${name}! Welcome to the MCP server.`,
        },
      ],
    };
  }
  throw new Error(`Unknown resource: ${uri}`);
});
```

## Example Prompt Implementation

```typescript
server.setRequestHandler("prompts/list", async () => ({
  prompts: [
    {
      name: "summarize",
      description: "Create a summary of provided text",
      arguments: [
        {
          name: "text",
          description: "Text to summarize",
          required: true,
        },
      ],
    },
  ],
}));

server.setRequestHandler("prompts/get", async (request) => {
  if (request.params.name === "summarize") {
    const text = request.params.arguments?.text || "";
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Please provide a concise summary of the following text:\n\n${text}`,
          },
        },
      ],
    };
  }
  throw new Error(`Unknown prompt: ${request.params.name}`);
});
```

## OpenAPI-Generated Tool Example

When you use `--from-openapi`, the tool looks like this:

```typescript
// src/index.ts
import { callApi } from "./http.js";

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "getPetById") {
    const validated = z.object({ 
      petId: z.number().int() 
    }).parse(request.params.arguments);
    
    const result = await callApi({
      method: "GET",
      path: `/pet/${validated.petId}`,
    });
    
    return {
      content: [{ type: "text", text: JSON.stringify(result, null, 2) }],
    };
  }
  // ... other tools
});
```

```typescript
// src/http.ts
import { config } from "dotenv";
config();

const API_BASE_URL = process.env.API_BASE_URL || "";
const API_AUTH_HEADER = process.env.API_AUTH_HEADER || "";
const API_AUTH_VALUE = process.env.API_AUTH_VALUE || "";

interface CallApiOptions {
  method: string;
  path: string;
  query?: Record<string, any>;
  body?: any;
}

export async function callApi(options: CallApiOptions): Promise<any> {
  const { method, path, query, body } = options;
  
  const url = new URL(path, API_BASE_URL);
  if (query) {
    Object.entries(query).forEach(([key, value]) => {
      if (value !== undefined) url.searchParams.set(key, String(value));
    });
  }

  const headers: Record<string, string> = {
    "Content-Type": "application/json",
  };
  if (API_AUTH_HEADER && API_AUTH_VALUE) {
    headers[API_AUTH_HEADER] = API_AUTH_VALUE;
  }

  const response = await fetch(url.toString(), {
    method,
    headers,
    body: body ? JSON.stringify(body) : undefined,
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${await response.text()}`);
  }

  return response.json();
}
```

## curl-Generated Tool Example

When you use `--from-curl`:

```bash
npx mcp-quickstart search-tool --from-curl "curl 'https://api.example.com/v1/search?q=test&limit=10' -H 'Authorization: Bearer SECRET'"
```

Generates:

```typescript
// src/index.ts
server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "api_call") {
    const validated = z.object({
      q: z.string().optional(),
      limit: z.string().optional(),
    }).parse(request.params.arguments);
    
    const result = await callApi({
      method: "GET",
      path: "/v1/search",
      query: validated,
    });
    
    return {
      content: [{ type: "text", text: JSON.stringify(result, null, 2) }],
    };
  }
});
```

With `.env.example`:

```bash
API_BASE_URL=https://api.example.com
API_AUTH_HEADER=Authorization
API_AUTH_VALUE=Bearer YOUR_TOKEN_HERE
```

## Configuration

### Environment Variables

```bash
# .env (for OpenAPI/curl-generated servers)
API_BASE_URL=https://api.example.com
API_AUTH_HEADER=Authorization
API_AUTH_VALUE=Bearer YOUR_API_KEY
```

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/absolute/path/to/my-server/dist/index.js"]
    }
  }
}
```

Or for development:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["--loader", "tsx", "/absolute/path/to/my-server/src/index.ts"]
    }
  }
}
```

### Cursor

Add to `.cursorrules` or workspace settings:

```json
{
  "mcp": {
    "servers": {
      "my-server": {
        "command": "node",
        "args": ["/absolute/path/to/my-server/dist/index.js"]
      }
    }
  }
}
```

## Cloudflare Workers Deployment

### Initial setup

```bash
npx mcp-quickstart edge-server --transport cloudflare -y
cd edge-server
npm install
npx wrangler login
```

### Configure secrets (for API-backed servers)

```bash
npx wrangler secret put API_AUTH_VALUE
# Enter your API key when prompted
```

### Deploy

```bash
npm run deploy
# → https://edge-server.<your-subdomain>.workers.dev/mcp
```

### Use the deployed server

Point your AI client to the `/mcp` endpoint:

```json
{
  "mcpServers": {
    "edge-server": {
      "url": "https://edge-server.<your-subdomain>.workers.dev/mcp",
      "transport": "streamable-http"
    }
  }
}
```

## Testing

### TypeScript

```bash
npm test                 # Run all tests
npm test -- --watch      # Watch mode
npm test -- --coverage   # Coverage report
```

Example test from generated project:

```typescript
// __tests__/tools.test.ts
import { describe, it, expect } from "vitest";
import { calculateTextStats } from "../src/tools.js";

describe("calculateTextStats", () => {
  it("counts characters correctly", () => {
    const result = calculateTextStats({ text: "Hello, world!" });
    expect(result.characters).toBe(13);
  });

  it("counts words correctly", () => {
    const result = calculateTextStats({ text: "Hello world" });
    expect(result.words).toBe(2);
  });

  it("handles empty text", () => {
    const result = calculateTextStats({ text: "" });
    expect(result.characters).toBe(0);
    expect(result.words).toBe(0);
  });
});
```

### Python

```bash
uv run pytest            # Run all tests
uv run pytest --cov      # Coverage
```

## Common Patterns

### Adding a new tool

1. Add business logic to `src/tools.ts`:

```typescript
export interface CalculateSumInput {
  numbers: number[];
}

export function calculateSum(input: CalculateSumInput): number {
  return input.numbers.reduce((a, b) => a + b, 0);
}
```

2. Register in `src/index.ts`:

```typescript
server.setRequestHandler("tools/list", async () => ({
  tools: [
    // ... existing tools
    {
      name: "calculate_sum",
      description: "Sum an array of numbers",
      inputSchema: {
        type: "object",
        properties: {
          numbers: {
            type: "array",
            items: { type: "number" },
            description: "Numbers to sum",
          },
        },
        required: ["numbers"],
      },
    },
  ],
}));

server.setRequestHandler("tools/call", async (request) => {
  // ... existing tools
  if (request.params.name === "calculate_sum") {
    const validated = z.object({
      numbers: z.array(z.number()),
    }).parse(request.params.arguments);
    const result = calculateSum(validated);
    return {
      content: [{ type: "text", text: String(result) }],
    };
  }
});
```

3. Add tests in `__tests__/tools.test.ts`:

```typescript
it("sums numbers correctly", () => {
  expect(calculateSum({ numbers: [1, 2, 3] })).toBe(6);
});
```

### Error handling in tools

```typescript
server.setRequestHandler("tools/call", async (request) => {
  try {
    if (request.params.name === "my_tool") {
      // ... tool logic
    }
  } catch (error) {
    return {
      content: [
        {
          type: "text",
          text: `Error: ${error instanceof Error ? error.message : String(error)}`,
        },
      ],
      isError: true,
    };
  }
});
```

### HTTP transport setup

For stdio → HTTP migration:

```typescript
// Replace StdioServerTransport with:
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";
import express from "express";

const app = express();
app.use(express.json());

app.post("/mcp", async (req, res) => {
  const transport = new SSEServerTransport("/mcp", res);
  await server.connect(transport);
  // Handle SSE connection
});

app.listen(3000, () => {
  console.log("MCP server on http://localhost:3000/mcp");
});
```

## Troubleshooting

### "Module not found" errors

Ensure `tsconfig.json` has:

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "node",
    "esModuleInterop": true
  }
}
```

And `package.json` has:

```json
{
  "type": "module"
}
```

### Inspector not connecting

```bash
# Kill existing inspector process
pkill -f mcp-inspector

# Restart
npm run inspect
```

### OpenAPI generation errors

- Ensure the spec is valid OpenAPI 3.x (validate at editor.swagger.io)
- Check that `$ref` paths are resolvable
- For remote URLs, ensure they're publicly accessible

### Cloudflare deployment fails

```bash
# Ensure you're logged in
npx wrangler whoami

# Check wrangler.toml is present
cat wrangler.toml

# Try manual deployment
npx wrangler deploy
```

### API auth not working

For OpenAPI/curl-generated servers, check `.env`:

```bash
# Verify variables are set
echo $API_BASE_URL
echo $API_AUTH_VALUE

# For Workers, use secrets not env vars:
npx wrangler secret put API_AUTH_VALUE
npx wrangler secret list
```

### Python tests not found

```bash
# Ensure pytest is in dependencies
uv add --dev pytest

# Run from project root
uv run pytest
```

## Advanced: Custom Templates

To modify the generated scaffolds, clone the repo:

```bash
git clone https://github.com/G12789/mcp-quickstart.git
cd mcp-quickstart
npm install

# Templates are in templates/<lang>-<transport>/
# Files prefixed with _ (e.g., _gitignore) are renamed on generation

# Test your changes
node src/index.js test-out --lang ts -y
```
