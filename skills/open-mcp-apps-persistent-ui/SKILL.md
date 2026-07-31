---
name: open-mcp-apps-persistent-ui
description: Build and manage persistent, reusable AI-generated UI components with the open-mcp-apps MCP engine
triggers:
  - create a persistent UI component
  - build an MCP app widget
  - make a reusable interface for
  - set up open-mcp-apps
  - create a kanban board that persists
  - build a habit tracker with open-mcp-apps
  - write a custom MCP app
  - install an app into open-mcp-apps
---

# open-mcp-apps Persistent UI Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**open-mcp-apps** is an MCP server that enables AI assistants to create persistent, reusable UI components. When a user asks for a UI (kanban board, habit tracker, reading list), the AI writes a single-file HTML app, saves it to a registry, and binds it to persistent SQLite-backed data. The app persists across conversations and hosts.

Key capabilities:
- **App registry**: AI can write and save new HTML apps on demand
- **Persistent data**: Apps bind to versioned SQLite collections with idempotent mutations
- **Shell runtime**: Serves apps with MCP App bridge, host theming, and data API
- **17 built-in apps**: Ready-made components (companion, study cards, family week planner)

## Installation

### Standard Installation (One Command)

```bash
curl -fsSL https://raw.githubusercontent.com/2nd1st/open-mcp-apps/main/install.sh | sh
```

The installer:
- Prompts for which hosts to register (Claude Desktop, Claude Code, Codex)
- Creates a fixed per-user data store (shared across hosts)
- Registers the MCP server in each host's config

**Skip prompts** (auto-yes):
```bash
curl -fsSL https://raw.githubusercontent.com/2nd1st/open-mcp-apps/main/install.sh | sh -s -- --yes
```

**Target specific host**:
```bash
curl -fsSL https://raw.githubusercontent.com/2nd1st/open-mcp-apps/main/install.sh | sh -s -- --host codex
```

### Manual Installation (Clone)

```bash
git clone https://github.com/2nd1st/open-mcp-apps
cd open-mcp-apps
node install.mjs
```

### Post-Install

1. **Fully quit and reopen** the host (Cmd-Q on macOS, not just close window)
2. **First-run permissions**: When tools appear, click "Always allow" for each
3. **Batch permissions**: Settings → Connectors → open-mcp-apps → Tool permissions

### Data Store Location

- **macOS**: `~/Library/Application Support/open-mcp-apps/open-mcp-apps.db`
- **Windows**: `%APPDATA%\open-mcp-apps\open-mcp-apps.db`
- **Linux**: `$XDG_DATA_HOME/open-mcp-apps/open-mcp-apps.db` or `~/.local/share/open-mcp-apps/`

### Uninstall

```bash
node uninstall.mjs           # Unregister, keep data
node uninstall.mjs --purge   # Delete everything (irreversible)
node uninstall.mjs --check   # Preview changes
```

**Full reset**: Delete the `.db` file (and `-wal`/`-shm` siblings) while host is fully quit.

## Core MCP Tools

The server exposes these tools to the AI:

### `list_apps`
Lists all installed apps with metadata.

```json
{
  "name": "list_apps"
}
```

Returns: Array of apps with `name`, `title`, `description`, `version`, `author`, `trust_tier`.

### `get_app`
Retrieves an app's full source code.

```json
{
  "name": "get_app",
  "arguments": {
    "name": "kanban-board"
  }
}
```

### `get_app_guide`
Retrieves the authoring guide the AI should read before writing a new app.

```json
{
  "name": "get_app_guide"
}
```

Returns: Complete guide with `window.oma` API, design patterns, constraints.

### `save_app`
Saves a new or updated app to the registry.

```json
{
  "name": "save_app",
  "arguments": {
    "name": "habit-tracker",
    "title": "Habit Tracker",
    "description": "Track daily habits with streaks",
    "html": "<html>...</html>",
    "version": "1.0.0",
    "collections": ["habits"]
  }
}
```

### `delete_app`
Removes an app from the registry (data persists).

```json
{
  "name": "delete_app",
  "arguments": {
    "name": "old-app"
  }
}
```

### `open_app` (or `open_<name>`)
Opens an app with optional initial items.

```json
{
  "name": "open_kanban",
  "arguments": {
    "initial_items": {
      "tasks": [
        {"id": "1", "title": "Review PR", "status": "todo"},
        {"id": "2", "title": "Fix bug", "status": "in-progress"}
      ]
    }
  }
}
```

The AI uses this after creating or when reopening an app.

## Common Patterns for AI Agents

### Pattern 1: Create a New App from User Request

When user says: "make me a reading tracker"

```
1. Call get_app_guide to read the authoring contract
2. Write single-file HTML app following the guide
3. Call save_app with the HTML and collections list
4. Call open_reading_tracker with sample initial_items
```

Example HTML structure for an app:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Reading Tracker</title>
</head>
<body>
  <div class="container">
    <h1>My Reading List</h1>
    <ul id="books"></ul>
    <form id="add-form">
      <input id="title" placeholder="Book title">
      <input id="author" placeholder="Author">
      <button type="submit">Add</button>
    </form>
  </div>

  <script>
    // window.oma is injected by the engine
    const { items, mutate, subscribe } = window.oma;

    // Load collection
    const books = items('books');

    // Render
    function render() {
      const list = document.getElementById('books');
      list.innerHTML = books.map(b => 
        `<li>${b.title} by ${b.author} <button onclick="remove('${b.id}')">✕</button></li>`
      ).join('');
    }

    // Add book
    document.getElementById('add-form').onsubmit = (e) => {
      e.preventDefault();
      mutate('books', {
        command: 'add_book',
        item: {
          id: crypto.randomUUID(),
          title: document.getElementById('title').value,
          author: document.getElementById('author').value
        }
      });
      e.target.reset();
    };

    // Remove book
    window.remove = (id) => {
      mutate('books', {
        command: 'remove_book',
        item_id: id
      });
    };

    // Subscribe to changes
    subscribe('books', render);
    render();
  </script>
</body>
</html>
```

### Pattern 2: Reuse Existing App

When user says: "show me my reading tracker again"

```
1. Call list_apps to check if 'reading-tracker' exists
2. If exists: call open_reading_tracker (no initial_items needed)
3. If not: follow Pattern 1 to create it
```

### Pattern 3: Update an App

When user says: "add a rating field to my reading tracker"

```
1. Call get_app with name="reading-tracker"
2. Modify the HTML to add rating functionality
3. Call save_app with incremented version
4. Call open_reading_tracker to show updated version
```

### Pattern 4: Install from Library

The built-in library app shows 17 ready-made apps. To use them:

```
1. User opens library (open_library tool)
2. User clicks install in the library UI
3. App is installed and immediately available
```

Or programmatically:

```bash
# Install a custom app from file
node install-app.mjs ./my-custom-app.html
node install-app.mjs ./untrusted-app.html --sandboxed
```

## window.oma API Reference

Every app has access to `window.oma` with these methods:

### `items(collection_name)`
Returns array of all items in the collection.

```javascript
const tasks = window.oma.items('tasks');
// Returns: [{id: '1', title: 'Task 1', ...}, ...]
```

### `mutate(collection_name, command)`
Idempotently mutates the collection.

```javascript
window.oma.mutate('tasks', {
  command: 'add_task',
  command_id: crypto.randomUUID(), // Optional, auto-generated
  expected_version: window.oma.version('tasks'), // OCC
  item: {
    id: crypto.randomUUID(),
    title: 'New task',
    status: 'todo'
  }
});
```

### `subscribe(collection_name, callback)`
Listens for changes.

```javascript
window.oma.subscribe('tasks', () => {
  console.log('Tasks changed, re-render');
  render();
});
```

### `version(collection_name)`
Returns current version number (for optimistic concurrency control).

```javascript
const v = window.oma.version('tasks'); // e.g., 42
```

### `theme`
Object with `mode` ('light' or 'dark') and design tokens.

```javascript
if (window.oma.theme.mode === 'dark') {
  document.body.classList.add('dark');
}
```

### File API (for apps with `capabilities: ['files']`)

```javascript
// Upload file
window.oma.files.upload(file_object).then(file_id => {
  console.log('Uploaded:', file_id);
});

// Get file metadata
window.oma.files.get(file_id).then(metadata => {
  console.log(metadata.name, metadata.size, metadata.mime_type);
});

// Download URL
const url = window.oma.files.url(file_id);
// Returns: /files/<file_id>

// List all files
window.oma.files.list().then(files => {
  files.forEach(f => console.log(f.name));
});

// Delete
window.oma.files.delete(file_id);
```

## Configuration

### Environment Variables

Set in the `env` block of your host's MCP server entry:

```json
{
  "mcpServers": {
    "open-mcp-apps": {
      "command": "node",
      "args": ["/path/to/open-mcp-apps/src/server.mjs"],
      "env": {
        "OMA_VIEWER": "0",
        "PORT": "9000"
      }
    }
  }
}
```

- **`OMA_VIEWER`**: Set to `0` to disable the browser viewer entirely
- **`PORT`**: Change the viewer port (default: `8787`)

### Browser Viewer

The viewer runs at `http://127.0.0.1:8787` (or custom `PORT`). It serves:
- `/view/<app-name>` - Individual app pages
- `/mcp` - Stateless HTTP MCP endpoint

**Security**: Bound to `127.0.0.1` only. No password because any local process can access the SQLite file directly. Treat tunnel URLs as secrets if you expose it.

## Advanced: Writing Apps Manually

For apps beyond the AI's context window, write in your own editor:

```bash
# Create my-app.html with full bundling/tooling
# Keep it ≤200 KB, self-contained, no network requests

# Install as trusted (full capabilities)
node install-app.mjs ./my-app.html

# Install as sandboxed (no capabilities)
node install-app.mjs ./my-app.html --sandboxed

# List provenance
node install-app.mjs --list
```

**Key constraints**:
- Single HTML file, ≤200 KB
- No external network requests
- Engine injects CSS, design tokens, and `window.oma`
- Provenance (trusted vs sandboxed) cannot be changed after install

See [`RUNTIME.md`](https://github.com/2nd1st/open-mcp-apps/blob/main/RUNTIME.md) for full contract.

## Troubleshooting

### Apps not appearing after install
- **Fully quit the host** (Cmd-Q, not just close window)
- Check `~/.config/Claude/claude_desktop_config.json` (or equivalent) for server entry
- Verify server process: `ps aux | grep open-mcp-apps`

### "Permission denied" on first tool use
- Click "Always allow" for each tool
- Or batch-approve: Settings → Connectors → open-mcp-apps → Tool permissions

### Data not persisting
- Ensure host is fully quit before deleting/moving the `.db` file
- Check for `-wal` and `-shm` siblings (SQLite write-ahead log)

### Port 8787 already in use
- Another `open-mcp-apps` instance is running (they share data, no issue)
- Another process owns the port: set `PORT` in env to use a different port

### App doesn't load in viewer
- Check browser console for errors
- Ensure app HTML is valid (no unclosed tags)
- Verify collections exist: the AI should seed initial items

### Version conflicts after update
- Fully quit host
- If persists: delete `open-mcp-apps.db` and restart (data lost)

### "Rate limit" or "Token budget" errors
- The AI is reading large guide/sources. This is expected first time
- Subsequent calls use cached context

## Testing

Run the full test suite:

```bash
npm test
```

Individual test files:

```bash
node test/server-smoke.mjs   # 419 assertions (stdio MCP)
node test/http-smoke.mjs     #  53 assertions (HTTP transport)
node test/provenance.mjs     #  39 assertions (trust tiers)
node test/seed-smoke.mjs     #  14 assertions (seed pipeline)
node test/files-smoke.mjs    #  45 assertions (file store)
```

## Design Principles

- **UI and data persist separately**: Apps are views, collections are truth
- **Idempotent mutations**: Every change is a domain command with `command_id`
- **Optimistic concurrency**: `expected_version` prevents conflicts
- **Zero config for users**: AI handles app creation, no manual setup
- **Single-file apps**: ≤200 KB HTML, no build step for AI-authored apps
- **Provenance matters**: Sandboxed apps run in restricted iframe, no file/network access

## Example: Full Kanban Board Creation

User says: "make me a kanban board"

AI workflow:

1. **Check if exists**:
   ```json
   {"name": "list_apps"}
   ```

2. **Read guide** (first time only):
   ```json
   {"name": "get_app_guide"}
   ```

3. **Write app** (HTML with columns: todo, in-progress, done):
   ```html
   <!DOCTYPE html>
   <html>
   <head><title>Kanban Board</title></head>
   <body>
     <div class="board">
       <div class="column" data-status="todo">
         <h2>To Do</h2>
         <div class="tasks" id="todo"></div>
       </div>
       <div class="column" data-status="in-progress">
         <h2>In Progress</h2>
         <div class="tasks" id="in-progress"></div>
       </div>
       <div class="column" data-status="done">
         <h2>Done</h2>
         <div class="tasks" id="done"></div>
       </div>
     </div>
     <form id="add-task">
       <input id="task-title" placeholder="Task">
       <button>Add to To Do</button>
     </form>
     <script>
       const { items, mutate, subscribe } = window.oma;
       
       function render() {
         ['todo', 'in-progress', 'done'].forEach(status => {
           const col = document.getElementById(status);
           const tasks = items('tasks').filter(t => t.status === status);
           col.innerHTML = tasks.map(t => `
             <div class="task" draggable="true" data-id="${t.id}">
               ${t.title}
               <button onclick="deleteTask('${t.id}')">✕</button>
             </div>
           `).join('');
         });
       }
       
       document.getElementById('add-task').onsubmit = (e) => {
         e.preventDefault();
         mutate('tasks', {
           command: 'add_task',
           item: {
             id: crypto.randomUUID(),
             title: document.getElementById('task-title').value,
             status: 'todo'
           }
         });
         e.target.reset();
       };
       
       window.deleteTask = (id) => {
         mutate('tasks', { command: 'delete_task', item_id: id });
       };
       
       // Drag-and-drop
       document.querySelectorAll('.column').forEach(col => {
         col.ondrop = (e) => {
           const id = e.dataTransfer.getData('text/plain');
           const newStatus = col.dataset.status;
           mutate('tasks', {
             command: 'move_task',
             item_id: id,
             new_status: newStatus
           });
         };
         col.ondragover = (e) => e.preventDefault();
       });
       
       subscribe('tasks', render);
       render();
     </script>
   </body>
   </html>
   ```

4. **Save app**:
   ```json
   {
     "name": "save_app",
     "arguments": {
       "name": "kanban-board",
       "title": "Kanban Board",
       "description": "Visual task board with drag-and-drop",
       "html": "<html>...</html>",
       "version": "1.0.0",
       "collections": ["tasks"]
     }
   }
   ```

5. **Open with initial tasks**:
   ```json
   {
     "name": "open_kanban",
     "arguments": {
       "initial_items": {
         "tasks": [
           {"id": "1", "title": "Review PR", "status": "todo"},
           {"id": "2", "title": "Fix bug", "status": "in-progress"},
           {"id": "3", "title": "Deploy", "status": "done"}
         ]
       }
     }
   }
   ```

The board now persists. In any future chat: "open my kanban board" → AI calls `open_kanban` → all tasks still there.

---

**Repository**: https://github.com/2nd1st/open-mcp-apps  
**Homepage**: https://openmcp.app  
**License**: AGPL-3.0
