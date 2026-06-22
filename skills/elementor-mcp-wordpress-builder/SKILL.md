---
name: elementor-mcp-wordpress-builder
description: Build and manage WordPress Elementor pages programmatically through MCP server with 118+ AI-ready tools for page design, widgets, layouts, and theme building.
triggers:
  - create an Elementor page in WordPress
  - add widgets to my Elementor site
  - build a WordPress page with Elementor containers
  - update Elementor page layout programmatically
  - manage WordPress theme templates with Elementor
  - apply brand kit to Elementor site
  - list available Elementor widgets and tools
  - connect MCP server to WordPress Elementor
---

# Elementor MCP WordPress Builder Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Elementor MCP Server is a WordPress plugin that exposes Elementor's page building, widget management, and theme builder capabilities through the Model Context Protocol (MCP). It provides 118+ tools for AI agents to create, manipulate, and manage Elementor page designs programmatically without manual editor interaction.

**Key capabilities:**
- Create and manage pages, posts, and templates
- Add and configure 62+ widgets (free) or 101+ (with Pro)
- Build layouts with flexbox containers
- Manage global design tokens (colors, typography)
- Apply brand kits for instant site restyling
- Create theme templates (headers, footers, singles, archives)
- Build popups with triggers and conditions (Pro)
- Generate custom widgets from AI specs (Pro)
- Search and insert stock images from Openverse
- Inject custom CSS/JS at element, page, or site level

## Installation

### Prerequisites

| Requirement | Version |
|-------------|---------|
| WordPress | >= 6.9 |
| PHP | >= 8.0 |
| Elementor | >= 3.20 |

### Steps

1. **Install Elementor:**
   ```bash
   # Via WP-CLI
   wp plugin install elementor --activate
   ```

2. **Install WordPress MCP Adapter:**
   The adapter is bundled with the plugin — no separate installation needed.

3. **Install Elementor MCP Plugin:**
   - Download latest release from [GitHub Releases](https://github.com/msrbuilds/elementor-mcp/releases)
   - Upload via **WordPress Admin > Plugins > Add New > Upload Plugin**
   - Or via WP-CLI:
     ```bash
     wp plugin install /path/to/elementor-mcp.zip --activate
     ```

4. **Create Application Password:**
   - Go to **Users > Profile > Application Passwords**
   - Generate new password (save it securely)

5. **Configure MCP Connection:**
   - Navigate to **EMCP Tools > Connection** in WordPress admin
   - Enter username and Application Password
   - Copy generated config for your AI client

## Configuration

### MCP Server Endpoint

```
https://your-site.com/wp-json/mcp/elementor-mcp-server
```

### Claude Desktop Configuration

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "elementor-mcp": {
      "type": "http",
      "url": "https://your-site.com/wp-json/mcp/elementor-mcp-server",
      "headers": {
        "Authorization": "Basic BASE64_CREDENTIALS_HERE"
      }
    }
  }
}

```

Generate `BASE64_CREDENTIALS_HERE`:
```bash
echo -n "username:app-password" | base64
```

### Cursor Configuration

`.cursor/mcp.json` in project root:

```json
{
  "mcpServers": {
    "elementor-mcp": {
      "url": "https://your-site.com/wp-json/mcp/elementor-mcp-server",
      "headers": {
        "Authorization": "Basic BASE64_CREDENTIALS_HERE"
      }
    }
  }
}
```

### Node.js Proxy (Recommended for Remote Sites)

For remote WordPress sites or protocol compatibility:

```json
{
  "mcpServers": {
    "elementor-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@msrbuilds/emcp-proxy@latest"],
      "env": {
        "WP_URL": "https://your-site.com",
        "WP_USERNAME": "admin",
        "WP_APP_PASSWORD": "xxxx xxxx xxxx xxxx xxxx xxxx",
        "MCP_PROTOCOL_VERSION": "2024-11-05"
      }
    }
  }
}
```

### Low-Tools Mode

For MCP clients with tool count limits (Antigravity, Gemini API):

1. Go to **EMCP Tools > Tools** in WordPress admin
2. Enable **Low-tools Mode** toggle
3. Reduces active tools from 118+ to ~50 essentials

## Core MCP Tools

### Discovery & Querying

**list-widgets** — Get all available Elementor widgets:
```json
{
  "tool": "list-widgets"
}
```

**get-widget-schema** — Inspect widget configuration schema:
```json
{
  "tool": "get-widget-schema",
  "arguments": {
    "widget_type": "heading"
  }
}
```

**get-page-structure** — Retrieve page element tree:
```json
{
  "tool": "get-page-structure",
  "arguments": {
    "post_id": 123
  }
}
```

**get-element-settings** — Get specific element configuration:
```json
{
  "tool": "get-element-settings",
  "arguments": {
    "post_id": 123,
    "element_id": "abc123"
  }
}
```

### Page Management

**create-page** — Create new Elementor page:
```json
{
  "tool": "create-page",
  "arguments": {
    "title": "Services Landing Page",
    "status": "publish",
    "content": "Initial content"
  }
}
```

**update-page-settings** — Modify page-level settings:
```json
{
  "tool": "update-page-settings",
  "arguments": {
    "post_id": 123,
    "settings": {
      "page_layout": "full_width",
      "custom_css": ".my-class { color: blue; }"
    }
  }
}
```

**clear-page-content** — Remove all elements from page:
```json
{
  "tool": "clear-page-content",
  "arguments": {
    "post_id": 123
  }
}
```

### Layout Building

**add-container** — Add flexbox container:
```json
{
  "tool": "add-container",
  "arguments": {
    "post_id": 123,
    "settings": {
      "content_width": "full",
      "flex_direction": "column",
      "flex_gap": {
        "size": 20,
        "unit": "px"
      },
      "background_background": "classic",
      "background_color": "#f5f5f5",
      "padding": {
        "top": 40,
        "right": 20,
        "bottom": 40,
        "left": 20,
        "unit": "px"
      }
    },
    "parent_id": null,
    "position": 0
  }
}
```

**update-container** — Modify existing container:
```json
{
  "tool": "update-container",
  "arguments": {
    "post_id": 123,
    "element_id": "abc123",
    "settings": {
      "flex_direction": "row",
      "flex_justify_content": "space-between"
    }
  }
}
```

**move-element** — Reposition element:
```json
{
  "tool": "move-element",
  "arguments": {
    "post_id": 123,
    "element_id": "abc123",
    "new_parent_id": "xyz789",
    "new_position": 1
  }
}
```

**remove-element** — Delete element:
```json
{
  "tool": "remove-element",
  "arguments": {
    "post_id": 123,
    "element_id": "abc123"
  }
}
```

**duplicate-element** — Clone element:
```json
{
  "tool": "duplicate-element",
  "arguments": {
    "post_id": 123,
    "element_id": "abc123",
    "new_parent_id": "parent456",
    "new_position": 2
  }
}
```

### Widget Management

**add-widget** — Universal widget insertion:
```json
{
  "tool": "add-widget",
  "arguments": {
    "post_id": 123,
    "widget_type": "heading",
    "settings": {
      "title": "Welcome to Our Services",
      "header_size": "h1",
      "align": "center",
      "title_color": "#333333",
      "typography_typography": "custom",
      "typography_font_size": {
        "size": 48,
        "unit": "px"
      },
      "typography_font_weight": "700"
    },
    "parent_id": "container123",
    "position": 0
  }
}
```

**add-heading** — Convenience shortcut for headings:
```json
{
  "tool": "add-heading",
  "arguments": {
    "post_id": 123,
    "title": "Our Mission",
    "header_size": "h2",
    "align": "left",
    "parent_id": "container123"
  }
}
```

**add-text-editor** — Add rich text content:
```json
{
  "tool": "add-text-editor",
  "arguments": {
    "post_id": 123,
    "editor": "<p>We deliver cutting-edge solutions for modern businesses.</p>",
    "parent_id": "container123"
  }
}
```

**add-button** — Insert call-to-action button:
```json
{
  "tool": "add-button",
  "arguments": {
    "post_id": 123,
    "text": "Get Started",
    "link": {
      "url": "https://example.com/signup",
      "is_external": true,
      "nofollow": false
    },
    "align": "center",
    "button_type": "success",
    "parent_id": "container123"
  }
}
```

**add-image** — Insert image widget:
```json
{
  "tool": "add-image",
  "arguments": {
    "post_id": 123,
    "image": {
      "id": 456,
      "url": "https://example.com/wp-content/uploads/hero.jpg"
    },
    "image_size": "full",
    "align": "center",
    "parent_id": "container123"
  }
}
```

**update-widget** — Modify existing widget:
```json
{
  "tool": "update-widget",
  "arguments": {
    "post_id": 123,
    "element_id": "widget789",
    "settings": {
      "title": "Updated Heading Text",
      "title_color": "#0066cc"
    }
  }
}
```

### Pro Widgets (Elementor Pro Required)

**add-nav-menu** — Add navigation menu:
```json
{
  "tool": "add-nav-menu",
  "arguments": {
    "post_id": 123,
    "menu": "primary",
    "layout": "horizontal",
    "parent_id": "header-container"
  }
}
```

**add-loop-grid** — Create dynamic content loop:
```json
{
  "tool": "add-loop-grid",
  "arguments": {
    "post_id": 123,
    "template_id": 789,
    "query_post_type": "post",
    "query_posts_per_page": 6,
    "columns": 3,
    "parent_id": "container123"
  }
}
```

**add-popup** — Create popup template:
```json
{
  "tool": "add-popup",
  "arguments": {
    "title": "Newsletter Signup",
    "triggers": [
      {
        "type": "exit_intent",
        "settings": {}
      }
    ],
    "timing": {
      "delay": 5
    }
  }
}
```

### Atomic Elements (Elementor 4.0+)

**add-atomic-widget** — Add atomic element:
```json
{
  "tool": "add-atomic-widget",
  "arguments": {
    "post_id": 123,
    "widget_type": "n-button",
    "settings": {
      "button_text": "Learn More",
      "button_link": {
        "url": "/about"
      }
    },
    "parent_id": "container123"
  }
}
```

Available atomic widgets: `n-flex`, `n-div`, `n-heading`, `n-paragraph`, `n-button`, `n-image`, `n-svg`, `n-youtube`, `n-video`, `n-divider`

### Global Settings

**get-global-settings** — Retrieve site-wide design tokens:
```json
{
  "tool": "get-global-settings"
}
```

**update-global-colors** — Modify color palette:
```json
{
  "tool": "update-global-colors",
  "arguments": {
    "colors": [
      {
        "_id": "primary",
        "title": "Brand Primary",
        "color": "#0066cc"
      },
      {
        "_id": "secondary",
        "title": "Brand Secondary",
        "color": "#ff6600"
      }
    ]
  }
}
```

**update-global-typography** — Set typography presets:
```json
{
  "tool": "update-global-typography",
  "arguments": {
    "typography": [
      {
        "_id": "primary",
        "title": "Primary Heading",
        "typography_typography": "custom",
        "typography_font_family": "Roboto",
        "typography_font_size": {
          "size": 36,
          "unit": "px"
        },
        "typography_font_weight": "700"
      }
    ]
  }
}
```

### Brand Kits

**apply-brand-kit** — Apply coordinated design system (10 free kits, 50 in Pro):
```json
{
  "tool": "apply-brand-kit",
  "arguments": {
    "kit_name": "modern-tech"
  }
}
```

Available free kits: `modern-tech`, `classic-elegance`, `bold-creative`, `minimal-clean`, `warm-organic`, `corporate-pro`, `vibrant-startup`, `luxury-premium`, `earthy-natural`, `playful-fun`

**backup-brand-kit** — Save current design tokens:
```json
{
  "tool": "backup-brand-kit",
  "arguments": {
    "backup_name": "pre-redesign-2024"
  }
}
```

**restore-brand-kit** — Restore previous backup:
```json
{
  "tool": "restore-brand-kit",
  "arguments": {
    "backup_name": "pre-redesign-2024"
  }
}
```

### Templates

**save-template** — Export page or section as reusable template:
```json
{
  "tool": "save-template",
  "arguments": {
    "post_id": 123,
    "template_title": "Hero Section Layout",
    "template_type": "section",
    "element_id": "section456"
  }
}
```

**apply-template** — Import template to page:
```json
{
  "tool": "apply-template",
  "arguments": {
    "post_id": 123,
    "template_id": 789,
    "parent_id": "container123",
    "position": 0
  }
}
```

### Theme Builder

**create-theme-template** — Build header/footer/single/archive templates:
```json
{
  "tool": "create-theme-template",
  "arguments": {
    "title": "Site Header",
    "template_type": "header",
    "display_conditions": {
      "include": [
        {
          "type": "general",
          "value": "all"
        }
      ]
    }
  }
}
```

### Stock Images

**search-stock-images** — Find Creative Commons images:
```json
{
  "tool": "search-stock-images",
  "arguments": {
    "query": "business team collaboration",
    "limit": 10,
    "license": "cc0"
  }
}
```

**sideload-image** — Import image to Media Library:
```json
{
  "tool": "sideload-image",
  "arguments": {
    "image_url": "https://example.com/image.jpg",
    "title": "Team Photo",
    "alt_text": "Our team collaborating"
  }
}
```

### Custom Code

**add-element-css** — Add CSS to specific element:
```json
{
  "tool": "add-element-css",
  "arguments": {
    "post_id": 123,
    "element_id": "widget789",
    "css": "selector { border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }"
  }
}
```

**add-page-css** — Inject page-level CSS:
```json
{
  "tool": "add-page-css",
  "arguments": {
    "post_id": 123,
    "css": ".custom-section { background: linear-gradient(45deg, #667eea 0%, #764ba2 100%); }"
  }
}
```

**add-custom-code-snippet** — Create site-wide code injection:
```json
{
  "tool": "add-custom-code-snippet",
  "arguments": {
    "title": "Analytics Script",
    "code": "<script>console.log('Page loaded');</script>",
    "location": "head",
    "priority": 10
  }
}
```

### Composite Tools

**build-page-from-structure** — Create complete page from JSON structure:
```json
{
  "tool": "build-page-from-structure",
  "arguments": {
    "post_id": 123,
    "structure": {
      "containers": [
        {
          "settings": {
            "content_width": "full",
            "background_background": "classic",
            "background_color": "#ffffff"
          },
          "children": [
            {
              "widget_type": "heading",
              "settings": {
                "title": "Welcome",
                "header_size": "h1"
              }
            },
            {
              "widget_type": "text-editor",
              "settings": {
                "editor": "<p>Your content here.</p>"
              }
            }
          ]
        }
      ]
    }
  }
}
```

## Common Patterns

### Building a Complete Landing Page

```json
// Step 1: Create page
{
  "tool": "create-page",
  "arguments": {
    "title": "Product Launch",
    "status": "draft"
  }
}
// Returns: { "post_id": 123 }

// Step 2: Add hero container
{
  "tool": "add-container",
  "arguments": {
    "post_id": 123,
    "settings": {
      "content_width": "full",
      "min_height": {
        "size": 600,
        "unit": "px"
      },
      "flex_direction": "column",
      "flex_justify_content": "center",
      "flex_align_items": "center",
      "background_background": "gradient",
      "background_color": "#667eea",
      "background_color_b": "#764ba2"
    }
  }
}
// Returns: { "element_id": "hero-container" }

// Step 3: Add hero heading
{
  "tool": "add-heading",
  "arguments": {
    "post_id": 123,
    "title": "Revolutionary Product Launch",
    "header_size": "h1",
    "align": "center",
    "title_color": "#ffffff",
    "parent_id": "hero-container"
  }
}

// Step 4: Add CTA button
{
  "tool": "add-button",
  "arguments": {
    "post_id": 123,
    "text": "Pre-Order Now",
    "link": {
      "url": "/preorder"
    },
    "button_type": "success",
    "size": "lg",
    "parent_id": "hero-container"
  }
}

// Step 5: Add features section
{
  "tool": "add-container",
  "arguments": {
    "post_id": 123,
    "settings": {
      "flex_direction": "row",
      "flex_gap": {
        "size": 30,
        "unit": "px"
      }
    }
  }
}
```

### Applying Dynamic Content (Pro)

```json
// Create loop grid for blog posts
{
  "tool": "add-loop-grid",
  "arguments": {
    "post_id": 123,
    "template_id": 456,
    "query_post_type": "post",
    "query_posts_per_page": 9,
    "query_orderby": "date",
    "query_order": "DESC",
    "columns": 3,
    "columns_tablet": 2,
    "columns_mobile": 1,
    "parent_id": "blog-container"
  }
}
```

### Theming Entire Site

```json
// Step 1: Create header template
{
  "tool": "create-theme-template",
  "arguments": {
    "title": "Global Header",
    "template_type": "header",
    "display_conditions": {
      "include": [
        { "type": "general", "value": "all" }
      ]
    }
  }
}

// Step 2: Build header structure
{
  "tool": "add-container",
  "arguments": {
    "post_id": 789,
    "settings": {
      "flex_direction": "row",
      "flex_justify_content": "space-between"
    }
  }
}

// Step 3: Add logo
{
  "tool": "add-image",
  "arguments": {
    "post_id": 789,
    "image": { "id": 101 },
    "parent_id": "header-container"
  }
}

// Step 4: Add navigation (Pro)
{
  "tool": "add-nav-menu",
  "arguments": {
    "post_id": 789,
    "menu": "primary",
    "layout": "horizontal",
    "parent_id": "header-container"
  }
}
```

### Batch Operations

```json
// Update multiple elements at once
{
  "tool": "batch-update-elements",
  "arguments": {
    "post_id": 123,
    "updates": [
      {
        "element_id": "heading1",
        "settings": { "title_color": "#0066cc" }
      },
      {
        "element_id": "heading2",
        "settings": { "title_color": "#0066cc" }
      },
      {
        "element_id": "button1",
        "settings": { "button_type": "primary" }
      }
    ]
  }
}
```

## Troubleshooting

### Connection Issues

**Problem:** `401 Unauthorized` response

**Solution:** Verify Application Password encoding:
```bash
# Correct format (no spaces in username:password)
echo -n "admin:xxxx xxxx xxxx xxxx" | base64

# Test connection
curl -H "Authorization: Basic YOUR_BASE64_HERE" \
  https://your-site.com/wp-json/mcp/elementor-mcp-server
```

**Problem:** `404 Not Found` on MCP endpoint

**Solution:**
```bash
# Flush rewrite rules
wp rewrite flush

# Verify plugin is active
wp plugin list --status=active | grep elementor-mcp
```

### Tool Count Issues

**Problem:** Client rejects connection due to too many tools

**Solution:** Enable Low-tools Mode in **EMCP Tools > Tools** or filter tools manually:
```json
{
  "tool": "toggle-tools",
  "arguments": {
    "enabled_tools": [
      "create-page",
      "add-container",
      "add-widget",
      "get-page-structure",
      "update-widget"
    ]
  }
}
```

### Widget Schema Errors

**Problem:** Widget won't accept certain settings

**Solution:** Inspect widget schema first:
```json
{
  "tool": "get-widget-schema",
  "arguments": {
    "widget_type": "heading"
  }
}
```

Check returned JSON Schema for valid property names, types, and constraints.

### Container Nesting Issues

**Problem:** Elements appear in wrong location

**Solution:** Always verify parent container exists:
```json
// Get page structure first
{ "tool": "get-page-structure", "arguments": { "post_id": 123 } }

// Find correct parent_id in returned tree
// Then add element with verified parent_id
```

### Dynamic Tags Not Working (Pro)

**Problem:** Dynamic content not rendering

**Solution:** List available tags first:
```json
{
  "tool": "list-dynamic-tags"
}
```

Then bind using exact tag name:
```json
{
  "tool": "update-widget",
  "arguments": {
    "post_id": 123,
    "element_id": "widget789",
    "settings": {
      "title": "{{dynamic-tag:post-title}}"
    }
  }
}
```

### Brand Kit Not Applying

**Problem:** `apply-brand-kit` returns error

**Solution:** Check kit availability:
```json
// Free version has 10 kits
// Pro version has 50 kits

// List available kits via WordPress admin:
// EMCP Tools > Brand Kits tab
```

### Custom Widget Builder Errors (Pro)

**Problem:** Widget doesn't appear in editor

**Solution:** Verify widget spec structure:
```json
{
  "widget_id": "unique-id",
  "widget_name": "My Widget",
  "widget_category": "custom",
  "controls": [
    {
      "id": "text",
      "type": "text",
      "label": "Content"
    }
  ],
  "template": "<div>{{ settings.text }}</div>"
}
```

All required fields: `widget_id`, `widget_name`, `widget_category`, `controls` array, `template` string.

## Environment Variables

When using Node.js proxy (`npx @msrbuilds/emcp-proxy`):

| Variable | Required | Description |
|----------|----------|-------------|
| `WP_URL` | Yes | WordPress site URL (e.g., `https://example.com`) |
| `WP_USERNAME` | Yes | WordPress username |
| `WP_APP_PASSWORD` | Yes | Application Password (spaces allowed) |
| `MCP_PROTOCOL_VERSION` | No | Override protocol version (default: `2025-06-18`) |
| `MCP_LOG_FILE` | No | Path for debug logging (e.g., `/tmp/emcp.log`) |

## Resources

- **Documentation:** https://emcp.msrbuilds.com
- **GitHub Repository:** https://github.com/msrbuilds/elementor-mcp
- **Issue Tracker:** https://github.com/msrbuilds/elementor-mcp/issues
- **Support Portal:** Available in **EMCP Tools > Get Support** tab
- **MCP Protocol Spec:** https://modelcontextprotocol.io
- **WordPress MCP Adapter:** https://github.com/WordPress/mcp-adapter
- **Elementor Developer Docs:** https://developers.elementor.com

## License

GPL-2.0-or-later — Same as WordPress and Elementor
