# Export Specialist — AI Skill v1.0

**Role:** Produces all required export files from the final mastered video — platform-specific formats for YouTube, Spotify audio distribution, and social media clips.
**Phase:** Distribution
**Inputs:** `Assembly/Music_Video_final.mp4`, `Sound/master_mastered.wav`
**Outputs:** `Distribution/[title]_YouTube_1080p.mp4`, `Distribution/[title]_Spotify.wav`, platform-specific variants
**Tools:** `ffmpeg`
**Human Gate:** NO — automated batch export
**Reads from PROJECT_STATE:** `song_title`, `artist_name`, `asset_registry.assembly`, `asset_registry.audio`
**Writes to PROJECT_STATE:** `asset_registry.distribution`, `current_stage → youtube_upload`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Export Targets

### YouTube Master (Primary)
```bash
ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -c:v libx264 -preset slow -crf 16 \
  -profile:v high -level:v 4.2 \
  -pix_fmt yuv420p \
  -c:a aac -b:a 384k -ar 48000 \
  -movflags +faststart \
  "Distribution/[project_id]_YouTube_1080p.mp4" -y
```

| Parameter | Value | Reason |
|---|---|---|
| Codec | H.264 High Profile | Maximum compatibility, YouTube re-encodes anyway |
| CRF | 16 | High quality master before YouTube re-encoding |
| Audio | AAC 384kbps 48kHz | Above YouTube minimum — quality is preserved through their encoder |
| Faststart | yes | Enables streaming before full download |
| Resolution | 1920×1080 | Upload 1080p minimum |

### Spotify / Music Distribution Audio
```bash
ffmpeg -i "Sound/master_mastered.wav" \
  -c:a pcm_s16le -ar 44100 \
  "Distribution/[project_id]_Distribution_master.wav" -y
```

Most distributors (DistroKid, TuneCore, Amuse) accept WAV 16-bit 44.1 kHz as their preferred input. Verify on your distributor's requirements page before uploading.

### Instagram / TikTok Reel (9:16)
```bash
# Crop to 9:16 from center of 16:9 frame
ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -vf "crop=1080:1920:420:0,scale=1080:1920" \
  -c:v libx264 -preset slow -crf 20 \
  -c:a aac -b:a 192k \
  -t 60 \
  "Distribution/[project_id]_Reel_60s.mp4" -y
```

Note: `-t 60` caps at 60 seconds (Instagram Reel max). For TikTok: use `-t 60` for standard, `-t 180` for longer format.

### YouTube Shorts (9:16, max 60s)
```bash
ffmpeg -i "Assembly/Music_Video_final.mp4" \
  -vf "crop=1080:1920:420:0,scale=1080:1920" \
  -c:v libx264 -preset slow -crf 20 \
  -c:a aac -b:a 192k \
  -t 59 \
  "Distribution/[project_id]_Shorts.mp4" -y
```

---

## Step 1 — Pre-Export Checks

```
1. Assembly/Music_Video_final.mp4 exists and is > 50MB
2. Sound/master_mastered.wav exists
3. Distribution/ folder exists
4. Disk space: estimate 500MB needed for all exports
   - YouTube: ~800MB (high quality)
   - WAV audio: ~40MB
   - Reel/Shorts: ~100MB each
```

---

## Step 2 — Run All Exports

Execute all four export commands sequentially. Log file sizes on completion:

```
📦 EXPORT COMPLETE

Distribution/[project_id]_YouTube_1080p.mp4     →  [X] MB  ✅
Distribution/[project_id]_Distribution_master.wav →  [X] MB  ✅
Distribution/[project_id]_Reel_60s.mp4          →  [X] MB  ✅
Distribution/[project_id]_Shorts.mp4            →  [X] MB  ✅
```

---

## Step 3 — Verify Exports

```bash
# Quick check on each file:
ffprobe -v quiet -show_entries format=duration:format=size -of csv "Distribution/[file]"
```

Verify:
- YouTube file: duration matches original, has video+audio streams
- WAV file: is 44100 Hz, 16-bit stereo
- Reel/Shorts: duration ≤ 60s, 9:16 ratio (1080×1920)

---

## Step 4 — Update PROJECT_STATE

```yaml
asset_registry:
  distribution:
    youtube_master: "Distribution/[project_id]_YouTube_1080p.mp4"
    audio_master: "Distribution/[project_id]_Distribution_master.wav"
    reel: "Distribution/[project_id]_Reel_60s.mp4"
    shorts: "Distribution/[project_id]_Shorts.mp4"
current_stage: "youtube_upload"
next_action: "Run youtube_uploader skill to prepare metadata."
```

---

## Platform Spec Quick Reference

| Platform | Video Format | Resolution | Max Duration | Audio |
|---|---|---|---|---|
| YouTube | H.264 MP4 | 1920×1080 min, 3840×2160 preferred | No limit | AAC 384kbps |
| Instagram Feed | H.264 MP4 | 1080×1080 or 1080×1350 | 60 min | AAC 128kbps min |
| Instagram Reel | H.264 MP4 | 1080×1920 | 15 min | AAC |
| TikTok | H.264 MP4 | 1080×1920 | 10 min (standard) | AAC |
| YouTube Shorts | H.264 MP4 | 1080×1920 | 60s | AAC |
| Spotify (audio) | WAV / FLAC | N/A | N/A | 16-bit/44.1kHz |
| Apple Music | WAV / FLAC | N/A | N/A | 24-bit/44.1kHz preferred |
