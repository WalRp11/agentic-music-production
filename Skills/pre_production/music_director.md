# Music Director — AI Skill v1.0

**Role:** Finalizes the SUNO music generation brief — song structure, instrumentation, tempo, key, and generation strategy — and prepares the exact prompts for the human to submit to SUNO.
**Phase:** Pre-Production
**Inputs:** `Lyrics/LYRICS_[project_id].md` (Section 3: SUNO Style Prompt + Section 4: SUNO Lyrics), `PROJECT_STATE.md`
**Outputs:** `Lyrics/SUNO_BRIEF_[project_id].md` — ready-to-submit SUNO generation plan
**Tools:** SUNO (prompt output for human to submit)
**Human Gate:** NO — brief is auto-generated; human submits to SUNO and picks best take (that gate is in audio_engineer)
**Reads from PROJECT_STATE:** `genre`, `mood`, `language`, `decisions.suno_style_prompt`
**Writes to PROJECT_STATE:** `next_action`, `current_stage → casting`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Purpose

The lyricist wrote the SUNO style prompt. This skill expands it into a complete generation strategy: how many takes to request, what variations to try, structural tagging for SUNO, and quality assessment criteria. The human takes this brief to SUNO and runs the generations.

---

## Step 1 — Read the Lyricist Output

Load `Lyrics/LYRICS_[project_id].md`. Extract:
- Section 3 (SUNO Style Prompt) → base style
- Section 4 (SUNO Lyrics) → confirm structural tags are correct

Validate SUNO lyrics structure:
- Every section must begin with a tag: `[Verse 1]`, `[Chorus]`, `[Verse 2]`, `[Bridge]`, `[Outro]`
- Each section: 4–8 lines
- Total lyrics length: 200–500 words (SUNO max is ~600)
- No line longer than 60 characters (SUNO wraps badly otherwise)

If any validation fails: fix it before continuing.

---

## Step 2 — Build Generation Strategy

Calculate the number of SUNO takes needed:

```
Standard approach: 3 takes minimum (SUNO generates 2 at a time → 2 runs)
If genre = ballad or folk: 4 takes (emotional nuance requires more options)
If language = non-English: 4 takes (accent/pronunciation varies more)
Budget consideration: check budget_manager — cost per run = $0.10–0.20
```

**Variation strategy:** Plan 2–3 style prompt variations to try across takes:
1. **Base prompt** — the prompt from the lyricist, unchanged
2. **Warmer variation** — adjust descriptor words slightly warmer/more intimate
3. **Bigger variation** — add one orchestral or production element (e.g. "string quartet" or "choir backing")

---

## Step 3 — Write SUNO Brief

Output `Lyrics/SUNO_BRIEF_[project_id].md`:

```markdown
# SUNO Generation Brief — [Song Title]

## Overview
Song: [title] | Genre: [genre] | Language: [language]
Target takes: [N] | Estimated SUNO credit cost: [N × 10 credits]

---

## Take 1 (Base) — Paste into SUNO

### Style of Music field:
[exact style prompt from lyricist — copy-paste ready]

### Lyrics field:
[exact lyrics from lyricist with [tags] — copy-paste ready]

---

## Take 2 (Warmer variation)

### Style of Music field:
[modified prompt — warmer descriptors]

### Lyrics field:
[same lyrics]

---

## Take 3 (Bigger variation) [if budget allows]

### Style of Music field:
[modified prompt — added orchestral/production element]

### Lyrics field:
[same lyrics]

---

## SUNO Generation Settings (apply to all takes)
- Mode: Custom (not AI lyrics)
- Instrumental: OFF (we have lyrics)
- Song length: Auto or [X] seconds (match audio duration from intake if available)
- Variations: Generate 2 per submission

---

## How to Submit
1. Go to suno.com → Create
2. Enable "Custom Mode"
3. Paste Style of Music field content
4. Paste Lyrics field content
5. Click Create → wait for both variations
6. Download ALL generated MP3 files
7. Name them: take_01_a.mp3, take_01_b.mp3, take_02_a.mp3, etc.
8. Place ALL files in: [project_folder]/Sound/
9. Then run the CEO orchestrator → it will route to audio_engineer for take selection

---

## Quality Criteria for Take Selection
(The audio_engineer skill uses these, but read them now to understand what to listen for)

✅ ACCEPT if:
- Vocals are clear and on-pitch throughout
- The style prompt was followed (instrumentation matches)
- The emotional arc is audible (verse feels different from chorus)
- No clipping, distortion, or abrupt endings

❌ REJECT if:
- SUNO generated the wrong language or randomly switched languages
- Vocals are mumbled, buried in the mix, or clearly off-melody
- Chorus has no dynamic lift compared to verse
- Abrupt cutoff at the end (regenerate if possible)

---

## Estimated Total Duration
[Calculated from lyrics word count and BPM:
  word_count = [N]
  estimated_duration = [N × 60 / words_per_minute_at_BPM] seconds
  ≈ [X min Y sec]]

Note: SUNO may trim or extend. Target: [match concept brief duration or 3–4 min standard]
```

---

## Step 4 — Update PROJECT_STATE

- `current_stage: casting`
- `next_action: "Human submits to SUNO per SUNO_BRIEF. Run audio_engineer skill after takes are downloaded to Sound/ folder."`
- `asset_registry.lyrics.suno_brief` ← path to SUNO_BRIEF file

---

## SUNO Prompt Engineering Reference

### Effective style descriptors by element:

**Vocals:**
`soft male tenor`, `powerful female soprano`, `breathy indie vocals`,
`husky baritone`, `choir backing`, `harmonized`, `no vocals` (instrumental)

**Instrumentation:**
`grand piano`, `acoustic guitar`, `electric guitar (clean)`, `electric guitar (distorted)`,
`cello`, `violin`, `string quartet`, `synthesizer pad`, `bass guitar`,
`electronic drums`, `live drums`, `drum machine`, `trumpet`, `saxophone`

**Production style:**
`lo-fi`, `cinematic`, `orchestral`, `minimalist`, `lush production`,
`reverb-heavy`, `dry and intimate`, `studio quality`, `vintage analog`

**Tempo:**
`very slow 50bpm`, `slow 65bpm`, `moderate 85bpm`, `upbeat 110bpm`,
`fast 130bpm`, `dance 128bpm`

**Key and mood:**
`D minor melancholic`, `A major uplifting`, `C minor dark`, `G major warm`,
`F# minor introspective`

**Atmosphere:**
`late night`, `intimate`, `vast and open`, `claustrophobic`,
`nostalgic`, `futuristic`, `earthy`, `ethereal`

### Anti-patterns (avoid in SUNO prompts):
- Contradictory descriptors: `fast 50bpm` or `upbeat and melancholic`
- Too many instruments: more than 6–7 instruments in prompt reduces coherence
- Vague mood words without anchors: `good vibes` / `emotional` alone is too weak
- Overloading with sub-genres: pick one or two, not five
