---
description: Extract frames from a video file for visual analysis and UI debugging
argument-hint: <video-file> [start-seconds] [end-seconds] [fps]
allowed-tools: Bash, Read, Glob, Agent
disable-model-invocation: true
---

# Video Frame Extractor

Extract frames from a video file so you can analyze them visually. Useful for debugging UI issues from screen recordings.

## Arguments

The user provided: `$ARGUMENTS`

Parse the arguments as positional values:
- **First argument** (`$0`): path to the video file (required)
- **Second argument** (`$1`): start time in seconds (optional)
- **Third argument** (`$2`): end time in seconds (optional)
- **Fourth argument** (`$3`): frames per second to extract (optional)

If no video file path was provided, ask the user to provide one and stop.

## Environment variable overrides

Before applying built-in defaults, check for these environment variables using `echo $FREX_FPS`, `echo $FREX_OUTPUT_DIR`, and `echo $FREX_MAX_FRAMES`:

- `FREX_FPS` — default frames per second (built-in default: `10`)
- `FREX_OUTPUT_DIR` — output directory for frames (built-in default: `/tmp/claude-frames/<timestamp>`)
- `FREX_MAX_FRAMES` — max number of frames to read and display (built-in default: `50`)

**Precedence:** explicit arguments > environment variables > built-in defaults.

## Step 1: Check ffmpeg

Run:

```bash
which ffmpeg
```

If ffmpeg is NOT found, print this message and stop — do not proceed further:

```
ffmpeg is not installed. Install it with one of:

  macOS:    brew install ffmpeg
  Ubuntu:   sudo apt install ffmpeg
  Windows:  winget install ffmpeg

Or download from: https://ffmpeg.org/download.html
```

## Step 2: Validate the video file

Check that the video file exists using `test -f`. If it does not exist, tell the user the file was not found and stop.

Get video metadata by running:

```bash
ffprobe -v error -show_entries format=duration -show_entries stream=width,height -of csv=p=0 "<video-file>"
```

Report the video duration and resolution to the user. If ffprobe fails, continue anyway — the extraction may still work.

## Step 3: Extract frames

Determine the output directory. If `FREX_OUTPUT_DIR` is set, use that. Otherwise, create a timestamped directory:

```bash
mkdir -p /tmp/claude-frames/$(date +%s)
```

Store the directory path. Then run ffmpeg to extract frames based on the parsed arguments:

- If BOTH start and end times are provided:
  ```bash
  ffmpeg -y -ss <start> -to <end> -i "<video-file>" -vf fps=<fps> "<output-dir>/frame_%03d.png" 2>&1
  ```

- If ONLY start time is provided (no end time):
  ```bash
  ffmpeg -y -ss <start> -i "<video-file>" -vf fps=<fps> "<output-dir>/frame_%03d.png" 2>&1
  ```

- If NO start/end times are provided:
  ```bash
  ffmpeg -y -i "<video-file>" -vf fps=<fps> "<output-dir>/frame_%03d.png" 2>&1
  ```

If ffmpeg fails, show the error output and suggest common fixes (wrong file format, corrupted file, unsupported codec).

## Step 4: Summary

List the extracted frames:

```bash
ls "<output-dir>"/frame_*.png 2>/dev/null | wc -l
```

Print a summary including:
- Number of frames extracted
- Output directory path
- Video resolution (from Step 2)
- Time range extracted (or "full video" if no range specified)
- FPS used

## Step 5: Analyze extracted frames with a subagent

**Important:** Do NOT read the frame images directly in the main context. Instead, launch an Agent to read and analyze them. This keeps the image data out of the main context window.

Use the Agent tool with `subagent_type: "general-purpose"` and provide a prompt like:

> You are analyzing video frames extracted for UI debugging.
>
> The frames are located in: `<output-dir>`
>
> 1. Use the Glob tool to list all `frame_*.png` files in that directory.
> 2. If there are more than `<FREX_MAX_FRAMES>` frames (default: 50), only read the first N.
> 3. Use the Read tool to read each frame image file — the Read tool displays images visually.
> 4. For each frame, briefly describe what you see (UI state, any anomalies, visual differences from adjacent frames).
> 5. At the end, provide a summary of the visual progression across all frames and flag anything that looks like a bug, glitch, or transient UI state.
> 6. List the file paths of all frames so the user can reference specific ones.

After the agent returns its analysis, relay the findings to the user. If the user wants to inspect specific frames more closely, you can launch another agent or read individual frames directly.
