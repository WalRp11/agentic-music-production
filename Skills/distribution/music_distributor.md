# Music Distributor — AI Skill v1.0

**Role:** Guides the human through distributing the mastered audio track to streaming platforms (Spotify, Apple Music, Tidal, etc.) via a music distribution aggregator.
**Phase:** Distribution
**Inputs:** `Distribution/[project_id]_Distribution_master.wav`, `Lyrics/LYRICS_[project_id].md` (song metadata), `PROJECT_STATE.md`
**Outputs:** `Distribution/distribution_metadata.md` — complete distributor submission package
**Tools:** None (human-executed on distributor platform)
**Human Gate:** YES — human executes the upload; rights and royalty decisions are always human decisions
**Reads from PROJECT_STATE:** `song_title`, `artist_name`, `genre`, `language`, `target_channel`
**Writes to PROJECT_STATE:** `current_stage → social`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## ⚠️ Legal Notice

Music distribution involves rights and royalties. This skill provides workflow guidance only. The human is always responsible for:
- Confirming they own all rights to the music (SUNO-generated tracks — verify SUNO's current commercial license terms)
- Choosing the correct rights split if collaborators are involved
- Registering with a Performing Rights Organization (PRO) — ASCAP, BMI, SOCAN, GEMA, etc.
- Deciding on explicit content labeling

---

## Supported Distributors

This skill is platform-agnostic. The workflow applies to all major aggregators:

| Distributor | Pricing | Noteworthy |
|---|---|---|
| DistroKid | ~$22/year (unlimited) | Fastest distribution, keeps 100% royalties |
| TuneCore | Per-release (~$10) | Strong reporting, keeps 100% royalties |
| Amuse | Free tier + paid | Good for starting out |
| CD Baby | Per-release (~$10) | Long track record, sync licensing options |
| United Masters | Free + revenue share | Strong brand partnership focus |

**Recommendation:** DistroKid for volume (unlimited releases per year); TuneCore for singles with strong metadata needs.

---

## Step 1 — Prepare Distribution Metadata

Before going to the distributor platform, gather all required fields:

Output `Distribution/distribution_metadata.md`:

```markdown
# Distribution Submission Package — [Song Title]

## Audio File
File: Distribution/[project_id]_Distribution_master.wav
Format: WAV 16-bit 44100 Hz stereo
Duration: [X min Y sec]
Loudness: -14 LUFS (mastered)

---

## Required Metadata Fields

### Track Info
Title: [song_title]
  ↳ DO NOT add "(Official)" or version tags — save those for alt versions
Primary artist: [artist_name]
Featured artists: [none / list]
Explicit content: No  ← verify this matches your lyrics

### Release Info  
Release type: Single
Release date: [suggest 3–7 days from today for indexing, or coordinate with YouTube publish date]
Label: [your name / company name OR "Self-Released"]

### Genre (pick primary + secondary):
Primary genre: [genre from PROJECT_STATE]
Secondary genre: [optional — e.g. "Cinematic" or "Singer-Songwriter"]

### ISRC Code
[ ] I have an ISRC code: [paste here]
[x] Let distributor auto-assign one  ← default for first release

### UPC / EAN
[ ] I have a UPC: [paste here]
[x] Let distributor auto-assign one

### Territories
Worldwide: Yes (recommended unless there are specific restrictions)

### Stores to Target (enable all):
✅ Spotify        ✅ Apple Music      ✅ YouTube Music
✅ Amazon Music   ✅ Tidal            ✅ Deezer
✅ Pandora        ✅ iHeartRadio      ✅ TikTok/Reels
(and all others offered by your distributor)

---

## Album Artwork (required by all distributors)
Size: 3000×3000 pixels minimum (square)
Format: JPG or PNG
File: Marketing/Thumbnail/cover_art_3000x3000.png ← create from thumbnail (Step 2 below)
Requirements: No website URLs, no pricing, no streaming platform logos in artwork

---

## Lyrics (optional but recommended)
Submit lyrics to distributors that support it (DistroKid, TuneCore):
Source: Lyrics/LYRICS_[project_id].md → Section 4 (SUNO Lyrics — strip structural tags)
Format: Plain text, time-synced if available

---

## PRO Registration (Human Task)
If you are registered with a PRO (ASCAP, BMI, SOCAN, GEMA, etc.):
→ Register this track with your PRO after distribution
→ Use the ISRC code assigned by the distributor

If you are NOT yet registered with a PRO:
→ Consider registering before your catalog grows
→ Free in most countries (ASCAP: one-time $50 fee; BMI: free for songwriters)

---

## SUNO Commercial License Check (Human Must Verify)
SUNO's commercial rights policy may change. Before distribution, verify:
→ Visit: suno.com/terms
→ Confirm your plan includes commercial distribution rights
→ Pro/Premier plans typically include commercial use — free plans may not
→ Note the verification date below:
Date verified: [fill in]
Plan at time: [fill in]
```

---

## Step 2 — Create Square Cover Art

Distribution requires square (1:1) artwork. Derive from the thumbnail:

```bash
# Crop 16:9 thumbnail to 1:1 square (center crop)
ffmpeg -i "Marketing/Thumbnail/thumbnail.png" \
  -vf "crop=ih:ih:(iw-ih)/2:0, scale=3000:3000" \
  "Marketing/Thumbnail/cover_art_3000x3000.png" -y
```

The human should inspect `cover_art_3000x3000.png` and confirm it looks good as a square. If the key visual is cut off, adjust the crop offset manually.

---

## Step 3 — Human Upload Guide

```
🎵 DISTRIBUTION UPLOAD GUIDE

Files ready:
  Audio:    Distribution/[project_id]_Distribution_master.wav
  Artwork:  Marketing/Thumbnail/cover_art_3000x3000.png
  Metadata: Distribution/distribution_metadata.md

Steps on your chosen distributor (DistroKid example):
  1. Log in → Upload → New Single
  2. Upload WAV file
  3. Upload cover art (3000×3000 PNG)
  4. Fill in all metadata fields from distribution_metadata.md
  5. Select stores (enable all)
  6. Set release date
  7. Submit for review (DistroKid: typically <24h)

After approval:
  → Copy your Spotify/Apple Music link and add it to the YouTube video description
  → Update PROJECT_STATE.md: add distributor links to human_log
  → Proceed to social_media_manager for promotional clip cuts
```

---

## Step 4 — Update PROJECT_STATE

```yaml
current_stage: "social"
next_action: "Run social_media_manager skill to create promotional clips."
```

Add to `human_log` after upload is confirmed:
```yaml
- timestamp: "[now]"
  type: approval
  stage: distribution
  message: "Submitted to [distributor name]. Expected live date: [date]. ISRC: [code]."
```
