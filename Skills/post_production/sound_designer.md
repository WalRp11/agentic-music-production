# Sound Designer — AI Skill v1.0

**Role:** Normalizes the master audio track to platform loudness standards, checks for quality issues, and produces the mastered audio file used in the final video and for distribution.
**Phase:** Post-Production
**Inputs:** `Sound/master.mp3`, `Assembly/Music_Video_graded.mp4`
**Outputs:** `Sound/master_mastered.wav`, `Assembly/Music_Video_mastered.mp4` (video with mastered audio)
**Tools:** `ffmpeg` (loudnorm filter), optional: external mastering service
**Human Gate:** NO — automated
**Reads from PROJECT_STATE:** `asset_registry.audio.master`, `asset_registry.assembly.color_graded`
**Writes to PROJECT_STATE:** `asset_registry.audio.mastered`, `asset_registry.assembly.mastered_video`, `current_stage → motion_design`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Platform Loudness Standards

| Platform | Integrated Loudness | True Peak | LRA |
|---|---|---|---|
| YouTube | −14 LUFS | −1.0 dBTP | ≤ 14 LU |
| Spotify | −14 LUFS | −1.0 dBTP | ≤ 14 LU |
| Apple Music | −16 LUFS | −1.0 dBTP | ≤ 14 LU |
| TikTok / Reels | −14 LUFS | −1.0 dBTP | ≤ 14 LU |
| Tidal | −14 LUFS | −1.0 dBTP | ≤ 14 LU |

**Target: −14 LUFS integrated, −1.0 dBTP true peak.** This satisfies YouTube, Spotify, and TikTok simultaneously.

---

## Step 1 — Analyze Current Levels

```bash
ffmpeg -i "Sound/master.mp3" \
  -af "loudnorm=I=-14:TP=-1.0:LRA=11:print_format=json" \
  -f null - 2>&1 | tail -20
```

This prints the current loudness stats without modifying anything:
```json
{
  "input_i": "-16.24",      ← current integrated loudness
  "input_tp": "-2.10",      ← current true peak
  "input_lra": "8.50",      ← current LRA
  "input_thresh": "-26.40",
  "target_offset": "2.24"   ← how much gain to apply
}
```

Report to log:
```
🔊 AUDIO ANALYSIS
  Input LUFS:  [input_i] dB
  True Peak:   [input_tp] dBTP
  LRA:         [input_lra] LU
  Target:      -14 LUFS / -1.0 dBTP
  Correction:  [target_offset] dB
```

---

## Step 2 — Apply Two-Pass Loudness Normalization

**IMPORTANT: Always use the 2-pass method for accurate results.**

```bash
# Pass 1: Measure (same command as Step 1 but capture the JSON values)
MEASURED=$(ffmpeg -i "Sound/master.mp3" \
  -af "loudnorm=I=-14:TP=-1.0:LRA=11:print_format=json" \
  -f null - 2>&1 | grep -A 20 '^\{')

# Extract values (bash):
IL=$(echo $MEASURED | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['input_i'])")
ITP=$(echo $MEASURED | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['input_tp'])")
ILRA=$(echo $MEASURED | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['input_lra'])")
ITHRESH=$(echo $MEASURED | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['input_thresh'])")
OFFSET=$(echo $MEASURED | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['target_offset'])")

# Pass 2: Apply normalization with measured values
ffmpeg -i "Sound/master.mp3" \
  -af "loudnorm=I=-14:TP=-1.0:LRA=11:measured_I=${IL}:measured_TP=${ITP}:measured_LRA=${ILRA}:measured_thresh=${ITHRESH}:offset=${OFFSET}:linear=true:print_format=summary" \
  -ar 44100 -c:a pcm_s16le \
  "Sound/master_mastered.wav" -y
```

**Output format: WAV 16-bit 44100 Hz** — this is the distribution master. Lossy formats (MP3) should never be used as the master.

---

## Step 3 — Replace Audio in Graded Video

```bash
ffmpeg \
  -i "Assembly/Music_Video_graded.mp4" \
  -i "Sound/master_mastered.wav" \
  -c:v copy \
  -c:a aac -b:a 320k \
  -map 0:v:0 -map 1:a:0 \
  -shortest \
  "Assembly/Music_Video_mastered.mp4" -y
```

This replaces the audio track in the graded video with the mastered audio, keeping the video stream untouched.

---

## Step 4 — Verify Mastered Levels

```bash
ffmpeg -i "Sound/master_mastered.wav" \
  -af "loudnorm=I=-14:TP=-1.0:LRA=11:print_format=json" \
  -f null - 2>&1 | tail -20
```

Verify `input_i` is within ±0.5 LUFS of −14.0.

```
✅ MASTERING COMPLETE

Output: Sound/master_mastered.wav
  Final LUFS:  [value] (target: -14.0)
  True Peak:   [value] (target: ≤ -1.0)
  LRA:         [value] (target: ≤ 14 LU)
  Format:      WAV 16-bit 44100 Hz
  Status:      ✅ Within spec  OR  ⚠️ Off by [X] — re-run with adjusted offset

Video: Assembly/Music_Video_mastered.mp4
  Audio:       AAC 320kbps (mastered)
  Video:       Copy from graded (untouched)
```

---

## Step 5 — Update PROJECT_STATE

```yaml
asset_registry:
  audio:
    mastered: "Sound/master_mastered.wav"
  assembly:
    mastered_video: "Assembly/Music_Video_mastered.mp4"
current_stage: "motion_design"
next_action: "Run motion_designer skill."
```

---

## Optional: External Mastering Services

For higher quality mastering than FFmpeg's loudnorm can achieve (recommended for commercial release):

| Service | Cost | Turnaround |
|---|---|---|
| LANDR (auto-master) | ~$4/track | Instant |
| eMastered | ~$9/track | Instant |
| Matchering (open source, free) | Free | Local |
| Human mastering engineer | $50–200/track | 1–3 days |

If using external service:
1. Upload `Sound/master.mp3` to the service
2. Download the mastered file
3. Save as `Sound/master_mastered.wav` (request WAV output)
4. Run Step 3 to embed in video
5. Verify levels with Step 4

Note: External mastering services often target −14 LUFS automatically — verify before embedding.
