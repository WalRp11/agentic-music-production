# Graphic Designer — AI Skill v1.0

**Role:** Creates the YouTube thumbnail, channel art variants, and social media graphic assets for the project, maintaining brand consistency.
**Phase:** Post-Production
**Inputs:** `Assembly/Music_Video_final.mp4` (for frame extraction), `Lyrics/CONCEPT_[project_id].md`, `Brand/color_palette.json`, `Brand/channel_brief.md`, `PROJECT_STATE.md`
**Outputs:** `Marketing/Thumbnail/thumbnail.png`, `Marketing/Thumbnail/thumbnail_alt.png`, optional channel art variants
**Tools:** `ffmpeg` (frame extraction), Higgsfield CLI (thumbnail image generation), `ffmpeg` (text overlay on thumbnail)
**Human Gate:** YES — human must approve thumbnail before upload
**Reads from PROJECT_STATE:** `song_title`, `artist_name`, `mood`, `decisions.color_grade_style`
**Writes to PROJECT_STATE:** `asset_registry.marketing.thumbnail`, `current_stage → export`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## YouTube Thumbnail Specifications

| Property | Requirement |
|---|---|
| Resolution | 1280×720 pixels (minimum), 1920×1080 preferred |
| File format | PNG or JPG |
| File size | ≤ 2MB |
| Aspect ratio | 16:9 |
| Safe zone | Keep all text and key visuals within 1184×616 center area |
| Font readability | Text must be legible at 120×68px (mobile search result size) |

---

## Thumbnail Design Formula (Proven for Music)

**Rule of thirds + High contrast + Single focal point + Readable text**

Effective music video thumbnail structure:
```
[Key visual: artist face OR strongest image from video]
+ [Song title text — large, high contrast]
+ [Optional: artist name — smaller, below title]
```

**What works for music thumbnails:**
- Close-up face with strong eye contact (or strong emotional expression)
- Single dominant color with high contrast
- Large, bold title text (visible at 120px wide)
- Consistent brand font

**What does NOT work:**
- Text smaller than 40px relative to 1280-wide image
- More than 2–3 text elements
- Busy backgrounds that compete with the subject
- Colors that blend with YouTube's red/white UI

---

## Step 1 — Extract Best Frame from Video

```bash
# Extract frames from key emotional moments (chorus, bridge)
# Example: grab frame at 1:05 (65 seconds)
ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -ss 65 -vframes 1 \
  "Marketing/Thumbnail/frame_01.png"

# Grab a few options at different moments
ffmpeg -i "Assembly/Music_Video_final.mp4" -ss 25 -vframes 1 "Marketing/Thumbnail/frame_02.png"
ffmpeg -i "Assembly/Music_Video_final.mp4" -ss 95 -vframes 1 "Marketing/Thumbnail/frame_03.png"
```

Ask human: "Which of these frames (frame_01, frame_02, frame_03) would you like as the thumbnail base? Or should I generate a new image specifically for the thumbnail?"

---

## Step 2 — Option A: Use Video Frame as Base

If human selects a video frame:

```bash
# Scale to thumbnail resolution
ffmpeg -i "Marketing/Thumbnail/frame_01.png" \
  -vf "scale=1280:720" \
  "Marketing/Thumbnail/thumbnail_base.png"
```

Then add text overlay (Step 4).

---

## Step 3 — Option B: Generate Dedicated Thumbnail Image

If human prefers a purpose-built thumbnail image:

Build a Higgsfield prompt specifically optimized for thumbnail use:
- Close-up composition (face or key object fills 60%+ of frame)
- Strong contrast and bold colors
- No background clutter
- Horizontal format (16:9)

```bash
higgsfield generate image \
  --model gpt_image_2 \
  --prompt "[close-up of protagonist] OR [key visual from concept], [dominant emotion], 
    high contrast, strong lighting, [mood color] tones, centered composition, 
    music video thumbnail style, bold and visually striking, 16:9 horizontal,
    [global style keywords from locations document]" \
  --size 1280x720 \
  --output "Marketing/Thumbnail/thumbnail_generated.png"
```

Generate 2 variants with different compositions.

---

## Step 4 — Add Text Overlay

Apply song title to the selected base image:

```bash
ffmpeg -i "Marketing/Thumbnail/thumbnail_base.png" \
  -vf "
    drawtext=text='[SONG TITLE]':
      fontsize=96:fontcolor=white:
      x=(w-text_w)/2:y=h-120:
      shadowcolor=black:shadowx=3:shadowy=3:
      borderw=2:bordercolor=black,
    drawtext=text='[ARTIST NAME]':
      fontsize=52:fontcolor=#EEEEEE:
      x=(w-text_w)/2:y=h-60:
      shadowcolor=black:shadowx=2:shadowy=2
  " \
  "Marketing/Thumbnail/thumbnail.png"
```

**Alternative: Bottom gradient + text (more polished):**
```bash
ffmpeg -i "Marketing/Thumbnail/thumbnail_base.png" \
  -vf "
    drawbox=x=0:y=ih*0.65:w=iw:h=ih*0.35:color=black@0.65:t=fill,
    drawtext=text='[SONG TITLE]':
      fontsize=88:fontcolor=white:x=(w-text_w)/2:y=h*0.70:
      shadowcolor=black:shadowx=2:shadowy=2,
    drawtext=text='[ARTIST NAME]':
      fontsize=46:fontcolor=#CCCCCC:x=(w-text_w)/2:y=h*0.85:
      shadowcolor=black:shadowx=1:shadowy=1
  " \
  "Marketing/Thumbnail/thumbnail.png"
```

---

## Step 5 — Generate Alt Thumbnail

Create a second variant with different cropping or text placement for A/B testing:

```bash
# Alt: title at top instead of bottom
ffmpeg -i "Marketing/Thumbnail/thumbnail_base.png" \
  -vf "
    drawbox=x=0:y=0:w=iw:h=ih*0.28:color=black@0.65:t=fill,
    drawtext=text='[SONG TITLE]':
      fontsize=88:fontcolor=white:x=(w-text_w)/2:y=h*0.06:
      shadowcolor=black:shadowx=2:shadowy=2,
    drawtext=text='[ARTIST NAME]':
      fontsize=46:fontcolor=#CCCCCC:x=(w-text_w)/2:y=h*0.18:
      shadowcolor=black:shadowx=1:shadowy=1
  " \
  "Marketing/Thumbnail/thumbnail_alt.png"
```

---

## Step 6 — Human Approval Gate

```
🖼️  THUMBNAIL READY — Please review:

Main thumbnail:  Marketing/Thumbnail/thumbnail.png
Alt thumbnail:   Marketing/Thumbnail/thumbnail_alt.png

Mobile check: View both images at 120×68px — can you read the title?

Approve?
  - MAIN → use thumbnail.png for upload
  - ALT → use thumbnail_alt.png for upload
  - BOTH REJECTED → describe what you want changed
  - REGENERATE → generate completely new base image
```

---

## Step 7 — Update PROJECT_STATE

```yaml
asset_registry:
  marketing:
    thumbnail: "Marketing/Thumbnail/thumbnail.png"
    thumbnail_alt: "Marketing/Thumbnail/thumbnail_alt.png"
current_stage: "export"
next_action: "Run export_specialist skill."
```

---

## Social Media Graphic Variants (If Time Allows)

After thumbnail approval, optionally create these additional formats:

| Format | Size | Usage |
|---|---|---|
| Instagram Square | 1080×1080 | Feed post |
| Instagram Story / TikTok | 1080×1920 (9:16) | Story/Reel teaser |
| Twitter/X Banner | 1500×500 | Profile header |

For each: re-crop the thumbnail base to the target ratio and re-apply text at appropriate scale.
