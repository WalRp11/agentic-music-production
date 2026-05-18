# Video Generator — AI Skill v1.1

**Role:** Converts each source image into a beat-synced 10-second video clip using Higgsfield Seedance 2.0, with audio-driven prompt construction guided by the seedance_music_video skill.
**Phase:** Production
**Inputs:** `Images/c*.png` (all generated source images), `Lyrics/music-video-script.md` (video motion specs), `Assembly/cut_list.json`, `Sound/master.mp3` (audio reference for beat sync), `Skills/production/seedance_music_video.md` (prompt construction guide)
**Outputs:** `Clips/c[01-N]_[scene_name].mp4` — one raw clip per scene
**Tools:** Higgsfield CLI (`higgsfield generate video`)
**Human Gate:** NO — batch run (Capex gate already approved)
**Reads from PROJECT_STATE:** `asset_registry.images`, `decisions.capex_approved`
**Writes to PROJECT_STATE:** `asset_registry.clips`, `current_stage → video_edit`
**Version History:**
  v1.0 — Initial release (2026-05-17)
  v1.1 — Added @audio1 beat-sync, seedance_music_video prompt guide reference (2026-05-18)

---

## Pre-Flight Checks

```
1. Verify capex_approved = true
2. Verify all images in asset_registry.images exist on disk
3. Count: images found must equal entries in cut_list.json
4. Verify Higgsfield CLI authenticated: `higgsfield account status`
```

If image count doesn't match cut list: stop and report discrepancy. Do not generate partial.

---

## Camera Motion Vocabulary for Higgsfield

Use these exact motion descriptions in video prompts:

| Desired Effect | Higgsfield Prompt Motion Spec |
|---|---|
| Fully static | `"static shot, no camera movement"` |
| Slow push in | `"very slow zoom in, barely perceptible, 10 seconds"` |
| Slow pull back | `"very slow pull back, slow reveal, 10 seconds"` |
| Slow pan left | `"very slow pan left, steady, 10 seconds"` |
| Slow pan right | `"very slow pan right, steady, 10 seconds"` |
| Slow tilt up | `"very slow upward tilt, 10 seconds"` |
| Slow tilt down | `"very slow downward tilt, 10 seconds"` |
| Subtle float | `"very subtle floating movement, almost imperceptible, dreamy"` |

**Rules for motion selection:**
- Default to static or very slow — faster motion looks bad when cut together
- Use push-in for emotional reveals (character faces, key objects)
- Use pull-back for establishing shots and endings
- Use pan only for horizontal environments (cityscapes, rooms)
- Never use zoom + pan simultaneously — pick one
- Vary motion types: no more than 3 consecutive identical motions

---

## Step 1 — Build Generation Queue

### 1a — Read the beat-sync guide

Before writing any prompts, read `Skills/production/seedance_music_video.md` and derive from the project files:
- **Genre**: read `PROJECT_STATE.md` → `mood` field; match to the closest Genre Visual Language section in the guide
- **Narrative pattern**: read `Lyrics/music-video-script.md` → identify which Narrative Pattern (1–5) fits the story structure
- **Beat-sync hooks**: read `Lyrics/timestamps_aligned.json` → map each section (verse/chorus/bridge/outro) to its start time; apply the relevant 2-Second Hook types from the guide to chorus and peak-intensity scenes
- **Audio file**: identify the master audio file from `Sound/` folder → reference as `@audio1` in the `--audio` flag

### 1b — Construct prompts per clip

For each clip, build the prompt in three layers:
```
[Image scene description from script]
+ ", " + [motion spec from Camera Motion Vocabulary]
+ ", " + [beat-sync descriptor derived from timestamps_aligned.json and seedance_music_video.md]
```

**How to derive beat-sync descriptors:**
1. Read `Lyrics/timestamps_aligned.json` — note the `start` time of each section
2. For each clip in `cut_list.json`, check which section it belongs to (from `storyboard_arrangement.md`)
3. Apply intensity rules from `seedance_music_video.md`:
   - Instrumental/ambient sections → `minimal energy, ambient stillness`
   - Verse sections → `verse energy, [genre-appropriate] intimate motion, synced to [primary instrument] rhythm`
   - Chorus sections → `chorus peak at [timestamp], [hook type from guide], maximum emotional intensity`
   - Interlude/bridge → `breath moment, reduced energy, [contrasting motion from chorus]`
   - Outro → `energy fading, slow pull-back, resolve to stillness`
4. Add the project's color grade style as a suffix (read from `Assembly/storyboard_arrangement.md` → `Color grade target`)

**Full prompt construction template:**
```
"[scene description from script],
[motion spec],
@audio1, [section label] at [timestamp from timestamps_aligned.json], [intensity descriptor],
[color grade style from storyboard_arrangement.md]"
```

### 1c — CLI command template

For each entry in `cut_list.json`, create a generation command:

```bash
higgsfield generate video \
  --image "Images/c[NN]_[scene_name].png" \
  --audio "Sound/[master_audio_file]" \
  --prompt "[full prompt from Step 1b]" \
  --duration 10 \
  --output "Clips/c[NN]_[scene_name].mp4" \
  --model seedance_2
```

**Model selection:**
- `seedance_2` — default for all standard clips (best quality for cinematic motion)
- `kling_3` — alternative if seedance_2 fails or for more dramatic motion

---

## Step 2 — Generate Clips (Batch)

Process queue sequentially (not parallel — Higgsfield rate limits):

```
FOR each clip in cut_list.json:
  1. Run generation command
  2. Wait for completion
  3. Verify output file exists and is > 500KB (very small = generation failed)
  4. Log: "✅ c01 generated — [filesize]" OR "❌ c01 failed — retrying"
  5. If failed: retry once with different --seed
  6. If retry fails: flag for manual review, continue to next clip
  7. Log credit spend to budget_manager
```

Print progress:
```
🎬 VIDEO GENERATION PROGRESS

[██████░░░░] 12/22 clips  |  ✅ 11  ⚠️ 1  ❌ 0
Estimated remaining: ~[N] minutes
Spend so far: $[X]
```

---

## Step 3 — Quality Verification

After all clips are generated:

```bash
# Verify duration of each clip
ffprobe -v quiet -show_entries format=duration -of csv="p=0" "Clips/c01_[scene_name].mp4"
```

Flag issues:
- Duration < 9.5 seconds → possible truncation, regenerate
- Duration > 10.5 seconds → trim in video_editor (note in cut_list)
- File size < 500KB → likely failed generation, regenerate
- Any clip where the motion is obviously wrong (static when push-in was specified)

---

## Step 4 — Clip Register

After all clips pass verification, output a clip register to PROJECT_STATE:

```yaml
asset_registry:
  clips:
    - id: "c01"
      file: "Clips/c01_empty_alley.mp4"
      duration: 10.0
      status: "ready"
    - id: "c02"
      file: "Clips/c02_coffee_detail.mp4"
      duration: 10.0
      status: "ready"
    - id: "c04"
      file: "Clips/c04_window_rain.mp4"
      duration: 10.0
      status: "flagged — motion incorrect, regenerated with seed 42"
current_stage: "video_edit"
next_action: "Run video_editor skill."
spend_log:
  - worker: "video_generator"
    units: [N clips]
    cost_usd_estimate: [N × 0.12]
```

---

## Common Generation Failures and Fixes

| Failure | Cause | Fix |
|---|---|---|
| Clip is static despite motion prompt | Higgsfield ignored motion | Retry with more explicit motion spec + `--seed [random]` |
| Character moves unnaturally | Soul ID + motion conflict | Use static shot for this character scene |
| Background warps/melts | Complex texture in source image | Simplify source image prompt, reduce detail in background |
| Clip ends abruptly | Generation timeout | Retry; if persistent, use 8s version and extend with FFmpeg freeze frame |
| Color dramatically different from source image | Model color shift | Add color lock to prompt: "maintain exact color palette of source image" |
| Text appears in background | Model hallucination | Add to prompt: "no text, no subtitles, no watermarks" — also check source image |
