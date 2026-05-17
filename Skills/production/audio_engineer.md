# Audio Engineer — AI Skill v1.0

**Role:** Guides the human through selecting the best SUNO take, renames and organizes audio takes, and prepares the master audio file for all downstream production workers.
**Phase:** Production
**Inputs:** SUNO-generated audio files in `Sound/`, `Lyrics/SUNO_BRIEF_[project_id].md`, `PROJECT_STATE.md`
**Outputs:** `Sound/master.mp3` (or .wav) — the selected master audio file
**Tools:** `ffprobe`, `ffmpeg` (optional: normalize to consistent loudness for preview)
**Human Gate:** YES — human listens to all takes and selects one (or requests new SUNO generations)
**Reads from PROJECT_STATE:** `asset_registry.audio.suno_takes`, `decisions.suno_style_prompt`
**Writes to PROJECT_STATE:** `decisions.suno_selected_take`, `asset_registry.audio.master`, `current_stage → image_generation`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Purpose

SUNO generates multiple takes. The human's ear is the final judge. This skill structures the listening session, provides objective quality checks, and ensures the best take becomes the master file — clearly named and placed — before any video or image generation begins.

---

## Step 1 — Inventory the Takes

Scan `Sound/` for all audio files. List them:

```bash
# Get duration and bitrate of all takes
ffprobe -v quiet -print_format json -show_format "Sound/[file]"
```

Print inventory:
```
🎵 AUDIO TAKES FOUND IN Sound/

  take_01_a.mp3  —  [X min Y sec]  —  [kbps]
  take_01_b.mp3  —  [X min Y sec]  —  [kbps]
  take_02_a.mp3  —  [X min Y sec]  —  [kbps]
  ...

Total takes: [N]
```

If no files found: block with message: "No audio takes found in Sound/ folder. Please generate takes in SUNO per the SUNO_BRIEF file and place MP3 files in the Sound/ folder."

---

## Step 2 — Objective Quality Scan

For each take, run an automated quality check:

```bash
# Check for clipping (peak level)
ffmpeg -i "Sound/[file]" -af "volumedetect" -f null - 2>&1 | grep -E "max_volume|mean_volume"

# Check duration matches expected
ffprobe -v quiet -show_entries format=duration -of csv="p=0" "Sound/[file]"
```

Flag issues automatically:
```
⚠️  take_01_a: max_volume = 0.0 dBFS → possible clipping
⚠️  take_02_b: duration = 1:42 → shorter than expected [3:30] — possible SUNO truncation
✅  take_01_b: peak = -1.2 dBFS, duration = 3:28 — looks good technically
```

---

## Step 3 — Human Listening Session

Present the listening guide to the human:

```
🎧 TAKE SELECTION — Please listen to all takes and choose one.

Technically acceptable takes:
  ✅ take_01_a.mp3
  ✅ take_01_b.mp3
  ✅ take_02_a.mp3

Flagged (review carefully):
  ⚠️  take_02_b.mp3 — may be truncated

Listening criteria (in order of importance):

1. VOCAL CLARITY — Are the words audible throughout?
   Especially in the chorus — if the melody overrides the lyrics, reject.

2. EMOTIONAL ARC — Does the chorus feel bigger/different from the verse?
   A flat production where verse and chorus feel the same energy = reject.

3. LANGUAGE ACCURACY — Did SUNO correctly generate the full [language] lyrics?
   Any wrong language, mumbled words, or skipped sections = reject.

4. ENDING — Does the song end properly, or does it cut off abruptly?
   Abrupt ending with music still going = reject (consider re-generating this take).

5. STYLE MATCH — Does the instrumentation match the SUNO style prompt?
   Check: [key instrument from prompt] — is it audible?

After listening, reply with:
  SELECTED: [filename]
  OR: NONE — all rejected, I will regenerate in SUNO
  OR: REGENERATE [reason] — I want new takes with this change: [change]
```

---

## Step 4 — Process Selected Take

On human selection:

1. Copy selected file to `Sound/master.mp3` (keep original in place):
```bash
copy "Sound/[selected]" "Sound/master.mp3"
```

2. Optional — normalize for consistent preview loudness (does NOT affect final master):
```bash
ffmpeg -i "Sound/master.mp3" -af "loudnorm=I=-16:TP=-1.5:LRA=11" "Sound/master_preview.mp3"
```

3. Run final ffprobe report on master:
```bash
ffprobe -v quiet -print_format json -show_format -show_streams "Sound/master.mp3"
```

Report to human:
```
✅ MASTER AUDIO SET:  Sound/master.mp3
   Duration:   [X min Y sec] ([T] seconds)
   Bitrate:    [X kbps]
   Format:     [mp3/wav]
   Selected from: [original filename]

📌 All future workers will use this file as the audio source.
```

---

## Step 5 — Update PROJECT_STATE

```yaml
decisions:
  suno_selected_take: "[original filename]"
asset_registry:
  audio:
    master: "Sound/master.mp3"
    suno_takes: ["Sound/take_01_a.mp3", "Sound/take_01_b.mp3", ...]  # list all
current_stage: "image_generation"
next_action: "Capex gate → then run image_generator skill."
```

---

## If Human Requests Regeneration

Guide the human on SUNO prompt adjustments based on their feedback:

| Complaint | SUNO Prompt Adjustment |
|---|---|
| Vocals too quiet | Add: "prominent lead vocals, vocals upfront in mix" |
| Chorus not distinct enough | Add: "dynamic chorus with orchestral lift, full arrangement in chorus" |
| Wrong language / mumbled | Add language explicitly: "English lyrics only" or "[language] lyrics" — try different SUNO model |
| Too slow / too fast | Adjust BPM explicitly: "exact [N]bpm" |
| Wrong instrument | Remove from prompt and replace with more specific term |
| Song too short | Add: "full song, minimum 3 minutes, complete arrangement with outro" |

Update the SUNO_BRIEF file with the new variation and ask human to regenerate.
