# Video Editor — AI Skill v1.0

**Role:** Assembles all generated clips with the master audio track into the final music video using FFmpeg, following the cut list precisely.
**Phase:** Post-Production
**Inputs:** `Clips/c*.mp4`, `Sound/master.mp3`, `Assembly/cut_list.json`
**Outputs:** `Assembly/Music_Video_raw.mp4` — the assembled video (before color grade and sound design)
**Tools:** `ffmpeg`, `ffprobe`
**Human Gate:** NO — automated assembly
**Reads from PROJECT_STATE:** `asset_registry.clips`, `asset_registry.audio.master`, `asset_registry.assembly.cut_list_json`
**Writes to PROJECT_STATE:** `asset_registry.assembly.final_video`, `current_stage → color_grade`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Pre-Flight Checks

```bash
# Verify all clips exist
foreach clip in cut_list.json:
  if not exists: STOP — "Missing clip: [path]. Run video_generator first or regenerate missing clip."

# Verify master audio exists
if not exists "Sound/master.mp3": STOP — "Master audio missing. Run audio_engineer first."

# Verify clip durations sum approximately matches audio duration
total_clip_duration = sum of all clip durations
audio_duration = ffprobe duration of master.mp3
if abs(total_clip_duration - audio_duration) > 5:
  WARN: "Clip total ([X]s) differs from audio ([Y]s) by [Z]s. Assembly will have silence or cutoff. Verify cut_list.json."
```

---

## Step 1 — Trim Clips to Exact Durations

For each clip, apply the `trim_start` and `trim_end` from `cut_list.json`:

```bash
ffmpeg -i "Clips/c01_[scene_name].mp4" \
  -ss [trim_start] -t [trim_end - trim_start] \
  -c:v libx264 -preset slow -crf 18 -an \
  "Cuts/c01_[scene_name].mp4" -y
```

**Note:** `-an` removes audio — the master audio track is added at assembly time.

Name convention for cuts: same as clips but in `Cuts/` folder.

---

## Step 2 — Build Concat List

Write `Assembly/concat.txt`:

```
file '../Cuts/c01_empty_alley.mp4'
file '../Cuts/c02_coffee_detail.mp4'
file '../Cuts/c03_window_exterior.mp4'
...
```

One line per clip, in the order from `cut_list.json`. Paths are relative to the `Assembly/` folder.

```bash
# Generate concat.txt automatically from cut_list.json
# (Python or jq one-liner — example in Python):
python3 -c "
import json
with open('Assembly/cut_list.json') as f:
    cl = json.load(f)
with open('Assembly/concat.txt', 'w') as out:
    for c in cl['cuts']:
        out.write(f\"file '../Cuts/{c['id']}_{c['scene_name']}.mp4'\n\")
"
```

---

## Step 3 — Assemble Video with Audio

```bash
# Step A: Concatenate all clips into a silent video
ffmpeg -f concat -safe 0 \
  -i "Assembly/concat.txt" \
  -c:v libx264 -preset slow -crf 16 \
  "Assembly/video_silent.mp4" -y

# Step B: Verify silent video duration vs audio duration
ffprobe -v quiet -show_entries format=duration -of csv="p=0" "Assembly/video_silent.mp4"
ffprobe -v quiet -show_entries format=duration -of csv="p=0" "Sound/master.mp3"

# Step C: Merge audio and video
ffmpeg \
  -i "Assembly/video_silent.mp4" \
  -i "Sound/master.mp3" \
  -c:v copy \
  -c:a aac -b:a 320k \
  -shortest \
  "Assembly/Music_Video_raw.mp4" -y
```

**-shortest flag:** If video is slightly longer than audio (common), it trims to audio length. If audio is longer (rare), it trims to video. In either case, flag the discrepancy for human review if > 2 seconds.

---

## Step 4 — Verification

```bash
ffprobe -v quiet -print_format json -show_format -show_streams "Assembly/Music_Video_raw.mp4"
```

Verify:
- Has both video and audio streams
- Duration matches master audio duration (within ±1 second)
- Resolution: 1920×1080
- Video codec: h264
- Audio codec: aac

```
✅ ASSEMBLY COMPLETE

Output: Assembly/Music_Video_raw.mp4
Duration: [X min Y sec]
Resolution: 1920×1080
Audio: [Y kbps AAC]
File size: [Z MB]

Next: color_grade skill will process this file.
```

---

## Step 5 — Update PROJECT_STATE

```yaml
asset_registry:
  assembly:
    concat_txt: "Assembly/concat.txt"
    final_video: "Assembly/Music_Video_raw.mp4"
  cuts: ["Cuts/c01_empty_alley.mp4", "Cuts/c02_coffee_detail.mp4", ...]
current_stage: "color_grade"
next_action: "Run color_grader skill."
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Concat fails: "invalid data" | Clip codec mismatch | Re-encode all clips to same codec first: `-c:v libx264 -pix_fmt yuv420p` |
| Audio desync | Variable frame rate clips | Add `-vsync vfr` to concat step |
| Black frames at cut points | Clip transition frame | Trim 0.1s from start of each cut: `trim_start += 0.1` |
| Video shorter than audio | Not enough clips | Add more scenes to script; re-run video_generator for new scenes |
| Final file very large (> 2GB) | CRF too low | Re-run step 3 with `-crf 20` |
| Audio clipping | AAC encode issue | Re-run step 3B with `-c:a aac -b:a 256k -af loudnorm=I=-16` |
