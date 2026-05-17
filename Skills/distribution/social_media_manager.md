# Social Media Manager — AI Skill v1.0

**Role:** Creates platform-specific promotional clips, story graphics, and a posting schedule from the finished music video.
**Phase:** Distribution
**Inputs:** `Assembly/Music_Video_final.mp4`, `Sound/master_mastered.wav`, `Marketing/Thumbnail/thumbnail.png`, `Lyrics/LYRICS_[project_id].md`, `PROJECT_STATE.md`
**Outputs:** `Marketing/Trailer/trailer_60s.mp4`, `Marketing/Social_Clips/[platform]_*.mp4`, posting schedule
**Tools:** `ffmpeg`
**Human Gate:** NO — automated clip generation
**Reads from PROJECT_STATE:** `song_title`, `artist_name`, `target_channel`, `decisions.youtube_publish_date`
**Writes to PROJECT_STATE:** `asset_registry.marketing.social_clips`, `asset_registry.marketing.trailer`, `current_stage → analytics`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Clip Strategy

The goal is to maximize reach before and after the YouTube release. The standard clip package:

| Asset | Duration | Platform | Timing |
|---|---|---|---|
| 60s Trailer | 60s | YouTube Community, Instagram, Twitter | 3–5 days before YouTube release |
| 30s Teaser | 30s | Instagram Story, TikTok | 1–2 days before release |
| YouTube Short | 59s | YouTube Shorts | Day of release or day after |
| 15s Hook | 15s | TikTok, Instagram Reel | 1 week after release |
| Lyric clip | 60s | YouTube Community, Facebook | 1–2 weeks after release |

---

## Step 1 — Identify the Best 60 Seconds

Analyze the video for the most compelling 60-second window:

**Algorithm:**
1. The first chorus is usually the best hook — find its timestamp from `cut_list.json` section data
2. Ideally: start 10s before the chorus drops (to build tension) and end at the 60s mark into the chorus
3. If the chorus is after the 60s mark: use 0:00 to 0:60 (opening + first verse)

Ask human if automatic selection seems wrong: "I selected [start]–[end] as the trailer window. Does this capture the best part of the song?"

---

## Step 2 — Create Trailer (60s)

```bash
ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -ss [start_seconds] -t 60 \
  -c:v libx264 -preset slow -crf 18 \
  -c:a aac -b:a 256k \
  "Marketing/Trailer/trailer_60s.mp4" -y
```

Add "Coming [date]" or "Out Now" text overlay if needed:

```bash
# Add "OUT NOW" text + song title if release date has passed
ffmpeg -i "Marketing/Trailer/trailer_60s.mp4" \
  -vf "
    drawtext=text='OUT NOW':fontsize=72:fontcolor=white:x=(w-text_w)/2:y=40:
      shadowcolor=black:shadowx=2:shadowy=2:
      enable='between(t,50,60)':
      alpha='if(lt(t,50),0,if(lt(t,51),(t-50)/1,1))',
    drawtext=text='[SONG TITLE] — [ARTIST NAME]':
      fontsize=44:fontcolor=#EEEEEE:x=(w-text_w)/2:y=115:
      shadowcolor=black:shadowx=1:shadowy=1:
      enable='between(t,50,60)':
      alpha='if(lt(t,50),0,if(lt(t,51),(t-50)/1,1))'
  " \
  -c:v libx264 -preset slow -crf 18 -c:a copy \
  "Marketing/Trailer/trailer_60s_final.mp4" -y
```

---

## Step 3 — Create 9:16 Vertical Versions

For Instagram Story, TikTok, YouTube Shorts:

```bash
# 60s vertical (YouTube Shorts / Instagram Reel)
ffmpeg -i "Marketing/Trailer/trailer_60s.mp4" \
  -vf "crop=1080:1920:420:0" \
  -c:v libx264 -preset slow -crf 20 \
  -c:a aac -b:a 192k \
  "Marketing/Social_Clips/youtube_shorts.mp4" -y

# 30s teaser (Instagram Story)
ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -ss [chorus_start] -t 30 \
  -vf "crop=1080:1920:420:0" \
  -c:v libx264 -preset slow -crf 20 \
  -c:a aac -b:a 192k \
  "Marketing/Social_Clips/instagram_story_30s.mp4" -y

# 15s hook (TikTok / Reel)
ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -ss [most_catchy_moment] -t 15 \
  -vf "crop=1080:1920:420:0" \
  -c:v libx264 -preset slow -crf 20 \
  -c:a aac -b:a 192k \
  "Marketing/Social_Clips/tiktok_15s.mp4" -y
```

---

## Step 4 — Create Lyric Clip (Optional)

A 60s clip with the chorus lyrics displayed on screen as subtitles:

```bash
# Create a subtitle file (SRT) from the chorus lyrics
# Write manually or from timestamps.json

ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -ss [verse1_start] -t 60 \
  -vf "subtitles=Lyrics/chorus_lyrics.srt:force_style='FontName=Arial,FontSize=28,PrimaryColour=&Hffffff,OutlineColour=&H000000,Outline=2,Bold=1'" \
  -c:v libx264 -preset slow -crf 20 \
  -c:a aac -b:a 192k \
  "Marketing/Social_Clips/lyric_clip_60s.mp4" -y
```

---

## Step 5 — Generate Posting Schedule

Calculate dates based on `decisions.youtube_publish_date`:

```markdown
# Social Media Posting Schedule — [Song Title]

## Pre-Release
[publish_date - 5 days]  →  Post trailer_60s to YouTube Community + Instagram Feed
[publish_date - 3 days]  →  Post instagram_story_30s.mp4 to Instagram Story
[publish_date - 1 day]   →  Post "Tomorrow" announcement + thumbnail to all channels

## Release Day: [publish_date]
[publish_date, same time as YouTube] → YouTube video goes live (scheduled)
[publish_date + 2 hours]            → Post youtube_shorts.mp4 to YouTube Shorts
[publish_date + 4 hours]            → Post tiktok_15s.mp4 to TikTok
[publish_date + 6 hours]            → Reply to all comments — first 24h is critical

## Post-Release
[publish_date + 3 days]   →  Post lyric_clip_60s.mp4 to Instagram + Facebook
[publish_date + 7 days]   →  Run analytics_reporter to check performance
[publish_date + 14 days]  →  Post tiktok_15s.mp4 again with different caption
[publish_date + 30 days]  →  Analytics full review — inform next project concept
```

Save as `Distribution/posting_schedule.md`.

---

## Captions and Hashtag Templates

### Instagram / TikTok Caption Template
```
[Evocative line from concept brief — make it feel personal]

[SONG TITLE] out now 🎵 [Link in bio]

#[genre] #[mood] #newmusic #[artist_name] #officialvideo #[song_title_no_spaces]
[5–10 niche hashtags from YouTube keywords list]
```

### YouTube Community Post Template
```
[SONG TITLE] is out now — [one sentence about the song]

[YouTube video link]

[Call to action: "What does this song make you feel?" or thematic question]
```

---

## Step 6 — Update PROJECT_STATE

```yaml
asset_registry:
  marketing:
    trailer: "Marketing/Trailer/trailer_60s_final.mp4"
    social_clips:
      - "Marketing/Social_Clips/youtube_shorts.mp4"
      - "Marketing/Social_Clips/instagram_story_30s.mp4"
      - "Marketing/Social_Clips/tiktok_15s.mp4"
      - "Marketing/Social_Clips/lyric_clip_60s.mp4"
    posting_schedule: "Distribution/posting_schedule.md"
current_stage: "analytics"
next_action: "Monitor performance. Run analytics_reporter 48 hours after YouTube publish."
```
