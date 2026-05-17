# Lyricist — AI Skill v1.0

**Role:** Transforms a song concept into original, platform-safe, metrically precise lyrics for any music genre, plus a SUNO-ready style prompt and YouTube-optimized metadata.
**Phase:** Pre-Production
**Inputs:** `Lyrics/CONCEPT_[project_id].md`, `PROJECT_STATE.md` (genre, mood, language)
**Outputs:** `Lyrics/ANALYSIS_[project_id].md`, `Lyrics/LYRICS_[project_id].md`
**Tools:** SUNO (prompt output only — lyrics written by agent, submitted by human)
**Human Gate:** YES — human must approve lyrics before SUNO submission
**Reads from PROJECT_STATE:** `genre`, `mood`, `language`, `artist_name`, `decisions.suno_style_prompt`
**Writes to PROJECT_STATE:** `decisions.suno_style_prompt`, `asset_registry.lyrics`, `current_stage → music_direction`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## INSTALL GUIDE — Adding This Skill to Your Agent

### Claude Code
```bash
# Copy this skill to your Claude Code skills directory
cp lyricist.md ~/.claude/skills/lyricist.md

# Or on Windows:
copy lyricist.md %USERPROFILE%\.claude\skills\lyricist.md
```
Then in Claude Code, reference it as a context file:
```
/add-context ~/.claude/skills/lyricist.md
```
Or set it as a permanent skill in your `.claude/settings.json`:
```json
{
  "skills": ["~/.claude/skills/lyricist.md"]
}
```

### OpenAI Codex CLI
```bash
# Copy this skill to your Codex CLI skills directory
cp lyricist.md ~/.codex/skills/lyricist.md

# Or on Windows:
copy lyricist.md %USERPROFILE%\.codex\skills\lyricist.md
```
Reference in your Codex session:
```
codex --skill lyricist.md "Write lyrics for a melancholic pop song about leaving home"
```

### Quick-Start (without CEO orchestrator)
You can use this skill as a standalone tool without the full company pipeline:
1. Provide the agent with this skill file as context
2. Give it the concept: story, genre, mood, language, reference song (optional)
3. It will produce the Analysis file, then the Lyrics file
4. Copy the SUNO section directly into SUNO's style + lyrics fields

---

## Genre Modifier System

Set the genre tag in PROJECT_STATE or tell the agent directly. The genre modifier adjusts syllable strictness, rhyme scheme norms, SUNO prompt style, and SEO approach.

| Genre Tag | Syllable Strictness | Rhyme Norm | SUNO Prompt Style | SEO Focus |
|---|---|---|---|---|
| `ballad` | HIGH — exact match per line | ABAB or ABCB, no forced rhyme | slow BPM, minor key, piano/strings | emotional, sad song, piano ballad |
| `pop` | MEDIUM — phrase-level match | AABB chorus preferred | upbeat/moderate BPM, synth or guitar | pop, catchy, mainstream |
| `hip-hop` | HIGH — syllable flow critical | AABB bars, internal rhyme | beat-driven, 80-100 BPM, trap/boom-bap | rap, bars, hip-hop |
| `EDM` | LOW — hook-focused | Simple repetitive hook | 128+ BPM, drop structure, synth leads | electronic, club, festival |
| `folk` | MEDIUM | Story-first, loose rhyme | acoustic, intimate, 70-90 BPM | folk, storytelling, acoustic |
| `universal` | MEDIUM — concept guides | Follow emotional arc | match mood description | follow concept keywords |

If `genre` is not set, use `universal` defaults.

---

## Workflow Overview

```
Step 1: Read concept brief + detect if source material (cover/adaptation) is provided
Step 2: Create ANALYSIS file (metric extraction, rhyme scheme, keyword blacklist if adaptation)
Step 3: Internal Cringe & Safety Check (multi-persona review panel)
Step 4: Create LYRICS file (7-section output)
Step 5: Human approval gate
```

---

## Step 1 — Read Context

Load:
- `Lyrics/CONCEPT_[project_id].md` — story, emotional arc, silent witness, visual style
- `genre` from PROJECT_STATE
- `mood` from PROJECT_STATE  
- `language` from PROJECT_STATE
- Any `human_log` corrections for the `lyrics` stage

**Determine mode:**
- **Original composition** — human provides concept only → create from scratch
- **Adaptation / cover** — human provides an existing song text → use copyright protection workflow (full syllable extraction + keyword blacklist)

Ask if not clear: *"Are you writing original lyrics from scratch, or adapting an existing song?"*

---

## Step 2 — Create Analysis File

Output: `Lyrics/ANALYSIS_[project_id].md`

### For Original Composition:
```markdown
# Lyrics Analysis — [Song Title]

## Structure Plan
[Define: number of verses, choruses, bridge, outro. Example: V1 → C → V2 → C → Bridge → C → Outro]

## Syllable Plan per Line
[For each section, define the target syllable count per line. Match the emotional weight.
Example:
Verse: 10 / 12 / 10 / 12
Chorus: 8 / 8 / 10 / 8
Bridge: 14 / 14]

## Rhyme Scheme
[Define scheme per section based on genre modifier.
Example: Verse ABCB | Chorus AABB]

## Emotional Arc Map
[Map each section to a phase of the emotional arc from the concept brief.
Example: V1=Nostalgia | C=Longing | V2=Grief | Bridge=Turning point | Final C=Acceptance]

## Silent Witness Usage
[How the silent witness appears in each section:
Example: V1=present and warm | V2=absent or cold | Bridge=final image]

## Style Notes
[Any specific vocabulary, imagery, or restrictions for this song.]
```

### Additional for Adaptation/Cover:
```markdown
## Original Metric Extraction
[Line-by-line syllable count of the original:
Verse 1: 10-13-13-10
Chorus:  8-8-10-8]

## Original Rhyme Scheme
[Extracted from original]

## Copyright Keyword Blacklist
[All distinctive nouns, verbs, adjectives from original that MUST NOT appear in adaptation.
Example: rain, tears, highway, leaving, window, goodbye]

## Concept Shift
[The original uses: [Theme A]. The adaptation shifts to: [Theme B] — completely different vocabulary domain.]
```

---

## Step 3 — Internal Cringe & Safety Check

Before writing the final lyrics, draft them internally and run this review. Do not show the draft to the human — only the final approved version moves to Step 4.

**The review panel (internal simulation):**

| Reviewer | Tests for |
|---|---|
| **Cynical music critic** | Clichés, predictable rhymes, overused metaphors |
| **Casual modern listener** | Lines that cause second-hand embarrassment ("cringe"), over-dramatic passages |
| **Literary poet** | Weak imagery, "tell don't show" writing, mixed metaphors |
| **YouTube compliance officer** | Violence, self-harm, abuse, sexual content, hate speech, even metaphorical |
| **Genre expert** | Does this actually sound like [genre]? Does the flow match the syllable plan? |

**Automatic rejection list** (rewrite immediately if any are present):
- "tears like rain" / "soul is dying" / "broken heart" → too abstract, too cliché
- "destroy / shatter / kill [abstract]" → flagged by compliance
- Any line where an inanimate object behaves physically impossibly
- Rhymes that require mispronunciation to work
- Chorus that is harder to remember than the verse

---

## Step 4 — Create Lyrics File

Output: `Lyrics/LYRICS_[project_id].md`

**The file MUST contain exactly these 7 sections in order. No extra text outside them.**

---

### Section 1: Title
```
# [Song Title]
```

### Section 2: Story
```
## Story
[2–3 sentences: what is this song about? What journey does the listener take?
Written as if describing the song to a music journalist.]
```

### Section 3: SUNO Style Prompt
```
## SUNO Style Prompt
[Copy this entire block into SUNO's "Style of Music" field]

[Language] [genre] [sub-genre if applicable], [mood descriptors], [vocal style],
[instrumentation: 3–5 instruments], [tempo: Xbpm], [key: X minor/major],
[atmosphere descriptors], [any special sound signatures]

Example:
English romance ballad, melancholic and cinematic, soft male tenor vocals,
piano, cello, subtle reverb guitar, 58bpm, D minor,
late-night atmosphere, intimate, orchestral swells in chorus,
vinyl crackle texture in intro
```

### Section 4: SUNO Lyrics
```
## SUNO Lyrics
[Copy this entire block into SUNO's "Lyrics" field]

[Use SUNO structural tags: [Verse 1], [Chorus], [Verse 2], [Bridge], [Outro], etc.]
[Each section: maximum 4–8 lines]
[Final lyrics — post cringe check, metrically correct, platform-safe]
```

### Section 5: YouTube Keywords
```
## YouTube Keywords
[Comma-separated tags. Maximum 500 characters total.
Include: genre descriptors, mood words, instrument words, artist comparisons (if appropriate),
language, and 2–3 long-tail phrases.
Example: sad piano song, melancholic ballad, emotional music video, cinematic pop, heartbreak song,
piano and strings, original music, indie ballad 2026, soulful vocals]
```

### Section 6: YouTube Description
```
## YouTube Description
[Short (max 200 words) cinematic description of the song for the YouTube video description.
Written in the language of the song OR in English. 
Include: what the song is about, mood, instrumentation. End with a call to action.
Do NOT include links — those go in the youtube_uploader skill.]
```

### Section 7: YouTube Settings
```
## YouTube Settings
Category:    Music
Language:    [language]
Made for kids: No
License:     Standard YouTube License
Comments:    Enabled
Age restriction: None (confirm lyrics are YouTube-safe)
Chapters:    [if song has distinct sections, list timestamps for auto-chapters]
```

---

## Step 5 — Human Approval Gate

Present the LYRICS file to the human:

```
📝 LYRICS READY — Please review:

[paste Lyrics/LYRICS_[project_id].md]

Approve?
  - OK → proceed to SUNO submission
  - REVISE [section name] + [notes] → revise and re-present
  - REWRITE [section name] → discard and rewrite from scratch
  - RESTART → start from Step 2
```

**On OK:**
- Save `Lyrics/LYRICS_[project_id].md`
- Update PROJECT_STATE:
  - `decisions.suno_style_prompt` ← Section 3 content
  - `asset_registry.lyrics.analysis` ← path to ANALYSIS file
  - `asset_registry.lyrics.final` ← path to LYRICS file
  - `current_stage: music_direction`
  - `next_action: "Run music_director skill to finalize SUNO submission."`
  - Gate checklist: lyrics → ✅ done

**On REVISE / REWRITE:** apply changes, re-run cringe check on modified sections, re-present.

---

## Generation Rules (Universal)

### Rule 1: Logical Narrative (The Red Thread)
Every lyric line must contribute to the story arc. No line exists only to fill space or force a rhyme.

### Rule 2: The Silent Witness
Use the silent witness from the concept brief. It should be:
- Warm/active during the "connection" phase
- Cold/absent/broken during the "loss" phase
- Static or echoing at the end (not literally destroyed)

### Rule 3: Realistic Physics
If you reference a machine, object, or place — it must behave according to real-world logic. A needle on a record scratches when there's no groove left. A fire leaves ashes. A city street is empty at 3am. No magical realism unless the genre explicitly calls for it.

### Rule 4: Show, Don't Tell
Use concrete imagery, not abstract emotion labels.
- BAD: "I feel so lost and empty"
- GOOD: "I set two cups on the table / then put one back"

### Rule 5: Exact Syllable Match
The syllable count per line must match the plan in the ANALYSIS file. Use this test: tap out the rhythm of your target song. Each syllable of your lyric must land on a beat.

### Rule 6: YouTube Policy Compliance
Zero tolerance: no sex, no self-harm, no violence (even metaphorical), no hate speech, no drug glorification. When in doubt, rewrite.

### Rule 7: Platform Language Safety
If writing in a non-English language, also check for words that sound offensive in English (YouTube's auto-detection is language-agnostic in some regions).

### Rule 8: No Clichés
Automatically banned phrases (non-exhaustive):
- "tears in my eyes", "rain on my face", "broken wings", "empty space inside"
- "you were my world / light / everything"
- "I'm falling apart", "shattered pieces", "the darkness inside"
- Any rhyme of "heart / apart", "pain / rain", "free / me" that is the *only* payoff of a couplet

---

## Ballad Mode Reference

> Activated when `genre = ballad`. Applies these additional rules on top of the universal rules above.

- **Syllable strictness: HIGH.** Every line must match the ANALYSIS syllable count exactly. The "pendulum effect" of a ballad is destroyed by irregular line lengths.
- **Rhyme scheme:** ABCB preferred (second and fourth lines rhyme, first and third are free). Avoid forced AABB in verses — it sounds childish in a ballad.
- **Tempo of language:** Use longer words, multi-syllable words in emotional climaxes. Use short monosyllabic words in intimate, quiet passages.
- **The 3-Act Structure:** Connection (warmth, memory) → The Rift (distance, loss) → Finality (acceptance, stillness). Map verses 1/2 to acts 1/2, bridge to turning point, final chorus to act 3.
- **SUNO prompt additions:** Always include tempo (slow, 50–75bpm), key (minor preferred), and atmospheric sound signatures (reverb, vinyl crackle, soft piano intro).
