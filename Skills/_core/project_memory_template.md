# PROJECT_STATE — [SONG TITLE]
> **Template Version:** 1.1 | Copy this file to `Projects/[Song Title]/PROJECT_STATE.md` and fill in the fields.
>
> **Companion files in this project folder:**
> - `PROJECT_STATE.md` — this file. Permanent record. Managed by CEO.
> - `PROJECT_CONTEXT.md` — session scratchpad (temporary, not in human_log). Created by CEO at session start. Cleared/archived after 3 days.

---

## Identity

```yaml
song_title: ""
artist_name: ""
project_id: ""                    # slug, e.g. "blue_rain_2026"
genre: ""                         # ballad | pop | hip-hop | EDM | folk | other
mood: ""                          # e.g. "melancholic, cinematic, hopeful"
language: ""
target_channel: ""                # YouTube channel name this project belongs to
created: ""                       # ISO date, e.g. 2026-05-17
last_updated: ""
```

---

## Pipeline Stage

```yaml
intake_complete: false            # set to true by project_intake worker
current_stage: "intake"           # intake | concept | lyrics | music_direction | casting | locations | script | audio_production | lyric_alignment | creative_design | image_generation | video_generation | video_edit | color_grade | sound_design | motion_design | graphics | export | youtube_upload | distribution | social | analytics | retrospective | complete
blocked: false
blocked_reason: ""
next_action: "Run project_intake skill to analyze existing files and complete project setup."
```

### Stage Gate Checklist
| # | Stage | Status | Human Gate | Approved By |
|---|---|---|---|---|
| 1 | intake | ⬜ pending | NO — agent auto-completes | — |
| 2 | concept | ⬜ pending | YES — human approves brief | — |
| 3 | lyrics | ⬜ pending | YES — human approves lyrics | — |
| 4 | music_direction | ⬜ pending | NO — SUNO prompt only | — |
| 5 | casting | ⬜ pending | NO — auto or flag Soul ID | — |
| 6 | locations | ⬜ pending | NO — text mood boards | — |
| 7 | script | ⬜ pending | YES — human approves scene script | — |
| 8 | audio_production | ⬜ pending | YES — human picks best SUNO take | — |
| 9 | lyric_alignment | ⬜ pending | NO — Whisper auto-timestamps | — |
| 10 | creative_design | ⬜ pending | NO — storyboard arrangement auto | — |
| 11 | image_generation | ⬜ pending | YES — human approves each image | — |
| 12 | video_generation | ⬜ pending | NO — batch run | — |
| 13 | video_edit | ⬜ pending | NO — FFmpeg / DaVinci assembly | — |
| 14 | color_grade | ⬜ pending | NO — auto LUT | — |
| 15 | sound_design | ⬜ pending | NO — auto normalize | — |
| 16 | motion_design | ⬜ pending | NO — template apply | — |
| 17 | graphics | ⬜ pending | YES — human approves thumbnail | — |
| 18 | export | ⬜ pending | NO — format batch | — |
| 19 | youtube_upload | ⬜ pending | YES — human approves metadata | — |
| 20 | distribution | ⬜ pending | YES — human executes upload | — |
| 21 | social | ⬜ pending | NO — clip cuts auto | — |
| 22 | analytics | ⬜ pending | NO — reporting only | — |
| 23 | retrospective | ⬜ pending | YES — human approves skill changes | — |

---

## Decisions Log

```yaml
decisions:
  soul_id_required: false         # set by casting_director
  soul_id_reference_man: ""       # Higgsfield Soul ID string for male character (set by soul_identity skill)
  soul_id_reference_woman: ""     # Higgsfield Soul ID string for female character (set by soul_identity skill)
  soul_id_reference: ""           # legacy field — do not use for new projects; kept for back-compat
  suno_style_prompt: ""           # locked after music_direction stage
  suno_selected_take: ""          # filename of chosen audio take
  video_aspect_ratio: "16:9"      # 16:9 | 9:16 | 1:1
  color_grade_style: ""           # e.g. "cinematic teal-orange", "warm vintage"
  distributor: ""                 # DistroKid | TuneCore | Amuse | other
  youtube_publish_date: ""
  capex_approved: false           # human must set to true before costly batch runs
  capex_budget_usd: 0
```

---

## Human Log

> Every human instruction, correction, or approval is appended here with a timestamp.
> The CEO orchestrator reads this on every run to apply corrections.

```yaml
human_log:
  - timestamp: ""
    type: ""                      # approval | correction | instruction | capex_decision
    stage: ""
    message: ""
```

**Example entries:**
```yaml
human_log:
  - timestamp: "2026-05-17T14:30:00"
    type: approval
    stage: concept
    message: "Concept approved. Mood should feel late-night, urban loneliness."
  - timestamp: "2026-05-17T15:00:00"
    type: correction
    stage: lyrics
    message: "The chorus is too abstract. Rewrite it to be more concrete — use the image of an empty subway station."
  - timestamp: "2026-05-17T16:00:00"
    type: capex_decision
    stage: image_generation
    message: "Approved 40 image generations and 30 video clips. Budget: $15 Higgsfield credits."
```

---

## Asset Registry

```yaml
asset_registry:
  audio:
    master: ""                    # e.g. Sound/master.mp3
    suno_takes: []                # e.g. [Sound/take_01.mp3, Sound/take_02.mp3]
    timestamps_json: ""           # Lyrics/timestamps.json
  lyrics:
    analysis: ""                  # Lyrics/ANALYSIS_[title].md
    final: ""                     # Lyrics/LYRICS_[title].md
  script:
    video_script: ""              # Lyrics/music-video-script.md
    cut_list_json: ""             # Assembly/cut_list.json
  images: []                      # Images/*.png
  clips: []                       # Clips/*.mp4
  cuts: []                        # Cuts/*.mp4
  assembly:
    concat_txt: ""                # Assembly/concat.txt
    final_video: ""               # Assembly/Music_Video.mp4
    color_graded: ""              # Assembly/Music_Video_graded.mp4
    mastered_audio: ""            # Sound/master_mastered.wav
  marketing:
    thumbnail: ""                 # Marketing/Thumbnail/thumbnail.png
    trailer: ""                   # Marketing/Trailer/trailer.mp4
    social_clips: []              # Marketing/Social_Clips/*.mp4
  distribution:
    youtube_metadata: ""          # Distribution/youtube_metadata.md
    export_youtube: ""            # Distribution/[title]_YouTube_4K.mp4
    export_audio: ""              # Distribution/[title]_Spotify.wav
```

---

## Spend Log

```yaml
spend_log:
  total_usd_estimate: 0
  entries:
    - timestamp: ""
      worker: ""
      action: ""
      units: 0
      cost_usd_estimate: 0
```

---

## Notes

> Free-form notes from any worker or the human. Append, never delete.

```
[no notes yet]
```

---

## Genre Context

> Populated by CEO when genre module is loaded. Do not edit manually.

```yaml
genre_module_loaded: false
genre_module_path: ""             # e.g. Skills/genres/ballad.md
genre_parameters_active: false
```

