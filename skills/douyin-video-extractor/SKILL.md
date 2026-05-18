---
name: douyin-video-extractor
description: Extract watermark-free Douyin/TikTok videos and AI-powered transcripts using MCP server, WebUI, or CLI
triggers:
  - extract douyin video text
  - download watermark-free douyin video
  - get douyin video transcript
  - parse douyin share link
  - extract tiktok video captions
  - download douyin without watermark
  - transcribe douyin video audio
  - setup douyin mcp server
---

# Douyin Video Extractor Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Extract watermark-free videos and AI-powered transcripts from Douyin (抖音) share links. Supports MCP server integration, WebUI, and CLI modes with automatic audio splitting for large files.

## What This Project Does

- **Watermark-free downloads**: Get high-quality video download links without watermarks
- **AI transcription**: Extract video captions using SiliconFlow's SenseVoice ASR
- **Large file support**: Automatically splits audio >1 hour or >50MB into segments
- **Multiple interfaces**: WebUI, MCP server (Claude Desktop), and CLI

## Installation

### Basic Setup

```bash
# Clone repository
git clone https://github.com/yzfly/douyin-mcp-server.git
cd douyin-mcp-server

# Install dependencies
uv sync

# Set API key (for transcription features)
export API_KEY="sk-xxxxxxxxxxxxxxxx"
```

### System Requirements

- Python 3.10+
- FFmpeg (for audio processing)
- uv package manager

Install FFmpeg:
```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt install ffmpeg

# Windows (with Chocolatey)
choco install ffmpeg
```

## MCP Server Configuration

Add to Claude Desktop or compatible MCP client config:

```json
{
  "mcpServers": {
    "douyin-mcp": {
      "command": "uvx",
      "args": ["douyin-mcp-server"],
      "env": {
        "API_KEY": "sk-xxxxxxxxxxxxxxxx"
      }
    }
  }
}

```

### Available MCP Tools

| Tool Name | Function | Requires API Key |
|-----------|----------|------------------|
| `parse_douyin_video_info` | Parse video metadata | No |
| `get_douyin_download_link` | Get watermark-free download URL | No |
| `extract_douyin_text` | Extract video transcript | Yes |

### MCP Usage Example

```
User: Extract the transcript from https://v.douyin.com/xxxxx/

Claude: [Calls extract_douyin_text tool]
I've extracted the video transcript. Here's the content:
...
```

## CLI Usage

### Basic Commands

```bash
# Show help
uv run python douyin-video/scripts/douyin_downloader.py --help

# Get video information only (no API key needed)
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "https://v.douyin.com/xxxxx/" \
  -a info

# Download watermark-free video
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "https://v.douyin.com/xxxxx/" \
  -a download \
  -o ./videos

# Extract transcript (requires API_KEY)
export API_KEY="sk-xxxxxxxxxxxxxxxx"
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "https://v.douyin.com/xxxxx/" \
  -a extract \
  -o ./output

# Extract transcript AND save video
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "https://v.douyin.com/xxxxx/" \
  -a extract \
  -o ./output \
  --save-video
```

### CLI Arguments

```
-l, --link        Douyin share link (required)
-a, --action      Action: info|download|extract (default: info)
-o, --output      Output directory (default: ./output)
--save-video      Save video when extracting transcript
```

## WebUI Usage

### Start WebUI Server

```bash
# Method 1: With environment variable API key
export API_KEY="sk-xxxxxxxxxxxxxxxx"
uv run python web/app.py

# Method 2: Configure API key in browser UI
uv run python web/app.py
```

Access at **http://localhost:8080**

### WebUI Features

1. **Paste Share Link** - Input Douyin/TikTok share URL
2. **Get Info** - Parses video metadata and download link (no API needed)
3. **Extract Transcript** - Downloads video, extracts audio, runs ASR (requires API)
4. **Download/Copy** - Download video or copy transcript as Markdown

API key can be configured in-browser (persisted in localStorage) or via environment variable.

## Code Examples

### Python Integration

```python
from douyin_video.core import DouyinVideoProcessor

# Initialize processor
processor = DouyinVideoProcessor(api_key="sk-xxxxxxxxxxxxxxxx")

# Parse video info
share_link = "https://v.douyin.com/xxxxx/"
video_info = processor.parse_video_info(share_link)
print(f"Title: {video_info['title']}")
print(f"Download: {video_info['download_url']}")

# Extract transcript
transcript = processor.extract_transcript(
    share_link=share_link,
    output_dir="./output",
    save_video=True
)
print(transcript)
```

### Batch Processing Script

```python
import os
from pathlib import Path
from douyin_video.core import DouyinVideoProcessor

# Read API key from environment
api_key = os.getenv("API_KEY")
processor = DouyinVideoProcessor(api_key=api_key)

# Process multiple videos
links = [
    "https://v.douyin.com/xxxxx/",
    "https://v.douyin.com/yyyyy/",
]

output_dir = Path("./batch_output")
output_dir.mkdir(exist_ok=True)

for link in links:
    try:
        result = processor.extract_transcript(
            share_link=link,
            output_dir=str(output_dir),
            save_video=True
        )
        print(f"✓ Processed: {link}")
    except Exception as e:
        print(f"✗ Failed {link}: {e}")
```

## Output Format

When extracting transcripts, output is saved as:

```
output/
└── {video_id}/
    ├── transcript.md    # Markdown transcript
    └── video.mp4        # Video file (if --save-video used)
```

**transcript.md structure:**

```markdown
# {Video Title}

| Attribute | Value |
|-----------|-------|
| Video ID | `7600361826030865707` |
| Extracted | 2026-01-30 14:19:00 |
| Download | [Click to download](url) |

---

## Transcript

Transcribed text content from AI speech recognition...
```

## Large File Handling

For audio files exceeding API limits (1 hour or 50MB):

1. **Auto-detection**: Checks audio duration and file size
2. **FFmpeg splitting**: Divides into 9-minute segments
3. **Sequential processing**: Transcribes each segment
4. **Merging**: Combines all transcripts

Example FFmpeg split command used internally:

```bash
ffmpeg -i input.mp3 -f segment -segment_time 540 -c copy output_%03d.mp3
```

## Configuration

### Environment Variables

```bash
# Required for transcription features
export API_KEY="sk-xxxxxxxxxxxxxxxx"

# Optional: Custom WebUI port
export PORT=8080
```

### Get Free API Key

1. Sign up at [SiliconFlow](https://cloud.siliconflow.cn/i/TxUlXG3u)
2. New users get free quota for testing
3. Uses `FunAudioLLM/SenseVoiceSmall` model

## Common Patterns

### Extract Multiple Videos with Custom Output

```python
from pathlib import Path
from douyin_video.core import DouyinVideoProcessor

processor = DouyinVideoProcessor()

videos = {
    "tutorial": "https://v.douyin.com/xxxxx/",
    "review": "https://v.douyin.com/yyyyy/",
}

for name, link in videos.items():
    output_path = Path(f"./transcripts/{name}")
    output_path.mkdir(parents=True, exist_ok=True)
    
    processor.extract_transcript(
        share_link=link,
        output_dir=str(output_path),
        save_video=False
    )
```

### Info Only (No Download)

```python
# Quick metadata extraction without downloading
processor = DouyinVideoProcessor()
info = processor.parse_video_info("https://v.douyin.com/xxxxx/")

print(f"ID: {info['video_id']}")
print(f"Title: {info['title']}")
print(f"Download URL: {info['download_url']}")
```

## Troubleshooting

### FFmpeg Not Found

```bash
# Verify FFmpeg installation
ffmpeg -version

# If not installed, install it:
# macOS: brew install ffmpeg
# Ubuntu: sudo apt install ffmpeg
```

### API Key Errors

```python
# Verify API key is set
import os
print(os.getenv("API_KEY"))  # Should not be None

# If using WebUI, check browser console for API configuration
```

### Share Link Parsing Fails

- Ensure link is complete (e.g., `https://v.douyin.com/xxxxx/`)
- Try copying link again from Douyin app
- Check if video is public/accessible

### Audio Extraction Fails

```bash
# Test FFmpeg manually
ffmpeg -i video.mp4 -vn -acodec libmp3lame audio.mp3

# Check video file integrity
ffmpeg -i video.mp4 -v error -f null -
```

### Large File Timeout

For very long videos (>2 hours), consider:

```python
# Split manually into smaller time ranges
processor.extract_transcript(
    share_link=link,
    output_dir="./output",
    max_segment_minutes=5  # Smaller segments
)
```

## API Reference

### DouyinVideoProcessor Class

```python
class DouyinVideoProcessor:
    def __init__(self, api_key: str = None):
        """Initialize processor with optional API key"""
        
    def parse_video_info(self, share_link: str) -> dict:
        """Extract video metadata without downloading"""
        
    def get_download_link(self, share_link: str) -> str:
        """Get watermark-free download URL"""
        
    def extract_transcript(
        self,
        share_link: str,
        output_dir: str = "./output",
        save_video: bool = False
    ) -> str:
        """Extract video transcript using AI ASR"""
```

## License

Apache License 2.0 - See project repository for details.
