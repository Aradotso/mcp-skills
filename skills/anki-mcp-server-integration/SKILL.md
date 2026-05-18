---
name: anki-mcp-server-integration
description: Integrate Anki spaced repetition flashcards with AI assistants through Model Context Protocol for study sessions, deck management, and card creation
triggers:
  - help me review my anki deck
  - create flashcards in anki
  - sync anki with ai assistant
  - manage anki decks and notes
  - add cards to anki deck
  - setup anki mcp server
  - review spaced repetition cards
  - import media into anki
---

# Anki MCP Server Integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that enables AI assistants to interact with Anki, the spaced repetition flashcard application. Transform study sessions into dynamic conversations where AI can create, edit, and review flashcards naturally, explain concepts, and adapt to your learning style.

## Installation

### Prerequisites

- [Anki desktop app](https://apps.ankiweb.net/) installed and running
- [AnkiConnect plugin](https://ankiweb.net/shared/info/2055492159) installed in Anki (Tools → Add-ons → Get Add-ons → code: 2055492159)
- Node.js 20.19.0+

### For Claude Desktop (STDIO Mode)

**Option 1: MCPB Bundle (Recommended)**

Download `.mcpb` file from [releases](https://github.com/ankimcp/anki-mcp-server/releases), then in Claude Desktop:
- Settings → Extensions → drag and drop the `.mcpb` file
- Configure AnkiConnect URL if needed (default: `http://localhost:8765`)
- Restart Claude Desktop

**Option 2: NPM Configuration**

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "anki-mcp": {
      "command": "npx",
      "args": ["-y", "@ankimcp/anki-mcp-server", "--stdio"],
      "env": {
        "ANKI_CONNECT_URL": "http://localhost:8765"
      }
    }
  }
}
```

Location:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

### For Cursor IDE / Cline / Zed

Add to MCP configuration file:

```json
{
  "mcpServers": {
    "anki-mcp": {
      "command": "npx",
      "args": ["-y", "@ankimcp/anki-mcp-server", "--stdio"],
      "env": {
        "ANKI_CONNECT_URL": "http://localhost:8765"
      }
    }
  }
}
```

Configuration locations:
- Cursor: `~/.cursor/mcp.json`
- Cline: VS Code settings UI
- Zed: Extension marketplace

### For Web-based AI (HTTP Mode with ngrok)

```bash
# One-time setup
npm install -g @ankimcp/anki-mcp-server
npm install -g ngrok
ngrok config add-authtoken <YOUR_NGROK_TOKEN>

# Start server with public tunnel
ankimcp --ngrok
```

Share the ngrok URL with ChatGPT or Claude.ai as an MCP server endpoint.

## Configuration

### Environment Variables

```bash
# AnkiConnect URL (default: http://localhost:8765)
ANKI_CONNECT_URL=http://localhost:8765

# Enable read-only mode (no write operations)
READ_ONLY=true
```

### CLI Options

```bash
ankimcp [options]

--stdio                  # STDIO mode for MCP clients
--port <port>            # HTTP port (default: 3000)
--host <host>            # HTTP host (default: 127.0.0.1)
--anki-connect <url>     # AnkiConnect URL
--ngrok                  # Start ngrok tunnel
--read-only              # Prevent modifications
```

## Core Workflows

### 1. Interactive Review Session

```typescript
// User says: "Help me review my Spanish deck"

// AI workflow:
// 1. Sync with AnkiWeb
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "sync",
  arguments: {}
});

// 2. Get due cards from Spanish deck
const dueCards = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "get_due_cards",
  arguments: {
    deck: "Spanish"
  }
});

// 3. Present first card
const presentation = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "present_card",
  arguments: {
    card_id: dueCards.cards[0].cardId
  }
});

// AI shows question, user answers, then AI rates:
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "rate_card",
  arguments: {
    card_id: dueCards.cards[0].cardId,
    ease: 3 // 1=Again, 2=Hard, 3=Good, 4=Easy
  }
});
```

### 2. Batch Card Creation

```typescript
// User says: "Create 10 Arabic vocab cards with RTL styling"

// 1. List available note types
const models = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "modelNames",
  arguments: {}
});

// 2. Get fields for Basic note type
const fields = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "modelFieldNames",
  arguments: {
    modelName: "Basic"
  }
});

// 3. Create notes in batch (up to 100 at once)
const result = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "addNotes",
  arguments: {
    notes: [
      {
        deckName: "Arabic::Vocabulary",
        modelName: "Basic",
        fields: {
          Front: "مرحبا",
          Back: "Hello"
        },
        tags: ["arabic", "greetings"]
      },
      {
        deckName: "Arabic::Vocabulary",
        modelName: "Basic",
        fields: {
          Front: "شكرا",
          Back: "Thank you"
        },
        tags: ["arabic", "greetings"]
      }
      // ... up to 98 more notes
    ]
  }
});

// Result includes successful IDs and any errors
console.log(result.success); // [note_id1, note_id2, ...]
console.log(result.errors);  // Array of error objects if any failed
```

### 3. Media Import from File

```typescript
// User says: "Import this image from my Downloads folder into the selected note"

// 1. Get selected note from Anki browser
const selected = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "guiSelectedNotes",
  arguments: {}
});

// 2. Upload media file (auto-detects file path)
const media = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "storeMediaFile",
  arguments: {
    filename: "diagram.png",
    data: "/Users/username/Downloads/diagram.png" // File path (fastest)
    // OR url: "https://example.com/image.jpg"   // URL download
    // OR data: "base64string..."                // Base64 (slowest, avoid)
  }
});

// 3. Get note details
const noteInfo = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "notesInfo",
  arguments: {
    notes: [selected.result[0]]
  }
});

// 4. Update front field with image
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "updateNoteFields",
  arguments: {
    note: {
      id: selected.result[0],
      fields: {
        Front: `<img src="${media.result}"> ${noteInfo.result[0].fields.Front.value}`
      }
    }
  }
});
```

### 4. Deck Management

```typescript
// List all decks with statistics
const decks = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "listDecks",
  arguments: {
    includeStats: true
  }
});

// Get detailed deck statistics
const stats = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "deckStats",
  arguments: {
    decks: ["Spanish", "Spanish::Grammar"]
  }
});

// Create nested deck (max 2 levels: Parent::Child)
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "createDeck",
  arguments: {
    deck: "French::Vocabulary"
  }
});

// Move cards to different deck
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "changeDeck",
  arguments: {
    cards: [1234567890, 9876543210],
    deck: "French::Advanced"
  }
});
```

### 5. Note Search and Update

```typescript
// Search for notes using Anki query syntax
const notes = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "findNotes",
  arguments: {
    query: "deck:Spanish tag:verb" // Anki search syntax
  }
});

// Get detailed note information
const noteDetails = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "notesInfo",
  arguments: {
    notes: notes.result
  }
});

// Update note fields (CSS-aware, preserves formatting)
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "updateNoteFields",
  arguments: {
    note: {
      id: notes.result[0],
      fields: {
        Front: "¿Cómo estás?",
        Back: "How are you? (informal)"
      }
    }
  }
});

// Delete notes and their cards
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "deleteNotes",
  arguments: {
    notes: [1234567890, 9876543210]
  }
});
```

### 6. Tag Management

```typescript
// Get all existing tags (use first to avoid duplication)
const allTags = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "getTags",
  arguments: {}
});

// Add tags to notes (space-separated)
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "addTags",
  arguments: {
    notes: [1234567890, 9876543210],
    tags: "important grammar advanced"
  }
});

// Remove tags from notes
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "removeTags",
  arguments: {
    notes: [1234567890],
    tags: "beginner"
  }
});

// Rename tag across all notes
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "replaceTags",
  arguments: {
    notes: [1234567890, 9876543210],
    tag_to_replace: "old-tag",
    replace_with_tag: "new-tag"
  }
});

// Clean up unused tags from collection
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "clearUnusedTags",
  arguments: {}
});
```

### 7. Media Management

```typescript
// List media files with pattern filter
const mediaFiles = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "getMediaFilesNames",
  arguments: {
    pattern: "*.png"
  }
});

// Download media as base64
const mediaData = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "retrieveMediaFile",
  arguments: {
    filename: "diagram.png"
  }
});

// Upload media (three methods)
// Method 1: File path (FASTEST - recommended)
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "storeMediaFile",
  arguments: {
    filename: "photo.jpg",
    data: "/Users/username/Pictures/photo.jpg"
  }
});

// Method 2: URL (FAST - auto-downloads)
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "storeMediaFile",
  arguments: {
    filename: "diagram.png",
    url: "https://example.com/diagram.png"
  }
});

// Method 3: Base64 (SLOW - avoid if possible)
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "storeMediaFile",
  arguments: {
    filename: "audio.mp3",
    data: "base64encodedstring..."
  }
});

// Delete media file
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "deleteMediaFile",
  arguments: {
    filename: "old-diagram.png"
  }
});
```

## Common Patterns

### Check Available Decks Before Creating

```typescript
// Always list decks first to avoid duplicates
const decks = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "listDecks",
  arguments: {}
});

if (!decks.result.includes("MyDeck")) {
  await use_mcp_tool({
    server_name: "anki-mcp",
    tool_name: "createDeck",
    arguments: { deck: "MyDeck" }
  });
}
```

### Get Tags Before Adding

```typescript
// Fetch existing tags to maintain consistency
const tags = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "getTags",
  arguments: {}
});

// Use existing tags when adding notes
const newTags = tags.result.filter(t => t.includes("spanish"));
```

### Batch Operations for Efficiency

```typescript
// Instead of adding notes one by one, batch them
const notes = [];
for (let i = 0; i < 50; i++) {
  notes.push({
    deckName: "Vocabulary",
    modelName: "Basic",
    fields: { Front: `Word ${i}`, Back: `Definition ${i}` },
    tags: ["batch-import"]
  });
}

// Single batch operation
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "addNotes",
  arguments: { notes }
});
```

### Safe Note Updates

```typescript
// Always get note info before updating to preserve content
const info = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "notesInfo",
  arguments: { notes: [noteId] }
});

const currentFields = info.result[0].fields;

// Update only specific fields, preserve others
await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "updateNoteFields",
  arguments: {
    note: {
      id: noteId,
      fields: {
        Front: currentFields.Front.value, // Keep existing
        Back: "Updated back content"      // Change this
      }
    }
  }
});
```

## Anki Search Query Syntax

When using `findNotes`, use Anki's search syntax:

```typescript
// Examples of valid queries
"deck:Spanish"                    // All notes in Spanish deck
"tag:verb"                        // Notes tagged with 'verb'
"deck:Spanish tag:verb"           // Spanish deck AND verb tag
"is:due"                          // Due for review
"added:7"                         // Added in last 7 days
"Front:*hola*"                    // Front field contains 'hola'
"deck:Spanish -tag:mastered"      // Spanish deck WITHOUT mastered tag
```

## Troubleshooting

### AnkiConnect Not Responding

```bash
# Verify AnkiConnect is installed
# In Anki: Tools → Add-ons → Check for "AnkiConnect"

# Check if Anki is running
curl http://localhost:8765

# Try custom port
export ANKI_CONNECT_URL=http://localhost:8766
ankimcp --stdio
```

### Permission Errors

AnkiConnect requires explicit permission for external access. If operations fail:
1. Open Anki
2. Tools → Add-ons → AnkiConnect → Config
3. Add your application to `webCorsOriginList` if using HTTP mode

### Media Upload Fails

```typescript
// Prefer file paths over base64
// ✅ GOOD
storeMediaFile({ filename: "img.png", data: "/path/to/img.png" })

// ✅ GOOD
storeMediaFile({ filename: "img.png", url: "https://example.com/img.png" })

// ❌ SLOW (avoid unless necessary)
storeMediaFile({ filename: "img.png", data: "base64..." })
```

### Read-Only Mode Not Working

```bash
# Verify environment variable
echo $READ_ONLY

# Or use CLI flag explicitly
ankimcp --stdio --read-only
```

### Batch Operation Partial Failures

```typescript
// addNotes supports partial success
const result = await use_mcp_tool({
  server_name: "anki-mcp",
  tool_name: "addNotes",
  arguments: { notes: [...] }
});

// Check which notes succeeded and which failed
result.success.forEach((id, idx) => {
  if (id) console.log(`Note ${idx}: Success - ID ${id}`);
});

result.errors.forEach((err, idx) => {
  if (err) console.log(`Note ${idx}: Failed - ${err.error}`);
});
```

### Deck Creation Limits

```typescript
// Only 2 levels supported: Parent::Child
// ✅ Valid
createDeck({ deck: "Languages::Spanish" })

// ❌ Invalid (too many levels)
createDeck({ deck: "Languages::Spanish::Verbs" })
```

## Best Practices

1. **Always sync before review sessions**: Use `sync` tool at start of review workflow
2. **Use file paths for media**: Avoid base64 when possible for performance
3. **Batch note creation**: Use `addNotes` instead of multiple `addNote` calls
4. **Check existing tags**: Call `getTags` before adding to avoid duplication
5. **Preserve note content**: Get `notesInfo` before `updateNoteFields` to avoid data loss
6. **Use read-only mode for exploration**: Enable `--read-only` when testing queries
7. **Handle partial failures**: Check `errors` array in `addNotes` response
8. **Verify decks exist**: Use `listDecks` before creating or moving cards

## Resources

- [Official Documentation](https://ankimcp.ai)
- [GitHub Repository](https://github.com/ankimcp/anki-mcp-server)
- [AnkiConnect API](https://foosoft.net/projects/anki-connect/)
- [Anki Manual](https://docs.ankiweb.net/)
