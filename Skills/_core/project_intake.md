# Project Intake — AI Skill v1.0

**Role:** Universal entry point for any project — scans the project folder, analyzes existing files, interviews the human to fill in missing metadata, reorganizes loose files into the correct subfolder structure, and writes a fully populated PROJECT_STATE.md.
**Phase:** Core
**Inputs:** Project folder path (may be empty or contain loose files)
**Outputs:** Populated `PROJECT_STATE.md`, organized subfolder structure, intake summary for human confirmation
**Tools:** `ffprobe`, `whisper` (optional), file system read/write
**Human Gate:** YES — intake interview and confirmation of populated state
**Reads from PROJECT_STATE:** all (to detect partial state)
**Writes to PROJECT_STATE:** all fields; sets `intake_complete: true` when done
**Version History:** v1.0 — Initial release (2026-05-17)

---

## When This Skill Runs

- Always — it is the mandatory first stage of every project.
- Triggered by the CEO when `intake_complete == false`.
- Works for: a completely empty project folder, a folder with only an audio file, a folder with a mixture of loose files from a previous partial session, and a fully structured folder from a previous project that needs its state refreshed.

---

## Phase 1 — Folder Scan

Recursively walk the entire project folder. Catalogue every file found:

```
FOR each file in project folder (recursive):
    Classify by extension:
        Audio:  .mp3 .wav .flac .aac .ogg → candidate for Sound/
        Image:  .png .jpg .jpeg .webp     → candidate for Images/
        Video:  .mp4 .mov .mkv            → candidate for Clips/ or Assembly/
        Text:   .md .txt .json            → candidate for Lyrics/ or root
        Other:  log, ignore
```

Print a structured inventory to the human:

```
📂 Inventory of: [project folder path]
   🎵 Audio files found:  [count] — [filenames]
   🖼️  Image files found:  [count] — [filenames]
   🎬 Video files found:  [count] — [filenames]
   📝 Text files found:   [count] — [filenames]
   ❓ Unrecognized files: [count] — [filenames]
```

---

## Phase 2 — Audio Analysis (if audio file found)

For each audio file found, run:

```bash
ffprobe -v quiet -print_format json -show_format -show_streams "[audio_file]"
```

Extract and record:
- `duration_seconds` — total song length
- `bitrate` — kbps
- `format` — mp3/wav/flac
- `sample_rate` — Hz
- `channels` — mono/stereo

**Optional whisper transcript** (only if no lyrics file found):
```bash
whisper "[audio_file]" --language [ask human or detect] --output_format json --output_dir Lyrics/
```
This produces `Lyrics/[filename].json` with word-level timestamps. Store path in `asset_registry.audio.timestamps_json`.

Report to human:
```
🎵 Audio analysis complete:
   Duration: [Xs / X min X sec]
   Format: [format] | Bitrate: [X kbps] | Sample rate: [X Hz]
   [If whisper ran]: Transcript extracted → Lyrics/[filename].json
```

---

## Phase 3 — Human Interview

Ask the following questions. For each question, show any detected value as a default the human can accept or override. Ask all questions in a single block — do not send one question at a time.

```
INTAKE INTERVIEW — Please answer the following. Press Enter to accept [detected value] if shown.

1. Song title?                          [detected: filename stem if obvious, else blank]
2. Artist / channel name?               [blank]
3. Genre?                               (ballad | pop | hip-hop | EDM | folk | other) [blank]
4. Mood / vibe? (free text)             [blank]
5. Language of lyrics?                  [detected: from whisper if ran, else blank]
6. Target YouTube channel?              [blank]
7. Which production stages are already DONE?
   (comma-separate from this list, or write "none"):
   concept, lyrics, music_direction, casting, locations, script,
   audio_production, image_generation, video_generation, video_edit,
   color_grade, sound_design, motion_design, graphics
   → [blank]
8. Any specific instructions or corrections I should know right now?  [blank]
9. Capex: Do you approve batch AI generation runs (image + video)?
   If yes, what is the approximate budget in USD?                      [blank]
```

---

## Phase 4 — File Reorganization

After the interview, move (not delete) all found files into correct subfolders. Never overwrite an existing file — if a conflict exists, add a `_imported` suffix.

| File type | Destination |
|---|---|
| Audio files | `Sound/` |
| Image files | `Images/` |
| Video files (short clips < 30s) | `Clips/` |
| Video files (longer, likely assembled) | `Assembly/` |
| `.md` / `.txt` text files | `Lyrics/` |
| `.json` files | `Lyrics/` (if looks like lyrics/timestamps) or `Assembly/` (if cut_list) |

Report every move:
```
📦 Reorganized files:
   Sound/master.mp3       ← moved from root
   Lyrics/lyrics.md       ← moved from root
   [no conflicts]
```

---

## Phase 5 — Write PROJECT_STATE.md

Copy `Skills/_core/project_memory_template.md` to `Projects/[project_id]/PROJECT_STATE.md` (if it doesn't already exist). Populate all fields from the interview answers and detected data:

```yaml
song_title: "[answer 1]"
artist_name: "[answer 2]"
project_id: "[slugified song title + year]"
genre: "[answer 3]"
mood: "[answer 4]"
language: "[answer 5]"
target_channel: "[answer 6]"
created: "[today's date ISO]"
last_updated: "[now ISO]"

intake_complete: true
current_stage: "[first stage not in 'done' list from answer 7]"
next_action: "Run [next_stage_worker] skill."

decisions:
  capex_approved: [true/false from answer 9]
  capex_budget_usd: [amount from answer 9 or 0]

asset_registry:
  audio:
    master: "[path to moved audio file if exists]"
  lyrics:
    final: "[path to moved lyrics file if exists]"
```

For each stage listed as "done" in answer 7, mark the corresponding gate row as `✅ done` in the Stage Gate Checklist table.

Append the human's answer 8 as the first entry in `human_log[]` if it contains instructions.

---

## Phase 6 — Confirmation Summary

Present the human with a complete intake summary before finalizing:

```
✅ INTAKE COMPLETE — Please review and confirm:

📋 Project: [song_title] | Artist: [artist_name]
🎵 Genre: [genre] | Mood: [mood] | Language: [language]
📺 Target channel: [target_channel]

🗂️  Files organized:
    Sound/[master file]
    Lyrics/[lyrics file if any]

📊 Pipeline status:
    Done:    [list of completed stages]
    Next:    [next_action]

💰 Capex: [approved/not approved] — budget: $[amount]

Type OK to confirm, or give me any corrections.
```

If the human says OK → set `intake_complete: true`, save `PROJECT_STATE.md`, report to CEO orchestrator.
If the human gives corrections → apply them, re-print summary, repeat until confirmed.

---

## Error Handling

| Situation | Action |
|---|---|
| No audio file found | Note in `PROJECT_STATE.md` — set `current_stage: concept`, tell human no audio yet |
| ffprobe not installed | Skip audio analysis, note in `blocked_reason`, tell human to install ffprobe |
| whisper fails | Skip transcript extraction, note in `blocked_reason`, continue with manual lyrics path |
| File conflict on move | Add `_imported` suffix, never overwrite |
| Human gives no answer to a question | Leave field blank, set `blocked_reason` for that field, continue intake with what is available |
