---
name: douyin-video-extractor
description: Extract watermark-free Douyin/TikTok videos and transcribe audio content using AI speech recognition
triggers:
  - extract text from a douyin video
  - download watermark-free douyin video
  - get douyin video transcript
  - parse douyin share link
  - transcribe douyin video audio
  - extract douyin video captions
  - download tiktok video without watermark
  - get video text from douyin url
---

# Douyin Video Extractor Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`douyin-mcp-server` extracts watermark-free videos from Douyin (Chinese TikTok) share links and uses AI to transcribe audio content into text. It supports three usage modes: WebUI, MCP server integration, and command-line interface.

**Key Features:**
- Extract high-quality watermark-free video download links
- AI-powered speech-to-text transcription using SenseVoice
- Automatic chunking for large audio files (>1 hour or >50MB)
- MCP integration for Claude Desktop and other AI assistants
- Web interface for browser-based usage

## Installation

### Prerequisites

```bash
# Install uv (Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install FFmpeg (required for audio processing)
# macOS
brew install ffmpeg

# Ubuntu/Debian
apt install ffmpeg

# Windows (with chocolatey)
choco install ffmpeg
```

### Setup

```bash
# Clone the repository
git clone https://github.com/yzfly/douyin-mcp-server.git
cd douyin-mcp-server

# Install dependencies
uv sync

# Set API key for transcription (optional, only needed for text extraction)
export API_KEY="sk-xxxxxxxxxxxxxxxx"
```

## Usage Modes

### 1. WebUI (Recommended for Interactive Use)

```bash
# Start the web server
uv run python web/app.py

# Access in browser: http://localhost:8080
```

**WebUI Features:**
- Parse video info without API key
- Extract transcripts with API key (configured in browser or env var)
- Download videos directly
- Export transcripts as Markdown

### 2. MCP Server (For AI Assistants)

Configure in `claude_desktop_config.json` or similar MCP client config:

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

**Available MCP Tools:**

- `parse_douyin_video_info` - Parse video metadata (no API key needed)
- `get_douyin_download_link` - Get watermark-free download URL (no API key needed)
- `extract_douyin_text` - Extract video transcript via AI (requires API key)

### 3. Command Line Interface

```bash
# Get video information (no API key required)
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "https://v.douyin.com/xxxxx/" \
  -a info

# Download watermark-free video
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "https://v.douyin.com/xxxxx/" \
  -a download \
  -o ./videos

# Extract transcript (requires API_KEY)
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "https://v.douyin.com/xxxxx/" \
  -a extract \
  -o ./output

# Extract transcript and save video
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "https://v.douyin.com/xxxxx/" \
  -a extract \
  -o ./output \
  --save-video
```

**CLI Arguments:**
- `-l, --link` - Douyin share link (required)
- `-a, --action` - Action: `info`, `download`, or `extract` (required)
- `-o, --output` - Output directory (default: `./output`)
- `--save-video` - Save video file when extracting transcript
- `--api-key` - Override API key from environment

## Python Integration

### Parse Video Info

```python
from douyin_video.parser import DouyinParser

# Initialize parser
parser = DouyinParser()

# Parse video information
share_link = "https://v.douyin.com/xxxxx/"
video_info = parser.parse_video_info(share_link)

print(f"Title: {video_info['title']}")
print(f"Video ID: {video_info['video_id']}")
print(f"Download URL: {video_info['download_url']}")
```

### Download Video

```python
from douyin_video.downloader import DouyinDownloader

downloader = DouyinDownloader()

# Download watermark-free video
video_url = "https://v.douyin.com/xxxxx/"
output_path = "./videos"
file_path = downloader.download_video(video_url, output_path)
print(f"Video saved to: {file_path}")
```

### Extract Transcript

```python
from douyin_video.transcriber import VideoTranscriber
import os

# Initialize with API key
api_key = os.getenv("API_KEY")
transcriber = VideoTranscriber(api_key=api_key)

# Extract transcript from video URL
video_url = "https://v.douyin.com/xxxxx/"
transcript = transcriber.extract_transcript(video_url)

print(f"Transcript: {transcript['text']}")
print(f"Video ID: {transcript['video_id']}")
print(f"Title: {transcript['title']}")

# Save as Markdown
transcriber.save_markdown(
    transcript=transcript,
    output_dir="./output"
)
```

### Handle Large Files

The library automatically handles large audio files:

```python
# Files >1 hour or >50MB are automatically chunked
# No special configuration needed
transcript = transcriber.extract_transcript(long_video_url)
# Chunks are processed and merged automatically
```

## Configuration

### API Key Setup

Get a free API key from [SiliconFlow](https://cloud.siliconflow.cn/i/TxUlXG3u) (new users get free credits).

**Option 1: Environment Variable**
```bash
export API_KEY="sk-xxxxxxxxxxxxxxxx"
```

**Option 2: WebUI Browser Storage**
1. Open WebUI
2. Click "API 未配置" button
3. Enter and save API key
4. Key persists in browser localStorage

**Option 3: CLI Argument**
```bash
uv run python douyin-video/scripts/douyin_downloader.py \
  --api-key "sk-xxxxxxxxxxxxxxxx" \
  -l "https://v.douyin.com/xxxxx/" \
  -a extract
```

### Output Format

Extracted transcripts are saved as Markdown:

```markdown
# Video Title

| 属性 | 值 |
|------|-----|
| 视频ID | `7600361826030865707` |
| 提取时间 | 2026-01-30 14:19:00 |
| 下载链接 | [点击下载](url) |

---

## 文案内容

Transcribed text content appears here...
```

## Common Patterns

### Batch Processing Multiple Videos

```python
from douyin_video.transcriber import VideoTranscriber
import os

api_key = os.getenv("API_KEY")
transcriber = VideoTranscriber(api_key=api_key)

video_urls = [
    "https://v.douyin.com/xxxxx1/",
    "https://v.douyin.com/xxxxx2/",
    "https://v.douyin.com/xxxxx3/",
]

for url in video_urls:
    try:
        transcript = transcriber.extract_transcript(url)
        transcriber.save_markdown(transcript, "./batch_output")
        print(f"✓ Processed: {transcript['title']}")
    except Exception as e:
        print(f"✗ Failed {url}: {e}")
```

### Error Handling

```python
from douyin_video.parser import DouyinParser
from douyin_video.exceptions import ParseError, DownloadError

parser = DouyinParser()

try:
    video_info = parser.parse_video_info(share_link)
except ParseError as e:
    print(f"Failed to parse video: {e}")
except DownloadError as e:
    print(f"Failed to download: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

### Custom Output Handling

```python
from douyin_video.transcriber import VideoTranscriber
import json

transcriber = VideoTranscriber(api_key=os.getenv("API_KEY"))
transcript = transcriber.extract_transcript(video_url)

# Save as JSON
with open("transcript.json", "w", encoding="utf-8") as f:
    json.dump(transcript, f, ensure_ascii=False, indent=2)

# Extract specific fields
video_id = transcript["video_id"]
text_content = transcript["text"]
download_url = transcript["download_url"]
```

## Troubleshooting

### FFmpeg Not Found

**Error:** `FileNotFoundError: [Errno 2] No such file or directory: 'ffmpeg'`

**Solution:**
```bash
# Verify FFmpeg installation
ffmpeg -version

# If not installed, install via package manager
brew install ffmpeg  # macOS
apt install ffmpeg   # Ubuntu
```

### API Key Not Working

**Error:** `Unauthorized: Invalid API key`

**Solution:**
1. Verify API key is correct
2. Check environment variable: `echo $API_KEY`
3. Ensure API key has sufficient credits at [SiliconFlow](https://cloud.siliconflow.cn)

### Large File Processing Fails

**Error:** `Request Entity Too Large` or timeout errors

**Solution:** The library automatically chunks large files, but ensure:
- FFmpeg is installed and accessible
- Sufficient disk space for temporary files
- Stable network connection for multiple API calls

### Video Link Not Parsing

**Error:** `Failed to parse video link`

**Solution:**
1. Ensure link is a valid Douyin share link (starts with `https://v.douyin.com/`)
2. Try copying the share link again from the Douyin app
3. Check if video is still available (not deleted)

### Permission Denied on Output Directory

**Error:** `PermissionError: [Errno 13] Permission denied`

**Solution:**
```bash
# Ensure output directory exists and is writable
mkdir -p ./output
chmod 755 ./output

# Or specify a different output directory
uv run python douyin-video/scripts/douyin_downloader.py \
  -l "url" -a extract -o ~/Documents/douyin_output
```

### WebUI Not Loading

**Error:** Browser shows connection refused or 404

**Solution:**
```bash
# Ensure server is running
uv run python web/app.py

# Check if port 8080 is available
lsof -i :8080

# Use different port if needed
PORT=8081 uv run python web/app.py
```

## Advanced Usage

### Custom Transcription Settings

```python
from douyin_video.transcriber import VideoTranscriber

transcriber = VideoTranscriber(
    api_key=os.getenv("API_KEY"),
    model="FunAudioLLM/SenseVoiceSmall",  # Default model
    chunk_duration=540  # 9 minutes per chunk (default)
)
```

### Programmatic MCP Server

```python
from douyin_video.mcp_server import DouyinMCPServer

server = DouyinMCPServer(api_key=os.getenv("API_KEY"))
await server.run()
```

## Related Resources

- [SiliconFlow API Documentation](https://cloud.siliconflow.cn/)
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [Project Repository](https://github.com/yzfly/douyin-mcp-server)
