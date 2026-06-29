# FFmpeg — Landscape PIP Composite (16:9 VSL)

All outputs: 1920×1080. Input: `$BASE` (avatar on background, 1920×1080), `$BROLL` (overlay clip).

## Audio-Per-Segment Architecture (MANDATORY)

**NEVER separate audio from video in composites.** Each segment must carry its own synced audio.

- AVATAR segments: cut from `avatar-on-bg.mp4` WITH audio, output-level seeking
- PIP/FULLSCREEN segments: audio from `$BASE` at matching timecode (`-map "1:a"`)
- Pre-render avatar-on-bg.mp4 with `-g 25` (keyframe every 1s) for seek accuracy

```bash
# Pre-render avatar-on-bg with keyframes for accurate seeking
ffmpeg -i "$BG_IMG" -i "$AVATAR_GREEN" \
  -filter_complex "
    [0:v]scale=1920:1080[bg];
    [1:v]colorkey=0x00FF00:0.25:0.05,colorkey=0x00FF00:0.40:0.01[keyed];
    [bg][keyed]overlay=0:0[composited]
  " \
  -map "[composited]" -map "1:a" \
  -c:v libx264 -pix_fmt yuv420p -g 25 -c:a aac -y "$AVATAR_ON_BG"
```

---

## Standard PIP Overlay (avatar bottom-right, b-roll fullscreen)

PIP settings: 384×330px at x=1512, y=726 (bottom-right, ~20% width).

```bash
START=60; END=90  # segment window

ffmpeg \
  -ss $START -i "$AVATAR_ON_BG" \
  -ss $START -i "$AVATAR_ON_BG" \
  -ss 0 -i "$BROLL" \
  -filter_complex "
    [0:v]trim=duration=$(($END-$START)),setpts=PTS-STARTPTS[base];
    [2:v]scale=1920:1080,trim=duration=$(($END-$START)),setpts=PTS-STARTPTS[broll_scaled];
    [1:v]colorkey=0x00FF00:0.25:0.05,colorkey=0x00FF00:0.40:0.01,crop=1258:1080:314:0,scale=384:330,trim=duration=$(($END-$START)),setpts=PTS-STARTPTS[pip];
    [base][broll_scaled]overlay=0:0[with_broll];
    [with_broll][pip]overlay=1512:726
  " \
  -map "[out]" -map "1:a" \
  -t $(($END-$START)) -c:v libx264 -pix_fmt yuv420p -c:a aac -y "seg-fullscreen-pip.mp4"
```

---

## Multi-Segment VSL Assembly

### Segment Types

**AVATAR segment** (talking head only):
```bash
START=0; END=30
ffmpeg -ss $START -i "$AVATAR_ON_BG" -t $(($END-$START)) \
  -c:v libx264 -pix_fmt yuv420p -c:a aac -y "seg-01-avatar.mp4"
```

**FULLSCREEN segment** (b-roll fills frame, avatar PIP in corner):
```bash
START=30; END=90; BROLL="$PROD/interim/broll/segments/wbst01.mp4"
ffmpeg \
  -ss $START -i "$AVATAR_ON_BG" \
  -ss $START -i "$AVATAR_ON_BG" \
  -i "$BROLL" \
  -filter_complex "
    [0:v]trim=duration=60,setpts=PTS-STARTPTS[base];
    [2:v]scale=1920:1080,loop=-1:1,trim=duration=60,setpts=PTS-STARTPTS[broll];
    [1:v]colorkey=0x00FF00:0.25:0.05,colorkey=0x00FF00:0.40:0.01,crop=1258:1080:314:0,scale=384:330,trim=duration=60,setpts=PTS-STARTPTS[pip];
    [base][broll]overlay=0:0[with_broll];
    [with_broll][pip]overlay=1512:726[out]
  " \
  -map "[out]" -map "1:a" -t 60 \
  -c:v libx264 -pix_fmt yuv420p -c:a aac -y "seg-02-broll.mp4"
```

**Concatenate all segments:**
```bash
# Build concat list
for seg in seg-*.mp4; do echo "file '$seg'"; done > concat-list.txt
ffmpeg -f concat -safe 0 -i concat-list.txt -c copy -y "$FINAL"
```

---

## Gap-Free B-Roll Windows

Extend each segment's enable window to the START of the next segment:
- Eliminates 0.5–1.5s avatar flashes between b-roll segments
- Keep intentional AVATAR transitions (> 1.5s with scripted dialogue)

```bash
# enable=between(t,START,NEXT_START) — not between(t,START,END)
overlay=x=0:y=0:enable='between(t,30,90)'  # extends to next segment start
```

---

## Avatar Crop / Zoom (CRITICAL)

**NEVER crop+stretch** — distorts avatar proportions.
**ALWAYS zoom-in then crop:**

```bash
# 1.15x zoom (removes chair arms, 162px from bottom)
scale=2208:1242,crop=1920:1080:144:0
```

For PIP insert, run `pip-crop-detect` script to get accurate body-only crop values per render.

---

## Seeking Rules

- **Output-level seeking** (`-i file -ss $START`): accurate to timecode, slower decode
- **Input-level seeking** (`-ss $START -i file`): snaps to nearest keyframe — avoid for 5+ min videos
- Use `-g 25` on pre-renders so keyframes are every 1s

For 5+ minute base videos: always use output-level seeking.
