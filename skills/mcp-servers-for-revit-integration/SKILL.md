---
name: mcp-servers-for-revit-integration
description: Connect AI assistants to Autodesk Revit via Model Context Protocol to read, create, modify, and delete Revit elements programmatically.
triggers:
  - "help me interact with Revit using MCP"
  - "set up mcp-servers-for-revit integration"
  - "create Revit elements through AI"
  - "query Revit model data"
  - "configure Revit MCP server"
  - "build custom Revit command set"
  - "connect Claude to Revit"
  - "automate Revit tasks with MCP"
---

# mcp-servers-for-revit Integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection

## Overview

mcp-servers-for-revit is a Model Context Protocol (MCP) implementation that enables AI assistants to interact with Autodesk Revit. It consists of three components:

1. **MCP Server** (TypeScript) - Exposes tools to AI clients via stdio
2. **Revit Plugin** (C#) - WebSocket bridge running inside Revit
3. **Command Set** (C#) - Executes actual Revit API operations

The system supports Revit 2020-2026 and provides 20+ tools for reading model data, creating elements, modifying properties, and exporting information.

## Installation

### Prerequisites

- Node.js 18+
- Autodesk Revit 2020-2026
- .NET Framework 4.8 (Revit 2020-2024) or .NET 8 (Revit 2025-2026)

### Quick Install (Release)

1. Download the ZIP for your Revit version from [Releases](https://github.com/mcp-servers-for-revit/mcp-servers-for-revit/releases):
   ```
   mcp-servers-for-revit-v1.0.0-Revit2025.zip
   ```

2. Extract to Revit addins folder:
   ```
   %AppData%\Autodesk\Revit\Addins\2025\
   ```

3. Configure MCP server in Claude Desktop (`claude_desktop_config.json`):
   ```json
   {
     "mcpServers": {
       "mcp-server-for-revit": {
         "command": "cmd",
         "args": ["/c", "npx", "-y", "mcp-server-for-revit"]
       }
     }
   }
   ```

4. For Claude Code (run in terminal):
   ```bash
   claude mcp add mcp-server-for-revit -- cmd /c npx -y mcp-server-for-revit
   ```

5. Start Revit and click **Always Load** when prompted

6. In Revit ribbon, click **Settings** on mcp-servers-for-revit tab, enable desired commands, and save

### Development Install

Clone and build from source:

```bash
git clone https://github.com/mcp-servers-for-revit/mcp-servers-for-revit.git
cd mcp-servers-for-revit

# Build MCP server
cd server
npm install
npm run build

# Build Revit plugin + command set
# Open mcp-servers-for-revit.sln in Visual Studio
# Select configuration: Release R25 (for Revit 2025)
# Build solution
```

## Available Tools

### Data Extraction

**get_current_view_info**
```typescript
// Returns info about active view
{
  "name": "get_current_view_info",
  "arguments": {}
}
```

**get_current_view_elements**
```typescript
// Get elements in current view
{
  "name": "get_current_view_elements",
  "arguments": {
    "category": "Walls" // Optional: filter by category
  }
}
```

**get_selected_elements**
```typescript
// Get currently selected elements
{
  "name": "get_selected_elements",
  "arguments": {}
}
```

**get_available_family_types**
```typescript
// List available family types
{
  "name": "get_available_family_types",
  "arguments": {
    "category": "Doors" // Required category
  }
}
```

**get_material_quantities**
```typescript
// Calculate material quantities
{
  "name": "get_material_quantities",
  "arguments": {
    "elementIds": [123456, 789012]
  }
}
```

**analyze_model_statistics**
```typescript
// Get model complexity metrics
{
  "name": "analyze_model_statistics",
  "arguments": {}
}
```

**ai_element_filter**
```typescript
// Intelligent element querying
{
  "name": "ai_element_filter",
  "arguments": {
    "query": "all structural columns on level 2"
  }
}
```

### Element Creation

**create_point_based_element**
```typescript
// Create doors, windows, furniture
{
  "name": "create_point_based_element",
  "arguments": {
    "familyTypeId": 654321,
    "location": {
      "x": 10.0,
      "y": 20.0,
      "z": 0.0
    },
    "facingOrientation": {
      "x": 1.0,
      "y": 0.0,
      "z": 0.0
    }
  }
}
```

**create_line_based_element**
```typescript
// Create walls, beams, pipes
{
  "name": "create_line_based_element",
  "arguments": {
    "familyTypeId": 456789,
    "startPoint": {
      "x": 0.0,
      "y": 0.0,
      "z": 0.0
    },
    "endPoint": {
      "x": 20.0,
      "y": 0.0,
      "z": 0.0
    },
    "levelId": 123456
  }
}
```

**create_surface_based_element**
```typescript
// Create floors, ceilings, roofs
{
  "name": "create_surface_based_element",
  "arguments": {
    "familyTypeId": 987654,
    "boundary": [
      {"x": 0.0, "y": 0.0},
      {"x": 20.0, "y": 0.0},
      {"x": 20.0, "y": 15.0},
      {"x": 0.0, "y": 15.0}
    ],
    "levelId": 123456
  }
}
```

**create_grid**
```typescript
// Create grid system
{
  "name": "create_grid",
  "arguments": {
    "direction": "vertical",
    "count": 5,
    "spacing": 20.0,
    "startPoint": {
      "x": 0.0,
      "y": 0.0,
      "z": 0.0
    },
    "length": 50.0
  }
}
```

**create_level**
```typescript
// Create levels
{
  "name": "create_level",
  "arguments": {
    "elevation": 14.0,
    "name": "Level 2"
  }
}
```

**create_room**
```typescript
// Place rooms
{
  "name": "create_room",
  "arguments": {
    "levelId": 123456,
    "location": {
      "x": 10.0,
      "y": 10.0
    },
    "name": "Conference Room",
    "number": "201"
  }
}
```

**create_dimensions**
```typescript
// Create dimension annotations
{
  "name": "create_dimensions",
  "arguments": {
    "elementIds": [123456, 789012],
    "viewId": 345678
  }
}
```

**create_structural_framing_system**
```typescript
// Create beam framing
{
  "name": "create_structural_framing_system",
  "arguments": {
    "beamTypeId": 456789,
    "boundary": [
      {"x": 0.0, "y": 0.0},
      {"x": 40.0, "y": 0.0},
      {"x": 40.0, "y": 30.0},
      {"x": 0.0, "y": 30.0}
    ],
    "spacing": 10.0,
    "levelId": 123456
  }
}
```

### Element Modification

**delete_element**
```typescript
// Delete elements
{
  "name": "delete_element",
  "arguments": {
    "elementIds": [123456, 789012]
  }
}
```

**operate_element**
```typescript
// Perform operations (select, hide, setColor)
{
  "name": "operate_element",
  "arguments": {
    "operation": "setColor",
    "elementIds": [123456],
    "color": {
      "red": 255,
      "green": 0,
      "blue": 0
    }
  }
}
```

**color_elements**
```typescript
// Color by parameter value
{
  "name": "color_elements",
  "arguments": {
    "parameterName": "Fire Rating",
    "colorScheme": {
      "1 Hour": {"red": 255, "green": 255, "blue": 0},
      "2 Hour": {"red": 255, "green": 0, "blue": 0}
    }
  }
}
```

### Annotation

**tag_all_walls**
```typescript
// Tag all walls in view
{
  "name": "tag_all_walls",
  "arguments": {
    "viewId": 345678
  }
}
```

**tag_all_rooms**
```typescript
// Tag all rooms in view
{
  "name": "tag_all_rooms",
  "arguments": {
    "viewId": 345678
  }
}
```

### Data Export

**export_room_data**
```typescript
// Export room information to CSV
{
  "name": "export_room_data",
  "arguments": {
    "outputPath": "C:\\exports\\rooms.csv"
  }
}
```

**store_project_data**
```typescript
// Store project metadata to local DB
{
  "name": "store_project_data",
  "arguments": {
    "metadata": {
      "projectName": "Office Building",
      "client": "Acme Corp"
    }
  }
}
```

**store_room_data**
```typescript
// Store room metadata to local DB
{
  "name": "store_room_data",
  "arguments": {
    "roomId": 123456,
    "metadata": {
      "department": "Engineering",
      "capacity": 12
    }
  }
}
```

**query_stored_data**
```typescript
// Query stored data
{
  "name": "query_stored_data",
  "arguments": {
    "query": "SELECT * FROM rooms WHERE department = 'Engineering'"
  }
}
```

### Advanced

**send_code_to_revit**
```typescript
// Execute C# code in Revit context
{
  "name": "send_code_to_revit",
  "arguments": {
    "code": "var doc = Application.ActiveUIDocument.Document; return new FilteredElementCollector(doc).OfClass(typeof(Wall)).GetElementCount();"
  }
}
```

## Command Set Development

### Creating Custom Commands

Commands inherit from `RevitMCPCommandSet.BaseCommand`:

```csharp
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using RevitMCPCommandSet;
using System.Collections.Generic;

public class CreateCustomElement : BaseCommand
{
    public override string Name => "create_custom_element";
    
    public override string Description => "Creates a custom element";
    
    public override Dictionary<string, object> Execute(
        UIApplication uiApp,
        Dictionary<string, object> parameters)
    {
        var doc = uiApp.ActiveUIDocument.Document;
        
        using (var trans = new Transaction(doc, "Create Custom Element"))
        {
            trans.Start();
            
            // Extract parameters
            var typeId = GetElementId(parameters, "familyTypeId");
            var location = GetXYZ(parameters, "location");
            
            // Create element
            var familySymbol = doc.GetElement(typeId) as FamilySymbol;
            if (!familySymbol.IsActive)
            {
                familySymbol.Activate();
            }
            
            var instance = doc.Create.NewFamilyInstance(
                location,
                familySymbol,
                StructuralType.NonStructural
            );
            
            trans.Commit();
            
            return new Dictionary<string, object>
            {
                ["success"] = true,
                ["elementId"] = instance.Id.IntegerValue
            };
        }
    }
}
```

### Register Command in command.json

```json
{
  "commands": [
    {
      "name": "create_custom_element",
      "description": "Creates a custom element",
      "parameters": {
        "familyTypeId": {
          "type": "number",
          "required": true
        },
        "location": {
          "type": "object",
          "required": true
        }
      }
    }
  ]
}
```

### Build and Deploy

```bash
# Build for Revit 2025
dotnet build commandset -c Release.R25

# Copy to plugin Commands folder
cp commandset/bin/Release.R25/RevitMCPCommandSet.dll plugin/bin/AddIn 2025 Release/Commands/RevitMCPCommandSet/2025/
```

## Testing

Tests use Nice3point.TUnit.Revit for integration testing:

```csharp
using Autodesk.Revit.DB;
using Nice3point.Revit.Api;
using Nice3point.Revit.Executors;
using RevitMCPCommandSet;
using TUnit.Core;

public class CustomElementTests : RevitApiTest
{
    private static Document _doc;

    [Before(HookType.Class)]
    [HookExecutor<RevitThreadExecutor>]
    public static void Setup()
    {
        _doc = Application.NewProjectDocument(UnitSystem.Imperial);
    }

    [After(HookType.Class)]
    [HookExecutor<RevitThreadExecutor>]
    public static void Cleanup()
    {
        _doc?.Close(false);
    }

    [Test]
    public async Task CreateCustomElement_ValidParameters_CreatesElement()
    {
        var command = new CreateCustomElement();
        var parameters = new Dictionary<string, object>
        {
            ["familyTypeId"] = 123456,
            ["location"] = new Dictionary<string, object>
            {
                ["x"] = 10.0,
                ["y"] = 20.0,
                ["z"] = 0.0
            }
        };

        var result = command.Execute(Application.UIApplication, parameters);

        await Assert.That(result["success"]).IsEqualTo(true);
        await Assert.That(result).ContainsKey("elementId");
    }
}
```

Run tests:
```bash
# Revit 2026 must be running
dotnet test -c Debug.R26 -r win-x64 tests/commandset
```

## Common Patterns

### Query Elements by Category

```typescript
// 1. Get available wall types
const wallTypes = await use_mcp_tool({
  server_name: "mcp-server-for-revit",
  tool_name: "get_available_family_types",
  arguments: {
    category: "Walls"
  }
});

// 2. Get all walls in current view
const walls = await use_mcp_tool({
  server_name: "mcp-server-for-revit",
  tool_name: "get_current_view_elements",
  arguments: {
    category: "Walls"
  }
});
```

### Create and Tag Rooms

```typescript
// 1. Create room
const roomResult = await use_mcp_tool({
  server_name: "mcp-server-for-revit",
  tool_name: "create_room",
  arguments: {
    levelId: 123456,
    location: { x: 10.0, y: 10.0 },
    name: "Conference Room",
    number: "201"
  }
});

// 2. Tag all rooms in view
await use_mcp_tool({
  server_name: "mcp-server-for-revit",
  tool_name: "tag_all_rooms",
  arguments: {
    viewId: 345678
  }
});
```

### Color Code by Parameter

```typescript
// Color walls by fire rating
await use_mcp_tool({
  server_name: "mcp-server-for-revit",
  tool_name: "color_elements",
  arguments: {
    parameterName: "Fire Rating",
    colorScheme: {
      "1 Hour": { red: 255, green: 255, blue: 0 },
      "2 Hour": { red: 255, green: 0, blue: 0 },
      "None": { red: 128, green: 128, blue: 128 }
    }
  }
});
```

### Extract and Export Data

```typescript
// 1. Get material quantities
const quantities = await use_mcp_tool({
  server_name: "mcp-server-for-revit",
  tool_name: "get_material_quantities",
  arguments: {
    elementIds: [123456, 789012]
  }
});

// 2. Export room data
await use_mcp_tool({
  server_name: "mcp-server-for-revit",
  tool_name: "export_room_data",
  arguments: {
    outputPath: "C:\\exports\\rooms.csv"
  }
});
```

## Troubleshooting

### MCP Server Not Connecting

1. Verify Node.js installation:
   ```bash
   node --version  # Should be 18+
   ```

2. Test server manually:
   ```bash
   npx -y mcp-server-for-revit
   ```

3. Check Claude Desktop config syntax:
   ```json
   {
     "mcpServers": {
       "mcp-server-for-revit": {
         "command": "cmd",
         "args": ["/c", "npx", "-y", "mcp-server-for-revit"]
       }
     }
   }
   ```

### Revit Plugin Not Loading

1. Check addin manifest location:
   ```
   %AppData%\Autodesk\Revit\Addins\2025\mcp-servers-for-revit.addin
   ```

2. Verify DLL paths in `.addin` file match actual file locations

3. Check Revit version matches build configuration

4. Enable plugin in Revit ribbon Settings

### WebSocket Connection Fails

1. Check if plugin is loaded (ribbon tab visible)

2. Verify WebSocket port (default: 8081) is not blocked

3. Restart Revit after configuration changes

### Command Execution Errors

1. Enable command in Settings dialog

2. Check parameter types match command requirements

3. Verify element IDs exist in current document

4. Ensure transactions are not already open

### Build Errors

**.NET SDK Version:**
```bash
# Command set requires .NET 10 SDK for Revit 2026
winget install Microsoft.DotNet.SDK.10
```

**Missing Revit API References:**
- Install Revit version matching build configuration
- NuGet packages restore automatically via Nice3point.Revit.Api

**Platform Target:**
```bash
# Use x64 platform on ARM64 machines
dotnet test -c Debug.R26 -r win-x64
```

## Release Process

Create release using bump script:

```powershell
# Update version and create tag
./scripts/release.ps1 -Version 1.2.3

# Push to trigger CI/CD
git push origin main --tags
```

The GitHub Actions workflow will:
- Build plugin for Revit 2020-2026
- Create GitHub release with ZIP assets
- Publish MCP server to npm

Manual npm publish (if needed):
```bash
cd server
npm version 1.2.3
npm publish --provenance
```
