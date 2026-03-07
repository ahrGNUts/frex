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

First, add the marketplace:

```
/plugin marketplace add ahrGNUts/frex
```

Then install the plugin:

```
/plugin install frex
```

For development/testing:

```bash
claude --plugin-dir /path/to/frex
```

## Usage

```
/frex <video-file> [--start N] [--end N] [--fps N] [--context "..."]
```

| Flag | Required | Default | Description |
|------|----------|---------|-------------|
| video-file | Yes | — | Path to the video file (first argument) |
| `--start` | No | 0 | Start time in seconds |
| `--end` | No | end of video | End time in seconds |
| `--fps` | No | 10 | Frames per second to extract |
| `--context` | No | — | What to look for in the frames (free-form text) |

### Examples

Extract frames from entire video at 10 fps:
```
/frex recording.mp4
```

Extract frames from seconds 10 to 30:
```
/frex recording.mp4 --start 10 --end 30
```

Extract 2 frames per second from a specific range:
```
/frex demo.mov --start 5 --end 15 --fps 2
```

Just set fps, skip other options:
```
/frex recording.mp4 --fps 5
```

Look for a specific issue:
```
/frex recording.mp4 --fps 5 --context check if the tooltip flickers when hovering
```

## Configuration

Set environment variables to override defaults. Explicit arguments always take precedence.

| Variable | Default | Description |
|----------|---------|-------------|
| `FREX_FPS` | `10` | Default frames per second |
| `FREX_OUTPUT_DIR` | `<system-temp>/claude-frames/<timestamp>` | Output directory for frames |
| `FREX_MAX_FRAMES` | `50` | Max frames to display in session |

Example — set default fps to 2:

```bash
export FREX_FPS=2
```

## Output

Frames are saved as JPG files:

```
$TMPDIR/claude-frames/1709654321/    # macOS/Linux
%TEMP%\claude-frames\1709654321\     # Windows
├── frame_001.jpg
├── frame_002.jpg
├── frame_003.jpg
└── ...
```

Claude reads each frame visually and can describe what it sees, helping you identify UI issues without manually scrubbing through a video.

## Caveats

- **Frame count can get large fast.** At the default 10 fps, a 60-second clip produces 600 frames. Use `--start`/`--end` to narrow the range, or lower `--fps` for longer recordings.
- **Long clips may produce diminishing returns.** The subagent that analyzes frames is subject to context limits. `FREX_MAX_FRAMES` (default: 50) caps how many frames are read, but even so, shorter focused clips tend to give better results than full recordings.
- **Frames are written to disk.** Output goes to your system's temp directory under `claude-frames/` by default. These are not automatically cleaned up — delete them manually or point `FREX_OUTPUT_DIR` somewhere you periodically clear.

## License

GPL-2.0
