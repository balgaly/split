# split

Video splitter skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — split large video recordings into equal parts with no quality loss.

## Features

- **Auto part-count** — scales by duration: up to 2 hours = 2 parts, up to 3 hours = 3 parts, etc.
- **No re-encoding** — uses ffmpeg stream copy (`-c copy`) for instant splitting with zero quality loss
- **Idempotent** — detects existing parts and skips re-splitting
- **Auto-installs ffmpeg** — if ffmpeg isn't found, installs it via `winget` (Windows) automatically

## Installation

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/balgaly/split.git ~/.claude/skills/split
```

Or manually: download the ZIP, extract the inner `split/` folder into `~/.claude/skills/`.

## Usage

```
/split "path/to/video.mp4"
```

That's it. The skill handles everything: detecting ffmpeg, probing the video duration, calculating the optimal number of parts, splitting, and verifying the output.

## How It Works

| Video Duration | Parts |
|----------------|-------|
| Up to 2 hours  | 2     |
| Up to 3 hours  | 3     |
| Up to 4 hours  | 4     |
| ...            | ...   |

Formula: `parts = max(2, ceil(duration_in_hours))`

Output files are named `<original> - Part1.mp4`, `<original> - Part2.mp4`, etc., placed alongside the original file.

## Requirements

- **ffmpeg** on PATH, or **Windows with winget** for automatic installation
- Claude Code CLI

## License

[MIT](LICENSE)
