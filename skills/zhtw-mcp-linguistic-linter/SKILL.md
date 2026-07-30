---
name: zhtw-mcp-linguistic-linter
description: Traditional Chinese (zh-TW) linguistic linter enforcing Taiwan Ministry of Education standards through MCP
triggers:
  - check this text for Traditional Chinese Taiwan standards
  - lint this zh-TW document for mainland Chinese terms
  - fix Traditional Chinese punctuation and vocabulary
  - validate this text against Taiwan MoE standards
  - review this for cross-strait terminology issues
  - check Traditional Chinese character variants and spacing
  - lint markdown for zh-TW compliance
  - detect mainland Chinese vocabulary in this text
---

# zhtw-mcp-linguistic-linter

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A linguistic linter for Traditional Chinese (zh-TW) that enforces Taiwan Ministry of Education (MoE) standards on vocabulary, punctuation, and character shapes. It runs as an MCP server inside AI coding assistants and catches Mainland Chinese (zh-CN) regional drift before it reaches the user.

## What It Does

**zhtw-mcp** enforces three official Taiwan standards:

1. **Punctuation**: Taiwan-style `「」` corner brackets instead of `""` curly quotes, full-width punctuation (`，` `。` `：`)
2. **Character shapes**: MoE standard forms (裏→裡, 着→著)
3. **Vocabulary**: Cross-strait normalization (軟件→軟體, 內存→記憶體, 默認→預設)

The tool includes 1100+ vocabulary rules and 15 casing rules. For ambiguous terms, it uses the AI assistant itself to help decide context.

### Key Use Cases

- Lint Traditional Chinese documentation
- Auto-fix cross-strait terminology drift
- Enforce Taiwan MoE standards in CI/CD
- Review AI-generated Chinese text for regional correctness
- Quality gates for zh-TW content

## Installation

### Pre-built Binaries

**macOS / Linux:**
```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/sysprog21/zhtw-mcp/releases/latest/download/zhtw-mcp-installer.sh | sh
```

**Windows (PowerShell):**
```powershell
powershell -ExecutionPolicy Bypass -c "irm https://github.com/sysprog21/zhtw-mcp/releases/latest/download/zhtw-mcp-installer.ps1 | iex"
```

### Building from Source

Requires Rust 1.91+:

```bash
git clone https://github.com/sysprog21/zhtw-mcp.git
cd zhtw-mcp
make
# Binary at target/release/zhtw-mcp
```

### Quick Install with Make

```bash
make install      # build, install to ~/.local/bin, register with detected MCP clients
make uninstall    # remove binary and MCP registrations
make status       # check installation state
```

### MCP Client Registration

**Claude Code:**
```bash
claude mcp add zhtw-mcp -- /path/to/zhtw-mcp
```

**Codex CLI:**
```bash
codex mcp add zhtw -- /path/to/zhtw-mcp
```

**OpenCode:**
```bash
opencode mcp add zhtw-mcp /path/to/zhtw-mcp
```

**Generic (`.mcp.json` in project root):**
```json
{
  "mcpServers": {
    "zhtw-mcp": {
      "command": "/home/user/.local/bin/zhtw-mcp",
      "args": []
    }
  }
}
```

## CLI Usage

### Basic Commands

```bash
# Lint a file
zhtw-mcp lint README.md

# Auto-fix in place
zhtw-mcp lint file.md --fix

# Preview fixes without modifying
zhtw-mcp lint file.md --fix --dry-run

# Show telemetry summary
zhtw-mcp lint file.md --telemetry

# Clear judgment cache
zhtw-mcp cache clear
```

### Advanced CLI Options

```bash
# Use strict MoE profile
zhtw-mcp lint doc.md --profile strict

# Relaxed mode for UI strings
zhtw-mcp lint ui.md --relaxed

# Detect AI writing artifacts
zhtw-mcp lint article.md --detect-ai

# Specify content type
zhtw-mcp lint notes.txt --content-type markdown

# Set max errors for quality gate
zhtw-mcp lint doc.md --max-errors 5

# Convert Simplified to Traditional (S2T)
zhtw-mcp s2t input.md -o output.md
```

### Configuration File

Create `.zhtw.toml` in your project root:

```toml
# Profile: "base" or "strict"
profile = "base"

# Flags
relaxed = false
detect_ai = false

# Fix mode: "lexical_safe" or "none"
fix_mode = "lexical_safe"

# Max errors for quality gate (0 = unlimited)
max_errors = 0

# Content type: "auto", "plain", "markdown"
content_type = "auto"
```

## MCP Tool API

When running as an MCP server, the AI assistant calls the `zhtw` tool with JSON parameters.

### Tool: `zhtw`

**Parameters:**

```typescript
{
  text: string;              // Required: text to lint
  profile?: "base" | "strict";  // Default: "base"
  relaxed?: boolean;         // Default: false
  detect_ai?: boolean;       // Default: false
  fix_mode?: "lexical_safe" | "none";  // Default: "none"
  max_errors?: number;       // Default: 0 (unlimited)
  content_type?: "auto" | "plain" | "markdown";  // Default: "auto"
  include_telemetry?: boolean;  // Default: false
}
```

**Response:**

```typescript
{
  accepted: boolean;         // Quality gate verdict
  issues: Array<{
    line: number;
    column: number;
    message: string;
    suggestion?: string;
    rule_type: string;
  }>;
  corrected_text?: string;   // Only if fix_mode != "none"
  telemetry?: {
    input_tokens: number;
    output_tokens?: number;
    cache_creation_tokens?: number;
    cache_read_tokens?: number;
    // ...
  };
}
```

## Common Patterns

### 1. Basic Linting

**User says:** "Check this paragraph for mainland terms"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "這個軟件的默認設置存儲在內存中。"
  }
}
```

**Response:**
```json
{
  "accepted": false,
  "issues": [
    {
      "line": 1,
      "column": 3,
      "message": "Mainland term: '軟件' → '軟體'",
      "suggestion": "軟體",
      "rule_type": "vocabulary"
    },
    {
      "line": 1,
      "column": 6,
      "message": "Mainland term: '默認' → '預設'",
      "suggestion": "預設",
      "rule_type": "vocabulary"
    },
    {
      "line": 1,
      "column": 12,
      "message": "Mainland term: '內存' → '記憶體'",
      "suggestion": "記憶體",
      "rule_type": "vocabulary"
    }
  ]
}
```

### 2. Auto-fix Text

**User says:** "Fix the zh-TW issues in this document"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "這是一個測試文件,包含錯誤的標點.",
    "fix_mode": "lexical_safe"
  }
}
```

**Response:**
```json
{
  "accepted": true,
  "issues": [],
  "corrected_text": "這是一個測試文件，包含錯誤的標點。"
}
```

### 3. Strict MoE Enforcement

**User says:** "Check this with strict MoE rules"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "這裏有一些内容需要檢查。",
    "profile": "strict"
  }
}
```

**Response includes character variant issues:**
```json
{
  "accepted": false,
  "issues": [
    {
      "line": 1,
      "column": 2,
      "message": "Non-standard character variant: '裏' → '裡'",
      "suggestion": "裡",
      "rule_type": "variant"
    },
    {
      "line": 1,
      "column": 7,
      "message": "Simplified character: '内' → '內'",
      "suggestion": "內",
      "rule_type": "variant"
    }
  ]
}
```

### 4. UI String Linting (Relaxed Mode)

**User says:** "Lint this UI string, skip grammar"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "設定範圍: 1-100",
    "relaxed": true
  }
}
```

Relaxed mode:
- Disables colon/dunhao enforcement
- Uses en-dash for ranges
- Skips grammar checks

### 5. AI Writing Review

**User says:** "Review this for AI writing artifacts"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "首先，我們需要注意的是，在這個過程中，重要的是要確保系統的安全性。",
    "detect_ai": true
  }
}
```

Detects:
- Filler phrases (首先, 在這個過程中)
- Semantic safety words (需要注意的是, 重要的是)
- Excessive copula/passive voice
- Density-based patterns

### 6. Markdown-Aware Linting

**User says:** "Lint this markdown, skip code blocks"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "這是說明文件.\n\n```rust\nlet code = \"不檢查\";\n```\n\n更多內容,這裏。",
    "content_type": "markdown"
  }
}
```

Excludes from scanning:
- Fenced code blocks
- Inline code spans
- HTML blocks
- YAML frontmatter

### 7. Quality Gate with Error Threshold

**User says:** "Reject if more than 3 zh-TW errors"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "文檔內容...",
    "max_errors": 3
  }
}
```

**Response:**
```json
{
  "accepted": false,
  "issues": [
    // 4+ issues listed
  ]
}
```

### 8. Telemetry for Cost Tracking

**User says:** "Lint this and include telemetry"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "長文檔內容...",
    "include_telemetry": true
  }
}
```

**Response includes:**
```json
{
  "accepted": true,
  "issues": [],
  "telemetry": {
    "input_tokens": 1250,
    "output_tokens": 0,
    "cache_creation_tokens": 0,
    "cache_read_tokens": 0,
    "total_cost_usd": 0.000375
  }
}
```

## MCP Resources

The server exposes two read-only resources for AI assistants:

### `zh-tw://style-guide/moe`

Taiwan Ministry of Education style guide covering:
- Punctuation rules
- Character shape standards
- Vocabulary preferences

### `zh-tw://dictionary/ambiguous`

Cross-strait term disambiguation dictionary with context examples.

**Access in prompt:**
```
Please consult zh-tw://style-guide/moe before making suggestions.
```

## Rule Types

The linter reports issues with these `rule_type` values:

| Type | Description | Example |
|------|-------------|---------|
| `vocabulary` | Cross-strait vocabulary | 軟件→軟體 |
| `punctuation` | Punctuation marks | `.`→`。` |
| `spacing` | CJK-Latin/digit spacing | `zh tw`→`zh-TW` |
| `variant` | Character variants (strict) | 裏→裡 |
| `casing` | Proper noun casing | `github`→`GitHub` |
| `grammar` | Grammar issues | 台→臺 (in 臺灣) |
| `political` | Politically colored terms | 祖國, 內地 |
| `filler` | AI filler phrases (detect_ai) | 首先, 總的來說 |
| `safety` | Semantic safety (detect_ai) | 需要注意的是 |

## Profiles and Flags

### Profiles (Mutually Exclusive)

**`base` (default):**
- Cross-strait vocabulary
- Punctuation
- Casing
- Grammar
- Political terms

**`strict`:**
- Everything in `base`
- Character variants (裏→裡)
- Full MoE enforcement

### Flags (Orthogonal)

**`relaxed`:**
- Disables colon/dunhao enforcement
- Disables grammar checks
- Uses en-dash for ranges
- For UI strings, software menus

**`detect_ai`:**
- Filler phrase detection
- Semantic safety words
- Copula/passive voice checks
- Density-based pattern detection
- For reviewing AI-generated content

**Combinations:**
- `base` + `relaxed` → Lenient UI linting
- `strict` + `relaxed` → Variant normalization, lenient punctuation
- `base` + `detect_ai` → Standard linting + AI artifact detection
- `strict` + `detect_ai` → Full MoE + AI artifact detection

## CI/CD Integration

### GitHub Actions

```yaml
name: zh-TW Lint
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install zhtw-mcp
        run: |
          curl --proto '=https' --tlsv1.2 -LsSf \
            https://github.com/sysprog21/zhtw-mcp/releases/latest/download/zhtw-mcp-installer.sh | sh
          echo "$HOME/.cargo/bin" >> $GITHUB_PATH
      - name: Lint documentation
        run: |
          zhtw-mcp lint docs/**/*.md --max-errors 0
```

### GitLab CI

```yaml
zhtw-lint:
  image: rust:latest
  script:
    - curl --proto '=https' --tlsv1.2 -LsSf https://github.com/sysprog21/zhtw-mcp/releases/latest/download/zhtw-mcp-installer.sh | sh
    - export PATH="$HOME/.cargo/bin:$PATH"
    - zhtw-mcp lint docs/**/*.md --max-errors 0
```

### Pre-commit Hook

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash
set -e

# Find all staged markdown files
FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.md$' || true)

if [ -n "$FILES" ]; then
  echo "Linting zh-TW files..."
  zhtw-mcp lint $FILES --max-errors 0
fi
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

## Simplified to Traditional Conversion

Convert Simplified Chinese (zh-CN) to Traditional (zh-TW):

```bash
# File conversion
zhtw-mcp s2t input.md -o output.md

# Stdin/stdout
echo "软件开发" | zhtw-mcp s2t

# With profile
zhtw-mcp s2t input.md -o output.md --profile strict
```

**Rust API:**

```rust
use zhtw_mcp::converter::Converter;

fn main() {
    let converter = Converter::new();
    let traditional = converter.convert("软件开发");
    println!("{}", traditional);  // "軟體開發"
}
```

## Rust Library Usage

If you want to embed the linter in your own Rust project:

**Add to `Cargo.toml`:**

```toml
[dependencies]
zhtw-mcp = "0.1"
```

**Basic usage:**

```rust
use zhtw_mcp::{LintEngine, LintOptions, Profile};

fn main() {
    let engine = LintEngine::new();
    let options = LintOptions {
        profile: Profile::Base,
        relaxed: false,
        detect_ai: false,
        fix_mode: None,
        max_errors: 0,
        content_type: "auto".to_string(),
    };
    
    let text = "這個軟件的默認配置。";
    let result = engine.lint(text, &options);
    
    println!("Accepted: {}", result.accepted);
    for issue in result.issues {
        println!("{}:{} - {}", issue.line, issue.column, issue.message);
        if let Some(suggestion) = issue.suggestion {
            println!("  Suggestion: {}", suggestion);
        }
    }
}
```

**Auto-fix:**

```rust
use zhtw_mcp::{LintEngine, LintOptions, FixMode};

fn main() {
    let engine = LintEngine::new();
    let mut options = LintOptions::default();
    options.fix_mode = Some(FixMode::LexicalSafe);
    
    let text = "這是測試,包含錯誤.";
    let result = engine.lint(text, &options);
    
    if let Some(corrected) = result.corrected_text {
        println!("Fixed: {}", corrected);  // "這是測試，包含錯誤。"
    }
}
```

## Troubleshooting

### MCP Server Not Detected

**Check registration:**
```bash
claude mcp list
# or
codex mcp list
```

**Re-register:**
```bash
claude mcp remove zhtw-mcp
claude mcp add zhtw-mcp -- /path/to/zhtw-mcp
```

**Check process:**
```bash
ps aux | grep zhtw-mcp
```

### False Positives

**Override rules in config:**

Create `.zhtw.toml`:

```toml
# Disable specific rule types
[overrides]
disable_rules = ["political", "grammar"]

# Allow specific terms
allow_terms = ["內地"]  # if discussing Hong Kong geography
```

**Use relaxed mode for UI strings:**

```bash
zhtw-mcp lint ui-strings.md --relaxed
```

### Cache Issues

**Clear judgment cache:**

```bash
zhtw-mcp cache clear
```

Cache location:
- Linux: `~/.cache/zhtw-mcp/`
- macOS: `~/Library/Caches/zhtw-mcp/`
- Windows: `%LOCALAPPDATA%\zhtw-mcp\cache\`

### Performance Issues

**Batch processing:**

Instead of linting many small files individually, concatenate and lint in one pass:

```bash
cat docs/*.md | zhtw-mcp lint --content-type markdown
```

**Disable AI detection if not needed:**

AI detection uses LLM sampling for ambiguous terms. Disable for faster processing:

```bash
zhtw-mcp lint doc.md  # AI detection off by default
```

### Ambiguous Term Conflicts

For terms that vary by domain context (並發/並行, 進程/行程), the linter consults the AI assistant. If decisions are inconsistent:

1. Check cache: `zhtw-mcp cache clear`
2. Provide more context in surrounding text
3. Use explicit overrides in config

## Best Practices

### For Documentation

```bash
# Use strict profile for formal documentation
zhtw-mcp lint docs/*.md --profile strict --fix

# Quality gate in CI
zhtw-mcp lint docs/*.md --max-errors 0
```

### For UI Strings

```bash
# Use relaxed mode
zhtw-mcp lint src/i18n/zh-TW.json --relaxed

# Preserve technical terms
echo 'allow_terms = ["API", "JSON"]' >> .zhtw.toml
```

### For AI-Generated Content

```bash
# Enable AI detection
zhtw-mcp lint generated.md --detect-ai

# Combine with auto-fix
zhtw-mcp lint generated.md --detect-ai --fix
```

### For Mixed Content (Code + Docs)

```markdown
# Use markdown content type to skip code blocks
zhtw-mcp lint README.md --content-type markdown
```

## Examples from Real Projects

### Example 1: Linting a README

**Input:**
```markdown
# 我的項目

這是一個開源軟件,默認使用內存存儲.

## 安裝

使用 npm 安裝:

\`\`\`bash
npm install my-package
\`\`\`
```

**Command:**
```bash
zhtw-mcp lint README.md --content-type markdown
```

**Output:**
```
README.md:3:9: Mainland term: '軟件' → '軟體' [vocabulary]
README.md:3:11: Half-width comma should be full-width: ',' → '，' [punctuation]
README.md:3:13: Mainland term: '默認' → '預設' [vocabulary]
README.md:3:18: Mainland term: '內存' → '記憶體' [vocabulary]
README.md:3:22: Half-width period should be full-width: '.' → '。' [punctuation]
```

**Auto-fix:**
```bash
zhtw-mcp lint README.md --content-type markdown --fix
```

**Fixed output:**
```markdown
# 我的項目

這是一個開源軟體，預設使用記憶體存儲。

## 安裝

使用 npm 安裝：

\`\`\`bash
npm install my-package
\`\`\`
```

### Example 2: CI Quality Gate

**GitHub Actions workflow:**

```yaml
name: Documentation Quality
on:
  pull_request:
    paths:
      - 'docs/**/*.md'
      - 'README*.md'

jobs:
  lint-zh-tw:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install zhtw-mcp
        run: |
          curl --proto '=https' --tlsv1.2 -LsSf \
            https://github.com/sysprog21/zhtw-mcp/releases/latest/download/zhtw-mcp-installer.sh | sh
          echo "$HOME/.cargo/bin" >> $GITHUB_PATH
      
      - name: Lint Traditional Chinese
        run: |
          set -e
          CHANGED_FILES=$(git diff --name-only origin/main...HEAD | grep '\.md$' || true)
          if [ -n "$CHANGED_FILES" ]; then
            zhtw-mcp lint $CHANGED_FILES --profile strict --max-errors 0
          fi
```

### Example 3: MCP Assistant Workflow

**User:** "Review this article draft for Traditional Chinese quality"

**Assistant:**

```
I'll check the article for zh-TW compliance using strict MoE standards and AI writing detection.
```

**Tool call:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "<article content>",
    "profile": "strict",
    "detect_ai": true
  }
}
```

**Assistant response:**

```
Found 12 issues:

Vocabulary (8):
- Line 5: '軟件' → '軟體'
- Line 12: '默認' → '預設'
- Line 23: '內存' → '記憶體'
...

AI artifacts (4):
- Line 3: Filler phrase "首先" at start of sentence
- Line 15: Semantic safety phrase "需要注意的是"
- Line 28: Excessive passive voice (4 instances in paragraph)
...

Would you like me to apply auto-fixes?
```

**User:** "Yes, fix them"

**Assistant calls:**
```json
{
  "name": "zhtw",
  "arguments": {
    "text": "<article content>",
    "profile": "strict",
    "detect_ai": true,
    "fix_mode": "lexical_safe"
  }
}
```

**Assistant:** "Applied all lexical fixes. Manual review needed for 4 AI artifact issues (filler phrases require context to remove)."

## Additional Resources

- **Full CLI reference:** See project `docs/cli.md`
- **MCP tool details:** See project `docs/mcp.md`
- **Rule reference:** See project `docs/rules.md`
- **Internals:** See project `docs/internals.md`
- **Release workflow:** See project `docs/release.md`

## Summary

Use **zhtw-mcp** to:

1. **Lint** Traditional Chinese text for cross-strait terminology drift
2. **Auto-fix** punctuation, vocabulary, and spacing issues
3. **Enforce** Taiwan Ministry of Education standards
4. **Detect** AI writing artifacts in generated content
5. **Integrate** with CI/CD pipelines for quality gates
6. **Convert** Simplified Chinese to Traditional

The tool runs as both a CLI and MCP server, making it accessible from command line, scripts, and AI coding assistants.
