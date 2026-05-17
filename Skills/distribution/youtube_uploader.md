# YouTube Uploader — AI Skill v1.0

**Role:** Prepares the complete YouTube upload package — SEO-optimized title, description, tags, chapters, and settings — ready for the human to paste into YouTube Studio.
**Phase:** Distribution
**Inputs:** `Lyrics/LYRICS_[project_id].md` (Sections 5–7), `Lyrics/CONCEPT_[project_id].md`, `Assembly/cut_list.json` (for chapter timestamps), `Brand/channel_brief.md`, `PROJECT_STATE.md`
**Outputs:** `Distribution/youtube_metadata.md` — the complete upload package
**Tools:** None (document generation only — human executes upload)
**Human Gate:** YES — human approves all metadata before upload
**Reads from PROJECT_STATE:** `song_title`, `artist_name`, `genre`, `mood`, `target_channel`, `decisions.youtube_publish_date`
**Writes to PROJECT_STATE:** `asset_registry.distribution.youtube_metadata`, `current_stage → distribution`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## YouTube SEO Principles

1. **Title formula:** `[Emotional hook] | [Song Title] — [Artist Name] (Official Music Video)`
   - The emotional hook at the start captures search intent
   - "Official Music Video" signals authority and improves CTR vs auto-generated content

2. **Description formula:** Story + Lyrics + Links + Tags
   - First 2 lines are visible before "Show more" — make them compelling
   - Include full lyrics (boosts searchability and YouTube keyword indexing)
   - Timestamps for chapters improve watch-time signals

3. **Tags:** Mix broad (genre, mood) + specific (artist comparisons, instrumentation) + long-tail phrases

4. **First 48 hours matter most** — schedule for peak hours in your target audience's timezone (typically Fri/Sat 14:00–18:00 local time)

---

## Step 1 — Build Chapter Timestamps

Read `Assembly/cut_list.json` and group scenes by their section (from `music-video-script.md`):

```
0:00 — Intro
0:15 — Verse 1
1:00 — Chorus
1:30 — Verse 2
2:15 — Bridge
2:40 — Final Chorus
3:10 — Outro
```

YouTube auto-detects chapters if:
- At least 3 timestamps
- Video is 10+ minutes (for shorter videos, add them anyway — they show in description)
- First timestamp starts at 0:00

---

## Step 2 — Generate YouTube Metadata

Output `Distribution/youtube_metadata.md`:

```markdown
# YouTube Upload Package — [Song Title]
> Copy each section below directly into YouTube Studio fields.
> Do NOT upload until human has approved this document.

---

## VIDEO FILE
File: Distribution/[project_id]_YouTube_1080p.mp4

## THUMBNAIL FILE
File: Marketing/Thumbnail/thumbnail.png
Alt:  Marketing/Thumbnail/thumbnail_alt.png

---

## TITLE
[Emotional descriptor — 2-3 words] | [Song Title] — [Artist Name] (Official Music Video)

Examples by genre:
  Ballad:    "Empty Station | [Song Title] — [Artist] (Official Music Video)"
  Pop:       "Can't Stop Running | [Song Title] — [Artist] (Official Music Video)"
  Hip-Hop:   "No Looking Back | [Song Title] — [Artist] (Official Music Video)"

Chosen title: [GENERATED TITLE]
Character count: [N] / 100 max  ← keep under 70 for mobile display

---

## DESCRIPTION
[First 2 lines — most important — visible before "Show more"]
[Line 1: most evocative sentence about the song's emotion/story]
[Line 2: call to action or artist statement]

[blank line]
🎵 Stream & Download: [distributor link — add after distribution step]
📺 Subscribe: [channel URL]

[blank line]
⏱️ CHAPTERS
[timestamp list from Step 1]

[blank line]
📝 LYRICS
[full lyrics from LYRICS file Section 4 — stripped of SUNO tags]

[blank line]
🎬 PRODUCTION NOTES
Music: [Artist Name] × SUNO AI
Video: [Artist Name] × Higgsfield AI
Produced by: [Artist Name / Company Name]

[blank line]
#[genre] #[mood] #[artist_name_no_spaces] #[song_title_no_spaces] #officialmusic

---

## TAGS (copy as comma-separated list)
[From LYRICS Section 5 — YouTube Keywords]

---

## SETTINGS
Category:       Music
Language:       [language]
Made for kids:  No
Visibility:     [Scheduled / Public — fill in date]
Publish date:   [decisions.youtube_publish_date — or "publish immediately"]
Comments:       On (hold potentially inappropriate comments for review)
Age restriction: No
License:        Standard YouTube License
Allow embedding: Yes
Notify subscribers: Yes

---

## END CARDS (add at 0:20 before end)
Element 1: Subscribe button — centered
Element 2: Latest video — top right corner

## CARDS (add at 30% into video)
Type: Channel subscription prompt

---

## PRE-PUBLISH CHECKLIST
☐ Video file uploaded successfully
☐ Thumbnail set to: thumbnail.png
☐ Title verified (under 70 chars for mobile)
☐ Description preview checked (first 2 lines compelling)
☐ Chapters verified in description (all timestamps correct)
☐ Tags entered (max 500 chars total)
☐ Category set to Music
☐ Language set
☐ End cards added
☐ Scheduled publish time set
☐ "Made for kids" = No
```

---

## Step 3 — Human Approval Gate

```
📺 YOUTUBE METADATA READY

Title: [generated title]
Description preview (first 2 lines):
  [line 1]
  [line 2]

Tags: [first 5 tags] ... ([total count] total)
Chapters: [chapter list]
Schedule: [date/time or "immediate"]

Approve?
  - OK → final metadata saved, ready to upload
  - REVISE TITLE → [new title]
  - REVISE DESCRIPTION → [notes]
  - CHANGE DATE → [new date/time]
```

**On OK:**
- Save `Distribution/youtube_metadata.md` as final
- Update PROJECT_STATE:
  - `asset_registry.distribution.youtube_metadata` ← path
  - `decisions.youtube_publish_date` ← confirmed date
  - `current_stage: distribution`
  - Stage Gate: youtube_upload → ✅ done

---

## Step 4 — Upload Instructions for Human

```
🚀 UPLOAD GUIDE — YouTube Studio

1. Go to: youtube.com/upload
2. Drag & drop: Distribution/[project_id]_YouTube_1080p.mp4
3. While uploading, fill in fields from Distribution/youtube_metadata.md:
   - Title field ← copy TITLE section
   - Description field ← copy DESCRIPTION section
4. Thumbnail: upload Marketing/Thumbnail/thumbnail.png
5. Playlists: add to your [genre] playlist
6. Tags: paste from TAGS section
7. More options → Language + Category + Comments settings
8. Visibility: Scheduled → [publish date from metadata]
9. Add End Cards and Cards as specified
10. Click SAVE → video is scheduled

After publish:
→ Add distributor link to description (after distribution step)
→ Pin a comment with song description
→ Run analytics_reporter after 48 hours for first performance check
```

Update PROJECT_STATE after upload: add upload confirmation to `human_log`.
