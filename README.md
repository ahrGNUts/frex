# frex — Video Frame Extractor for Claude Code

A Claude Code plugin that extracts frames from video files for visual analysis and UI debugging.

## What it does

The `/frex` command extracts individual frames from a video file using ffmpeg, then displays them directly in your Claude Code session. Claude can see and analyze each frame, making it useful for:

- Debugging UI issues from screen recordings
- Analyzing visual regressions frame by frame
- Reviewing animations and transitions
- Inspecting transient states that are hard to screenshot manually

## Requirements

**ffmpeg** must be installed on your system:

- macOS: `brew install ffmpeg`
- Ubuntu/Debian: `sudo apt install ffmpeg`
- Windows: `winget install ffmpeg`
- Or download from [ffmpeg.org](https://ffmpeg.org/download.html)

## Installation

```bash
claude plugin install frex
```

For development/testing:

```bash
claude --plugin-dir /path/to/frex
```

## Usage

```
/frex <video-file> [start-seconds] [end-seconds] [fps]
```

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| video-file | Yes | — | Path to the video file |
| start-seconds | No | 0 | Start time in seconds |
| end-seconds | No | end of video | End time in seconds |
| fps | No | 10 | Frames per second to extract |

### Examples

Extract frames from entire video at 10 fps:
```
/frex recording.mp4
```

Extract frames from seconds 10 to 30:
```
/frex recording.mp4 10 30
```

Extract 2 frames per second from a specific range:
```
/frex demo.mov 5 15 2
```

## Configuration

Set environment variables to override defaults. Explicit arguments always take precedence.

| Variable | Default | Description |
|----------|---------|-------------|
| `FREX_FPS` | `10` | Default frames per second |
| `FREX_OUTPUT_DIR` | `/tmp/claude-frames/<timestamp>` | Output directory for frames |
| `FREX_MAX_FRAMES` | `50` | Max frames to display in session |

Example — set default fps to 2:

```bash
export FREX_FPS=2
```

## Output

Frames are saved as PNG files:

```
/tmp/claude-frames/1709654321/
├── frame_001.png
├── frame_002.png
├── frame_003.png
└── ...
```

Claude reads each frame visually and can describe what it sees, helping you identify UI issues without manually scrubbing through a video.

## License

GPL-2.0
