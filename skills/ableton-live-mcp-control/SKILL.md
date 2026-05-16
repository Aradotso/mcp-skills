---
name: ableton-live-mcp-control
description: Control Ableton Live with AI agents via MCP - create MIDI clips, insert audio, add tracks/devices, analyze signals, automate mixing
triggers:
  - "control Ableton Live with AI"
  - "create MIDI clips in Ableton"
  - "automate Ableton Live production"
  - "set up Ableton MCP server"
  - "analyze audio signals in Ableton"
  - "generate music in Ableton with AI"
  - "add tracks and effects to Ableton"
  - "create audio spectrograms from Ableton"
---

# Ableton Live MCP Control

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This MCP server enables AI agents to control Ableton Live through arbitrary Python execution in Ableton's Live Object Model. It provides both high-level tools for common tasks and low-level access to the entire Ableton API. The server allows creating MIDI clips, inserting audio files, adding tracks/devices, analyzing audio signals, generating spectrograms, and automating mixing/mastering workflows.

## Installation

The MCP server requires Ableton Live (tested with Suite 12.3.8) on macOS or Windows.

**Automated Setup (Recommended):**
```
Set up the https://github.com/bschoepke/ableton-live-mcp MCP server for me
```

**Manual Setup:**

1. Clone the repository:
```bash
git clone https://github.com/bschoepke/ableton-live-mcp.git
cd ableton-live-mcp
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Install the Ableton Live Remote Script:
   - Copy the `RemoteScripts/MCP` folder to Ableton's Remote Scripts directory:
     - macOS: `~/Music/Ableton/User Library/Remote Scripts/`
     - Windows: `%USERPROFILE%\Documents\Ableton\User Library\Remote Scripts\`

4. Configure Ableton Live:
   - Open Ableton Live
   - Go to Preferences → Link/Tempo/MIDI
   - Add "MCP" as a Control Surface

5. Add to your MCP settings file (e.g., `claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "ableton-live": {
      "command": "python",
      "args": ["-m", "ableton_live_mcp"],
      "cwd": "/path/to/ableton-live-mcp"
    }
  }
}
```

**Important:** Back up your Live Set before using this MCP, as it can edit your set directly.

## Key MCP Tools

### evaluate_python
Execute arbitrary Python code in Ableton Live's environment.

```python
# Example: Get the current tempo
result = evaluate_python("Live.Song.Song.tempo")

# Example: Create a new MIDI track
code = """
song = Live.Song.Song
track = song.create_midi_track(-1)
track.name = 'My New Track'
"""
evaluate_python(code)
```

### create_midi_clip
Create a MIDI clip with notes on a specified track.

```python
# Create a simple C major chord progression
create_midi_clip(
    track_index=0,
    clip_slot=0,
    length_bars=4,
    notes=[
        {"pitch": 60, "start": 0.0, "duration": 1.0, "velocity": 100},  # C
        {"pitch": 64, "start": 0.0, "duration": 1.0, "velocity": 100},  # E
        {"pitch": 67, "start": 0.0, "duration": 1.0, "velocity": 100},  # G
    ]
)
```

### insert_audio_file
Insert an audio file into a track.

```python
# Insert a vocal sample
insert_audio_file(
    track_index=1,
    clip_slot=0,
    file_path="/path/to/vocal.wav"
)
```

### add_track
Add a new audio or MIDI track with optional devices.

```python
# Add a MIDI track with a synth
add_track(
    track_type="midi",
    name="Bass Synth",
    devices=["Operator", "Reverb"]
)
```

### get_live_set_info
Get comprehensive information about the current Live Set.

```python
# Get full Live Set details
info = get_live_set_info()
# Returns: tracks, scenes, tempo, time signature, devices, etc.
```

### capture_audio
Capture audio from an "Agent Audio Tap" Max for Live device.

```python
# Capture 10 seconds of audio from track 0
audio_data = capture_audio(
    track_index=0,
    device_index=0,
    duration_seconds=10.0
)
# Returns: base64-encoded WAV data
```

### get_device_parameters
Get all parameters for a device on a track.

```python
# Get parameters for the first device on track 0
params = get_device_parameters(
    track_index=0,
    device_index=0
)
```

### set_device_parameter
Set a device parameter value.

```python
# Set the cutoff frequency of a filter
set_device_parameter(
    track_index=0,
    device_index=0,
    parameter_name="Filter Freq",
    value=1200.0
)
```

## Common Patterns

### Creating a Complete Track with Instruments

```python
# Create a bass track with Operator synth
code = """
song = Live.Song.Song
track = song.create_midi_track(-1)
track.name = 'Bass'

# Add Operator synth
operator = track.devices[0]  # Default instrument
operator.name = 'Bass Synth'

# Create a bass pattern
clip = track.clip_slots[0].create_clip(4.0)
clip.name = 'Bass Pattern'

# Add notes
for i in range(16):
    if i % 4 == 0:
        clip.set_notes(((36, i * 0.25, 0.25, 100, False),))  # C1 on downbeats
"""
evaluate_python(code)
```

### Analyzing Audio with Spectrograms

```python
# First, add Agent Audio Tap device to a track in Ableton
# Then capture and analyze audio
import base64
import numpy as np
import matplotlib.pyplot as plt
from scipy import signal

# Capture audio
audio_b64 = capture_audio(track_index=0, device_index=0, duration_seconds=10.0)

# Decode audio
audio_bytes = base64.b64decode(audio_b64)

# Generate spectrogram
code = """
import wave
import numpy as np
from scipy import signal

with wave.open('temp.wav', 'rb') as wf:
    audio_data = np.frombuffer(wf.readframes(wf.getnframes()), dtype=np.int16)
    sample_rate = wf.getframerate()

f, t, Sxx = signal.spectrogram(audio_data, sample_rate)
# Save or display spectrogram
"""
```

### Setting Up a Mastering Chain

```python
# Create master processing chain
code = """
song = Live.Song.Song
master = song.master_track

# Add EQ Eight
eq = master.devices.append(song.create_device('EQ Eight'))

# Add Compressor
comp = master.devices.append(song.create_device('Compressor'))
comp.parameters[0].value = -12.0  # Threshold

# Add Limiter
limiter = master.devices.append(song.create_device('Limiter'))
limiter.parameters[1].value = -0.3  # Ceiling
"""
evaluate_python(code)
```

### Creating MIDI Clips Programmatically

```python
# Generate a chord progression
code = """
import Live

song = Live.Song.Song
track = song.tracks[0]
clip = track.clip_slots[0].create_clip(8.0)

# Chord progression: C - Am - F - G
chords = [
    [(60, 64, 67), 0.0],  # C major
    [(57, 60, 64), 2.0],  # A minor
    [(53, 57, 60), 4.0],  # F major
    [(55, 59, 62), 6.0],  # G major
]

notes = []
for chord, start_beat in chords:
    for pitch in chord:
        notes.append((pitch, start_beat, 2.0, 100, False))

clip.set_notes(tuple(notes))
clip.name = 'Chord Progression'
"""
evaluate_python(code)
```

### Automating Mixing Parameters

```python
# Create automation for volume fadeout
code = """
song = Live.Song.Song
track = song.tracks[0]
clip = track.clip_slots[0].clip

# Create volume automation
envelope = clip.automation_envelope(track.mixer_device.volume)
envelope.insert_step(0.0, 0, 1.0)   # Start at full volume
envelope.insert_step(7.0, 0, 0.0)   # Fade to silence at bar 7
"""
evaluate_python(code)
```

### Working with Audio Effects

```python
# Add and configure a reverb chain
code = """
song = Live.Song.Song
track = song.tracks[1]

# Add Reverb
reverb = track.devices.append(song.create_device('Reverb'))
reverb.parameters[0].value = 3.5   # Decay time
reverb.parameters[1].value = 0.3   # Dry/Wet

# Add EQ after reverb
eq = track.devices.append(song.create_device('EQ Eight'))
# High-pass filter to clean up reverb
eq.parameters[1].value = 300.0  # Frequency
"""
evaluate_python(code)
```

### Sidechain Compression Setup

```python
# Set up sidechain compression (kick to bass)
code = """
song = Live.Song.Song
bass_track = song.tracks[1]
kick_track = song.tracks[0]

# Add Compressor to bass track
comp = bass_track.devices.append(song.create_device('Compressor'))

# Enable sidechain
comp.parameters[9].value = 1.0  # Sidechain enabled

# Set sidechain source to kick track
available_routings = comp.available_input_routing_types
for routing in available_routings:
    if 'Audio From' in routing.display_name and kick_track.name in routing.display_name:
        comp.input_routing_type = routing
        break

# Configure compression
comp.parameters[0].value = -18.0  # Threshold
comp.parameters[1].value = 4.0    # Ratio
comp.parameters[2].value = 0.1    # Attack
comp.parameters[3].value = 0.2    # Release
"""
evaluate_python(code)
```

## Working with Third-Party Plugins

```python
# Add VST/AU instruments
code = """
song = Live.Song.Song
track = song.tracks[0]

# Add Serum (example)
serum = track.devices.append(song.create_device('Serum'))

# Add Keyscape for piano
keyscape = track.devices.append(song.create_device('Keyscape'))
"""
evaluate_python(code)
```

## Querying Live Set Information

```python
# Get detailed information about all tracks
info = get_live_set_info()

# The response includes:
# - tempo
# - time_signature
# - tracks (name, type, devices, clip_slots)
# - scenes
# - master_track info
# - return_tracks

# Example: Find all tracks with a specific device
code = """
song = Live.Song.Song
tracks_with_reverb = []

for i, track in enumerate(song.tracks):
    for device in track.devices:
        if 'Reverb' in device.name:
            tracks_with_reverb.append((i, track.name))
            break

result = tracks_with_reverb
"""
result = evaluate_python(code)
```

## Audio Signal Analysis

```python
# Analyze frequency content of a track
code = """
import numpy as np
from scipy import signal as sp_signal

# Assuming audio is captured via Agent Audio Tap
# (audio_data would be from capture_audio tool)

def analyze_frequency_content(audio_data, sample_rate=44100):
    # Compute FFT
    fft = np.fft.rfft(audio_data)
    freqs = np.fft.rfftfreq(len(audio_data), 1/sample_rate)
    magnitudes = np.abs(fft)
    
    # Find dominant frequencies
    peaks, _ = sp_signal.find_peaks(magnitudes, height=np.max(magnitudes)*0.1)
    dominant_freqs = freqs[peaks]
    
    return {
        'dominant_frequencies': dominant_freqs.tolist(),
        'frequency_range': [freqs[0], freqs[-1]],
        'peak_magnitude': float(np.max(magnitudes))
    }

# Use this function after capturing audio
"""
```

## Configuration

The MCP server connects to Ableton via OSC (Open Sound Control) on localhost.

**Default settings:**
- OSC receive port: 11000
- OSC send port: 11001

To change ports, modify the Remote Script configuration in Ableton or update the MCP server settings.

**Environment Variables:**
- No specific environment variables required
- Ensure Python path includes the MCP module directory

## Troubleshooting

### MCP Not Connecting to Ableton
1. Verify the MCP Remote Script is installed in Ableton's Remote Scripts folder
2. Check that "MCP" is selected as a Control Surface in Ableton Preferences
3. Restart Ableton Live after installing the Remote Script
4. Check firewall settings allow local OSC communication

### Evaluation Errors
```python
# Always wrap risky code in try/except when using evaluate_python
code = """
try:
    song = Live.Song.Song
    # Your code here
except Exception as e:
    result = f'Error: {str(e)}'
"""
```

### Audio Capture Issues
1. Ensure "Agent Audio Tap" Max for Live device is added to the track
2. Verify the device is enabled and receiving audio
3. Check that the track is not muted
4. Confirm audio is actually playing during capture

### Device Not Found
```python
# List all devices on a track first
code = """
song = Live.Song.Song
track = song.tracks[0]
device_names = [d.name for d in track.devices]
result = device_names
"""
devices = evaluate_python(code)
```

### Parameter Changes Not Taking Effect
- Ensure parameter names match exactly (case-sensitive)
- Some parameters may have limited ranges - check min/max values
- Verify the device is enabled
- Use `get_device_parameters()` to see available parameters

### Live Set Corruption Prevention
- Always back up your Live Set before extensive MCP operations
- Test code on a duplicate/test project first
- Use version control for your Live Sets
- Save frequently during AI-assisted production sessions

## Advanced Usage

### Custom Python Libraries
You can import and use additional Python libraries in `evaluate_python`:

```python
code = """
import numpy as np
import math

# Generate MIDI notes based on harmonic series
fundamental = 220  # A3
harmonics = [fundamental * i for i in range(1, 9)]

# Convert to MIDI note numbers
midi_notes = [int(round(69 + 12 * math.log2(f / 440))) for f in harmonics]
result = midi_notes
"""
notes = evaluate_python(code)
```

### Batch Processing Multiple Tracks

```python
# Apply same effect chain to multiple tracks
code = """
song = Live.Song.Song
effect_chain = ['EQ Eight', 'Compressor', 'Reverb']

for i in range(4):  # First 4 tracks
    track = song.tracks[i]
    for effect_name in effect_chain:
        track.devices.append(song.create_device(effect_name))
"""
evaluate_python(code)
```
