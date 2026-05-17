# Motion Designer — AI Skill v1.0

**Role:** Adds animated title cards, artist name, lower thirds, channel intro, and outro to the mastered music video using FFmpeg text and filter overlays.
**Phase:** Post-Production
**Inputs:** `Assembly/Music_Video_mastered.mp4`, `Brand/color_palette.json`, `Brand/channel_brief.md`, `Lyrics/LYRICS_[project_id].md` (song title)
**Outputs:** `Assembly/Music_Video_final.mp4` — the fully branded final video
**Tools:** `ffmpeg` (drawtext, fade filters)
**Human Gate:** NO — templates applied from brand guide
**Reads from PROJECT_STATE:** `song_title`, `artist_name`, `decisions.color_grade_style`
**Writes to PROJECT_STATE:** `asset_registry.assembly.final_video`, `current_stage → graphics`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Standard Motion Elements

Every music video produced by this company includes these branded elements:

| Element | When | Duration |
|---|---|---|
| Song title card | 0:00–0:05 | 5s fade in / hold 3s / fade out |
| Artist name | Below title, same card | With title |
| Subtitle / tagline | Optional — below artist name | With title |
| Outro card | Last 5 seconds | Channel name + "Subscribe" |
| Lower third (artist) | Optional — appears at 0:10 if no title card | 3s |

---

## Font and Style Rules

Read `Brand/color_palette.json` for colors.

**Font fallback stack (FFmpeg `fontfile` parameter):**
```
1. Brand font (if set in channel_brief.md)
2. System: Arial / Helvetica / Liberation Sans
3. FFmpeg built-in: default
```

**Text style defaults (override in brand_guardian.md if different):**
```
Title text:      fontsize=80, bold, color=white, shadow color=black, shadowx=2, shadowy=2
Artist text:     fontsize=48, regular, color=#CCCCCC, shadow minimal
Outro text:      fontsize=56, bold, color=white
All text:        letter_spacing=4, uppercase recommended
```

---

## Step 1 — Build Title Card Overlay

FFmpeg drawtext for opening title (song title + artist name):

```bash
ffmpeg -i "Assembly/Music_Video_mastered.mp4" \
  -vf "
    drawtext=text='[SONG TITLE]':
      fontsize=80:fontcolor=white:x=(w-text_w)/2:y=(h/2)-60:
      shadowcolor=black:shadowx=2:shadowy=2:
      enable='between(t,0,8)':
      alpha='if(lt(t,1),t/1, if(gt(t,7),max(0,(8-t)/1),1))',
    drawtext=text='[ARTIST NAME]':
      fontsize=48:fontcolor=#CCCCCC:x=(w-text_w)/2:y=(h/2)+10:
      shadowcolor=black:shadowx=1:shadowy=1:
      enable='between(t,0,8)':
      alpha='if(lt(t,1),t/1, if(gt(t,7),max(0,(8-t)/1),1))'
  " \
  -c:v libx264 -preset slow -crf 16 \
  -c:a copy \
  "Assembly/Music_Video_titled.mp4" -y
```

**Fade timing breakdown:**
- `t=0 to t=1`: fade in (1 second)
- `t=1 to t=7`: fully visible
- `t=7 to t=8`: fade out (1 second)

---

## Step 2 — Build Outro Card

Read the total video duration from PROJECT_STATE. The outro occupies the last 5 seconds:

```bash
DURATION=$(ffprobe -v quiet -show_entries format=duration -of csv="p=0" "Assembly/Music_Video_titled.mp4")
OUTRO_START=$(echo "$DURATION - 5" | bc)

ffmpeg -i "Assembly/Music_Video_titled.mp4" \
  -vf "
    drawtext=text='[CHANNEL NAME]':
      fontsize=56:fontcolor=white:x=(w-text_w)/2:y=(h/2)-30:
      shadowcolor=black:shadowx=2:shadowy=2:
      enable='gte(t,${OUTRO_START})':
      alpha='if(lt(t,${OUTRO_START}+1),(t-${OUTRO_START})/1, if(gt(t,${DURATION}-1),max(0,(${DURATION}-t)/1),1))',
    drawtext=text='SUBSCRIBE':
      fontsize=36:fontcolor=#AAAAAA:x=(w-text_w)/2:y=(h/2)+20:
      enable='gte(t,${OUTRO_START})':
      alpha='if(lt(t,${OUTRO_START}+1),(t-${OUTRO_START})/1, if(gt(t,${DURATION}-1),max(0,(${DURATION}-t)/1),1))'
  " \
  -c:v libx264 -preset slow -crf 16 \
  -c:a copy \
  "Assembly/Music_Video_final.mp4" -y
```

---

## Step 3 — Opening Fade-In (Black to Video)

Add a 1.5-second black fade-in at the very start if the video doesn't already start from black:

```bash
ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -vf "fade=t=in:st=0:d=1.5" \
  -c:v libx264 -preset slow -crf 16 -c:a copy \
  "Assembly/Music_Video_final.mp4" -y
```

---

## Step 4 — Optional: Outro Background Overlay

If the outro needs a semi-transparent dark overlay behind the text (for readability):

```bash
-vf "
  drawbox=x=0:y=(h/2)-50:w=iw:h=100:color=black@0.5:t=fill:
    enable='gte(t,${OUTRO_START})',
  [text overlays from Step 2]
"
```

---

## Step 5 — Verify Final Video

```bash
ffprobe -v quiet -print_format json -show_format -show_streams "Assembly/Music_Video_final.mp4"
```

Check:
- Duration matches mastered video (text overlays should not change duration)
- Resolution: 1920×1080
- Both audio and video streams present

```
✅ MOTION DESIGN COMPLETE

Output: Assembly/Music_Video_final.mp4
Title card: "[song_title]" + "[artist_name]" — 0:00–0:08
Outro card: "[channel_name]" + "SUBSCRIBE" — last 5s
Duration unchanged: [X min Y sec]
```

---

## Step 6 — Update PROJECT_STATE

```yaml
asset_registry:
  assembly:
    final_video: "Assembly/Music_Video_final.mp4"
current_stage: "graphics"
next_action: "Run graphic_designer skill to create thumbnail."
```

---

## Customization Notes

To change font, color, or text: edit this skill's template or override in `Brand/brand_guardian.md` → Motion Design section.
The motion_designer always reads `Brand/color_palette.json` first — if `accent` color is defined there, it may be used for the title underline or a subtle decorative element.

For more complex animations (kinetic typography, logo bugs, animated lyrics): these require DaVinci Resolve Fusion or After Effects — flag as "manual step" and note in human_log.
