# Scriptwriter — AI Skill v1.0

**Role:** Produces the frame-accurate music video script — a scene-by-scene breakdown with timestamps, shot descriptions, and a machine-readable cut list — that drives every downstream production worker.
**Phase:** Pre-Production
**Inputs:** `Lyrics/LYRICS_[project_id].md`, `Lyrics/CONCEPT_[project_id].md`, `Lyrics/CASTING_[project_id].md`, `Lyrics/LOCATIONS_[project_id].md`, audio timestamps (if available from intake whisper), `PROJECT_STATE.md`
**Outputs:** `Lyrics/music-video-script.md`, `Assembly/cut_list.json`
**Tools:** `ffprobe` (for audio duration verification), text analysis
**Human Gate:** YES — human must approve scene script before image/video generation begins
**Reads from PROJECT_STATE:** `asset_registry.audio.master`, `asset_registry.audio.timestamps_json`, `decisions.soul_id_required`
**Writes to PROJECT_STATE:** `asset_registry.script`, `asset_registry.assembly.cut_list_json`, `current_stage → audio_production`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Critical Rules (Hard Failures Prevented)

These rules exist because of real production failures. Violating them guarantees expensive rework.

### RULE S1 — Clip Count Math (Non-Negotiable)
**Before writing a single scene, calculate the exact number of unique clips needed:**
```
unique_clips_needed = ceil(total_song_duration / clip_length_seconds)
```
Where `clip_length_seconds` = 10 (Higgsfield standard).

Example: song is 214 seconds → `ceil(214 / 10) = 22 unique clips minimum`.

**Never write a script with fewer unique clips than this number.** Repeating the same clip back-to-back ruins the video.

### RULE S2 — No Consecutive Repeat
No clip may appear consecutively with itself. If a section needs more visual coverage than unique clips provide, write additional scene variants of the same location with different framing or subject emphasis.

### RULE S3 — Scene Description = Prompt Ready
Every scene description in the script is copy-paste ready for Higgsfield. Do not write vague scene descriptions — they will produce vague results.

### RULE S4 — Timestamp Accuracy
Every scene is mapped to exact seconds (start:end). These timestamps come from whisper output if available, or from rough section timing calculated from lyrics structure and BPM. Flag any timestamp as `[ESTIMATED]` if not from audio analysis.

### RULE S5 — One Visual Idea Per Scene
Each 10-second clip shows ONE action, ONE subject, ONE camera movement. No cuts within a clip, no scene that needs two different camera positions.

---

## Step 1 — Get Audio Duration

```bash
ffprobe -v quiet -print_format json -show_format "[Sound/master.mp3]"
```
Extract `duration` in seconds. If no audio file exists yet: use estimated duration from music_director SUNO brief.

Calculate required unique clips:
```
clip_count = ceil(duration / 10)
Print: "Song duration: [Xs]. Required unique clips: [clip_count]"
```

---

## Step 2 — Build Timestamp Map

If `Lyrics/timestamps.json` exists (from whisper): use it directly.
If not: estimate section timestamps from lyrics structure:

```
Total duration: [T] seconds
Sections: [list from SUNO Lyrics tags]
Estimated duration per section:
  Intro (if any): ~15s
  Verse: ~30–45s
  Chorus: ~20–30s
  Bridge: ~20s
  Outro: ~15–20s
  Distribute to sum = T
```

Print the timestamp map for the human to review.

---

## Step 3 — Write Scene Script

Output `Lyrics/music-video-script.md`:

```markdown
# Music Video Script — [Song Title]
**Duration:** [T]s | **Unique clips required:** [N] | **Clip length:** 10s each
**Audio:** [path to master file]

---

## Timestamp Map
| Section | Start | End | Duration |
|---|---|---|---|
| Intro | 00:00 | 00:15 | 15s |
| Verse 1 | 00:15 | 01:00 | 45s |
| Chorus 1 | 01:00 | 01:30 | 30s |
| ... | ... | ... | ... |

---

## Scene Breakdown

### Scene 01 — [Short Name]
**Timestamp:** 00:00 – 00:10 [ESTIMATED / CONFIRMED]
**Section:** Intro
**Location:** [Location name from CASTING brief]
**Shot type:** [Wide / Medium / Close-up / Detail]
**Subject:** [What is in frame — specific]
**Action:** [What moves, if anything — camera or subject]
**Emotion:** [The feeling this shot communicates]

**Higgsfield Image Prompt:**
```
[Full prompt: 30–80 words. Location fragment + subject + lighting + action + style.
Example: "Empty coffee shop interior before opening, early morning blue light,
chairs upturned on tables, dust particles in a shaft of window light,
static wide shot, film grain, 35mm, melancholic atmosphere, cinematic"]
```

**Higgsfield Video Motion Spec:**
```
Camera: [static / slow push-in / slow pull-back / slow pan left / slow pan right / slow tilt up]
Speed: [very slow]
Subject motion: [none / subtle — e.g. "steam rising from a cup"]
```

---
[Repeat for every scene — minimum [clip_count] scenes]
```

---

## Step 4 — Write Cut List JSON

Output `Assembly/cut_list.json`:

```json
{
  "project_id": "[project_id]",
  "audio_file": "Sound/master.mp3",
  "total_duration_seconds": 0,
  "clip_length_seconds": 10,
  "total_clips": 0,
  "cuts": [
    {
      "id": "c01",
      "scene_name": "[Scene 01 short name]",
      "start_seconds": 0,
      "end_seconds": 10,
      "source_clip": "Clips/c01_[scene_name].mp4",
      "trim_start": 0,
      "trim_end": 10,
      "transition": "cut"
    }
  ]
}
```

Generate one entry per clip. The `source_clip` path is where `video_generator` will deposit the file. The `video_editor` will read this JSON to build the assembly.

---

## Step 5 — Human Approval Gate

Present to human:

```
🎬 SCENE SCRIPT READY

Song duration: [T]s | Unique clips: [N]

[Paste the full Timestamp Map table]
[Paste the first 3 Scene entries as examples]
[... and N more scenes (full script saved to Lyrics/music-video-script.md)]

COST ESTIMATE:
  [N] images × $0.04 = $[image_cost]
  [N] video clips × $0.12 = $[clip_cost]
  Total generation estimate: $[total]

Please review and approve:
  - OK → pipeline will proceed to audio_production, then image/video generation
  - REVISE [scene number] + [notes] → modify specific scenes
  - ADD SCENE → I will insert additional scene variants
  - CHANGE [section] LOCATION → remap to different location
```

**On OK:**
- Save both output files
- Update PROJECT_STATE
- Stage Gate: script → ✅ done
- Set `current_stage: audio_production`
- Set `next_action: "Human submits to SUNO per SUNO_BRIEF file. After takes are downloaded to Sound/, run audio_engineer skill."`

---

## Scene Writing Quality Standards

**Strong scene descriptions:**
- Contain a specific, observable subject (not "sadness" but "an empty chair at a set table")
- Describe light direction and quality
- Imply emotion through physical detail, not abstract labels
- Are achievable in a single static or slow-moving 10-second clip

**Scene arc principle:**
- Scenes in Verse 1: establishing, observational, wide or medium shots
- Scenes in Chorus: slightly more dynamic, medium to close-up, slightly more subject motion
- Scenes in Verse 2: more intimate than Verse 1, closer shots
- Bridge: the most unusual or revealing image of the video
- Final Chorus / Outro: return to wide shots from a slightly different angle — the world looks the same but feels different

**Shot variety rule:**
No more than 3 consecutive scenes of the same shot type (wide/medium/close). Variety prevents visual monotony.
