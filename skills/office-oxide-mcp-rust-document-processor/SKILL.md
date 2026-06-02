---
name: office-oxide-mcp-rust-document-processor
description: Rust-native MCP server for Office document processing (Excel, Word, PowerPoint, PDF) with sub-millisecond performance
triggers:
  - process office documents with rust
  - read and write excel word powerpoint files
  - fill pdf forms programmatically
  - generate office documents from data
  - extract data from office files
  - create xlsx docx pptx files with rust
  - automate office document workflows
  - convert office documents to json markdown
---

# office-oxide-mcp-rust-document-processor

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`office-oxide-mcp` is a Rust-native MCP (Model Context Protocol) server for processing Office documents (Excel, Word, PowerPoint, PDF). It provides sub-millisecond performance for reading, writing, and manipulating office files locally without external dependencies. Think of it as an open-source alternative to Aspose with native Rust performance.

**Key capabilities:**
- **Read**: Extract content from XLSX, DOCX, PPTX, PDF to JSON/Markdown/Text/Chunks
- **Write**: Create and modify Excel spreadsheets, Word documents, PowerPoint presentations
- **PDF Forms**: Fill AcroForm/XFA fields, overlay text at coordinates, analyze layout
- **Skills System**: Pre-built templates for invoices, reports, dashboards, pitch decks
- **Coherence Engine**: Maintain consistency across linked cells and documents

**Performance**: ~10-50× faster than Python equivalents (openpyxl, python-docx, etc.)

## Installation

### Install Binary

```bash
cargo install office-oxide-mcp
```

Or download from [GitHub Releases](https://github.com/Aimino-Tech/office-oxide-mcp/releases).

### Configure MCP Client

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):
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

**Cursor** (`~/.cursor/config.json`):
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

**VS Code Copilot** (`.vscode/mcp.json`):
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

### Build from Source

```bash
git clone https://github.com/Aimino-Tech/office-oxide-mcp.git
cd office-oxide-mcp
cargo build --release
# Binary at target/release/office-oxide-mcp
```

## Core MCP Tools

### Document Information

**`list_formats`** - List all supported formats:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "list_formats"
  },
  "id": 1
}
```

**`get_document_info`** - Get file metadata:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "get_document_info",
    "arguments": {
      "file_path": "/path/to/document.xlsx"
    }
  },
  "id": 2
}
```

### Reading Documents

**`office_read`** - Read document content:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/data.xlsx",
      "output_format": "json"
    }
  },
  "id": 3
}
```

**Output formats**: `json`, `markdown`, `text`, `chunks` (for RAG embeddings)

## Excel Operations

### Create Spreadsheet

**`office_create_xlsx`** - Create new Excel file:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_create_xlsx",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Q1 Revenue"
    }
  },
  "id": 4
}
```

### Write Data

**`office_write_cell`** - Write single cell:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_write_cell",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Q1 Revenue",
      "cell": "A1",
      "value": "Revenue",
      "format": "bold"
    }
  },
  "id": 5
}
```

**`office_write_range`** - Write range of cells:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_write_range",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Q1 Revenue",
      "start_cell": "A2",
      "data": [
        ["Jan", 120000, 95000],
        ["Feb", 135000, 102000],
        ["Mar", 148000, 110000]
      ]
    }
  },
  "id": 6
}
```

### Formatting

**`office_format_range`** - Apply formatting:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_format_range",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Q1 Revenue",
      "range": "B2:C4",
      "format": {
        "number_format": "$#,##0.00",
        "bold": false,
        "font_size": 11
      }
    }
  },
  "id": 7
}
```

### Charts

**`office_create_chart`** - Create chart:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_create_chart",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Q1 Revenue",
      "chart_type": "column",
      "data_range": "A1:C4",
      "title": "Quarterly Revenue vs Cost",
      "position": "E2"
    }
  },
  "id": 8
}
```

**Chart types**: `column`, `bar`, `line`, `pie`, `scatter`, `area`

### Sheet Management

**`office_add_sheet`** - Add worksheet:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_add_sheet",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Q2 Revenue"
    }
  },
  "id": 9
}
```

**`office_merge_cells`** - Merge cells:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_merge_cells",
    "arguments": {
      "file_path": "/path/to/output.xlsx",
      "sheet_name": "Q1 Revenue",
      "range": "A1:D1"
    }
  },
  "id": 10
}
```

## Word Operations

### Create Document

**`office_create_docx`** - Create Word document:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_create_docx",
    "arguments": {
      "file_path": "/path/to/report.docx",
      "title": "Annual Business Report 2024"
    }
  },
  "id": 11
}
```

### Write from Markdown

**`office_write_docx_from_md`** - Convert Markdown to DOCX:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_write_docx_from_md",
    "arguments": {
      "file_path": "/path/to/report.docx",
      "markdown": "# Executive Summary\n\nQ1 2024 revenue increased **23%** to $1.2M.\n\n## Key Metrics\n\n- Revenue: $1,200,000\n- Growth: 23%\n- Customers: 450"
    }
  },
  "id": 12
}
```

### Add Elements

**`office_add_table`** - Insert table:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_add_table",
    "arguments": {
      "file_path": "/path/to/report.docx",
      "rows": [
        ["Metric", "Q1", "Q2", "Q3"],
        ["Revenue", "$1.2M", "$1.4M", "$1.6M"],
        ["Profit", "$320K", "$410K", "$520K"]
      ],
      "style": "grid"
    }
  },
  "id": 13
}
```

**`office_add_image`** - Insert image:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_add_image",
    "arguments": {
      "file_path": "/path/to/report.docx",
      "image_path": "/path/to/chart.png",
      "width": 400,
      "height": 300
    }
  },
  "id": 14
}
```

## PowerPoint Operations

### Create Presentation

**`office_create_pptx`** - Create PowerPoint:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_create_pptx",
    "arguments": {
      "file_path": "/path/to/deck.pptx",
      "title": "Q4 Strategy Review"
    }
  },
  "id": 15
}
```

### Add Slides

**`office_add_slide`** - Add slide:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_add_slide",
    "arguments": {
      "file_path": "/path/to/deck.pptx",
      "layout": "title_and_content",
      "title": "Market Overview",
      "content": "• Total addressable market: $2.5B\n• Current penetration: 3.2%\n• Growth rate: 18% YoY"
    }
  },
  "id": 16
}
```

**Layouts**: `title`, `title_and_content`, `section_header`, `two_content`, `comparison`, `blank`

**`office_add_chart`** - Add chart to slide:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_add_chart",
    "arguments": {
      "file_path": "/path/to/deck.pptx",
      "slide_index": 2,
      "chart_type": "bar",
      "data": {
        "categories": ["Q1", "Q2", "Q3", "Q4"],
        "series": [
          {"name": "Revenue", "values": [1.2, 1.4, 1.6, 1.9]},
          {"name": "Profit", "values": [0.32, 0.41, 0.52, 0.68]}
        ]
      },
      "title": "Quarterly Performance"
    }
  },
  "id": 17
}
```

## PDF Operations

### Read PDF

**`office_read`** - Extract PDF content:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/document.pdf",
      "output_format": "markdown"
    }
  },
  "id": 18
}
```

### Form Filling

**`office_list_pdf_fields`** - List form fields:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_list_pdf_fields",
    "arguments": {
      "file_path": "/path/to/form.pdf"
    }
  },
  "id": 19
}
```

**`office_fill_pdf_form`** - Fill form fields:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_fill_pdf_form",
    "arguments": {
      "file_path": "/path/to/form.pdf",
      "output_path": "/path/to/filled_form.pdf",
      "fields": {
        "full_name": "John Smith",
        "date_of_birth": "1985-03-15",
        "email": "john.smith@example.com",
        "address": "123 Main St, Boston, MA 02101"
      }
    }
  },
  "id": 20
}
```

### Overlay Text on Flat PDFs

**`office_analyze_pdf_layout`** - Find coordinates for text placement:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_analyze_pdf_layout",
    "arguments": {
      "file_path": "/path/to/template.pdf"
    }
  },
  "id": 21
}
```

**`office_overlay_pdf_text`** - Insert text at coordinates:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_overlay_pdf_text",
    "arguments": {
      "file_path": "/path/to/template.pdf",
      "output_path": "/path/to/filled.pdf",
      "fields": [
        {"page": 1, "x": 120, "y": 680, "text": "John Smith", "font_size": 12},
        {"page": 1, "x": 120, "y": 650, "text": "1985-03-15", "font_size": 10},
        {"page": 1, "x": 120, "y": 620, "text": "123 Main St", "font_size": 10}
      ]
    }
  },
  "id": 22
}
```

## Skills System

Pre-built templates for common document workflows.

**`skill_list`** - List available skills:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "skill_list"
  },
  "id": 23
}
```

**Available skills**:
- `excel.basic` - Financial statements, dashboards, reports
- `word.invoice` - Invoices
- `word.report` - Business reports, contracts
- `ppt.deck` - Pitch decks, QBR presentations

**`skill_run`** - Execute skill:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "skill_run",
    "arguments": {
      "skill_name": "excel.basic",
      "template": "pnl",
      "output_path": "/path/to/pnl.xlsx",
      "data": {
        "company": "Acme Corp",
        "period": "Q1 2024",
        "revenue": [
          {"category": "Products", "amount": 850000},
          {"category": "Services", "amount": 320000}
        ],
        "expenses": [
          {"category": "Cost of Sales", "amount": 520000},
          {"category": "Operating", "amount": 280000}
        ]
      }
    }
  },
  "id": 24
}
```

## Common Patterns

### Generate Excel Report from Data

```json
// 1. Create workbook
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_create_xlsx",
    "arguments": {
      "file_path": "/tmp/sales_report.xlsx",
      "sheet_name": "Sales"
    }
  },
  "id": 1
}

// 2. Write headers
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_write_range",
    "arguments": {
      "file_path": "/tmp/sales_report.xlsx",
      "sheet_name": "Sales",
      "start_cell": "A1",
      "data": [["Month", "Revenue", "Cost", "Profit"]]
    }
  },
  "id": 2
}

// 3. Write data rows
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_write_range",
    "arguments": {
      "file_path": "/tmp/sales_report.xlsx",
      "sheet_name": "Sales",
      "start_cell": "A2",
      "data": [
        ["Jan", 120000, 85000, "=B2-C2"],
        ["Feb", 135000, 92000, "=B3-C3"],
        ["Mar", 148000, 98000, "=B4-C4"]
      ]
    }
  },
  "id": 3
}

// 4. Format currency columns
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_format_range",
    "arguments": {
      "file_path": "/tmp/sales_report.xlsx",
      "sheet_name": "Sales",
      "range": "B2:D4",
      "format": {"number_format": "$#,##0"}
    }
  },
  "id": 4
}

// 5. Add chart
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_create_chart",
    "arguments": {
      "file_path": "/tmp/sales_report.xlsx",
      "sheet_name": "Sales",
      "chart_type": "column",
      "data_range": "A1:D4",
      "title": "Q1 Sales Performance",
      "position": "F2"
    }
  },
  "id": 5
}
```

### Convert Excel to JSON for Analysis

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/data.xlsx",
      "output_format": "json"
    }
  },
  "id": 1
}
```

Response structure:
```json
{
  "sheets": [
    {
      "name": "Sales",
      "rows": [
        {"Month": "Jan", "Revenue": 120000, "Cost": 85000, "Profit": 35000},
        {"Month": "Feb", "Revenue": 135000, "Cost": 92000, "Profit": 43000}
      ]
    }
  ]
}
```

### Generate Word Report from Template

```json
// Create document
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_create_docx",
    "arguments": {
      "file_path": "/tmp/monthly_report.docx",
      "title": "Monthly Business Report - March 2024"
    }
  },
  "id": 1
}

// Write content from markdown
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_write_docx_from_md",
    "arguments": {
      "file_path": "/tmp/monthly_report.docx",
      "markdown": "# Executive Summary\n\nMarch revenue reached $148K, representing 9.6% growth over February.\n\n## Financial Highlights\n\n- **Revenue**: $148,000 (+9.6%)\n- **Operating Margin**: 33.8%\n- **New Customers**: 12\n\n## Outlook\n\nQ2 pipeline indicates continued growth trajectory."
    }
  },
  "id": 2
}

// Add performance table
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_add_table",
    "arguments": {
      "file_path": "/tmp/monthly_report.docx",
      "rows": [
        ["Metric", "Jan", "Feb", "Mar"],
        ["Revenue", "$120K", "$135K", "$148K"],
        ["Margin", "29.2%", "31.9%", "33.8%"]
      ]
    }
  },
  "id": 3
}
```

### Batch Fill PDF Forms

```json
// First, analyze form structure
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_list_pdf_fields",
    "arguments": {
      "file_path": "/path/to/application_form.pdf"
    }
  },
  "id": 1
}

// Fill for each applicant
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_fill_pdf_form",
    "arguments": {
      "file_path": "/path/to/application_form.pdf",
      "output_path": "/path/to/filled/applicant_001.pdf",
      "fields": {
        "applicant_name": "Sarah Johnson",
        "ssn": "XXX-XX-XXXX",
        "address": "456 Oak Ave, Seattle, WA 98101",
        "phone": "(206) 555-0123",
        "email": "sarah.j@example.com",
        "income": "85000",
        "signature_date": "2024-03-15"
      }
    }
  },
  "id": 2
}
```

## Troubleshooting

### MCP Server Not Found

**Problem**: AI agent can't find `office-oxide-mcp` command

**Solution**: Ensure binary is in PATH:
```bash
which office-oxide-mcp
# If not found, add cargo bin to PATH
export PATH="$HOME/.cargo/bin:$PATH"
```

Or use full path in config:
```json
{
  "mcpServers": {
    "office": {
      "command": "/Users/username/.cargo/bin/office-oxide-mcp",
      "args": ["--transport", "stdio"]
    }
  }
}
```

### File Permission Errors

**Problem**: Cannot write to output path

**Solution**: Check directory permissions:
```bash
mkdir -p /path/to/output
chmod 755 /path/to/output
```

### Chart Not Appearing in Excel

**Problem**: Chart created but not visible

**Solution**: Ensure `data_range` includes headers and data:
```json
{
  "data_range": "A1:C10",  // Include header row A1
  "chart_type": "column"
}
```

### PDF Form Fields Not Filling

**Problem**: `office_fill_pdf_form` completes but fields empty

**Solution**:
1. List fields first to get exact field names:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_list_pdf_fields",
    "arguments": {"file_path": "/path/to/form.pdf"}
  },
  "id": 1
}
```

2. Use exact field names from response (case-sensitive)
3. For flat PDFs without form fields, use `office_overlay_pdf_text` instead

### Large File Performance

**Problem**: Processing large XLSX file (>100MB) is slow

**Solution**: Use chunked reading:
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "office_read",
    "arguments": {
      "file_path": "/path/to/large.xlsx",
      "output_format": "chunks",
      "chunk_size": 1000  // Process 1000 rows at a time
    }
  },
  "id": 1
}
```

### Memory Issues

**Problem**: Server crashes with large documents

**Solution**: Process in batches or use streaming:
```bash
# Increase stack size if needed
ulimit -s 16384
office-oxide-mcp --transport stdio
```

## Performance Tips

1. **Batch operations**: Use `office_write_range` instead of multiple `office_write_cell` calls
2. **Reuse files**: Open file once, make multiple edits, save once
3. **JSON format**: Use `output_format: "json"` for structured data extraction (faster than markdown)
4. **Chunks for RAG**: Use `output_format: "chunks"` for embedding large documents
5. **PDF analysis**: Run `office_analyze_pdf_layout` once, save coordinates for reuse

## Environment Variables

```bash
# Optional: Custom temp directory for large file processing
export OFFICE_OXIDE_TEMP_DIR="/path/to/temp"

# Optional: Log level (error, warn, info, debug, trace)
export RUST_LOG=office_oxide_mcp=info

# Optional: Max concurrent operations
export OFFICE_OXIDE_MAX_CONCURRENT=4
```

## Resources

- **GitHub**: https://github.com/Aimino-Tech/office-oxide-mcp
- **Discussions**: https://github.com/Aimino-Tech/office-oxide-mcp/discussions
- **Examples**: `showcase/` directory in repository
- **License**: MIT OR Apache-2.0
