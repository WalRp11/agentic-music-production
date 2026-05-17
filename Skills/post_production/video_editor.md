# Video Editor — AI Skill v2.0

**Role:** Assembles all generated clips with the master audio track into the final music video, with creative-mode-aware timeline building and automated marker placement.
**Phase:** Post-Production
**Inputs:** `Clips/c*.mp4`, `Sound/master.mp3`, `Assembly/cut_list.json`, `Assembly/storyboard_arrangement.md` (from Creative Designer)
**Outputs:** `Assembly/Music_Video_raw.mp4` (ffmpeg fast path) OR a DaVinci Resolve `.drp` project file ready for color grade and final export
**Tools:** **DaVinci Resolve Free** (primary — Python scripting API), `ffmpeg` + `ffprobe` (fallback / pre-processing)
**Human Gate:** YES for DaVinci Resolve path — human reviews timeline and markers before export. NO for ffmpeg fast path.
**Reads from PROJECT_STATE:** `asset_registry.clips`, `asset_registry.audio.master`, `asset_registry.assembly.cut_list_json`
**Writes to PROJECT_STATE:** `asset_registry.assembly.final_video`, `current_stage → color_grade`
**Version History:**
  v1.0 — Initial release: ffmpeg-only concatenation (2026-05-17)
  v2.0 — DaVinci Resolve Python API as primary editor; ffmpeg retained as fast-path fallback; creative mode markers; beat-sync workflow (2026-05-17)

---

## Tool Decision: DaVinci Resolve vs. ffmpeg

| Use DaVinci Resolve (primary) | Use ffmpeg (fallback) |
|---|---|
| Creative edit with transitions, markers, color grade handoff | Quick raw assembly for testing or headless CI |
| Human will review timeline before export | Fully automated pipeline with no human review |
| Cut list has `mode` or `technique` fields from Creative Designer | Cut list is a simple ordered list with no creative metadata |
| Color grader will use the same Resolve project | Output is only needed as a temp file |

**DaVinci Resolve Free** is the standard tool for this company. It is free (no license fee), supports Python scripting via `DaVinciResolveScript`, handles color grading natively (passing the project directly to the color_grader skill), and is the industry standard. Install from [blackmagicdesign.com/products/davinciresolve](https://www.blackmagicdesign.com/products/davinciresolve).

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

## Path A — DaVinci Resolve (Primary Creative Edit)

### Setup: Environment Variables (Windows)

Set these before running any Resolve Python script. Add to your PowerShell profile or run once per session:

```powershell
$env:RESOLVE_SCRIPT_API = "$env:PROGRAMDATA\Blackmagic Design\DaVinci Resolve\Support\Developer\Scripting"
$env:RESOLVE_SCRIPT_LIB = "C:\Program Files\Blackmagic Design\DaVinci Resolve\fusionscript.dll"
$env:PYTHONPATH     = "$env:PYTHONPATH;$env:RESOLVE_SCRIPT_API\Modules"
```

**DaVinci Resolve must be running** before the script is invoked. It does not need to have a project open — the script creates or loads one.

---

### Step DR-1 — Pre-trim All Clips with ffmpeg

Even when using Resolve as the primary editor, use ffmpeg to pre-trim each clip to its exact duration. This avoids Resolve re-interpreting in/out points incorrectly on variable-frame-rate clips from Higgsfield.

```bash
# For each clip in cut_list.json:
ffmpeg -i "Clips/c01_[scene_name].mp4" \
  -ss [trim_start] -t [duration] \
  -c:v libx264 -preset fast -crf 18 -an \
  "Cuts/c01_[scene_name].mp4" -y
```

---

### Step DR-2 — Build the Resolve Timeline via Python

This script reads `Assembly/cut_list.json` and builds a DaVinci Resolve timeline with:
- All clips in cut order
- Master audio on Audio Track 1
- Creative mode markers (colour-coded by mode and technique)

```python
#!/usr/bin/env python3
"""
build_resolve_timeline.py
Run AFTER: all Cuts/ clips exist, DaVinci Resolve is open
"""
import sys, json, os
import DaVinciResolveScript as dvr_script

CUT_LIST   = "Assembly/cut_list.json"
AUDIO_FILE = os.path.abspath("Sound/master.mp3")
CUTS_DIR   = os.path.abspath("Cuts")
PROJECT_NAME = "Music_Video_Edit"  # change per project

# Marker colours per creative mode (Resolve colour names)
MODE_COLORS = {
    "literal":      "Green",
    "metaphorical": "Blue",
    "counterpoint": "Red",
}
TECHNIQUE_COLORS = {
    "breath_shot":       "Purple",
    "visual_echo":       "Orange",
    "progressive_zoom":  "Cyan",
    "ghost_image":       "Pink",
    "empty_frame":       "Yellow",
}

def main():
    resolve = dvr_script.scriptapp("Resolve")
    pm      = resolve.GetProjectManager()

    # Create or load project
    project = pm.LoadProject(PROJECT_NAME)
    if not project:
        project = pm.CreateProject(PROJECT_NAME)
    if not project:
        print("ERROR: Could not create or load Resolve project."); sys.exit(1)

    # Set timeline settings (24fps, 1920x1080)
    project.SetSetting("timelineFrameRate", "24")
    project.SetSetting("timelineResolutionWidth",  "1920")
    project.SetSetting("timelineResolutionHeight", "1080")

    pool    = project.GetMediaPool()
    storage = resolve.GetMediaStorage()

    # Load cut_list
    with open(CUT_LIST, encoding="utf-8") as f:
        cl = json.load(f)
    clips_data = cl["clips"]

    # Import all pre-trimmed clips into Media Pool
    clip_paths = []
    for c in clips_data:
        clip_file = os.path.join(CUTS_DIR, f"c{c['clip_number']:02d}_{c['scene_description'].replace(' ', '_')}.mp4")
        clip_paths.append(clip_file)

    pool_items = storage.AddItemListToMediaPool(clip_paths)
    if not pool_items:
        print("ERROR: No clips imported. Check Cuts/ folder."); sys.exit(1)

    # Import audio
    audio_items = storage.AddItemListToMediaPool([AUDIO_FILE])

    # Create timeline from clips
    timeline = pool.CreateTimelineFromClips(f"{PROJECT_NAME}_v1", pool_items)
    if not timeline:
        print("ERROR: Could not create timeline."); sys.exit(1)
    project.SetCurrentTimeline(timeline)

    # Append audio to timeline (track 1 audio)
    # Note: AppendToTimeline adds to current timeline; audio handled via Fairlight or manually
    if audio_items:
        pool.AppendToTimeline(audio_items)

    # Add creative mode markers per clip
    fps = 24
    for c in clips_data:
        frame_in  = int(c["time_in"]  * fps)
        mode      = c.get("mode", "")
        technique = c.get("technique", "")
        note      = c.get("creative_justification", "")[:100]  # Resolve marker note limit

        color = MODE_COLORS.get(mode, "White")
        timeline.AddMarker(frame_in, color, f"[{mode.upper()}] c{c['clip_number']:02d}", note, 1)

        if technique and technique in TECHNIQUE_COLORS:
            timeline.AddMarker(frame_in + 1, TECHNIQUE_COLORS[technique],
                               f"[{technique.upper()}]", "", 1)

    print(f"\n✅ Timeline '{PROJECT_NAME}_v1' created in DaVinci Resolve.")
    print(f"   {len(clips_data)} clips, markers added.")
    print("   Green=Literal  Blue=Metaphorical  Red=Counterpoint")
    print("   Purple=BreathShot  Orange=VisualEcho  Cyan=ProgressiveZoom")
    print("   Next: audio sync in Fairlight → hand off to color_grader skill.")

if __name__ == "__main__":
    main()
```

---

### Step DR-3 — Audio Sync in DaVinci Resolve

After the script runs:
1. Switch to the **Cut** or **Edit** page in DaVinci Resolve
2. The video clips are on Video Track 1
3. Find `master.mp3` in the Media Pool → drag to Audio Track 1
4. Snap its start to frame 0
5. Lock Audio Track 1 (`right-click track header → Lock`)

---

### Step DR-4 — Human Review of Markers

The human editor reviews the timeline using the marker colour guide:

| Marker Colour | Meaning |
|---|---|
| Green | Literal mode — clip directly illustrates the lyric |
| Blue | Metaphorical mode — clip represents the lyric's feeling |
| Red | Counterpoint mode — clip deliberately contrasts with the lyric |
| Purple | Breath Shot — hold this clip longer than feels comfortable |
| Orange | Visual Echo — return to this image for emotional resonance |
| Cyan | Progressive Zoom — sequence of tightening shots |

The editor may adjust clip lengths, add transitions, or override creative choices. The markers are guides, not locks.

---

### Step DR-5 — Export (Deliver Page)

When the edit is approved:
1. Open **Deliver** page in DaVinci Resolve
2. Add render job: Format = MP4, Codec = H.264, Resolution = 1920×1080, Quality = Restrict to 40 Mbps
3. Audio: Include audio, AAC 320 kbps
4. Output: `Assembly/Music_Video_raw.mp4`
5. Start render → wait for completion
6. Save Resolve project: `File → Save Project`

**Save the `.drp` project file** — it is handed directly to the `color_grader` skill, which opens it in Resolve's Color page.

---

## Path B — ffmpeg Fast Path (Fallback)

Use this only when DaVinci Resolve is not installed or a quick raw assembly is needed for review.

### Step FF-1 — Trim Clips
```bash
ffmpeg -i "Clips/c01_[scene_name].mp4" \
  -ss [trim_start] -t [trim_end - trim_start] \
  -c:v libx264 -preset slow -crf 18 -an \
  "Cuts/c01_[scene_name].mp4" -y
```

**Note:** `-an` removes audio — the master audio track is added at assembly time.

Name convention for cuts: same as clips but in `Cuts/` folder.

---

### Step FF-2 — Build Concat List

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
    for c in cl['clips']:
        out.write(f\"file '../Cuts/c{c['clip_number']:02d}_{c['scene_description'].replace(' ', '_')}.mp4'\\n\")
"
```

---

### Step FF-3 — Assemble Video with Audio

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

**-shortest flag:** If video is slightly longer than audio (common), it trims to audio length.

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
