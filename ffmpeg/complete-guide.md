# FFmpeg Complete Guide

FFmpeg is the industry-standard toolkit for recording, converting, streaming, and analyzing audio and video. This guide covers installation, core concepts, everyday commands, advanced encoding, filters, streaming, hardware acceleration, and troubleshooting.

## 📋 Table of Contents

- [What is FFmpeg?](#-what-is-ffmpeg)
- [Installation](#-installation)
- [Core Concepts](#-core-concepts)
- [Inspecting Media](#-inspecting-media)
- [Command Syntax](#-command-syntax)
- [Common Tasks](#-common-tasks)
- [Encoding & Quality](#-encoding--quality)
- [Compressing Video File Size](#-compressing-video-file-size)
- [Cutting, Concatenation & Timing](#-cutting-concatenation--timing)
- [Audio Operations](#-audio-operations)
- [Video Transformations](#-video-transformations)
- [Filters](#-filters)
- [Subtitles & Metadata](#-subtitles--metadata)
- [Images & GIFs](#-images--gifs)
- [Streaming](#-streaming)
- [Hardware Acceleration](#-hardware-acceleration)
- [Scripting & Automation](#-scripting--automation)
- [Performance Tips](#-performance-tips)
- [Troubleshooting](#-troubleshooting)
- [Quick Reference](#-quick-reference)

---

## 🎬 What is FFmpeg?

FFmpeg is not a single program—it is a suite of libraries and CLI tools:

| Tool | Purpose |
| ---- | ------- |
| **ffmpeg** | Convert, encode, decode, mux, demux, stream, filter |
| **ffprobe** | Inspect containers, streams, codecs, metadata, frames |
| **ffplay** | Play media (useful for quick preview and filter debugging) |

Typical use cases:

- Convert `.mov` → `.mp4` for web playback
- Extract audio from video
- Resize or compress for mobile upload limits
- Generate HLS segments for adaptive streaming
- Burn subtitles into a video
- Batch-process hundreds of files in a shell script

---

## 📦 Installation

### macOS (Homebrew)

```bash
brew install ffmpeg

# Verify
ffmpeg -version
ffprobe -version
```

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install -y ffmpeg

ffmpeg -version
```

### Windows

**Option A — winget:**

```powershell
winget install Gyan.FFmpeg
```

**Option B — Chocolatey:**

```powershell
choco install ffmpeg
```

Add FFmpeg to your `PATH`, then open a new terminal and run `ffmpeg -version`.

### Docker (reproducible builds)

```bash
docker run --rm -v "$PWD:/data" -w /data jrottenberg/ffmpeg:7-scratch \
  -i input.mp4 -c:v libx264 -crf 23 output.mp4
```

### Build from source (optional)

Only needed for bleeding-edge codecs or custom flags. See [https://trac.ffmpeg.org/wiki/CompilationGuide](https://trac.ffmpeg.org/wiki/CompilationGuide).

> **Tip:** Run `ffmpeg -encoders` and `ffmpeg -decoders` to see what your build supports. Builds vary—especially on macOS vs Linux vs minimal Docker images.

---

## 🧠 Core Concepts

Understanding these terms prevents most "wrong output" mistakes.

### Container vs codec

| Term | Meaning | Examples |
| ---- | ------- | -------- |
| **Container (format)** | File wrapper that holds streams | MP4, MKV, WebM, MOV |
| **Codec** | Algorithm that compresses a stream | H.264, H.265, VP9, AAC, Opus |

A single `.mp4` file often contains:

- 1 **video** stream (e.g. H.264)
- 1 **audio** stream (e.g. AAC)
- Optional **subtitle** or **data** streams

### Stream

A stream is one continuous track inside a file. FFmpeg addresses streams with **`-map`** (see below).

### Transcode vs remux

| Operation | Re-encodes? | Speed | Quality loss |
| --------- | ----------- | ----- | ------------ |
| **Remux** (`-c copy`) | No — copies bitstreams | Very fast | None |
| **Transcode** | Yes — decodes and re-encodes | Slower | Possible |

Remux only works when the target container supports the existing codec (e.g. H.264 + AAC into MP4).

### Pixel format & color

Video frames have a **pixel format** (`yuv420p` is most compatible for H.264 web delivery). Wrong pix_fmt can cause players to reject a file or show green/purple tint.

### Key terms in commands

| Flag | Meaning |
| ---- | ------- |
| `-i` | Input file or URL |
| `-c` / `-codec` | Codec for all streams or per-type (`-c:v`, `-c:a`) |
| `-c copy` | Stream copy (no re-encode) |
| `-f` | Force container format (e.g. `mp4`, `hls`) |
| `-t` | Duration limit |
| `-ss` | Start time (seek) |
| `-y` | Overwrite output without asking |
| `-n` | Never overwrite |
| `-hide_banner` | Shorter startup output |

---

## 🔍 Inspecting Media

Always inspect before converting.

### Human-readable summary

```bash
ffprobe -hide_banner input.mp4
```

### JSON (script-friendly)

```bash
ffprobe -v quiet -print_format json -show_format -show_streams input.mp4
```

### Duration, resolution, codecs only

```bash
ffprobe -v error -select_streams v:0 -show_entries \
  stream=width,height,codec_name,r_frame_rate -of csv=p=0 input.mp4

ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 input.mp4
```

### List all formats and codecs FFmpeg supports

```bash
ffmpeg -formats    # containers
ffmpeg -codecs     # encoders/decoders
ffmpeg -encoders   # encoders only
ffmpeg -filters    # filter list
```

---

## ⌨️ Command Syntax

General pattern:

```text
ffmpeg [global options] [input options] -i input [output options] output
```

**Order matters** for some options: input options before `-i` apply to that input; output options after `-i` apply to the output file.

### Multiple inputs

```bash
ffmpeg -i video.mp4 -i audio.wav -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

### Stream mapping

| Map | Meaning |
| --- | ------- |
| `0:v:0` | First video stream of first input |
| `0:a:0` | First audio stream of first input |
| `1:a:0` | First audio stream of second input |

Without `-map`, FFmpeg picks a default mapping—which may drop subtitles or attach the wrong audio track.

### Useful global options

```bash
ffmpeg -hide_banner -loglevel warning -stats -i input.mp4 ...
```

| Log level | When to use |
| --------- | ------------- |
| `quiet` | Scripts—errors only |
| `warning` | Daily use |
| `info` | Default |
| `debug` | Diagnosing filter/codec issues |

---

## ⚡ Common Tasks

Replace `input.mp4` / `output.mp4` with your paths. Add `-y` to overwrite without prompting.

### Convert to H.264 + AAC (web-friendly MP4)

```bash
ffmpeg -i input.mp4 -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k output.mp4
```

### Fast remux (no re-encode)

```bash
ffmpeg -i input.mkv -c copy output.mp4
```

> MKV → MP4 remux fails if codecs aren't MP4-compatible (e.g. some DTS audio). Re-encode audio: `-c:a aac`.

### Convert to WebM (VP9 + Opus)

```bash
ffmpeg -i input.mp4 -c:v libvpx-vp9 -crf 30 -b:v 0 -c:a libopus -b:a 128k output.webm
```

VP9 is slower than H.264 but royalty-free.

### Extract audio only

```bash
# MP3
ffmpeg -i input.mp4 -vn -c:a libmp3lame -q:a 2 output.mp3

# AAC
ffmpeg -i input.mp4 -vn -c:a copy output.aac

# WAV (uncompressed)
ffmpeg -i input.mp4 -vn output.wav
```

`-vn` disables video.

### Extract video only (no audio)

```bash
ffmpeg -i input.mp4 -an -c:v copy output_noaudio.mp4
```

### Mute video (keep video, silent audio)

```bash
ffmpeg -i input.mp4 -c:v copy -an output_silent.mp4
```

### Replace audio track

```bash
ffmpeg -i video.mp4 -i narration.wav -c:v copy -map 0:v:0 -map 1:a:0 -shortest output.mp4
```

`-shortest` finishes when the shortest stream ends.

### Combine video + image as thumbnail intro (example)

```bash
ffmpeg -loop 1 -i cover.jpg -i clip.mp4 -filter_complex \
  "[0:v]scale=1280:720,setpts=PTS-STARTPTS,fps=30[v0];[1:v]scale=1280:720[v1];[v0][v1]concat=n=2:v=1:a=0[v]" \
  -map "[v]" -map 1:a? -c:v libx264 -crf 23 -c:a aac -shortest output.mp4
```

---

## 🎚️ Encoding & Quality

### Constant Rate Factor (CRF) — recommended for files

CRF targets **quality**, not bitrate. Lower = better quality, larger file.

| Codec | Typical CRF | Notes |
| ----- | ----------- | ----- |
| H.264 (`libx264`) | 18–28 | 23 is a common default |
| H.265 (`libx265`) | 24–32 | ~40–50% smaller than H.264 at similar quality |
| VP9 (`libvpx-vp9`) | 30–35 | Slow; use `-b:v 0` with CRF-style mode |

```bash
ffmpeg -i input.mp4 -c:v libx264 -crf 20 -preset slow -c:a aac -b:a 192k high_quality.mp4
```

**x264 presets** (`-preset`): `ultrafast` … `placebo`. Slower = better compression efficiency (smaller file at same CRF), not necessarily better visual quality.

### Target bitrate (streaming / broadcast)

```bash
ffmpeg -i input.mp4 -c:v libx264 -b:v 2500k -maxrate 2500k -bufsize 5000k \
  -c:a aac -b:a 128k output.mp4
```

### Two-pass encoding (H.264)

Better bitrate adherence for size caps:

```bash
ffmpeg -y -i input.mp4 -c:v libx264 -b:v 2500k -pass 1 -an -f mp4 /dev/null
ffmpeg -i input.mp4 -c:v libx264 -b:v 2500k -pass 2 -c:a aac -b:a 128k output.mp4
```

On Windows use `NUL` instead of `/dev/null`.

### Audio quality

| Method | Example |
| ------ | ------- |
| AAC bitrate | `-c:a aac -b:a 192k` |
| MP3 VBR | `-c:a libmp3lame -q:a 2` (0 best, 9 worst) |
| Copy | `-c:a copy` when container matches |

### Resolution & aspect ratio

```bash
# Scale to 1280 width, preserve aspect (-2 keeps even height for h264)
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -crf 23 -c:a copy output.mp4

# Fit inside 1920x1080 box with padding
ffmpeg -i input.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" \
  -c:v libx264 -crf 23 -c:a copy output.mp4
```

### Frame rate

```bash
# Force 30 fps (re-timing; may blend/drop frames)
ffmpeg -i input.mp4 -r 30 -c:v libx264 -crf 23 -c:a copy output_30fps.mp4

# fps filter (more control)
ffmpeg -i input.mp4 -vf fps=30 -c:v libx264 -crf 23 -c:a copy output_30fps.mp4
```

> For step-by-step size reduction (target MB, email/WhatsApp limits, maximum compression), see [Compressing Video File Size](#-compressing-video-file-size).

---

## 📉 Compressing Video File Size

This section focuses on **making files smaller** while keeping acceptable quality. File size is determined by **duration × bitrate** (video + audio). To shrink a file, you lower bitrate, use a more efficient codec, reduce resolution/frame rate, or shorten the clip.

### What affects file size

| Factor | Impact on size | How to reduce |
| ------ | ---------------- | ------------- |
| **Duration** | Linear | Trim with `-ss` / `-t` (see [Cutting](#-cutting-concatenation--timing)) |
| **Resolution** | Large (4K vs 720p can be 4–8×) | Scale down with `-vf scale=` |
| **Frame rate** | Moderate | `-r 24` or `fps=24` for talking-head / screen content |
| **Video codec** | Large | H.265 (~40–50% smaller than H.264 at similar quality) |
| **CRF / bitrate** | Large | Raise CRF or cap `-b:v` |
| **Audio** | Small–moderate | `-b:a 96k` or `64k`, mono `-ac 1` |
| **Extra streams** | Variable | Drop subtitles/data with `-map` |

### Step 1: Measure the original

```bash
# File size (human-readable)
ls -lh input.mp4

# Duration in seconds
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 input.mp4

# Resolution and video bitrate
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,bit_rate,codec_name -of csv=p=0 input.mp4
```

**Rough size math:**

```text
file_size (bits) ≈ (video_bitrate + audio_bitrate) × duration
file_size (MB)   ≈ (video_kbps + audio_kbps) × duration_sec / 8000
```

Example: 5 minutes (300 s) at 8000 kbps video + 128 kbps audio → ~(8128 × 300) / 8000 ≈ **305 MB**.

### Step 2: Choose a compression strategy

Use this decision flow:

```text
Need exact max size (e.g. 25 MB email limit)?
  → Target bitrate + two-pass (below)

OK with "smaller but good looking"?
  → CRF + maybe scale down (fastest workflow)

Need smallest file, quality secondary?
  → H.265 + higher CRF + 720p + lower audio
```

| Goal | Recommended approach |
| ---- | -------------------- |
| **Light compress** (~30% smaller) | H.264 CRF 26–28, same resolution |
| **Balanced** (~50–60% smaller) | 1080p→720p + CRF 26, AAC 128k |
| **Aggressive** (~70–80% smaller) | H.265, 720p or 480p, CRF 28–30, AAC 96k |
| **Exact cap** (16 / 25 / 50 MB) | Calculate bitrate → two-pass |

---

### Method 1: CRF (quality-based — easiest)

Raise **CRF** to reduce size. Each +6 on H.264 roughly **halves** the bitrate (and file size) for the same source type.

| H.264 CRF | Typical use | Size vs CRF 23 |
| --------- | ----------- | -------------- |
| 18–20 | High quality archive | Larger |
| 23 | Default / web | Baseline |
| 26–28 | Smaller web, social | ~40–60% smaller |
| 30–32 | Previews, thumbnails | Much smaller (visible softness) |

**Balanced web compress (good starting point):**

```bash
ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 26 -pix_fmt yuv420p \
  -c:a aac -b:a 128k -movflags +faststart compressed.mp4
```

**Smaller file (accept some quality loss):**

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -preset slow -crf 28 \
  -c:a aac -b:a 96k -movflags +faststart small.mp4
```

**Maximum CRF compress (H.264, 720p):**

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -preset slow -crf 30 \
  -c:a aac -b:a 64k -ac 1 -movflags +faststart tiny.mp4
```

> **Tip:** `-preset slow` (or `slower`) does **not** change quality at the same CRF—it improves compression efficiency, so the file is **smaller** for the same visual result. Encoding takes longer.

---

### Method 2: H.265 / HEVC (~40–50% smaller)

Best when recipients can play H.265 (modern phones, desktops; some older browsers struggle).

```bash
ffmpeg -i input.mp4 -c:v libx265 -preset medium -crf 28 -tag:v hvc1 \
  -c:a aac -b:a 128k -movflags +faststart hevc.mp4
```

| H.265 CRF | Rough H.264 equivalent |
| --------- | ---------------------- |
| 24 | CRF ~20 |
| 28 | CRF ~23–24 |
| 32 | CRF ~28 |

**Aggressive H.265 + 720p:**

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx265 -preset medium -crf 30 \
  -c:a aac -b:a 96k -movflags +faststart small_hevc.mp4
```

---

### Method 3: Lower resolution (often the biggest win)

Downscaling 4K → 1080p or 1080p → 720p often saves more than tweaking CRF alone.

| Source | Suggested max width | Example scale |
| ------ | ------------------- | ------------- |
| 4K (3840×2160) | 1920 (1080p) | `scale=1920:-2` |
| 1080p | 1280 (720p) | `scale=1280:-2` |
| 1080p / long talking head | 854 (480p) | `scale=854:-2` |
| Screen recording | 1280 or native | Keep text readable |

```bash
# 4K → 1080p, balanced quality
ffmpeg -i input.mp4 -vf "scale=1920:-2" -c:v libx264 -preset slow -crf 24 \
  -c:a aac -b:a 128k -movflags +faststart hd.mp4

# 1080p → 720p, smaller upload
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -preset slow -crf 26 \
  -c:a aac -b:a 96k -movflags +faststart sd.mp4
```

`-2` keeps aspect ratio and forces an **even** height (required for H.264/H.265).

---

### Method 4: Lower frame rate

Useful for slides, podcasts with static visuals, or security footage—not ideal for sports.

```bash
# 60 fps → 30 fps
ffmpeg -i input.mp4 -vf "fps=30,scale=1280:-2" -c:v libx264 -crf 26 \
  -c:a aac -b:a 128k -movflags +faststart fps30.mp4

# 30 fps → 24 fps (cinema-style; slightly smaller)
ffmpeg -i input.mp4 -vf "fps=24,scale=1280:-2" -c:v libx264 -crf 26 \
  -c:a aac -b:a 128k -movflags +faststart fps24.mp4
```

---

### Method 5: Compress audio

Audio is usually 5–15% of total size but easy to trim:

```bash
# Stereo 192k → 128k (transparent for most speech/music)
-c:a aac -b:a 128k

# Voice / podcast — mono 96k
-c:a aac -b:a 96k -ac 1

# Minimal audio (acceptable for background music)
-c:a aac -b:a 64k -ac 1
```

Combine with video settings; don't use `-c:a copy` if you need a smaller audio track.

---

### Method 6: Target a specific file size (two-pass)

When you need **≤ N MB** (email attachment, Discord, LMS upload), compute video bitrate:

```text
video_bitrate (kbps) = (target_size_MB × 8192) / duration_sec - audio_kbps - safety_margin
```

Use a **safety margin** of 50–100 kbps so the container overhead doesn't push you over.

**Example:** 10-minute video (600 s), target **25 MB**, audio **96 kbps**:

```text
(25 × 8192) / 600 - 96 ≈ 341 - 96 ≈ 245 kbps video
```

**Two-pass H.264 to ~25 MB:**

```bash
DURATION=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 input.mp4)
TARGET_MB=25
AUDIO_K=96
VIDEO_K=$(echo "scale=0; ($TARGET_MB * 8192 / $DURATION) - $AUDIO_K - 80" | bc)

ffmpeg -y -i input.mp4 -c:v libx264 -b:v ${VIDEO_K}k -maxrate ${VIDEO_K}k -bufsize $((VIDEO_K * 2))k \
  -pass 1 -an -f mp4 /dev/null

ffmpeg -i input.mp4 -c:v libx264 -b:v ${VIDEO_K}k -maxrate ${VIDEO_K}k -bufsize $((VIDEO_K * 2))k \
  -pass 2 -c:a aac -b:a ${AUDIO_K}k -movflags +faststart output_25mb.mp4
```

On Windows, replace `/dev/null` with `NUL` and compute `VIDEO_K` manually or in PowerShell.

**Also scale down** if the calculated bitrate is very low (< 400 kbps at 1080p)—quality will break down; use 720p:

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -b:v 800k -maxrate 800k -bufsize 1600k \
  -pass 1 -an -f mp4 /dev/null
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -b:v 800k -maxrate 800k -bufsize 1600k \
  -pass 2 -c:a aac -b:a 96k -movflags +faststart capped.mp4
```

---

### Method 7: Strip extras (no re-encode or light remux)

Removing unused streams doesn't recompress video but can shave megabytes:

```bash
# Video + one audio track only (drop subs, extra audio, chapters data)
ffmpeg -i input.mp4 -map 0:v:0 -map 0:a:0 -c copy trimmed_streams.mp4

# Strip metadata
ffmpeg -i input.mp4 -map_metadata -1 -c copy no_meta.mp4
```

For real size reduction of the video itself, you must **re-encode** (`-c:v libx264` or `libx265`), not `-c copy`.

---

### Ready-made compression recipes

#### Recipe A: "Half the size" (same resolution, good quality)

```bash
ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 26 -c:a aac -b:a 128k \
  -movflags +faststart half-ish.mp4
```

#### Recipe B: Email-friendly (~25 MB for ~10 min, adjust as needed)

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -preset slow -crf 28 \
  -c:a aac -b:a 96k -ac 1 -movflags +faststart email.mp4
```

If still too large, use [Method 6](#method-6-target-a-specific-file-size-two-pass) or shorten the clip.

#### Recipe C: WhatsApp / messaging (short clip, ~720p)

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -preset fast -crf 28 \
  -c:a aac -b:a 96k -movflags +faststart whatsapp.mp4
```

Check current WhatsApp size limits—they change by platform and file type.

#### Recipe D: Web embed (fast start, reasonable size)

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx264 -preset slow -crf 24 -pix_fmt yuv420p \
  -c:a aac -b:a 128k -movflags +faststart web.mp4
```

#### Recipe E: Smallest practical MP4 (H.265 + 720p + high CRF)

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2" -c:v libx265 -preset medium -crf 32 \
  -c:a aac -b:a 64k -ac 1 -movflags +faststart smallest.mp4
```

---

### Compare before and after

```bash
ls -lh input.mp4 compressed.mp4

# Per-stream bitrates
ffprobe -v error -show_entries format=size,duration -show_entries stream=codec_type,bit_rate \
  -of json input.mp4 compressed.mp4
```

Quick quality check: play both side by side in VLC or QuickTime. For objective metrics (optional, requires `libvmaf` build):

```bash
# SSIM between original and compressed (lower score = more difference)
ffmpeg -i input.mp4 -i compressed.mp4 -lavfi ssim -f null -
```

---

### Compression workflow (recommended)

1. **Inspect** — duration, resolution, current size (`ffprobe`, `ls -lh`).
2. **Pick target** — e.g. "under 50 MB" or "~50% smaller".
3. **Scale first** if source is 4K or 1080p and target is mobile/web.
4. **Encode** — start with CRF 26–28 + AAC 128k; or two-pass for hard caps.
5. **Check size** — if still too large, raise CRF by 2 or scale to 720p.
6. **Avoid double compression** — don't re-encode the same file repeatedly; always keep the **original master**.

### Common mistakes

| Mistake | Why it's bad | Fix |
| ------- | -------------- | --- |
| `-c copy` expecting smaller file | Only remuxes; no video compression | Use `libx264` / `libx265` |
| CRF 18 on already-compressed video | Huge file, little quality gain | Use CRF 24–28 on delivery encodes |
| 1080p at 300 kbps (two-pass) | Blocky mush | Scale to 720p/480p or raise target MB |
| Odd dimensions (e.g. 1279×719) | Encoder errors or player issues | Use `scale=1280:-2` |
| Ignoring audio | Misses easy savings | `-b:a 96k -ac 1` for voice |

---

## ✂️ Cutting, Concatenation & Timing

### Fast cut (stream copy) — keyframe-aligned

Cuts on **keyframes** only unless you re-encode. Very fast.

```bash
ffmpeg -ss 00:01:30 -i input.mp4 -t 00:00:45 -c copy clip.mp4
```

- `-ss` before `-i`: fast seek (less accurate)
- `-ss` after `-i`: accurate seek (slower)

### Accurate cut (re-encode)

```bash
ffmpeg -ss 00:01:30 -i input.mp4 -t 45 -c:v libx264 -crf 23 -c:a aac clip.mp4
```

### Concat demuxer (same codec, same resolution)

1. Create `files.txt`:

```text
file 'part1.mp4'
file 'part2.mp4'
file 'part3.mp4'
```

2. Concatenate:

```bash
ffmpeg -f concat -safe 0 -i files.txt -c copy merged.mp4
```

Paths in `files.txt` must be escaped for special characters. Use absolute paths to avoid surprises.

### Concat filter (different formats/resolutions)

Re-encodes to unify streams:

```bash
ffmpeg -i a.mp4 -i b.mp4 -filter_complex "[0:v][0:a][1:v][1:a]concat=n=2:v=1:a=1[v][a]" \
  -map "[v]" -map "[a]" -c:v libx264 -crf 23 -c:a aac merged.mp4
```

---

## 🔊 Audio Operations

### Change volume

```bash
ffmpeg -i input.mp4 -af "volume=0.5" -c:v copy quiet.mp4
```

### Normalize (EBU R128-style loudnorm)

```bash
ffmpeg -i input.mp4 -af loudnorm=I=-16:TP=-1.5:LRA=11 -c:v copy normalized.mp4
```

Two-pass loudnorm is recommended for broadcast; see FFmpeg `loudnorm` filter docs for measured values.

### Mix two audio sources

```bash
ffmpeg -i video.mp4 -i music.mp3 -filter_complex \
  "[1:a]volume=0.3[music];[0:a][music]amix=inputs=2:duration=first[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac mixed.mp4
```

### Resample / change channels

```bash
ffmpeg -i input.mp4 -ar 48000 -ac 2 -c:a aac -c:v copy output.mp4
```

---

## 🖼️ Video Transformations

### Crop

```bash
# crop=width:height:x:y
ffmpeg -i input.mp4 -vf "crop=1280:720:0:60" -c:a copy cropped.mp4
```

### Rotate

```bash
# 90° clockwise
ffmpeg -i input.mp4 -vf "transpose=1" -c:a copy rotated.mp4
```

### Flip

```bash
ffmpeg -i input.mp4 -vf "hflip" -c:a copy mirrored.mp4
```

### Watermark (overlay)

```bash
ffmpeg -i video.mp4 -i logo.png -filter_complex \
  "overlay=10:10" -c:a copy watermarked.mp4
```

### Picture-in-picture

```bash
ffmpeg -i main.mp4 -i cam.mp4 -filter_complex \
  "[1:v]scale=320:180[pip];[0:v][pip]overlay=W-w-20:20" -c:a copy pip.mp4
```

### Remove black bars (auto crop detect)

```bash
# Detect (run and read crop= values from log)
ffmpeg -i input.mp4 -vf cropdetect -f null -

# Apply
ffmpeg -i input.mp4 -vf "crop=1280:720:0:56" -c:a copy cropped.mp4
```

---

## 🎛️ Filters

Filters are chained with commas (`,`) on one chain, or semicolons (`;`) in `-filter_complex` for graphs with multiple inputs/outputs.

### Scale + denoise + sharpen (example chain)

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-2,hqdn3d=1.5:1.5:6:6,unsharp=5:5:1.0" \
  -c:v libx264 -crf 22 -c:a copy cleaned.mp4
```

### Blur faces (requires drawbox or external ML)

FFmpeg alone doesn't detect faces; integrate OpenCV or use `delogo` for fixed regions:

```bash
ffmpeg -i input.mp4 -vf "delogo=x=100:y=50:w=200:h=80" -c:a copy delogo.mp4
```

### Fade in / out

```bash
ffmpeg -i input.mp4 -vf "fade=t=in:st=0:d=1,fade=t=out:st=9:d=1" -c:a copy faded.mp4
```

### Color correction (eq)

```bash
ffmpeg -i input.mp4 -vf "eq=brightness=0.06:saturation=1.2" -c:a copy graded.mp4
```

### Preview filter with ffplay

```bash
ffplay -i input.mp4 -vf "hue=h=90"
```

Press `q` to quit.

---

## 📝 Subtitles & Metadata

### Extract subtitles

```bash
ffmpeg -i input.mkv -map 0:s:0 -c copy subs.srt
```

### Burn-in (hardcode) subtitles

```bash
ffmpeg -i video.mp4 -vf "subtitles=subs.srt" -c:a copy burned.mp4
```

Escape paths on Windows; use forward slashes or quoted paths.

### Copy metadata; set title

```bash
ffmpeg -i input.mp4 -c copy -metadata title="My Video" -metadata artist="Studio" output.mp4
```

### Strip all metadata

```bash
ffmpeg -i input.mp4 -map_metadata -1 -c copy clean.mp4
```

---

## 🖼️ Images & GIFs

### Extract single frame

```bash
ffmpeg -ss 00:00:05 -i input.mp4 -frames:v 1 frame.jpg
```

### Thumbnail sheet (contact sheet)

```bash
ffmpeg -i input.mp4 -vf "fps=1/10,scale=320:-1,tile=4x3" preview.jpg
```

### Images → video (slideshow)

```bash
ffmpeg -framerate 1/5 -i img%03d.jpg -c:v libx264 -pix_fmt yuv420p -r 30 slideshow.mp4
```

### Video → GIF (palette for quality)

```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i input.mp4 -i palette.png -lavfi "fps=10,scale=480:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif
```

### MP4 → animated WebP

```bash
ffmpeg -i input.mp4 -vcodec libwebp -lossless 0 -q:v 75 -loop 0 output.webp
```

---

## 📡 Streaming

### HLS (adaptive-friendly segments)

```bash
ffmpeg -i input.mp4 \
  -c:v libx264 -preset veryfast -crf 22 \
  -c:a aac -b:a 128k \
  -hls_time 6 -hls_playlist_type vod \
  -hls_segment_filename "seg_%03d.ts" \
  playlist.m3u8
```

For multi-bitrate HLS, generate several renditions (1080p, 720p, 480p) and master playlist—often done with a script or packaging service.

### RTMP (live to YouTube, Twitch, etc.)

```bash
ffmpeg -re -i input.mp4 -c:v libx264 -preset veryfast -maxrate 3000k -bufsize 6000k \
  -pix_fmt yuv420p -g 50 -c:a aac -b:a 160k -ar 44100 -f flv "rtmp://live.example.com/app/stream_key"
```

`-re` reads input at native frame rate (real-time).

### Pipe (no intermediate file)

```bash
curl -s url | ffmpeg -i pipe:0 -c copy -f mp4 -movflags frag_keyframe+empty_moov pipe:1 > out.mp4
```

---

## ⚡ Hardware Acceleration

Hardware encoders are **much faster** but offer less fine control than `libx264`.

### Apple VideoToolbox (macOS)

```bash
ffmpeg -i input.mp4 -c:v h264_videotoolbox -b:v 5000k -c:a copy output.mp4
```

### NVIDIA NVENC (Linux/Windows)

```bash
ffmpeg -i input.mp4 -c:v h264_nvenc -preset p4 -cq 23 -c:a copy output.mp4
```

### Intel QSV

```bash
ffmpeg -i input.mp4 -c:v h264_qsv -global_quality 23 -c:a copy output.mp4
```

### VAAPI (Linux)

Often requires uploading frames to GPU memory—see [FFmpeg VAAPI wiki](https://trac.ffmpeg.org/wiki/Hardware/VAAPI).

Check available hardware encoders:

```bash
ffmpeg -encoders | grep -E 'nvenc|qsv|videotoolbox|vaapi|amf'
```

> **Note:** If hardware encode fails, fall back to `libx264` for portability.

---

## 🤖 Scripting & Automation

### Batch convert all MP4 in a folder (bash)

```bash
for f in *.mp4; do
  ffmpeg -i "$f" -c:v libx264 -crf 23 -c:a aac -b:a 128k "converted_${f}"
done
```

### Use `-nostdin` in CI

Prevents FFmpeg from waiting for interactive input when stdin is closed:

```bash
ffmpeg -nostdin -i input.mp4 -c copy output.mp4
```

### Progress to a file

```bash
ffmpeg -progress progress.txt -nostats -i input.mp4 -c copy output.mp4
```

### Shell function: safe web export

```bash
web_export() {
  in="$1"
  out="${2:-web_$(basename "$in")}"
  ffmpeg -nostdin -y -i "$in" \
    -c:v libx264 -preset medium -crf 23 -pix_fmt yuv420p \
    -movflags +faststart \
    -c:a aac -b:a 128k \
    "$out"
}
```

`-movflags +faststart` moves MP4 metadata to the beginning for faster web playback start.

### Node.js (fluent-ffmpeg)

```javascript
const ffmpeg = require("fluent-ffmpeg");

ffmpeg("input.mp4")
  .outputOptions(["-c:v libx264", "-crf 23", "-preset medium", "-c:a aac", "-b:a 128k"])
  .save("output.mp4")
  .on("end", () => console.log("done"))
  .on("error", (err) => console.error(err));
```

---

## 🚀 Performance Tips

1. **Use `-c copy`** whenever you only need trim/concat and keyframes align.
2. **Avoid unnecessary upscaling** — scaling 720p → 4K wastes time and doesn't add detail.
3. **Match preset to job**: `veryfast` for previews, `slow` for archival compression.
4. **Parallelize batch jobs** with `xargs -P` or GNU `parallel`, not one giant FFmpeg command.
5. **Pin thread count** on servers: `-threads 4` (tune to CPU cores).
6. **Use hardware encode** for real-time or high-volume transcoding when quality targets allow.

---

## 🐛 Troubleshooting

### `Unknown encoder 'libx264'`

Your FFmpeg build lacks GPL codecs. Install a full build (Homebrew `ffmpeg`, not `ffmpeg-minimal`).

### `Conversion failed` / `Invalid argument`

Run with `-loglevel debug` once. Common causes: wrong pixel format, incompatible audio for MP4, odd dimensions (H.264 needs even width/height).

### Audio out of sync after cut

Re-encode instead of copy, or place `-ss` after `-i` for accuracy:

```bash
ffmpeg -i input.mp4 -ss 90 -t 45 -c:v libx264 -crf 23 -c:a aac clip.mp4
```

### `moov atom not found`

Incomplete download or corrupted file. Re-download or repair with untrunc tools if you have a reference file.

### `Permission denied` / `No such file`

Quote paths with spaces: `-i "My Video.mp4"`.

### Huge output file

CRF too low or `-preset placebo` with unnecessary resolution. Raise CRF (e.g. 23 → 28) or lower resolution. See [Compressing Video File Size](#-compressing-video-file-size).

### Compressed file still too large

1. Scale down: `scale=1280:-2` or `scale=854:-2`
2. Raise CRF by 2 (e.g. 26 → 28)
3. Try H.265 (`libx265 -crf 28`)
4. Lower audio: `-b:a 96k -ac 1`
5. For a hard cap, use two-pass with calculated bitrate

### Subtitle filter path errors on Windows

Use:

```bash
-vf "subtitles='C\:/path/subs.srt'"
```

### FFmpeg hangs in CI

Add `-nostdin`. Ensure the process isn't waiting for `overwrite? [y/N]`.

### Check encoding speed (benchmark)

```bash
time ffmpeg -i input.mp4 -c:v libx264 -preset fast -crf 23 -c:a copy -f null -
```

---

## 📎 Quick Reference

### Social / platform starting points (starting points—not platform guarantees)

| Target | Video | Audio | Extra |
| ------ | ----- | ----- | ----- |
| Web embed | H.264, yuv420p | AAC 128k | `-movflags +faststart` |
| Instagram feed | H.264, max ~1080 width | AAC | Keep under duration/size limits |
| YouTube upload | H.264 or H.265 high bitrate | AAC/LPCM | Prefer high resolution source |
| WhatsApp | H.264, moderate bitrate | AAC | Short clips, size caps |

### Handy one-liners

```bash
# Strip everything except video
ffmpeg -i in.mp4 -map 0:v:0 -c copy video_only.mp4

# Force keyframe every 2 sec (gOP=60 at 30fps)
ffmpeg -i in.mp4 -c:v libx264 -g 60 -keyint_min 60 -sc_threshold 0 -c:a copy out.mp4

# Show frame count
ffprobe -v error -select_streams v:0 -count_packets -show_entries stream=nb_read_packets -of csv=p=0 in.mp4

# Convert to mono voice-friendly
ffmpeg -i in.mp4 -ac 1 -c:a aac -b:a 96k -c:v copy voice.mp4
```

### Official resources

- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
- [FFmpeg Wiki](https://trac.ffmpeg.org/wiki)
- [FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html)

---

## ✅ Next Steps

- Build a **batch transcoding** script for your asset pipeline
- Add **HLS packaging** for video-on-demand
- Profile **hardware encoders** on your deployment machines
- Pair with **ImageMagick** or **ExifTool** for thumbnail and metadata workflows

---

_Last updated: May 2026_
