# Genre Module — Pop (v1.0)

**Genre tag:** `pop` | Aliases: pop_radio, modern_pop, indie_pop
**Load condition:** When `genre` in PROJECT_STATE.md is `pop` or any alias
**Injected into:** lyricist, music_director, scriptwriter, image_generator, video_editor, color_grader
**Version History:** v1.0 — Initial release (2026-05-17)

> This file is a parameter overlay. It does NOT replace the general skill — it is loaded alongside it and provides genre-specific parameters to override the general skill's defaults.

---

## Lyricist Parameters

| Parameter | Pop setting |
|---|---|
| **Syllable strictness** | MEDIUM — priority is on hook memorability, not perfect syllable fit |
| **Rhyme scheme** | AABB or ABAB; chorus rhyme MUST be perfect (not near-rhyme) |
| **Line length** | Flexible — shorter lines for verse (4-7 syllables), longer for pre-chorus build |
| **Hook** | CRITICAL — chorus opening line must be the most memorable line. Test aloud repeatedly |
| **Verse count** | 2 verses + pre-chorus + chorus + bridge + final chorus (standard pop structure) |
| **Pre-chorus** | REQUIRED — builds energy from verse into chorus |
| **Repetition** | Chorus repeated 3x minimum in standard pop structure |
| **Tone** | Broad emotional range: hopeful, defiant, celebratory, bittersweet — avoid ambiguity |
| **Abstract vs concrete** | Mix — hook line should be abstract/universal; verse should be specific story |
| **Language** | If non-English: ensure hook phrase works phonetically in target language |

**SUNO lyric formatting:**
- Mark sections: `[Verse 1]`, `[Pre-Chorus]`, `[Chorus]`, `[Verse 2]`, `[Bridge]`, `[Outro]`
- Hook line of chorus: write it on its own line for emphasis
- Keep chorus to 4-8 lines maximum

---

## Music Director Parameters

| Parameter | Pop setting |
|---|---|
| **BPM** | 100-130 BPM |
| **Key** | Major preferred for upbeat; minor for emotional pop |
| **Lead instrument** | Synth or guitar lead in chorus; piano or muted guitar in verse |
| **Production style** | Modern pop production: layered synths, crisp drum sounds, wide stereo mix |
| **Vocals** | Compression: heavier than ballad. Pitch correction: light. Ad-libs acceptable in outro |
| **Percussion** | 4/4 kick pattern, snare on 2 and 4, hi-hat driving 8th notes |
| **Dynamics** | Verse-chorus contrast through frequency, not just volume. Drop elements in verse |
| **Outro** | Repeat chorus or stripped-back piano/vocal only version |

**SUNO style prompt additions for pop:**
```
modern pop, [X]bpm, [major/minor] key, catchy hook, punchy drums, 
layered synths, polished production, radio-ready
```

---

## Scriptwriter Parameters

| Parameter | Pop setting |
|---|---|
| **Scene pacing** | Fast to medium — 3-6s per scene in verse, 6-10s in chorus |
| **Framing emphasis** | Mix of wide, medium, and close-up. More variety than ballad |
| **Camera motion** | Dynamic: movement on beat, rack focus, follow shot acceptable |
| **Cut rhythm** | On the beat — especially for chorus cuts |
| **Location weight** | Flexible — urban, studio, natural all acceptable |
| **Emotion arc** | Energy builds from start to finish |
| **Action density** | Medium — characters can move, dance lightly, interact |
| **Visual concept** | Pop allows more stylized/metaphorical concepts vs realistic ballad storytelling |

---

## Image Generator Parameters

| Parameter | Pop setting |
|---|---|
| **Color temperature** | Warmer, more saturated than ballad |
| **Saturation** | 70-90% — vibrant but not over-saturated |
| **Lighting** | Dramatic: backlight, colored gel, dappled light acceptable |
| **Lens feel** | 50mm standard or slight wide — more of the world visible |
| **Depth of field** | Medium — background visible but not sharp |
| **Time of day** | Any — golden hour and night city scenes common |
| **Weather** | Any — clear sky acceptable for pop |
| **Model selection** | `gpt_image_2` standard; `text2image_soul_v2` for character shots |
| **Aspect ratio** | `16:9` always |
| **Resolution** | `2k` always |

**Pop prompt additions:**
```
vibrant, dynamic lighting, energetic atmosphere, cinematic color, 
stylized, modern aesthetic
```

---

## Video Editor Parameters

| Parameter | Pop setting |
|---|---|
| **Default cut length** | 4-8s per scene |
| **Minimum cut length** | 2s (fast-cut chorus sequences acceptable) |
| **Transition type** | Hard cut preferred. Use dissolve sparingly for emotional moments |
| **Text overlay** | Song title/artist at opening acceptable; avoid during video |
| **Slow motion** | Acceptable for hero moments — 50% or 25% speed for stylized effect |

---

## Color Grade Parameters

| Parameter | Pop setting |
|---|---|
| **Preset** | `cinematic_teal_orange` (default cinematic grade) — apply with less desaturation than ballad |
| **Saturation** | Master sat -5 to +5 from neutral (much less desaturation than ballad) |
| **Highlights** | Can push higher for pop — brightness reads as energy |
| **LUT intensity** | 50-65% |

---

## YouTube / Distribution Parameters

| Parameter | Pop setting |
|---|---|
| **Title tone** | Direct, energetic, hook phrase from the song title or lyrics |
| **Tags** | Include: "pop music video", "[language] pop", song title variations, artist name |
| **Thumbnail feel** | High contrast, subject facing camera, bright color accent |
| **Upload time** | Follow YouTube recommendations from `Brand/company_learnings.md [YOUTUBE]` |
