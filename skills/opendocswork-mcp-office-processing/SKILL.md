---
name: opendocswork-mcp-office-processing
description: Rust-native MCP server for Office document processing (Excel, Word, PowerPoint, PDF) with sub-millisecond performance
triggers:
  - process office documents with MCP
  - read excel files with rust
  - fill PDF forms programmatically
  - create word documents from markdown
  - generate powerpoint presentations
  - extract data from office files
  - automate office document workflows
  - manipulate xlsx docx pptx files
---

# opendocswork-mcp-office-processing

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`office-oxide-mcp` is a Rust-native MCP (Model Context Protocol) server that enables AI assistants to read, write, and manipulate Office documents (Excel, Word, PowerPoint, PDF) with sub-millisecond performance. It's designed as an open-source alternative to commercial document processing libraries, offering local-first, fast document operations.

**Key capabilities:**
- **Read** Excel, Word, PowerPoint, PDF → JSON/Markdown/Text
- **Write** Excel spreadsheets with formulas, charts, formatting
- **Create** Word documents from Markdown, manipulate styles, tables
- **Generate** PowerPoint presentations with slides, charts, images
- **Fill** PDF forms (AcroForm/XFA) and overlay text on flat PDFs
- **Extract** structured data and metadata from all formats
- **Skills system** for reusable document workflows

## Installation

### Cargo Install

```bash
cargo install office-oxide-mcp
```

### From Source

```bash
git clone https://github.com/Aimino-Tech/office-oxide-mcp.git
cd office-oxide-mcp
cargo build --release
# Binary at target/release/office-oxide-mcp
```

### Binary Download

Download pre-built binaries from [GitHub Releases](https://github.com/Aimino-Tech/office-oxide-mcp/releases).

## Configuration

### Claude Desktop

Add to `claude_desktop_config.json`:

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

Add to Cursor MCP settings:

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

Add to `.vscode/mcp.json`:

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

### Environment Variables

```bash
# Optional: Set custom temp directory
export OFFICE_OXIDE_TEMP_DIR=/path/to/temp

# Optional: Enable debug logging
export RUST_LOG=office_oxide_mcp=debug
```

## Core MCP Tools

### Discovery & Metadata

#### `list_formats`

List all supported document formats and capabilities.

```json
{
  "method": "tools/call",
  "params": {
    "name": "list_formats"
  }
}
```

#### `get_document_info`

Get metadata about a document file.

```json
{
  "method": "tools/call",
  "params": {
    "name": "get_document_info",
    "arguments": {
      "file_path": "/path/to/document.xlsx"
    }
  }
}
```

**Response includes:** format, file size, modification time, read/write capabilities.

### Reading Documents

#### `office_read`

Universal document reader supporting multiple output formats.

**Read Excel to JSON:**

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/data.xlsx",
      "output_format": "json"
    }
  }
}
```

**Read Word to Markdown:**

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/report.docx",
      "output_format": "markdown"
    }
  }
}
```

**Read PDF to Text:**

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/document.pdf",
      "output_format": "text"
    }
  }
}
```

**Output format options:** `json`, `markdown`, `text`, `chunks`

## Excel Operations

### Creating Spreadsheets

#### `office_create_xlsx`

Create a new Excel workbook.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_create_xlsx",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Summary"
    }
  }
}
```

#### `office_write_cell`

Write a single cell value.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_write_cell",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Summary",
      "cell": "A1",
      "value": "Revenue",
      "value_type": "text"
    }
  }
}
```

**Value types:** `text`, `number`, `formula`, `date`, `boolean`

#### `office_write_range`

Write multiple cells at once.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_write_range",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Data",
      "start_cell": "A1",
      "data": [
        ["Name", "Age", "Salary"],
        ["Alice", 30, 75000],
        ["Bob", 25, 65000]
      ]
    }
  }
}
```

### Formatting

#### `office_format_range`

Apply formatting to cell ranges.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_format_range",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Summary",
      "range": "A1:C1",
      "format": {
        "bold": true,
        "font_size": 14,
        "bg_color": "#4472C4",
        "font_color": "#FFFFFF"
      }
    }
  }
}
```

**Format options:** `bold`, `italic`, `underline`, `font_size`, `font_name`, `bg_color`, `font_color`, `border`, `number_format`

#### `office_merge_cells`

Merge cell ranges.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_merge_cells",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Report",
      "range": "A1:D1"
    }
  }
}
```

#### `office_set_column_width`

Set column width.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_set_column_width",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Data",
      "column": "A",
      "width": 25.0
    }
  }
}
```

### Charts & Visualizations

#### `office_create_chart`

Create charts in Excel.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_create_chart",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Dashboard",
      "chart_type": "column",
      "data_range": "A1:B5",
      "title": "Monthly Sales",
      "position": "D2"
    }
  }
}
```

**Chart types:** `column`, `bar`, `line`, `pie`, `scatter`, `area`

### Sheet Management

#### `office_add_sheet`

Add a new worksheet.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_add_sheet",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "NewSheet"
    }
  }
}
```

#### `office_rename_sheet`

Rename existing worksheet.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_rename_sheet",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "old_name": "Sheet1",
      "new_name": "Q4Results"
    }
  }
}
```

## Word Document Operations

### Creating Documents

#### `office_create_docx`

Create a new Word document.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_create_docx",
    "arguments": {
      "file_path": "/path/to/output.docx"
    }
  }
}
```

#### `office_write_docx_from_md`

Create Word document from Markdown.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_write_docx_from_md",
    "arguments": {
      "file_path": "/path/to/output.docx",
      "markdown": "# Annual Report\n\n## Executive Summary\n\nRevenue increased by **25%** this year.\n\n- Q1: $1.2M\n- Q2: $1.5M\n- Q3: $1.8M\n- Q4: $2.1M"
    }
  }
}
```

### Text Manipulation

#### `office_replace_text`

Replace text in document.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_replace_text",
    "arguments": {
      "file_path": "/path/to/template.docx",
      "output_path": "/path/to/output.docx",
      "replacements": {
        "{{CUSTOMER_NAME}}": "Acme Corporation",
        "{{DATE}}": "2024-06-15",
        "{{AMOUNT}}": "$50,000"
      }
    }
  }
}
```

### Styling

#### `office_set_style`

Apply paragraph or character styles.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_set_style",
    "arguments": {
      "file_path": "/path/to/output.docx",
      "paragraph_index": 0,
      "style": {
        "font_size": 24,
        "bold": true,
        "color": "#1F4E78"
      }
    }
  }
}
```

### Tables & Images

#### `office_add_table`

Insert table into document.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_add_table",
    "arguments": {
      "file_path": "/path/to/output.docx",
      "rows": 3,
      "cols": 4,
      "data": [
        ["Quarter", "Revenue", "Expenses", "Profit"],
        ["Q1", "$1.2M", "$0.8M", "$0.4M"],
        ["Q2", "$1.5M", "$0.9M", "$0.6M"]
      ]
    }
  }
}
```

#### `office_add_image`

Insert image into document.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_add_image",
    "arguments": {
      "file_path": "/path/to/output.docx",
      "image_path": "/path/to/chart.png",
      "width": 400,
      "height": 300
    }
  }
}
```

## PowerPoint Operations

### Creating Presentations

#### `office_create_pptx`

Create new PowerPoint presentation.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_create_pptx",
    "arguments": {
      "file_path": "/path/to/presentation.pptx"
    }
  }
}
```

#### `office_add_slide`

Add slide to presentation.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_add_slide",
    "arguments": {
      "file_path": "/path/to/presentation.pptx",
      "layout": "title_slide",
      "title": "Q4 Business Review",
      "subtitle": "Executive Summary"
    }
  }
}
```

**Layout options:** `title_slide`, `title_content`, `section_header`, `two_column`, `comparison`, `blank`

#### `office_add_text_box`

Add text box to slide.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_add_text_box",
    "arguments": {
      "file_path": "/path/to/presentation.pptx",
      "slide_index": 1,
      "text": "Revenue increased 25% YoY",
      "x": 100,
      "y": 200,
      "width": 400,
      "height": 100
    }
  }
}
```

#### `office_add_chart`

Add chart to slide.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_add_chart",
    "arguments": {
      "file_path": "/path/to/presentation.pptx",
      "slide_index": 2,
      "chart_type": "column",
      "data": {
        "categories": ["Q1", "Q2", "Q3", "Q4"],
        "series": [
          {"name": "Revenue", "values": [1.2, 1.5, 1.8, 2.1]},
          {"name": "Expenses", "values": [0.8, 0.9, 1.0, 1.1]}
        ]
      },
      "title": "Quarterly Performance"
    }
  }
}
```

## PDF Operations

### Reading PDFs

Use `office_read` with PDF files:

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/document.pdf",
      "output_format": "markdown"
    }
  }
}
```

### PDF Form Filling

#### `office_list_pdf_fields`

List all form fields in PDF.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_list_pdf_fields",
    "arguments": {
      "file_path": "/path/to/form.pdf"
    }
  }
}
```

**Returns:** Array of fields with names, types, and current values.

#### `office_fill_pdf_form`

Fill PDF form fields (AcroForm/XFA).

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_fill_pdf_form",
    "arguments": {
      "file_path": "/path/to/form.pdf",
      "output_path": "/path/to/filled.pdf",
      "fields": {
        "full_name": "John Smith",
        "email": "john.smith@example.com",
        "date_of_birth": "1990-05-15",
        "agree_terms": true
      }
    }
  }
}
```

### PDF Text Overlay (Flat PDFs)

For PDFs without form fields, overlay text at specific coordinates.

#### `office_analyze_pdf_layout`

Analyze PDF to find text positions for overlay planning.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_analyze_pdf_layout",
    "arguments": {
      "file_path": "/path/to/flat_form.pdf"
    }
  }
}
```

**Returns:** Text positions, bounding boxes, and suggested coordinates.

#### `office_overlay_pdf_text`

Overlay text at specific coordinates.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_overlay_pdf_text",
    "arguments": {
      "file_path": "/path/to/flat_form.pdf",
      "output_path": "/path/to/filled.pdf",
      "fields": [
        {
          "text": "John Smith",
          "page": 0,
          "x": 150.0,
          "y": 700.0,
          "font_size": 12,
          "font": "Helvetica"
        },
        {
          "text": "john.smith@example.com",
          "page": 0,
          "x": 150.0,
          "y": 650.0,
          "font_size": 12,
          "font": "Helvetica"
        }
      ]
    }
  }
}
```

### PDF Export

#### `office_export_pdf`

Export Office documents to PDF.

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_export_pdf",
    "arguments": {
      "file_path": "/path/to/report.docx",
      "output_path": "/path/to/report.pdf"
    }
  }
}
```

## Skills System

Reusable document processing workflows.

### `skill_list`

List available skills.

```json
{
  "method": "tools/call",
  "params": {
    "name": "skill_list"
  }
}
```

**Built-in skills:**
- `excel.basic` - Basic spreadsheet operations
- `word.invoice` - Invoice generation
- `word.report` - Report creation
- `ppt.deck` - Presentation decks

### `skill_run`

Execute a skill with parameters.

```json
{
  "method": "tools/call",
  "params": {
    "name": "skill_run",
    "arguments": {
      "skill_name": "word.invoice",
      "params": {
        "output_path": "/path/to/invoice.docx",
        "invoice_number": "INV-2024-001",
        "customer_name": "Acme Corp",
        "items": [
          {"description": "Consulting Services", "quantity": 40, "rate": 150},
          {"description": "Software License", "quantity": 1, "rate": 5000}
        ],
        "due_date": "2024-07-15"
      }
    }
  }
}
```

## Common Patterns

### Financial Report Generation

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_create_xlsx",
    "arguments": {
      "file_path": "/tmp/financial_report.xlsx",
      "sheet_name": "P&L"
    }
  }
}
```

Then write headers:

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_write_range",
    "arguments": {
      "file_path": "/tmp/financial_report.xlsx",
      "sheet_name": "P&L",
      "start_cell": "A1",
      "data": [
        ["Category", "Q1", "Q2", "Q3", "Q4", "Total"],
        ["Revenue", 1200000, 1500000, 1800000, 2100000, "=SUM(B2:E2)"],
        ["Expenses", 800000, 900000, 1000000, 1100000, "=SUM(B3:E3)"],
        ["Profit", "=B2-B3", "=C2-C3", "=D2-D3", "=E2-E3", "=F2-F3"]
      ]
    }
  }
}
```

Format headers:

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_format_range",
    "arguments": {
      "file_path": "/tmp/financial_report.xlsx",
      "sheet_name": "P&L",
      "range": "A1:F1",
      "format": {
        "bold": true,
        "bg_color": "#4472C4",
        "font_color": "#FFFFFF"
      }
    }
  }
}
```

Add chart:

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_create_chart",
    "arguments": {
      "file_path": "/tmp/financial_report.xlsx",
      "sheet_name": "P&L",
      "chart_type": "column",
      "data_range": "A1:E4",
      "title": "Quarterly P&L",
      "position": "H2"
    }
  }
}
```

### Batch PDF Form Filling

```python
# Pseudo-code for batch operation
customers = [
    {"name": "Alice", "email": "alice@example.com", "amount": 1500},
    {"name": "Bob", "email": "bob@example.com", "amount": 2000}
]

for customer in customers:
    mcp_call("office_fill_pdf_form", {
        "file_path": "/templates/contract.pdf",
        "output_path": f"/output/contract_{customer['name']}.pdf",
        "fields": {
            "customer_name": customer["name"],
            "email": customer["email"],
            "contract_amount": str(customer["amount"]),
            "date": "2024-06-15"
        }
    })
```

### Document Template Processing

Read template, replace placeholders, export to PDF:

```json
[
  {
    "method": "tools/call",
    "params": {
      "name": "office_replace_text",
      "arguments": {
        "file_path": "/templates/proposal.docx",
        "output_path": "/tmp/proposal_draft.docx",
        "replacements": {
          "{{CLIENT}}": "Acme Corporation",
          "{{PROJECT}}": "Digital Transformation",
          "{{BUDGET}}": "$250,000",
          "{{TIMELINE}}": "6 months"
        }
      }
    }
  },
  {
    "method": "tools/call",
    "params": {
      "name": "office_export_pdf",
      "arguments": {
        "file_path": "/tmp/proposal_draft.docx",
        "output_path": "/output/proposal_final.pdf"
      }
    }
  }
]
```

## Troubleshooting

### "File not found" errors

Ensure absolute paths or correct relative paths from working directory:

```json
{
  "file_path": "/absolute/path/to/file.xlsx"
}
```

Or use environment variable expansion if supported by your MCP client.

### Excel formulas not calculating

Formulas are stored but may not show calculated values until opened in Excel. To force calculation, use cell references in subsequent operations or open in Excel.

### PDF form fields not filling

1. Check field names with `office_list_pdf_fields`
2. Ensure field names match exactly (case-sensitive)
3. For flat PDFs without forms, use `office_overlay_pdf_text` instead

### Memory issues with large files

The server is optimized for performance but very large files (>100MB) may require streaming:

```json
{
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/large.xlsx",
      "output_format": "chunks"
    }
  }
}
```

Process in chunks rather than loading entire file.

### Permission errors

Ensure the MCP server process has read/write permissions:

```bash
chmod 644 /path/to/input.xlsx
chmod 755 /path/to/output/
```

### Chart rendering issues

Some chart types require specific data structures. Use `excel.basic` skill as reference:

```json
{
  "method": "tools/call",
  "params": {
    "name": "skill_validate",
    "arguments": {
      "skill_name": "excel.basic"
    }
  }
}
```

### Markdown to Word conversion

Complex Markdown may require preprocessing. Supported elements:
- Headers (# ## ###)
- Bold (**text**)
- Italic (*text*)
- Lists (- item)
- Links ([text](url))
- Code blocks (```code```)

Unsupported: HTML tags, custom CSS, advanced tables.

## Performance Tips

1. **Batch operations**: Use `office_write_range` instead of multiple `office_write_cell` calls
2. **File handles**: Close/save files between operations if modifying repeatedly
3. **Chart data**: Prefer cell references over inline data for large datasets
4. **PDF overlay**: Analyze layout once, cache coordinates for batch operations
5. **Skills**: Use pre-built skills for common patterns (faster than manual tool calls)

## Version Compatibility

- Rust: 1.70+
- MCP Protocol: 0.1.x
- Office formats: Excel 2007+, Word 2007+, PowerPoint 2007+, PDF 1.4+

## Additional Resources

- [GitHub Repository](https://github.com/Aimino-Tech/office-oxide-mcp)
- [GitHub Discussions](https://github.com/Aimino-Tech/office-oxide-mcp/discussions)
- [Showcase Examples](https://github.com/Aimino-Tech/office-oxide-mcp/tree/main/showcase)
- [Contributing Guide](https://github.com/Aimino-Tech/office-oxide-mcp/blob/main/CONTRIBUTING.md)
