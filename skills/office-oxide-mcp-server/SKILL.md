---
name: office-oxide-mcp-server
description: Rust-native MCP server for Office document processing (Excel, Word, PowerPoint, PDF) with sub-millisecond performance
triggers:
  - process office documents with MCP
  - read excel spreadsheet data
  - fill PDF form fields automatically
  - generate Word documents with MCP
  - create PowerPoint presentations
  - extract data from XLSX files
  - automate office document workflows
  - use office-oxide-mcp server
---

# office-oxide-mcp-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

office-oxide-mcp is a Rust-native MCP (Model Context Protocol) server that provides blazing-fast Office document processing capabilities. It supports Excel (XLSX/XLS), Word (DOCX), PowerPoint (PPTX), and PDF operations with sub-millisecond performance — up to 46× faster than Python alternatives.

**Key capabilities:**
- Read/write Excel spreadsheets with formulas, charts, pivot tables
- Create/modify Word documents from Markdown or programmatically
- Generate PowerPoint presentations with layouts and charts
- Fill PDF forms (AcroForms, XFA) or overlay text at coordinates
- Extract structured data (JSON, Markdown, chunks) from all formats
- Skills system for reusable document generation patterns
- Coherence engine for propagating edits across linked data

## Installation

### From Binary

```bash
cargo install office-oxide-mcp
```

Or download from [GitHub Releases](https://github.com/Aimino-Tech/office-oxide-mcp/releases).

### From Source

```bash
git clone https://github.com/Aimino-Tech/office-oxide-mcp.git
cd office-oxide-mcp
cargo build --release
# Binary at target/release/office-oxide-mcp
```

## Configuration

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "office": {
      "command": "office-oxide-mcp",
      "args": ["--transport", "stdio"]
    }
  }
}
```

### Cursor

Add to `.cursor/mcp_config.json`:

```json
{
  "mcpServers": {
    "office-oxide-mcp": {
      "command": "office-oxide-mcp",
      "args": ["--transport", "stdio"]
    }
  }
}
```

### VS Code (Copilot)

Add to `.vscode/mcp_servers.json`:

```json
{
  "servers": {
    "office-oxide-mcp": {
      "command": "office-oxide-mcp",
      "args": ["--transport", "stdio"]
    }
  }
}
```

## Core Tools

### Document Information

**`get_document_info`** — Get metadata about any Office document

```json
{
  "file_path": "data/report.xlsx"
}
```

**`list_formats`** — List all supported formats and their capabilities

```json
{}
```

### Reading Documents

**`office_read`** — Read document content in various formats

```json
{
  "file_path": "data/financial.xlsx",
  "output_format": "json"
}
```

Output formats:
- `json` — Structured data with sheets/cells
- `markdown` — Human-readable markdown
- `chunks` — Semantic chunks for RAG/embeddings
- `text` — Plain text extraction

Example for Excel JSON output:
```json
{
  "sheets": [
    {
      "name": "Summary",
      "cells": [
        {"row": 0, "col": 0, "value": "Revenue", "formula": null},
        {"row": 0, "col": 1, "value": 125000, "formula": "=SUM(B2:B10)"}
      ]
    }
  ]
}
```

### Excel Operations

**`office_create_xlsx`** — Create new spreadsheet

```json
{
  "output_path": "output/report.xlsx"
}
```

**`office_write_cell`** — Write single cell

```json
{
  "file_path": "output/report.xlsx",
  "sheet": "Sheet1",
  "row": 0,
  "col": 0,
  "value": "Revenue",
  "value_type": "string"
}
```

**`office_write_range`** — Write multiple cells

```json
{
  "file_path": "output/report.xlsx",
  "sheet": "Sheet1",
  "start_row": 1,
  "start_col": 0,
  "data": [
    ["Q1", 25000],
    ["Q2", 30000],
    ["Q3", 35000],
    ["Q4", 35000]
  ]
}
```

**`office_format_range`** — Apply formatting

```json
{
  "file_path": "output/report.xlsx",
  "sheet": "Sheet1",
  "start_row": 0,
  "start_col": 0,
  "end_row": 0,
  "end_col": 1,
  "format": {
    "bold": true,
    "font_size": 12,
    "bg_color": "#E0E0E0"
  }
}
```

**`office_create_chart`** — Add chart

```json
{
  "file_path": "output/report.xlsx",
  "sheet": "Sheet1",
  "chart_type": "column",
  "data_range": "A2:B5",
  "title": "Quarterly Revenue",
  "x_offset": 300,
  "y_offset": 50
}
```

Chart types: `column`, `bar`, `line`, `pie`, `area`, `scatter`

**`office_add_sheet`** — Add new worksheet

```json
{
  "file_path": "output/report.xlsx",
  "sheet_name": "Analysis"
}
```

**`office_merge_cells`** — Merge cell range

```json
{
  "file_path": "output/report.xlsx",
  "sheet": "Sheet1",
  "start_row": 0,
  "start_col": 0,
  "end_row": 0,
  "end_col": 3
}
```

### Word Document Operations

**`office_create_docx`** — Create new Word document

```json
{
  "output_path": "output/contract.docx"
}
```

**`office_write_docx_from_md`** — Generate from Markdown

```json
{
  "file_path": "output/report.docx",
  "markdown": "# Annual Report\n\n## Executive Summary\n\nRevenue increased by **25%** this year.\n\n### Key Metrics\n\n- Total Revenue: $125,000\n- Net Profit: $35,000\n- Growth Rate: 25%"
}
```

**`office_add_table`** — Insert table

```json
{
  "file_path": "output/contract.docx",
  "rows": 4,
  "cols": 3,
  "data": [
    ["Item", "Quantity", "Price"],
    ["Product A", "10", "$100"],
    ["Product B", "5", "$200"],
    ["Total", "15", "$300"]
  ]
}
```

**`office_replace_text`** — Find and replace

```json
{
  "file_path": "output/contract.docx",
  "search": "{{CLIENT_NAME}}",
  "replace": "Acme Corporation"
}
```

**`office_add_header_footer`** — Add header/footer

```json
{
  "file_path": "output/report.docx",
  "header_text": "Confidential Report",
  "footer_text": "Page {PAGE} of {NUMPAGES}"
}
```

### PowerPoint Operations

**`office_create_pptx`** — Create presentation

```json
{
  "output_path": "output/pitch.pptx"
}
```

**`office_add_slide`** — Add slide

```json
{
  "file_path": "output/pitch.pptx",
  "layout": "title_and_content"
}
```

Layouts: `title`, `title_and_content`, `blank`, `two_content`, `comparison`

**`office_add_text_box`** — Add text to slide

```json
{
  "file_path": "output/pitch.pptx",
  "slide_index": 0,
  "text": "Q4 Strategy Review",
  "x": 50,
  "y": 50,
  "width": 600,
  "height": 100,
  "font_size": 32,
  "bold": true
}
```

**`office_add_chart`** (PPT) — Add chart to slide

```json
{
  "file_path": "output/pitch.pptx",
  "slide_index": 1,
  "chart_type": "column",
  "data": {
    "categories": ["Q1", "Q2", "Q3", "Q4"],
    "series": [
      {"name": "Revenue", "values": [25, 30, 35, 35]},
      {"name": "Profit", "values": [8, 10, 12, 15]}
    ]
  },
  "x": 100,
  "y": 150,
  "width": 500,
  "height": 300
}
```

### PDF Operations

**`office_list_pdf_fields`** — List form fields in PDF

```json
{
  "file_path": "forms/application.pdf"
}
```

Returns:
```json
{
  "fields": [
    {"name": "full_name", "type": "text", "value": "", "required": true},
    {"name": "date_of_birth", "type": "text", "value": "", "required": true},
    {"name": "address", "type": "text", "value": "", "required": false}
  ]
}
```

**`office_fill_pdf_form`** — Fill AcroForm/XFA fields

```json
{
  "file_path": "forms/application.pdf",
  "output_path": "output/application_filled.pdf",
  "fields": {
    "full_name": "John Doe",
    "date_of_birth": "1990-05-15",
    "address": "123 Main St, City, State 12345",
    "email": "john@example.com"
  }
}
```

**`office_analyze_pdf_layout`** — Get layout for coordinate overlay

```json
{
  "file_path": "forms/flat_form.pdf"
}
```

Returns bounding boxes for text placement on flat PDFs.

**`office_overlay_pdf_text`** — Overlay text at coordinates

```json
{
  "file_path": "forms/flat_form.pdf",
  "output_path": "output/form_filled.pdf",
  "fields": [
    {"text": "John Doe", "page": 0, "x": 150, "y": 720, "font_size": 12},
    {"text": "1990-05-15", "page": 0, "x": 150, "y": 680, "font_size": 12},
    {"text": "123 Main St", "page": 0, "x": 150, "y": 640, "font_size": 12}
  ]
}
```

**`office_export_pdf`** — Export Office doc to PDF

```json
{
  "file_path": "output/report.xlsx",
  "output_path": "output/report.pdf"
}
```

### Skills System

**`skill_list`** — List available skills

```json
{}
```

Built-in skills:
- `excel.basic` — Create formatted spreadsheets
- `word.invoice` — Generate invoices
- `word.report` — Generate business reports
- `ppt.deck` — Create presentation decks

**`skill_run`** — Execute a skill

```json
{
  "skill_name": "excel.basic",
  "parameters": {
    "title": "Q4 Revenue Report",
    "data": [
      ["Month", "Revenue", "Expenses", "Profit"],
      ["October", 35000, 20000, 15000],
      ["November", 38000, 22000, 16000],
      ["December", 42000, 24000, 18000]
    ],
    "output_path": "output/q4_report.xlsx"
  }
}
```

**`skill_validate`** — Validate skill parameters

```json
{
  "skill_name": "word.invoice",
  "parameters": {
    "invoice_number": "INV-2024-001",
    "client_name": "Acme Corp"
  }
}
```

## Common Patterns

### Pattern 1: Extract Excel Data for Analysis

```json
// Step 1: Read spreadsheet
{
  "tool": "office_read",
  "params": {
    "file_path": "data/sales_2024.xlsx",
    "output_format": "json"
  }
}

// Step 2: Process in code, then write summary
{
  "tool": "office_create_xlsx",
  "params": {
    "output_path": "output/sales_summary.xlsx"
  }
}

{
  "tool": "office_write_range",
  "params": {
    "file_path": "output/sales_summary.xlsx",
    "sheet": "Sheet1",
    "start_row": 0,
    "start_col": 0,
    "data": [
      ["Metric", "Value"],
      ["Total Sales", 125000],
      ["Average Order", 2500],
      ["Growth Rate", "15%"]
    ]
  }
}
```

### Pattern 2: Generate Invoice from Template

```json
// Use skill for structured invoice generation
{
  "tool": "skill_run",
  "params": {
    "skill_name": "word.invoice",
    "parameters": {
      "invoice_number": "INV-2024-042",
      "date": "2024-06-15",
      "client_name": "Tech Solutions Inc",
      "client_address": "456 Enterprise Blvd, Business City, BC 67890",
      "items": [
        {"description": "Consulting Services", "quantity": 40, "rate": 150, "amount": 6000},
        {"description": "Software License", "quantity": 5, "rate": 200, "amount": 1000}
      ],
      "subtotal": 7000,
      "tax_rate": 0.10,
      "tax_amount": 700,
      "total": 7700,
      "output_path": "output/invoice_042.docx"
    }
  }
}
```

### Pattern 3: Fill Government PDF Form

```json
// Step 1: Analyze form structure
{
  "tool": "office_list_pdf_fields",
  "params": {
    "file_path": "forms/tax_return.pdf"
  }
}

// Step 2: Fill form fields
{
  "tool": "office_fill_pdf_form",
  "params": {
    "file_path": "forms/tax_return.pdf",
    "output_path": "output/tax_return_filled.pdf",
    "fields": {
      "taxpayer_name": "Jane Smith",
      "ssn": "123-45-6789",
      "filing_status": "Single",
      "total_income": "85000",
      "deductions": "12550",
      "taxable_income": "72450"
    }
  }
}
```

### Pattern 4: Create Presentation from Data

```json
// Step 1: Create deck
{
  "tool": "office_create_pptx",
  "params": {
    "output_path": "output/quarterly_review.pptx"
  }
}

// Step 2: Title slide
{
  "tool": "office_add_slide",
  "params": {
    "file_path": "output/quarterly_review.pptx",
    "layout": "title"
  }
}

{
  "tool": "office_add_text_box",
  "params": {
    "file_path": "output/quarterly_review.pptx",
    "slide_index": 0,
    "text": "Q4 2024 Business Review",
    "x": 50,
    "y": 200,
    "width": 700,
    "height": 100,
    "font_size": 44,
    "bold": true
  }
}

// Step 3: Content slide with chart
{
  "tool": "office_add_slide",
  "params": {
    "file_path": "output/quarterly_review.pptx",
    "layout": "title_and_content"
  }
}

{
  "tool": "office_add_text_box",
  "params": {
    "file_path": "output/quarterly_review.pptx",
    "slide_index": 1,
    "text": "Revenue Growth",
    "x": 50,
    "y": 50,
    "width": 700,
    "height": 50,
    "font_size": 32,
    "bold": true
  }
}

{
  "tool": "office_add_chart",
  "params": {
    "file_path": "output/quarterly_review.pptx",
    "slide_index": 1,
    "chart_type": "column",
    "data": {
      "categories": ["Q1", "Q2", "Q3", "Q4"],
      "series": [
        {"name": "2024", "values": [85, 92, 98, 105]}
      ]
    },
    "x": 100,
    "y": 150,
    "width": 600,
    "height": 350
  }
}
```

### Pattern 5: Batch Process Multiple Documents

```json
// Read multiple spreadsheets and consolidate
{
  "tool": "office_read",
  "params": {
    "file_path": "data/sales_north.xlsx",
    "output_format": "json"
  }
}

{
  "tool": "office_read",
  "params": {
    "file_path": "data/sales_south.xlsx",
    "output_format": "json"
  }
}

// Create consolidated report
{
  "tool": "office_create_xlsx",
  "params": {
    "output_path": "output/sales_consolidated.xlsx"
  }
}

{
  "tool": "office_add_sheet",
  "params": {
    "file_path": "output/sales_consolidated.xlsx",
    "sheet_name": "North Region"
  }
}

{
  "tool": "office_add_sheet",
  "params": {
    "file_path": "output/sales_consolidated.xlsx",
    "sheet_name": "South Region"
  }
}

// Write data to respective sheets
// (continue with write_range calls for each region)
```

## Troubleshooting

### Server Won't Start

**Issue:** `office-oxide-mcp` command not found

**Solution:**
```bash
# Verify installation
cargo install --list | grep office-oxide-mcp

# Add cargo bin to PATH
export PATH="$HOME/.cargo/bin:$PATH"

# Or use full path
~/.cargo/bin/office-oxide-mcp --transport stdio
```

### File Permission Errors

**Issue:** Cannot read/write files

**Solution:**
- Ensure file paths are absolute or relative to MCP server working directory
- Check file permissions: `chmod 644 input.xlsx`
- Verify output directory exists: `mkdir -p output/`

### PDF Form Fields Not Filling

**Issue:** `office_fill_pdf_form` doesn't populate fields

**Solution:**
```json
// First, list fields to get exact names
{
  "tool": "office_list_pdf_fields",
  "params": {
    "file_path": "form.pdf"
  }
}

// Use exact field names from output (case-sensitive)
{
  "tool": "office_fill_pdf_form",
  "params": {
    "file_path": "form.pdf",
    "output_path": "output/form_filled.pdf",
    "fields": {
      "TextField1": "Value",  // Use exact name from list_pdf_fields
      "CheckBox2": "Yes"
    }
  }
}
```

### Chart Not Appearing in Excel

**Issue:** Chart created but not visible

**Solution:**
```json
// Ensure data range exists before creating chart
{
  "tool": "office_write_range",
  "params": {
    "file_path": "output.xlsx",
    "sheet": "Sheet1",
    "start_row": 0,
    "start_col": 0,
    "data": [
      ["Month", "Sales"],
      ["Jan", 100],
      ["Feb", 150],
      ["Mar", 200]
    ]
  }
}

// Then create chart with correct range (A1:B4 for above data)
{
  "tool": "office_create_chart",
  "params": {
    "file_path": "output.xlsx",
    "sheet": "Sheet1",
    "chart_type": "column",
    "data_range": "A1:B4",  // Include headers
    "title": "Monthly Sales",
    "x_offset": 300,
    "y_offset": 50
  }
}
```

### Performance Issues with Large Files

**Issue:** Slow processing on very large spreadsheets

**Solution:**
- Use `chunks` output format for large documents to process incrementally
- Read specific sheets only if possible (check schema first)
- For 10M+ cells, consider splitting into multiple files
- Increase system memory if available

### Skill Validation Failures

**Issue:** `skill_run` reports missing parameters

**Solution:**
```json
// Use skill_validate first to check requirements
{
  "tool": "skill_validate",
  "params": {
    "skill_name": "excel.basic",
    "parameters": {
      "title": "My Report"
      // Intentionally incomplete
    }
  }
}

// Response shows required fields:
// {"valid": false, "missing": ["data", "output_path"]}

// Then provide all required parameters
{
  "tool": "skill_run",
  "params": {
    "skill_name": "excel.basic",
    "parameters": {
      "title": "My Report",
      "data": [["A", "B"], [1, 2]],
      "output_path": "output/report.xlsx"
    }
  }
}
```

## Performance Tips

1. **Batch operations:** Use `office_write_range` instead of multiple `office_write_cell` calls
2. **Reuse files:** Open once, perform multiple operations, save once
3. **Format after data:** Write all data first, then apply formatting
4. **Use skills:** Pre-built skills are optimized for common patterns
5. **Read efficiently:** Use `json` format for structured access, `text` for content only

## Advanced: Coherence Engine

The coherence engine tracks semantic relationships between data in documents.

```json
// Check for inconsistencies
{
  "tool": "office_check_consistency",
  "params": {
    "file_path": "output/financial.xlsx"
  }
}

// Propagate edits to linked cells
{
  "tool": "office_propagate_edit",
  "params": {
    "file_path": "output/financial.xlsx",
    "sheet": "Summary",
    "row": 5,
    "col": 2,
    "new_value": 150000
  }
}
```

## Related Documentation

- [GitHub Repository](https://github.com/Aimino-Tech/office-oxide-mcp)
- [MCP Specification](https://modelcontextprotocol.io)
- [Use Cases Gallery](https://github.com/Aimino-Tech/office-oxide-mcp/tree/main/showcase)
