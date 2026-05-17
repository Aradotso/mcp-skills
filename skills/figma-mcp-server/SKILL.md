---
name: figma-mcp-server
description: Connect AI coding agents to Figma designs via MCP to generate code from frames, extract design tokens, use Code Connect, and write directly to the canvas
triggers:
  - "implement this Figma design"
  - "generate code from this Figma frame"
  - "get design tokens from Figma"
  - "extract variables from this Figma file"
  - "create a component in Figma"
  - "use Code Connect for this design"
  - "convert this frame to code"
  - "update Figma design with code"
---

# Figma MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Figma MCP server connects AI coding agents directly to Figma, enabling you to generate code from designs, extract design tokens, leverage Code Connect mappings, and write native Figma content back to the canvas. This skill teaches agents how to work with Figma files through the Model Context Protocol.

## What It Does

- **Generate code from Figma frames** — Select any frame and get structured code (React + Tailwind by default, adaptable to any framework)
- **Extract design context** — Pull variables, components, styles, and layout data
- **Code Connect integration** — Reference actual component implementations from your codebase
- **Write to canvas** (beta) — Create and modify frames, components, variables, and auto layout directly in Figma
- **Import from web** (rolling out) — Convert web pages into Figma designs

## Installation

### VS Code (requires GitHub Copilot)

1. Press `⌘ Shift P` → search `MCP:Add Server`
2. Select `HTTP`
3. Enter URL: `https://mcp.figma.com/mcp`
4. Enter server ID: `figma`
5. Choose global or workspace scope

Verify in `mcp.json`:
```json
{
  "servers": {
    "figma": {
      "type": "http",
      "url": "https://mcp.figma.com/mcp"
    }
  }
}
```

Test: Open agent mode (`⌥⌘B` or `⌃⌘I`) and type `#get_design_context`

### Cursor

**Recommended:** Install the Figma plugin (includes MCP + skills):
```
/add-plugin figma
```

**Manual:**
1. Open **Cursor → Settings → Cursor Settings → MCP**
2. Click **+ Add new global MCP server**
3. Add:
```json
{
  "mcpServers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp"
    }
  }
}
```

### Claude Code

**Recommended:** Install official plugin:
```bash
claude plugin install figma@claude-plugins-official
```

**Manual:**
```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

Authenticate:
```bash
# Inside claude CLI
/mcp auth figma
```

### Gemini CLI

```bash
gemini extensions install https://github.com/figma/mcp-server-guide
```

Authenticate:
```bash
gemini
# Then inside CLI:
/mcp auth figma
```

### Other Editors

Any tool supporting Streamable HTTP can connect using:
```json
{
  "mcpServers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp"
    }
  }
}
```

## Rate Limits

- **Starter/View/Collab seats:** 6 tool calls per month
- **Dev/Full seats (Pro/Org/Enterprise):** Per-minute limits (same as Tier 1 Figma REST API)
- **Write operations:** Exempt from rate limits during beta

## Core Tools

### `get_design_context`

Fetches structured design data (React + Tailwind representation) from a Figma node.

**Usage:**
```
Copy Figma frame URL → paste in prompt:
"Implement this design: https://www.figma.com/design/FILE_ID?node-id=123-456"
```

The agent extracts the `node-id` and calls the tool automatically.

**Explicit trigger:**
```
"Use get_design_context to fetch the design at this URL"
```

### `get_screenshot`

Returns a visual snapshot of the selected node.

**Usage:**
```
"Get a screenshot of this Figma component"
```

Always pair with `get_design_context` for complete context.

### `get_variable_defs`

Extracts design tokens (colors, spacing, typography) from a selection.

**Usage:**
```
"Get the variable names and values used in this frame"
"Extract design tokens from this Figma file"
```

**Example output structure:**
```javascript
{
  "colors": {
    "primary": "#3B82F6",
    "secondary": "#10B981"
  },
  "spacing": {
    "sm": "8px",
    "md": "16px",
    "lg": "24px"
  },
  "typography": {
    "headingLarge": {
      "fontFamily": "Inter",
      "fontSize": "32px",
      "fontWeight": "700"
    }
  }
}
```

### `get_metadata`

Fetches high-level node structure without full content (useful when responses are truncated).

**Usage:**
```
"Get the metadata for this Figma file"
"Show me the node structure"
```

Use to discover node IDs, then fetch specific nodes with `get_design_context`.

### Write Tools (beta, remote server only)

- **`create_frame`** — Create new frames
- **`create_component`** — Define reusable components
- **`update_auto_layout`** — Modify layout properties
- **`create_variable`** — Add design tokens

**Example flow:**
```
"Create a component in Figma for a primary button with our design system tokens"
```

Agent will:
1. Fetch variables with `get_variable_defs`
2. Use `create_component` to build the button
3. Apply tokens via `create_variable` if needed

## Working With Figma URLs

Figma URLs contain the node identifier. Agents parse this automatically:

**Frame URL format:**
```
https://www.figma.com/design/abc123?node-id=45-678
```

**The agent extracts:**
- `file_id`: `abc123`
- `node_id`: `45-678`

**Prompt patterns:**
```
"Implement this design: [paste URL]"
"Generate code for https://www.figma.com/design/..."
"Use Code Connect for this component: [URL]"
```

## Code Connect Integration

Code Connect maps Figma components to your actual component implementations. When configured, agents generate code using your real components instead of generic markup.

**Setup:**
1. Install Code Connect in your repo
2. Link components to Figma in your codebase
3. Figma MCP automatically detects and uses mappings

**Example mapping (React):**
```javascript
// Button.figma.tsx
import figma from '@figma/code-connect';
import { Button } from './Button';

figma.connect(Button, 
  'https://www.figma.com/design/abc123?node-id=12-34', 
  {
    props: {
      variant: figma.enum('Variant', {
        Primary: 'primary',
        Secondary: 'secondary'
      }),
      size: figma.enum('Size', {
        Small: 'sm',
        Large: 'lg'
      })
    },
    example: ({ variant, size }) => (
      <Button variant={variant} size={size}>
        Click me
      </Button>
    )
  }
);
```

**Agent behavior with Code Connect:**

Without:
```jsx
<button className="bg-blue-500 px-4 py-2 rounded">
  Click me
</button>
```

With:
```jsx
import { Button } from '@/components/ui/Button';

<Button variant="primary" size="md">
  Click me
</Button>
```

**Prompt:**
```
"Use Code Connect to implement this button component"
"Generate code using our actual component library"
```

## Real-World Workflows

### Implement a Feature Flow

```
"Implement this checkout flow using our existing components: 
https://www.figma.com/design/xyz?node-id=100-200

Use components from src/components/checkout
Follow our Next.js page structure
Use our design tokens for spacing and colors"
```

Agent will:
1. Call `get_design_context` for the frame
2. Call `get_variable_defs` for tokens
3. Call `get_screenshot` for visual reference
4. Check Code Connect for component mappings
5. Generate code using your conventions

### Extract Design System Tokens

```
"Extract all color, spacing, and typography variables from this design system:
https://www.figma.com/design/tokens?node-id=1-1

Generate a TypeScript tokens file for Tailwind config"
```

**Agent response:**
```typescript
// tokens.ts
export const tokens = {
  colors: {
    primary: {
      50: '#eff6ff',
      500: '#3b82f6',
      900: '#1e3a8a'
    },
    secondary: {
      500: '#10b981'
    }
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
    lg: '24px',
    xl: '32px'
  },
  typography: {
    display: {
      fontFamily: 'Inter',
      fontSize: '48px',
      fontWeight: '700',
      lineHeight: '1.2'
    },
    body: {
      fontFamily: 'Inter',
      fontSize: '16px',
      fontWeight: '400',
      lineHeight: '1.5'
    }
  }
} as const;
```

### Create Components in Figma (Write to Canvas)

```
"Create a primary button component in Figma using our design system tokens.
Should have three variants: small, medium, large.
Use our primary blue and standard corner radius."
```

Agent will:
1. Fetch existing variables
2. Call `create_component` with proper structure
3. Apply auto layout
4. Link to variables

### Sync Design and Code

```
"This Card component has been updated in code (src/components/Card.tsx).
Update the Figma component to match:
- Add elevation prop
- Support optional image
- Update spacing to use 16px instead of 12px"
```

Agent reads your code, then calls write tools to update Figma.

## Configuration and Rules

### Project-Level Instructions

Add to your project's agent instructions file (`.cursorrules`, `.clinerules`, etc.):

```yaml
## Figma MCP Integration

### Required workflow
1. Always run get_design_context first for the exact node
2. If truncated, use get_metadata → then re-fetch specific nodes
3. Run get_screenshot for visual reference
4. Download assets as needed
5. Translate React + Tailwind output to our framework

### Framework conventions
- Use Next.js App Router structure
- Components go in src/components/ui/
- Use our Stack and Box layout primitives
- Import design tokens from @/lib/tokens

### Styling
- Use Tailwind classes via className
- Reference design tokens: colors.primary.500
- Never hardcode spacing — use token variables

### Code Connect
- Always check for Code Connect mappings
- Reuse existing components from src/components/
- Match prop names to our component API

### File organization
- One component per file
- Export as named export
- Include TypeScript types
- Add JSDoc for complex props

### Never hardcode
- Colors → use tokens.colors.*
- Spacing → use tokens.spacing.*
- Typography → use tokens.typography.*
```

### Asset Handling

Images returned by `get_design_context` are base64-encoded:

```javascript
// Agent pattern for assets
const assetData = designContext.images[0];
if (assetData.startsWith('data:image')) {
  // Write to file
  fs.writeFileSync(
    `public/assets/${nodeName}.png`,
    Buffer.from(assetData.split(',')[1], 'base64')
  );
}
```

**Rule for agents:**
```
When Figma MCP returns images:
1. Decode base64 data
2. Save to public/assets/ with semantic names
3. Reference via relative path in code
4. Use Next.js Image component with proper sizing
```

## Framework Translation

Default output is React + Tailwind. Translate to your stack in the prompt:

**SwiftUI:**
```
"Generate SwiftUI code from this Figma frame"
```

**Vue + Tailwind:**
```
"Convert this to Vue 3 composition API with Tailwind"
```

**React Native:**
```
"Generate React Native StyleSheet code for this design"
```

**Chakra UI:**
```
"Use Chakra UI components for this layout"
```

**Custom design system:**
```
"Use our design system components from @company/ui-kit
Available components: Button, Card, Stack, Text, Input
Import from @company/ui-kit"
```

## Troubleshooting

### Agent not picking right tool

**Problem:** Generic code generated instead of using variables.

**Solution:**
```
"Use get_variable_defs to extract design tokens first, 
then generate code referencing those tokens"
```

### Truncated responses

**Problem:** `get_design_context` returns incomplete data.

**Solution:**
```
"Use get_metadata to see the node structure, 
then fetch only the specific child nodes I need"
```

### Missing Code Connect mappings

**Problem:** Components not reused even though Code Connect is set up.

**Solution:**
1. Verify Code Connect files are in repo
2. Check figma.connect() URLs match exact node IDs
3. Prompt: "Check for Code Connect mappings before generating code"

### Wrong component structure

**Problem:** Auto layout not translating correctly.

**Fix in Figma:**
1. Use Auto Layout with clear direction (horizontal/vertical)
2. Set explicit padding and gap values
3. Use fixed sizes or fill container intent
4. Add dev notes if layout is complex

**Prompt:**
```
"This frame uses auto layout. 
Generate a flex container with proper gap and padding.
Check the Figma auto layout settings before coding."
```

### Rate limit hit

**Problem:** "Rate limit exceeded" error.

**Solution:**
- Upgrade to Dev or Full seat
- Batch requests (use get_metadata → fetch specific nodes)
- Cache design context for repeated work

## Environment Variables

Figma MCP uses authentication automatically through the MCP client. No manual API keys required.

For custom implementations using Figma REST API directly:
```bash
# .env
FIGMA_ACCESS_TOKEN=your_personal_access_token
```

Get token: Figma → Settings → Account → Personal access tokens

## Best Practices

### Structure Figma Files for Better Code

- **Use components** for reusable elements
- **Link via Code Connect** for accurate component reuse
- **Use variables** for colors, spacing, typography
- **Name layers semantically** (CardContainer, not Group 5)
- **Use Auto Layout** to communicate responsive behavior
- **Add dev notes** for complex interactions

### Write Effective Prompts

```
❌ "Make this into code"
✅ "Generate Next.js code for this dashboard layout.
   Use components from src/components/dashboard.
   Apply our design tokens from @/lib/tokens.
   Follow the App Router structure."
```

### Verify Design Intent

Before generating code:
```
"Show me the auto layout settings and variables used in this frame"
"Get a screenshot and design context for this component"
```

### Keep Designs in Sync

When code changes:
```
"I updated the Button component in src/components/Button.tsx.
Update the Figma component to match:
[paste code]"
```

## Integration Patterns

### CI/CD Design Checks

```javascript
// scripts/check-design-sync.js
import { exec } from 'child_process';

const figmaNodeId = process.env.FIGMA_COMPONENT_NODE;
const prompt = `
Compare the Figma component at node ${figmaNodeId}
with src/components/Button.tsx.
Report any mismatches in props, variants, or styling.
`;

exec(`claude code "${prompt}"`, (error, stdout) => {
  if (stdout.includes('mismatch')) {
    process.exit(1);
  }
});
```

### Design System Documentation

```
"Generate Storybook stories from all button variants in this Figma frame.
Include controls for each variant and size.
Use our actual Button component from @/components/ui/Button."
```

### Automated Handoff

```
"Create a PR that:
1. Extracts design tokens from this Figma file
2. Updates src/theme/tokens.ts
3. Regenerates Tailwind config
4. Shows a preview of changed colors in Storybook"
```

## Advanced Usage

### Multi-Frame Workflows

```
"Implement these three screens as Next.js pages:
1. Login: https://figma.com/design/x?node-id=1-1
2. Dashboard: https://figma.com/design/x?node-id=1-2
3. Profile: https://figma.com/design/x?node-id=1-3

Share components between pages.
Use the same header and navigation."
```

### Design System Audits

```
"Audit this Figma file for inconsistencies:
- Colors not using variables
- Components without Code Connect
- Hardcoded spacing values
- Layers with generic names

Generate a report with locations and recommendations."
```

### Responsive Design Generation

```
"This frame has mobile, tablet, and desktop variants.
Generate responsive code that:
- Uses Tailwind breakpoints
- Maintains the same component structure
- Adjusts layout and spacing per breakpoint"
```

## Reference

**Official docs:** https://developers.figma.com/docs/figma-mcp-server/  
**Help center:** https://help.figma.com/hc/en-us/articles/32132100833559  
**Code Connect:** https://help.figma.com/hc/en-us/articles/23920389749655  
**Developer terms:** https://www.figma.com/legal/developer-terms/
