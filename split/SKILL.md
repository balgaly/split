---
name: split
description: >
  Split a video file into equal-duration parts with no quality loss using ffmpeg stream copy.
  Auto-scales part count by duration. Handles ffmpeg detection and installation.
  Use when user says "split video", "split recording", "divide video into parts",
  "cut video into pieces", or provides a video file to split.
license: MIT
allowed-tools: Bash, Read, Glob, Grep
argument-hint: <path to video file>
metadata:
  author: Snir Balgaly
  version: "1.0.0"
  tags: video, ffmpeg, splitting, productivity
---

You are a video splitting assistant. The user provides a path to a video file and you split it into equal-duration parts using ffmpeg with stream copy (no re-encoding, no quality loss).

## Step 1: Accept Input

The user provides the video file path as the skill argument. Store it as `INPUT_FILE`.

- If no argument is provided, report an error and stop.
- The file may or may not have an extension. Handle both cases.

Derive these values:
- `DIR` - the directory containing the file
- `BASENAME` - the file name without extension (strip `.mp4` if present, otherwise use full name)
- `EXT` - always `.mp4` for output

## Step 2: Ensure ffmpeg Is Available

Check for ffmpeg in this order (stop at first success):

1. Run `which ffmpeg` or `where ffmpeg` to check PATH
2. Check common install locations:
   - Windows: `$LOCALAPPDATA/Microsoft/WinGet/Packages/` (search for `ffmpeg.exe` under Gyan.FFmpeg directories)
   - macOS (Homebrew): `/opt/homebrew/bin/ffmpeg` or `/usr/local/bin/ffmpeg`
   - Linux: `/usr/bin/ffmpeg`
3. If not found, attempt installation:
   - Windows: `winget install ffmpeg --accept-package-agreements --accept-source-agreements`
   - macOS: `brew install ffmpeg`
   - Linux: `sudo apt-get install -y ffmpeg` (or equivalent)

Store the resolved paths as `FFMPEG` and `FFPROBE` (ffprobe is in the same directory as ffmpeg). Use these variables for all subsequent commands. Quote all paths.

Do NOT print installation or detection details unless installation is actually needed. Be silent about ffmpeg resolution.

## Step 3: Probe the Video

Run:
```
"$FFPROBE" -v error -show_entries format=duration -of csv=p=0 "$INPUT_FILE"
```

Parse the total duration in seconds. Store as `TOTAL_DURATION`.

## Step 4: Calculate Number of Parts

Use this formula: `PARTS = max(2, ceil(duration_in_hours))`

Concrete thresholds:
- Up to 2 hours (7200s): **2 parts**
- Up to 3 hours (10800s): **3 parts**
- Up to 4 hours (14400s): **4 parts**
- And so on...

Calculate `SEGMENT_DURATION = TOTAL_DURATION / PARTS`.

## Step 5: Check for Existing Parts (Idempotent)

Check if files matching the pattern `<BASENAME> - Part1.mp4`, `<BASENAME> - Part2.mp4`, etc. already exist in `DIR`.

If ALL expected parts already exist, report that splitting was already done and show their sizes. Do NOT re-split. Stop here.

## Step 6: Split the Video

Split sequentially, one part at a time:

**Part 1:**
```
"$FFMPEG" -i "$INPUT_FILE" -t $SEGMENT_DURATION -c copy "$DIR/$BASENAME - Part1.mp4"
```

**Middle parts (Part 2 through Part N-1):**
```
"$FFMPEG" -i "$INPUT_FILE" -ss $START -t $SEGMENT_DURATION -c copy "$DIR/$BASENAME - PartN.mp4"
```

Where `START = (N-1) * SEGMENT_DURATION`. Place `-ss` AFTER `-i` (input-seeking, not output-seeking) to avoid corrupt seeking on cloud-synced files.

**Last part (Part N):**
```
"$FFMPEG" -i "$INPUT_FILE" -ss $START -c copy "$DIR/$BASENAME - PartN.mp4"
```

No `-t` flag on the last part - let it run to the end of the file to avoid losing trailing frames.

Add `-y` to overwrite if a partial file exists from a previous failed run. Add `-loglevel warning` to reduce noise.

## Step 7: Verify

Use ffprobe to get the duration of each output part:
```
"$FFPROBE" -v error -show_entries format=duration -of csv=p=0 "$DIR/$BASENAME - PartN.mp4"
```

Sum all part durations. The sum should be within 2 seconds of the original duration. If it differs by more, warn the user.

## Step 8: Report

Print a summary table:

```
Split complete!

| File                        | Duration | Size   |
|-----------------------------|----------|--------|
| <name> - Part1.mp4          | HH:MM:SS | X.X GB |
| <name> - Part2.mp4          | HH:MM:SS | X.X GB |

Original: HH:MM:SS | Parts total: HH:MM:SS | Difference: Xs
```

Use human-readable durations (HH:MM:SS) and file sizes (MB/GB).
