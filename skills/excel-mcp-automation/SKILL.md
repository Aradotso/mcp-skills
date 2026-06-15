---
name: excel-mcp-automation
description: AI-powered Excel automation via COM API - 25 tools, 230 operations for Power Query, DAX, VBA, PivotTables, Charts, and more (Windows only)
triggers:
  - automate excel with AI
  - create excel pivottable
  - generate power query script
  - format excel ranges
  - work with excel vba
  - build excel charts
  - manage excel data model
  - create excel tables
---

# Excel MCP Automation

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What This Does

**ExcelMcp** is a Model Context Protocol (MCP) server and CLI that enables AI assistants to automate Microsoft Excel through natural language. It uses Excel's native COM API (100% safe, zero file corruption risk) to control the actual Excel application on Windows.

**Key Capabilities:**
- 🔄 Power Query (M code, atomic workflows, load destinations)
- 📊 Data Model/DAX (measures, relationships, model structure)
- 🎨 Excel Tables (lifecycle, filtering, sorting, structured references)
- 📈 PivotTables (creation, fields, aggregations, calculated members)
- 📉 Charts (all types, series, formatting, data labels, trendlines)
- 📝 VBA (modules, execution, version control)
- 📋 Ranges (values, formulas, formatting, validation, protection)
- 🔌 Connections (OLEDB/ODBC management)
- 🎚️ Slicers (interactive filtering)
- 📸 Screenshots (capture ranges/sheets as PNG)

**Requirements:**
- Windows OS (COM interop is Windows-specific)
- Microsoft Excel 2016 or later installed
- Desktop environment (not for server-side processing)

## Installation

### Option 1: VS Code Extension (Recommended for Conversational AI)

Install from [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=sbroenne.excel-mcp):

```bash
code --install-extension sbroenne.excel-mcp
```

### Option 2: Claude Desktop

Download `.mcpb` from [releases](https://github.com/sbroenne/mcp-server-excel/releases/latest) and double-click to install.

### Option 3: CLI Tool (Recommended for Coding Agents)

```powershell
# Download standalone executable (no .NET runtime required)
# From: https://github.com/sbroenne/mcp-server-excel/releases/latest
# Extract excelcli.exe to a directory in PATH

# OR install via .NET tool (requires .NET 10 runtime)
dotnet tool install --global Sbroenne.ExcelMcp.CLI

# Verify installation
excelcli --version
```

### Option 4: GitHub Copilot Plugin

```powershell
# Register plugin marketplace (one-time)
copilot plugin marketplace add sbroenne/mcp-server-excel-plugins

# Install CLI skill (for coding agents)
copilot plugin install excel-cli@mcp-server-excel-plugins

# Install MCP server skill (for conversational AI)
copilot plugin install excel-mcp@mcp-server-excel-plugins
```

### Option 5: Manual MCP Server Setup

```powershell
# Download mcp-excel.exe from releases
# Add to PATH, then configure your MCP client:

# Auto-configure for coding agents (requires Node.js)
npx add-mcp "mcp-excel" --name excel-mcp

# OR manually add to MCP client config (e.g., Claude Desktop)
# %APPDATA%/Claude/claude_desktop_config.json
```

```json
{
  "mcpServers": {
    "excel-mcp": {
      "command": "mcp-excel.exe"
    }
  }
}
```

## CLI vs MCP Server

| Interface | Best For | Token Usage |
|-----------|----------|-------------|
| **CLI** (`excelcli`) | Coding agents (Copilot, Cursor), scripting | 64% fewer tokens |
| **MCP Server** | Conversational AI (Claude Desktop, VS Code Chat) | More tokens, better tool discovery |

**Why CLI is better for coding agents:**
- Single tool interface (no large schemas sent to LLM)
- Direct command execution
- Auto-generated from Core code (1:1 feature parity with MCP)
- Bundled with excel-cli skill for guidance

## CLI Command Structure

All CLI commands follow the pattern:

```bash
excelcli <category> <operation> [options]
```

### Core Categories (22 total)

```bash
# File management
excelcli file create --path "Sales.xlsx"
excelcli file open --path "Sales.xlsx"
excelcli file close --save

# Range operations
excelcli range set-values --range "A1:B2" --values "[[\"Name\",\"Age\"],[\"Alice\",30]]"
excelcli range get-values --range "A1:C10"
excelcli range set-formulas --range "D2:D10" --formulas "=B2*C2"

# Table operations
excelcli table create --range "A1:D10" --name "SalesData" --has-headers
excelcli table add-column --table "SalesData" --name "Total" --formula "=[@Quantity]*[@Price]"
excelcli table apply-filter --table "SalesData" --column "Region" --values "[\"West\",\"East\"]"

# PivotTable operations
excelcli pivottable create --source-range "A1:D100" --destination "Sheet2!A1" --name "SalesPivot"
excelcli pivottable add-row-field --pivottable "SalesPivot" --field "Product"
excelcli pivottable add-data-field --pivottable "SalesPivot" --field "Sales" --function "Sum"

# Chart operations
excelcli chart create --type "ColumnClustered" --source-range "A1:B10" --destination "Sheet1!E1" --name "SalesChart"
excelcli chart add-series --chart "SalesChart" --name "Q1 Sales" --values "=Sheet1!$B$2:$B$10" --x-values "=Sheet1!$A$2:$A$10"

# Power Query
excelcli powerquery create --name "ImportCSV" --formula "let Source = Csv.Document(File.Contents(\"data.csv\")) in Source"
excelcli powerquery refresh --query "ImportCSV"
excelcli powerquery export-all --output-dir "./queries"

# Data Model / DAX
excelcli datamodel add-measure --table "Sales" --name "TotalRevenue" --formula "SUM(Sales[Amount])"
excelcli datamodel create-relationship --from-table "Orders" --from-column "ProductID" --to-table "Products" --to-column "ProductID"

# VBA
excelcli vba import-module --name "Utils" --code-path "utils.bas"
excelcli vba run-macro --macro "UpdatePrices"

# Formatting
excelcli rangeformat set-font --range "A1:D1" --bold --size 14
excelcli rangeformat set-number-format --range "B2:B10" --format "$#,##0.00"
excelcli rangeformat auto-fit-columns --range "A:D"

# Conditional Formatting
excelcli conditionalformatting add-rule --range "B2:B100" --rule-type "CellValue" --operator "GreaterThan" --formula1 "500" --format-color "Green"

# Slicers
excelcli slicer create --source "SalesData" --field "Region" --destination "Sheet1!F1" --name "RegionSlicer"

# Window Management
excelcli window show
excelcli window hide
excelcli window set-status --message "Processing sales data..."
```

## Real-World Examples

### Example 1: Create Sales Dashboard

```bash
# Create new workbook
excelcli file create --path "SalesDashboard.xlsx"

# Add sample data
excelcli range set-values --range "A1:D1" --values "[[\"Date\",\"Product\",\"Quantity\",\"Price\"]]"
excelcli range set-values --range "A2:D5" --values "[[\"2024-01-01\",\"Widget\",10,25.50],[\"2024-01-02\",\"Gadget\",5,45.00],[\"2024-01-03\",\"Widget\",8,25.50],[\"2024-01-04\",\"Gadget\",12,45.00]]"

# Convert to table
excelcli table create --range "A1:D5" --name "SalesData" --has-headers

# Add calculated column
excelcli table add-column --table "SalesData" --name "Total" --formula "=[@Quantity]*[@Price]"

# Create PivotTable
excelcli pivottable create --source-table "SalesData" --destination "Sheet2!A1" --name "ProductSummary"
excelcli pivottable add-row-field --pivottable "ProductSummary" --field "Product"
excelcli pivottable add-data-field --pivottable "ProductSummary" --field "Total" --function "Sum"

# Create chart
excelcli chart create --type "ColumnClustered" --pivottable "ProductSummary" --destination "Sheet2!E1" --name "ProductChart"

# Show Excel to see results
excelcli window show

# Save and close
excelcli file save
excelcli file close
```

### Example 2: Power Query ETL Workflow

```bash
# Create Power Query to import CSV
excelcli powerquery create --name "ImportSales" --formula "let Source = Csv.Document(File.Contents(\"C:\\Data\\sales.csv\"), [Delimiter=\",\", Encoding=65001, Headers=true]), ChangedType = Table.TransformColumnTypes(Source, {{\"Date\", type date}, {\"Amount\", type number}}) in ChangedType" --load-to-worksheet "RawData"

# Create another query to transform data
excelcli powerquery create --name "CleanSales" --formula "let Source = ImportSales, Filtered = Table.SelectRows(Source, each [Amount] > 0) in Filtered" --load-to-data-model

# Add DAX measure
excelcli datamodel add-measure --table "CleanSales" --name "TotalRevenue" --formula "SUM(CleanSales[Amount])"

# Refresh all queries
excelcli powerquery refresh-all

# Export M code for version control
excelcli powerquery export-all --output-dir "./power-queries"
```

### Example 3: Automated Report Generation (C# Script)

```csharp
using System.Diagnostics;

void GenerateMonthlyReport(string dataPath, string outputPath)
{
    // Open template
    RunExcelCli($"file open --path \"{dataPath}\"");
    
    // Refresh all connections
    RunExcelCli("connection refresh-all");
    
    // Update calculations
    RunExcelCli("calculation calculate-all");
    
    // Format report
    RunExcelCli("rangeformat set-font --range \"A1:Z1\" --bold --color \"White\"");
    RunExcelCli("rangeformat set-fill --range \"A1:Z1\" --color \"DarkBlue\"");
    RunExcelCli("rangeformat auto-fit-columns --range \"A:Z\"");
    
    // Apply conditional formatting to revenue column
    RunExcelCli("conditionalformatting add-rule --range \"F2:F1000\" --rule-type \"ColorScale\" --min-color \"Red\" --max-color \"Green\"");
    
    // Create summary pivot
    RunExcelCli("pivottable create --source-range \"A1:Z1000\" --destination \"Summary!A1\" --name \"MonthlySummary\"");
    RunExcelCli("pivottable add-row-field --pivottable \"MonthlySummary\" --field \"Department\"");
    RunExcelCli("pivottable add-data-field --pivottable \"MonthlySummary\" --field \"Revenue\" --function \"Sum\"");
    
    // Add chart
    RunExcelCli("chart create --type \"ColumnClustered\" --pivottable \"MonthlySummary\" --destination \"Summary!E1\" --name \"DeptRevenue\"");
    
    // Save as new file
    RunExcelCli($"file save --path \"{outputPath}\"");
    RunExcelCli("file close");
}

void RunExcelCli(string args)
{
    var psi = new ProcessStartInfo
    {
        FileName = "excelcli",
        Arguments = args,
        UseShellExecute = false,
        RedirectStandardOutput = true,
        RedirectStandardError = true
    };
    
    using var process = Process.Start(psi);
    process.WaitForExit();
    
    if (process.ExitCode != 0)
    {
        var error = process.StandardError.ReadToEnd();
        throw new Exception($"ExcelCli failed: {error}");
    }
}

// Usage
GenerateMonthlyReport("C:\\Reports\\Template.xlsx", "C:\\Reports\\January2024.xlsx");
```

### Example 4: VBA Macro Management

```bash
# Export existing VBA modules
excelcli vba export-module --name "DataValidation" --output-path "./vba/DataValidation.bas"
excelcli vba export-module --name "Utilities" --output-path "./vba/Utilities.bas"

# Later: Import updated modules
excelcli vba import-module --name "DataValidation" --code-path "./vba/DataValidation.bas" --replace

# Run macro
excelcli vba run-macro --macro "DataValidation.ValidateAllSheets"

# List all VBA modules
excelcli vba list-modules
```

## Configuration & Environment

ExcelMcp requires no special configuration files. Key behaviors:

**File Paths:**
- Always use absolute paths or ensure working directory is correct
- Power Query file references must use absolute paths

**Excel Instance:**
- Server creates/reuses a single Excel COM instance
- Close all Excel files before starting automation
- Use `window show` to watch AI work in real-time
- Use `window hide` for headless automation

**Environment Variables:**
```bash
# Optional: Customize Excel COM instance behavior
# (Most users don't need these)
EXCEL_VISIBLE=true    # Start Excel visible by default
EXCEL_SCREENUPDATING=false  # Disable screen updates for performance
```

## Common Patterns

### Pattern 1: Table → PivotTable → Chart Pipeline

```bash
# Create table from range
excelcli table create --range "A1:F100" --name "SalesData" --has-headers

# Add slicer for interactive filtering
excelcli slicer create --source "SalesData" --field "Region" --destination "Sheet1!H1"

# Create pivot from table
excelcli pivottable create --source-table "SalesData" --destination "Sheet2!A1" --name "Summary"
excelcli pivottable add-row-field --pivottable "Summary" --field "Product"
excelcli pivottable add-column-field --pivottable "Summary" --field "Quarter"
excelcli pivottable add-data-field --pivottable "Summary" --field "Sales" --function "Sum"

# Chart from pivot
excelcli chart create --type "ColumnClustered" --pivottable "Summary" --destination "Sheet2!F1"
```

### Pattern 2: Power Query → Data Model → DAX

```bash
# Import data to Data Model
excelcli powerquery create --name "Sales" --formula "let Source = Csv.Document(File.Contents(\"C:\\Data\\sales.csv\")) in Source" --load-to-data-model

excelcli powerquery create --name "Products" --formula "let Source = Csv.Document(File.Contents(\"C:\\Data\\products.csv\")) in Source" --load-to-data-model

# Create relationship
excelcli datamodel create-relationship --from-table "Sales" --from-column "ProductID" --to-table "Products" --to-column "ProductID"

# Add measures
excelcli datamodel add-measure --table "Sales" --name "TotalRevenue" --formula "SUM(Sales[Amount])"
excelcli datamodel add-measure --table "Sales" --name "AvgOrderValue" --formula "DIVIDE([TotalRevenue], COUNTROWS(Sales))"
```

### Pattern 3: Batch Formatting

```bash
# Format multiple header rows in one command
excelcli rangeformat set-font --range "A1:G1,A12:G12,A24:G24" --bold --size 12

# Apply number formats
excelcli rangeformat set-number-format --range "B2:B100" --format "$#,##0.00"  # Currency
excelcli rangeformat set-number-format --range "C2:C100" --format "0.0%"        # Percentage
excelcli rangeformat set-number-format --range "D2:D100" --format "mm/dd/yyyy"  # Date

# Auto-fit all columns
excelcli rangeformat auto-fit-columns --range "A:Z"
```

## Troubleshooting

### Excel instance already running
**Issue:** "Excel is already open with another file"

**Solution:**
```bash
# Close all Excel files first
excelcli file close --no-save
# Or manually close Excel application
```

### COM initialization failed
**Issue:** "Failed to create Excel COM object"

**Solution:**
- Ensure Microsoft Excel is installed (2016 or later)
- Run on Windows (COM is Windows-only)
- Check Excel is not open in another user session

### Power Query formula errors
**Issue:** "Power Query syntax error"

**Solution:**
```bash
# Use absolute paths in M formulas
# BAD:  File.Contents("data.csv")
# GOOD: File.Contents("C:\\Data\\data.csv")

# Escape quotes in JSON formulas
excelcli powerquery create --name "Test" --formula "let Source = \"Hello\" in Source"
```

### Range not found
**Issue:** "Range 'Sheet1!A1:B10' does not exist"

**Solution:**
```bash
# Ensure worksheet exists first
excelcli worksheet create --name "Sheet1"

# Use correct worksheet reference
excelcli range set-values --range "Sheet1!A1:B2" --values "[[1,2],[3,4]]"
```

### JSON parsing errors (CLI)
**Issue:** "Invalid JSON in --values argument"

**Solution:**
```bash
# Escape quotes properly in PowerShell/CMD
excelcli range set-values --range "A1:B2" --values "[[\"Name\",\"Age\"],[\"Alice\",30]]"

# Or use single quotes (PowerShell)
excelcli range set-values --range 'A1:B2' --values '[["Name","Age"],["Alice",30]]'
```

### Performance optimization
**Issue:** Slow automation for large datasets

**Solution:**
```bash
# Disable screen updating
excelcli window hide

# Set calculation to manual
excelcli calculation set-mode --mode "Manual"

# Do your work...

# Re-enable when done
excelcli calculation calculate-all
excelcli calculation set-mode --mode "Automatic"
excelcli window show
```

## MCP Server Tool Reference

When using the MCP server (not CLI), these are the available tools:

- `file` - Create, open, save, close workbooks
- `worksheet` - Create, delete, rename, move sheets
- `range` - Get/set values, formulas, named ranges
- `range_format` - Font, fill, borders, alignment, number formats
- `range_validation` - Data validation rules
- `range_protection` - Protect/unprotect ranges
- `table` - Create, modify, filter Excel tables
- `pivottable` - Create, configure PivotTables
- `pivottable_field` - Manage PivotTable fields
- `pivottable_calculated` - Add calculated fields/items
- `chart` - Create, configure charts
- `chart_series` - Manage chart data series
- `powerquery` - Create, refresh, export Power Query queries
- `datamodel` - Manage Data Model tables, relationships
- `datamodel_dax` - Create DAX measures
- `vba` - Import, export, run VBA macros
- `connection` - Manage data connections
- `slicer` - Create, configure slicers
- `conditionalformatting` - Add/clear formatting rules
- `screenshot` - Capture ranges/sheets as PNG
- `window` - Show/hide Excel, set status messages
- `calculation` - Get/set calculation mode, trigger recalc

Each tool has multiple operations. See [FEATURES.md](https://github.com/sbroenne/mcp-server-excel/blob/main/FEATURES.md) for complete reference.

## Additional Resources

- **Documentation:** [excelmcpserver.dev](https://excelmcpserver.dev/)
- **GitHub:** [sbroenne/mcp-server-excel](https://github.com/sbroenne/mcp-server-excel)
- **VS Code Extension:** [Marketplace](https://marketplace.visualstudio.com/items?itemName=sbroenne.excel-mcp)
- **Releases:** [Latest downloads](https://github.com/sbroenne/mcp-server-excel/releases/latest)
- **Feature Reference:** [FEATURES.md](https://github.com/sbroenne/mcp-server-excel/blob/main/FEATURES.md)
- **Development Guide:** [DEVELOPMENT.md](https://github.com/sbroenne/mcp-server-excel/blob/main/docs/DEVELOPMENT.md)

---

**Remember:** ExcelMcp controls the actual Excel application via COM. Always close Excel files before starting automation, and use `window show` to watch AI work in real-time during development.
