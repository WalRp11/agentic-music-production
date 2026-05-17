# Brand Guardian — AI Skill v1.0

**Role:** Enforces visual identity, naming conventions, and channel strategy across all projects — the single source of truth for brand consistency.
**Phase:** Core
**Inputs:** `Brand/color_palette.json`, `Brand/channel_brief.md`, any output file from a production worker
**Outputs:** Brand compliance notes, corrections to identity-violating decisions
**Tools:** None (advisory/enforcement skill — no generation)
**Human Gate:** NO — reads and enforces; escalates violations to human only if correction requires a major creative decision
**Reads from PROJECT_STATE:** `target_channel`, `genre`, `artist_name`, `decisions.color_grade_style`
**Writes to PROJECT_STATE:** Nothing directly — flags violations in `human_log` if found
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Core Brand Principles

### 1. One Genre Per Channel

Each YouTube channel targets exactly one genre-mood space. This is non-negotiable.

**Why:** YouTube's recommendation algorithm learns your channel's niche. Mixing genres confuses it and limits organic reach. A channel known for "cinematic ballads" will be recommended to listeners of cinematic ballads — mixing in trap music resets this signal.

**In practice:**
- Before creating a new project, confirm `target_channel` in PROJECT_STATE matches an established channel
- If the song's genre doesn't fit any existing channel, the CEO must ask the human to either: (a) create a new channel, or (b) adjust the concept to fit an existing channel
- `channel_brief.md` is the definitive list of channels and their allowed genres

**Rule enforced by:** CEO Orchestrator at concept_developer stage.

---

### 2. Visual Identity Consistency

All visual assets must feel like they come from the same visual universe within a channel.

**Check performed by:** `image_generator` skill (references location_scout's Global Visual Style Sheet).

**Brand Guardian audits:**
- Color grading style is consistent per channel (e.g., all Cinematic Teal-Orange, not mixed)
- Motion designer title card fonts don't change between releases without human approval
- Thumbnail formula remains consistent (same layout, same font treatment, same color temperature)

---

### 3. Artist Name Treatment

Canonical artist name: `[stored in PROJECT_STATE.artist_name]`

Rules:
- Always exactly as stored — no abbreviations, no ALL CAPS, no variations
- In thumbnail text: match exactly
- In YouTube metadata: Title + Description + Tags must all use the same spelling

---

## Color Palette Reference

Load `Brand/color_palette.json` for exact values. If the file is empty (first project), populate it after the first project completes:

```json
{
  "channels": {
    "[channel_name]": {
      "primary": "#RRGGBB",
      "secondary": "#RRGGBB",
      "accent": "#RRGGBB",
      "background": "#RRGGBB",
      "text_light": "#FFFFFF",
      "text_dark": "#111111",
      "color_grade_preset": "cinematic_teal_orange | warm_vintage | dark_moody | clean_pop | cold_drama",
      "thumbnail_font": "[font name]",
      "thumbnail_layout": "[description of thumbnail formula]"
    }
  }
}
```

---

## Typography Defaults

If `Brand/color_palette.json` doesn't specify fonts yet, use these defaults:

| Use | Font (FFmpeg drawtext) | Weight |
|---|---|---|
| Main title card | Arial | Bold |
| Artist name subtitle | Arial | Regular |
| Outro text | Arial | Regular |
| Thumbnail title | System sans-serif (done in image, not FFmpeg) | Bold |

---

## Motion Design Defaults

| Element | Default |
|---|---|
| Title card: In | Fade in over 1.0s |
| Title card: Hold | Until 0:08 |
| Title card: Out | Fade out over 0.5s |
| Outro: In | Fade in over 0.5s at last 5s |
| Outro: Hold | Last 5s |
| Opening | Black → fade in over 2.0s |

---

## Channel Brief Reference

`Brand/channel_brief.md` must contain:

```markdown
# Channel Brief

## Channel: [Channel Name 1]
- YouTube URL: [url]
- Genre focus: [genre]
- Mood range: [e.g. "melancholic, cinematic, hopeful"]
- Target audience: [description]
- Color grade: [preset name]
- Upload cadence: [e.g. "1 video per month"]

## Channel: [Channel Name 2]
[...]
```

---

## Brand Audit Checklist

Run this checklist when reviewing any output before upload:

```
BRAND AUDIT — [project_id]

Visual Identity:
☐ Color grade matches channel's assigned preset
☐ Thumbnail font matches channel's font
☐ Thumbnail layout formula consistent with previous releases on this channel
☐ Artist name spelled correctly and consistently in thumbnail

Metadata:
☐ Artist name exactly matches canonical spelling in all fields
☐ Title format follows: [Hook] | [Song Title] — [Artist Name] (Official Music Video)
☐ Genre tags consistent with channel's genre focus
☐ Description includes subscribe link

Channel Fit:
☐ Song's genre matches target_channel's allowed genres
☐ Emotional tone fits the channel's established identity

Result: ✅ APPROVED | ⚠️ MINOR ISSUES (list below) | ❌ BLOCKED (escalate to human)
```
