# frex — Video Frame Extractor for Claude Code

A Claude Code plugin that extracts frames from video files for visual analysis and UI debugging.

## What it does

The `/frex` command extracts individual frames from a video file using ffmpeg, then uses a subagent to analyze them based on the context you provide. This can be useful for:

- Debugging UI issues from screen recordings
- Analyzing visual regressions frame by frame
- Reviewing animations and transitions
- Inspecting transient states that are hard to screenshot manually
- Calling out a series of issues in a UI which may span multiple screens and would be time consuming to screenshot individually

## Why not just take single screenshots?
You certainly can (and should) if a single screenshot captures the issue you're seeing in a UI!

The specific use-cases for this plugin are for providing Claude context for UI quirks that are:
1. Only present for a very brief period (like a flickering UI component or a window that appears and immediately disappears)
2. A series of rapid sequential behaviors and/or interactions

Since this plugin turns a video file into stills, it works best for short clips depending on your context window length and token budget (see [Caveats](#Caveats) below).

The default subagent model is Sonnet, though you can specify a different model inside the --context description. Opus seems to be more perceptive when analyzing the video frames, but it can use considerably more tokens (see more [Caveats](#Caveats) below).

If you need longer video input than this plugin can provide, it might be better to work Gemini's native video processing into your CC workflow via MCP.

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
/frex <video-file> [--start N] [--end N] [--fps N] [--max-frames N] [--context "..."]
```

| Flag | Required | Default | Description |
|------|----------|---------|-------------|
| video-file | Yes | — | Path to the video file (first argument) |
| `--start` | No | 0 | Start time in seconds |
| `--end` | No | end of video | End time in seconds |
| `--fps` | No | 2 | Frames per second to extract |
| `--max-frames` | No | 50 | Max frames to analyze (number or `-1` for no limit) |
| `--context` | No | — | What to look for in the frames (free-form text) |

### Examples

Extract frames from entire video at default 2 fps:
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

Analyze all extracted frames (no limit):
```
/frex recording.mp4 --start 5 --end 15 --max-frames -1
```

Look for a specific issue:
```
/frex recording.mp4 --fps 5 --context check if the tooltip flickers when hovering
```

## Configuration

Set environment variables to override defaults. Explicit arguments always take precedence.

| Variable | Default | Description |
|----------|---------|-------------|
| `FREX_FPS` | `2` | Default frames per second |
| `FREX_OUTPUT_DIR` | `<system-temp>/claude-frames/<timestamp>` | Output directory for frames |
| `FREX_MAX_FRAMES` | `50` | Max frames to display in session |

Example — set default fps to 5:

```bash
export FREX_FPS=5
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

- **Frame count can get large fast.** At the default 2 fps, a 60-second clip produces 120 frames. Use `--start`/`--end` to narrow the range, or lower `--fps` for longer recordings.
- **Long clips may produce diminishing returns.** The subagent that analyzes frames is subject to context limits. `FREX_MAX_FRAMES` (default: 50) caps how many frames are read. You can override this per-invocation with `--max-frames`, or pass `-1` to remove the cap entirely. Even so, shorter focused clips tend to give better results than long recordings.
- **Subagent model type can have a significant impact on token usage and analysis outcome** During testing, Opus 4.6(1M) and Sonnet 4.6 were both asked to review the same 60 second clip @ 2fps using the same prompt. Both models downscaled the images because they were too large for Claude Code to read. Both models were given the same prompt. Opus used nearly 180k tokens to evaluate the frames. Sonnet used around 51k. Opus paid more attention to the entire screen in the recording and called out some things Sonnet didn't notice. Sonnet focused more on the main open window and the changes that were happening in it.
- **Frames are automatically downscaled if needed.** Claude Code has a 2000px per-side limit for multi-image reading, so extracted frames are scaled down to fit within 2000x2000 while preserving the original aspect ratio. This is always enforced and cannot be bypassed, but you can request a different aspect ratio or explicit dimensions (e.g. `--context "resize to 1280x720"`) as long as neither side exceeds 2000px.
- **Frames are written to disk.** Output goes to your system's temp directory under `claude-frames/` by default. These are not automatically cleaned up — delete them manually or point `FREX_OUTPUT_DIR` somewhere you periodically clear.

## License

MIT
