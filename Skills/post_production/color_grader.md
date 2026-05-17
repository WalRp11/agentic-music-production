# Color Grader — AI Skill v1.0

**Role:** Applies a cinematic color grade to the assembled music video using FFmpeg filters or DaVinci Resolve, based on the mood defined in the locations scout document.
**Phase:** Post-Production
**Inputs:** `Assembly/Music_Video_raw.mp4`, `Lyrics/LOCATIONS_[project_id].md` (Global Visual Style Sheet), `PROJECT_STATE.md`
**Outputs:** `Assembly/Music_Video_graded.mp4`
**Tools:** `ffmpeg` (primary), DaVinci Resolve (optional, human-operated)
**Human Gate:** NO — auto-applied based on style sheet
**Reads from PROJECT_STATE:** `decisions.color_grade_style`, `asset_registry.assembly.final_video`
**Writes to PROJECT_STATE:** `asset_registry.assembly.color_graded`, `current_stage → sound_design`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Color Grade Presets (FFmpeg)

Select the preset that best matches `decisions.color_grade_style`. Apply via FFmpeg `-vf` filter.

### Preset 1: Cinematic Teal-Orange (most common cinematic look)
```bash
-vf "curves=r='0/0 0.25/0.22 0.75/0.78 1/1':g='0/0 0.25/0.24 0.75/0.76 1/1':b='0/0 0.25/0.28 0.75/0.72 1/1',
colorbalance=rs=-0.1:gs=0.0:bs=0.1:rm=0.0:gm=0.0:bm=0.0:rh=0.15:gh=0.05:bh=-0.1,
eq=contrast=1.1:brightness=-0.03:saturation=0.85"
```

### Preset 2: Warm Vintage / Nostalgic
```bash
-vf "curves=r='0/0 0.3/0.32 0.7/0.74 1/1':g='0/0 0.3/0.28 0.7/0.72 1/1':b='0/0 0.3/0.22 0.7/0.65 1/1',
eq=contrast=1.05:brightness=0.02:saturation=0.75,
colorchannelmixer=.9:0.05:0.05:0:0.05:.9:0.05:0:0.1:0.1:.8:0"
```

### Preset 3: Dark / Moody (low light, de-saturated)
```bash
-vf "curves=r='0/0 0.2/0.16 0.8/0.82 1/0.96':g='0/0 0.2/0.18 0.8/0.80 1/0.95':b='0/0 0.2/0.22 0.8/0.82 1/0.98',
eq=contrast=1.2:brightness=-0.05:saturation=0.65,
vignette=PI/4"
```

### Preset 4: Clean / Bright Pop
```bash
-vf "curves=r='0/0 0.5/0.52 1/1':g='0/0 0.5/0.5 1/1':b='0/0 0.5/0.48 1/1',
eq=contrast=1.0:brightness=0.04:saturation=1.15"
```

### Preset 5: Cold / Desaturated Drama
```bash
-vf "curves=r='0/0 0.3/0.27 0.7/0.68 1/0.95':g='0/0 0.3/0.29 0.7/0.70 1/0.95':b='0/0 0.3/0.32 0.7/0.74 1/1',
eq=contrast=1.15:brightness=-0.02:saturation=0.55"
```

---

## Preset Selection Logic

```
Read decisions.color_grade_style from PROJECT_STATE.
Match to preset:

  contains "teal" OR "orange" OR "cinematic"    → Preset 1
  contains "warm" OR "vintage" OR "nostalgic"   → Preset 2
  contains "dark" OR "moody" OR "noir"          → Preset 3
  contains "pop" OR "bright" OR "clean"         → Preset 4
  contains "cold" OR "desaturated" OR "drama"   → Preset 5
  no match                                      → Preset 1 (default)
```

---

## Step 1 — Apply Color Grade

```bash
ffmpeg -i "Assembly/Music_Video_raw.mp4" \
  -vf "[selected preset filter string]" \
  -c:v libx264 -preset slow -crf 16 \
  -c:a copy \
  "Assembly/Music_Video_graded.mp4" -y
```

**Important:** `-c:a copy` preserves the audio track from the raw video without re-encoding.

---

## Step 2 — Letterbox / Cinematic Bars (optional)

If `decisions.color_grade_style` contains "cinematic" or "widescreen":

```bash
# Add 2.35:1 letterbox bars to 16:9 video
ffmpeg -i "Assembly/Music_Video_graded.mp4" \
  -vf "pad=iw:iw*9/21:(ow-iw)/2:(oh-ih)/2:black" \
  -c:v libx264 -preset slow -crf 16 -c:a copy \
  "Assembly/Music_Video_graded_wide.mp4" -y
```

Ask human: "The style calls for a cinematic look. Do you want 2.35:1 letterbox bars? (yes/no)"
If yes: use `_wide` version as the graded file. If no: keep standard 16:9.

---

## Step 3 — Verification

Compare a few frames visually (if human is available) or report automatically:

```
🎨 COLOR GRADE APPLIED

Preset used: [name]
Input: Assembly/Music_Video_raw.mp4
Output: Assembly/Music_Video_graded.mp4

Grade summary:
  Contrast: [+N%]
  Saturation: [N%]
  Color shift: [description from preset]
  Letterbox: [yes/no]
```

---

## DaVinci Resolve Path (Optional — Human-Operated)

If the human prefers manual grading in DaVinci Resolve:

1. Import `Assembly/Music_Video_raw.mp4` into DaVinci Resolve (Free or Studio)
2. Apply the grade based on the `LOCATIONS_[project_id].md` Global Visual Style Sheet
3. Export using the YouTube master preset (H.264 4K or 1080p, AAC 320kbps)
4. Save the export as `Assembly/Music_Video_graded.mp4`
5. Manually update PROJECT_STATE.md: set `asset_registry.assembly.color_graded` path

**DaVinci Resolve recommended node structure:**
```
[Input] → [Lift/Gamma/Gain (primary correction)] → [Hue vs Saturation] → [Output Shaper]
```

---

## Step 4 — Update PROJECT_STATE

```yaml
asset_registry:
  assembly:
    color_graded: "Assembly/Music_Video_graded.mp4"
current_stage: "sound_design"
next_action: "Run sound_designer skill."
```
