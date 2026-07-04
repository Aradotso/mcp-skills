---
name: voicebox-voice-mcp-agent
description: Local-first AI voice studio for voice cloning, speech synthesis, dictation, and MCP agent voice integration
triggers:
  - how do I use voicebox for voice cloning
  - set up voicebox TTS and STT locally
  - integrate voicebox with MCP agents
  - clone a voice with voicebox
  - configure voicebox dictation hotkey
  - generate speech with voicebox API
  - troubleshoot voicebox engine startup
  - add custom TTS engine to voicebox
---

# Voicebox Voice MCP Agent

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

Voicebox is a local-first AI voice studio that combines voice input (Whisper STT + global dictation) and voice output (7 TTS engines with voice cloning) in a single TypeScript/Rust application. All inference runs on your hardware by default; audio never leaves your machine unless explicitly configured. It provides:

- **Seven TTS engines**: Qwen3-TTS, Qwen CustomVoice, LuxTTS, Chatterbox Multilingual/Turbo, HumeAI TADA, Kokoro
- **Voice cloning**: Zero-shot cloning from short reference samples
- **Global dictation**: System-wide hotkey with paste-into-focused-field
- **MCP HTTP server**: Agent-driven voice workflows at `/mcp`
- **23 languages**: Multilingual synthesis
- **Audio effects**: Pitch, reverb, delay, chorus, compression, filters
- **Local LLM refinement**: Bundled Qwen3 for transcript cleanup

**Stack**: Bun monorepo, TypeScript, Tauri 2 (Rust), React 18, Zustand, TanStack Router, Vite

## Installation

### Prerequisites

```bash
# Install Bun (JavaScript runtime + package manager)
curl -fsSL https://bun.sh/install | bash

# Install Rust (for Tauri desktop builds)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install FFmpeg (audio processing)
# macOS:
brew install ffmpeg
# Ubuntu/Debian:
sudo apt install ffmpeg
# Windows: download from ffmpeg.org
```

### Quick Setup

```bash
# Clone repository
git clone https://github.com/jamiepine/voicebox.git
cd voicebox

# Install all dependencies
just setup

# Start development mode (web UI)
just dev
```

### Development Modes

```bash
# Web UI only (http://127.0.0.1:5173)
bun run dev:web

# Tauri desktop development
bun run dev

# Build for production
bun run build

# Desktop release build
just build:desktop
```

## Configuration

### Environment Variables

Create `.env` in project root:

```bash
# API Configuration
VOICEBOX_HOST=127.0.0.1
VOICEBOX_PORT=17493
VOICEBOX_CORS_ORIGINS=http://localhost:3000,http://localhost:5173
LOG_LEVEL=info

# HuggingFace (for gated models)
HF_TOKEN=your_huggingface_token_here

# Optional: Redis Persistence Cache (for web deployments)
VOICEBOX_REDIS_ENABLED=false
VOICEBOX_REDIS_URL=redis://127.0.0.1:6379
VOICEBOX_REDIS_KEY_PREFIX=voicebox:
```

### MCP Client Configuration

Create `.mcp.json` to configure MCP client access:

```json
{
  "mcpServers": {
    "voicebox": {
      "url": "http://127.0.0.1:17493/mcp",
      "description": "Voicebox voice synthesis and dictation server"
    }
  }
}
```

## API Usage

### REST API

The service exposes a REST API at `http://127.0.0.1:17493` when running.

#### Generate Speech

```typescript
// TypeScript example: Generate speech
interface GenerationRequest {
  text: string;
  profileId: string;
  engine?: string; // optional, uses profile default
  effects?: AudioEffect[];
}

interface AudioEffect {
  type: 'pitch' | 'reverb' | 'delay' | 'chorus' | 'compression';
  params: Record<string, number>;
}

async function generateSpeech(text: string, profileId: string): Promise<Blob> {
  const response = await fetch('http://127.0.0.1:17493/generations', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      text,
      profileId,
      effects: [
        { type: 'pitch', params: { semitones: 2 } },
        { type: 'reverb', params: { roomSize: 0.5, damping: 0.5 } }
      ]
    })
  });
  
  return await response.blob();
}

// Usage
const audioBlob = await generateSpeech(
  "Hello, this is a test of voice synthesis.",
  "default-profile-id"
);

// Play audio
const audioUrl = URL.createObjectURL(audioBlob);
const audio = new Audio(audioUrl);
audio.play();
```

#### Stream Generation Progress

```typescript
// Server-Sent Events for progress tracking
async function streamGeneration(text: string, profileId: string) {
  const response = await fetch('http://127.0.0.1:17493/generations/stream', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ text, profileId })
  });

  const reader = response.body!.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = JSON.parse(line.slice(6));
        console.log('Progress:', data.progress, '%');
        console.log('Status:', data.status);
      }
    }
  }
}
```

#### Voice Cloning

```typescript
// Clone voice from reference audio
async function cloneVoice(
  name: string,
  referenceAudio: File,
  targetEngine: string = 'qwen-customvoice'
): Promise<string> {
  const formData = new FormData();
  formData.append('name', name);
  formData.append('reference', referenceAudio);
  formData.append('engine', targetEngine);

  const response = await fetch('http://127.0.0.1:17493/voices/clone', {
    method: 'POST',
    body: formData
  });

  const { voiceId } = await response.json();
  return voiceId;
}

// Usage
const fileInput = document.querySelector('input[type="file"]') as HTMLInputElement;
const audioFile = fileInput.files![0];
const voiceId = await cloneVoice('My Custom Voice', audioFile);
console.log('Voice cloned with ID:', voiceId);
```

### Service Layer (Internal)

When extending Voicebox or adding features:

```typescript
// app/src/services/tts/types.ts
export interface TTSEngine {
  name: string;
  synthesize(text: string, options: SynthesisOptions): Promise<AudioBuffer>;
  clone?(reference: AudioBuffer): Promise<VoiceModel>;
  supportedLanguages: string[];
}

export interface SynthesisOptions {
  voiceId?: string;
  language?: string;
  speed?: number;
  pitch?: number;
}

// Example: Implementing a custom TTS engine
import type { TTSEngine, SynthesisOptions } from './types';

export class CustomTTSEngine implements TTSEngine {
  name = 'custom-tts';
  supportedLanguages = ['en', 'es', 'fr'];

  async synthesize(text: string, options: SynthesisOptions): Promise<AudioBuffer> {
    // Call your TTS model here
    const response = await fetch('http://your-tts-service/synthesize', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        text,
        voice_id: options.voiceId,
        language: options.language || 'en',
        speed: options.speed || 1.0,
        pitch: options.pitch || 0
      })
    });

    const arrayBuffer = await response.arrayBuffer();
    const audioContext = new AudioContext();
    return await audioContext.decodeAudioData(arrayBuffer);
  }

  async clone(reference: AudioBuffer): Promise<VoiceModel> {
    // Implement voice cloning logic
    throw new Error('Cloning not implemented for this engine');
  }
}

// Register engine
// app/src/services/tts/registry.ts
import { CustomTTSEngine } from './engines/custom';

export const ttsEngines: TTSEngine[] = [
  new CustomTTSEngine(),
  // ... other engines
];
```

## Platform Adapters

Voicebox uses platform adapters to abstract native capabilities:

```typescript
// app/src/platform/types.ts
export interface Platform {
  // Filesystem
  readFile(path: string): Promise<Uint8Array>;
  writeFile(path: string, data: Uint8Array): Promise<void>;
  
  // Audio capture
  startAudioCapture(): Promise<MediaStream>;
  stopAudioCapture(): void;
  
  // Hotkeys (desktop only)
  registerHotkey?(combo: string, callback: () => void): Promise<void>;
  unregisterHotkey?(combo: string): Promise<void>;
  
  // Updates (desktop only)
  checkForUpdates?(): Promise<UpdateInfo | null>;
  installUpdate?(): Promise<void>;
}

// Desktop implementation (Tauri)
// tauri/src/platform/desktop.ts
import { invoke } from '@tauri-apps/api/core';
import { register, unregister } from '@tauri-apps/plugin-global-shortcut';

export const desktopPlatform: Platform = {
  async readFile(path: string): Promise<Uint8Array> {
    return await invoke('read_file', { path });
  },

  async writeFile(path: string, data: Uint8Array): Promise<void> {
    await invoke('write_file', { path, data: Array.from(data) });
  },

  async startAudioCapture(): Promise<MediaStream> {
    return await navigator.mediaDevices.getUserMedia({ audio: true });
  },

  stopAudioCapture(): void {
    // Stop all tracks
  },

  async registerHotkey(combo: string, callback: () => void): Promise<void> {
    await register(combo, callback);
  },

  async unregisterHotkey(combo: string): Promise<void> {
    await unregister(combo);
  }
};
```

## MCP Integration

### MCP Server Endpoints

The MCP server runs at `http://127.0.0.1:17493/mcp` and exposes tools for agent-driven workflows:

```typescript
// Example: MCP tool definitions
{
  "tools": [
    {
      "name": "synthesize_speech",
      "description": "Generate speech audio from text using a specific voice profile",
      "inputSchema": {
        "type": "object",
        "properties": {
          "text": { "type": "string", "description": "Text to synthesize" },
          "profile_id": { "type": "string", "description": "Voice profile ID" },
          "engine": { "type": "string", "description": "Optional TTS engine override" }
        },
        "required": ["text", "profile_id"]
      }
    },
    {
      "name": "clone_voice",
      "description": "Clone a voice from reference audio",
      "inputSchema": {
        "type": "object",
        "properties": {
          "name": { "type": "string", "description": "Name for the cloned voice" },
          "reference_path": { "type": "string", "description": "Path to reference audio" },
          "engine": { "type": "string", "description": "TTS engine to use for cloning" }
        },
        "required": ["name", "reference_path"]
      }
    },
    {
      "name": "start_dictation",
      "description": "Start voice dictation and paste into focused field",
      "inputSchema": {
        "type": "object",
        "properties": {
          "language": { "type": "string", "description": "Language code (e.g., 'en', 'es')" }
        }
      }
    }
  ]
}
```

### MCP Client Usage

```typescript
// Example: Calling MCP tools from an agent
interface MCPRequest {
  tool: string;
  arguments: Record<string, unknown>;
}

async function callMCPTool(tool: string, args: Record<string, unknown>) {
  const response = await fetch('http://127.0.0.1:17493/mcp/execute', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ tool, arguments: args })
  });

  return await response.json();
}

// Generate speech via MCP
const result = await callMCPTool('synthesize_speech', {
  text: 'Hello from the MCP agent!',
  profile_id: 'default-voice',
  engine: 'qwen3-tts'
});

console.log('Audio URL:', result.audioUrl);
```

## State Management

Voicebox uses Zustand for state management:

```typescript
// app/src/stores/voice.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface VoiceProfile {
  id: string;
  name: string;
  engine: string;
  voiceId: string;
  settings: Record<string, unknown>;
}

interface VoiceStore {
  profiles: VoiceProfile[];
  activeProfileId: string | null;
  
  addProfile: (profile: VoiceProfile) => void;
  removeProfile: (id: string) => void;
  setActiveProfile: (id: string) => void;
  updateProfile: (id: string, updates: Partial<VoiceProfile>) => void;
}

export const useVoiceStore = create<VoiceStore>()(
  persist(
    (set) => ({
      profiles: [],
      activeProfileId: null,

      addProfile: (profile) =>
        set((state) => ({
          profiles: [...state.profiles, profile]
        })),

      removeProfile: (id) =>
        set((state) => ({
          profiles: state.profiles.filter((p) => p.id !== id),
          activeProfileId: state.activeProfileId === id ? null : state.activeProfileId
        })),

      setActiveProfile: (id) =>
        set({ activeProfileId: id }),

      updateProfile: (id, updates) =>
        set((state) => ({
          profiles: state.profiles.map((p) =>
            p.id === id ? { ...p, ...updates } : p
          )
        }))
    }),
    { name: 'voice-profiles' }
  )
);

// Usage in React components
import { useVoiceStore } from './stores/voice';

function VoiceSelector() {
  const { profiles, activeProfileId, setActiveProfile } = useVoiceStore();

  return (
    <select
      value={activeProfileId || ''}
      onChange={(e) => setActiveProfile(e.target.value)}
    >
      {profiles.map((profile) => (
        <option key={profile.id} value={profile.id}>
          {profile.name}
        </option>
      ))}
    </select>
  );
}
```

## Common Patterns

### Voice Generation with Effects

```typescript
// app/src/components/SpeechGenerator.tsx
import { useState } from 'react';
import { useVoiceStore } from '../stores/voice';

export function SpeechGenerator() {
  const [text, setText] = useState('');
  const [isGenerating, setIsGenerating] = useState(false);
  const [audioUrl, setAudioUrl] = useState<string | null>(null);
  const activeProfileId = useVoiceStore((s) => s.activeProfileId);

  async function handleGenerate() {
    if (!activeProfileId || !text) return;

    setIsGenerating(true);
    try {
      const response = await fetch('http://127.0.0.1:17493/generations', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          text,
          profileId: activeProfileId,
          effects: [
            { type: 'pitch', params: { semitones: 0 } },
            { type: 'reverb', params: { roomSize: 0.3, damping: 0.4 } }
          ]
        })
      });

      const blob = await response.blob();
      const url = URL.createObjectURL(blob);
      setAudioUrl(url);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setIsGenerating(false);
    }
  }

  return (
    <div>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Enter text to synthesize..."
        rows={5}
      />
      <button onClick={handleGenerate} disabled={isGenerating}>
        {isGenerating ? 'Generating...' : 'Generate Speech'}
      </button>
      {audioUrl && (
        <audio controls src={audioUrl}>
          Your browser does not support the audio element.
        </audio>
      )}
    </div>
  );
}
```

### Global Dictation Hotkey (Desktop)

```typescript
// tauri/src/lib/dictation.ts
import { register } from '@tauri-apps/plugin-global-shortcut';
import { invoke } from '@tauri-apps/api/core';

export async function setupDictationHotkey(combo: string = 'CommandOrControl+Shift+D') {
  await register(combo, async () => {
    console.log('Dictation hotkey triggered');
    
    // Start audio capture
    await invoke('start_dictation');
    
    // Audio capture and STT happen in Rust backend
    // Transcribed text is automatically pasted into focused field
  });
}

// Rust backend handler (tauri/src-tauri/src/dictation.rs)
#[tauri::command]
async fn start_dictation() -> Result<(), String> {
    // 1. Capture audio from microphone
    // 2. Run Whisper STT inference
    // 3. Get transcribed text
    // 4. Simulate paste into focused application
    Ok(())
}
```

### Voice Cloning Workflow

```typescript
// app/src/components/VoiceCloner.tsx
import { useState } from 'react';

export function VoiceCloner() {
  const [name, setName] = useState('');
  const [file, setFile] = useState<File | null>(null);
  const [isCloning, setIsCloning] = useState(false);
  const [result, setResult] = useState<string | null>(null);

  async function handleClone() {
    if (!name || !file) return;

    setIsCloning(true);
    try {
      const formData = new FormData();
      formData.append('name', name);
      formData.append('reference', file);
      formData.append('engine', 'qwen-customvoice');

      const response = await fetch('http://127.0.0.1:17493/voices/clone', {
        method: 'POST',
        body: formData
      });

      const { voiceId } = await response.json();
      setResult(`Voice cloned successfully! ID: ${voiceId}`);
    } catch (error) {
      console.error('Cloning failed:', error);
      setResult('Cloning failed. Check console for details.');
    } finally {
      setIsCloning(false);
    }
  }

  return (
    <div>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Voice name"
      />
      <input
        type="file"
        accept="audio/*"
        onChange={(e) => setFile(e.target.files?.[0] || null)}
      />
      <button onClick={handleClone} disabled={isCloning || !name || !file}>
        {isCloning ? 'Cloning...' : 'Clone Voice'}
      </button>
      {result && <p>{result}</p>}
    </div>
  );
}
```

## Testing

### TypeScript Unit Tests

```bash
# Run all TypeScript tests
bun run test

# Run with coverage
bun run test -- --coverage

# Run specific test file
bun test src/redis/cache.test.ts
```

Example test structure:

```typescript
// src/services/tts/engine.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { TTSEngineRegistry } from './registry';
import { MockTTSEngine } from './mocks';

describe('TTS Engine Registry', () => {
  let registry: TTSEngineRegistry;

  beforeEach(() => {
    registry = new TTSEngineRegistry();
    registry.register(new MockTTSEngine());
  });

  it('should register and retrieve engines', () => {
    const engine = registry.get('mock-tts');
    expect(engine).toBeDefined();
    expect(engine?.name).toBe('mock-tts');
  });

  it('should synthesize text', async () => {
    const engine = registry.get('mock-tts')!;
    const buffer = await engine.synthesize('Hello world', {
      language: 'en',
      speed: 1.0
    });
    expect(buffer).toBeInstanceOf(AudioBuffer);
  });
});
```

### Rust Tests

```bash
# Run Tauri Rust tests
cd tauri/src-tauri
cargo test

# Run with output
cargo test -- --nocapture
```

### End-to-End Checks

```bash
# Run full CI pipeline locally
bun run ci

# This runs:
# - typecheck (all workspaces)
# - biome lint + format
# - vitest unit tests
# - web build
```

## Troubleshooting

### Service Won't Start

**Problem**: API service fails to start or port 17493 is in use.

**Solutions**:

```bash
# Check if port is already in use
lsof -i :17493  # macOS/Linux
netstat -ano | findstr :17493  # Windows

# Kill existing process
kill -9 <PID>

# Use different port
export VOICEBOX_PORT=8080
bun run dev:web

# Check service health
curl http://127.0.0.1:17493/health
```

### Missing Sidecar Binaries (Tauri)

**Problem**: `bun run dev` fails with missing sidecar error.

**Solution**:

```bash
# Create placeholder binaries for development
bun run setup:dev

# Or build actual sidecar binaries
bun run build:sidecar

# Restart Tauri dev
bun run dev
```

### TTS Engine Failures

**Problem**: Speech generation fails or returns empty audio.

**Solutions**:

```typescript
// Check engine availability
const engines = await fetch('http://127.0.0.1:17493/engines').then(r => r.json());
console.log('Available engines:', engines);

// Verify profile configuration
const profiles = await fetch('http://127.0.0.1:17493/profiles').then(r => r.json());
console.log('Voice profiles:', profiles);

// Test with simple text first
const response = await fetch('http://127.0.0.1:17493/generations', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    text: 'Test',
    profileId: 'default-profile-id',
    engine: 'kokoro'  // Try lightest engine first
  })
});

if (!response.ok) {
  const error = await response.text();
  console.error('Generation error:', error);
}
```

### Model Download Failures

**Problem**: HuggingFace model downloads fail or timeout.

**Solutions**:

```bash
# Set HuggingFace token for gated models
export HF_TOKEN=your_token_here

# Check disk space (models can be large)
df -h  # Unix
Get-PSDrive  # PowerShell

# Manually download models
huggingface-cli download model-name --local-dir ./models

# Check network/proxy settings
export HTTP_PROXY=http://proxy:port
export HTTPS_PROXY=http://proxy:port
```

### Redis Persistence Errors (Web Deployments)

**Problem**: Connection errors when persistence cache is enabled.

**Solutions**:

```bash
# Disable Redis to use in-memory fallback
export VOICEBOX_REDIS_ENABLED=false

# Verify Redis is running
docker run --rm -p 6379:6379 redis:7-alpine

# Test connection
redis-cli -h 127.0.0.1 -p 6379 ping

# Check connection string
export VOICEBOX_REDIS_URL=redis://127.0.0.1:6379
bun run bootstrap:persistence
```

### TypeScript Build Errors

**Problem**: Type errors across workspaces.

**Solutions**:

```bash
# Run typecheck for all workspaces
bun run typecheck

# Check specific workspace
cd app && bun run typecheck
cd tauri && bun run typecheck

# Clean and reinstall
rm -rf node_modules bun.lockb
bun install

# Rebuild shared types
bun run build:types
```

### Audio Capture Issues (Desktop)

**Problem**: Microphone access denied or no audio captured.

**Solutions**:

1. Grant microphone permissions in system settings
2. Check browser/app permissions
3. Test with different audio input device:

```typescript
// List available devices
const devices = await navigator.mediaDevices.enumerateDevices();
const audioInputs = devices.filter(d => d.kind === 'audioinput');
console.log('Available microphones:', audioInputs);

// Use specific device
const stream = await navigator.mediaDevices.getUserMedia({
  audio: { deviceId: { exact: audioInputs[0].deviceId } }
});
```

### GPU Acceleration Issues

**Problem**: TTS inference is slow or not using GPU.

**Solutions**:

```bash
# Check CUDA availability (NVIDIA)
nvidia-smi

# Check MLX availability (Apple Silicon)
python3 -c "import mlx; print(mlx.core.metal.is_available())"

# Force CPU fallback for testing
export VOICEBOX_FORCE_CPU=true

# Check memory usage
# GPU memory may be insufficient for larger models
export VOICEBOX_MAX_BATCH_SIZE=1
```

## Project Commands Reference

```bash
# Setup & Installation
just setup                    # Install all dependencies
bun install                   # Install JS/TS dependencies only
bun run setup:dev            # Create dev placeholder binaries

# Development
just dev                      # Start web development mode
bun run dev:web              # Web UI only
bun run dev                  # Tauri desktop dev mode

# Building
bun run build                # Build all workspaces
bun run build:web            # Build web deployment
just build:desktop           # Build Tauri desktop app

# Testing & Quality
bun run test                 # Run TypeScript unit tests
bun run typecheck            # TypeScript strict mode check
bun run check               # Biome lint + format check
bun run ci                   # Full CI pipeline locally
just check                   # Repository-wide checks

# Deployment
bun run preview              # Preview web build
docker-compose up            # Start with Redis persistence

# Troubleshooting
curl http://127.0.0.1:17493/health  # Check service health
just clean                   # Clean build artifacts
```

## Key Files

- `app/src/services/tts/` — TTS engine implementations
- `app/src/services/stt/` — Speech-to-text (Whisper)
- `app/src/stores/` — Zustand state management
- `tauri/src/platform/` — Native capability implementations
- `src/redis/` — Persistence cache layer
- `.env.redis.example` — Redis configuration template
- `justfile` — Primary task orchestration
- `biome.json` — Lint and format configuration
