# Genre Module — Ballad (v1.0)

**Genre tag:** `ballad` | Aliases: cinematic_ballad, emotional_ballad, folk_ballad
**Load condition:** When `genre` in PROJECT_STATE.md is `ballad` or any alias
**Injected into:** lyricist, music_director, scriptwriter, image_generator, video_editor, color_grader
**Version History:** v1.0 — Initial release (2026-05-17)

> This file is a parameter overlay. It does NOT replace the general skill — it is loaded alongside it and provides genre-specific parameters to override the general skill's defaults.

---

## Lyricist Parameters

| Parameter | Ballad setting | Override? |
|---|---|---|
| **Syllable strictness** | HIGH — every syllable must fit naturally to melody | Strict |
| **Rhyme scheme** | ABAB or AABB preferred; allow ABCB for folk feel | Preferred |
| **Line length** | 7-10 syllables per melodic phrase | Strict |
| **Verse count** | 2-3 verses + chorus + optional bridge | Flexible |
| **Bridge** | REQUIRED — emotional peak before final chorus | Required |
| **Repetition** | Chorus repeated 2x minimum at end | Required |
| **Tone** | Melancholic, nostalgic, longing, acceptance — NOT hopeful resolution | Strong |
| **Abstract vs concrete** | Concrete imagery preferred — objects, places, sensations | Strong |
| **Clichés** | Avoid generic heartbreak phrases. Use culturally specific, concrete imagery tied to the song's setting and language | Strong |
| **Language rhythm** | For Russian: avoid consecutive stressed syllables. Check aloud for sing-ability | Required |

**SUNO lyric formatting:**
- Mark chorus with `[Chorus]`, verse with `[Verse N]`, bridge with `[Bridge]`
- Do NOT use `[Pre-chorus]` or `[Hook]` for ballads — SUNO handles these poorly in slow tempo
- Keep line breaks consistent with melodic phrase breaks

---

## Music Director Parameters

| Parameter | Ballad setting |
|---|---|
| **BPM** | 55-72 BPM |
| **Key** | Minor preferred: A minor, D minor, G minor. Major only for "bittersweet" resolution feel |
| **Lead instrument** | Piano or acoustic guitar — NOT synth lead |
| **Strings** | Cello or violin REQUIRED in chorus. Intro can be piano-only |
| **Vocals** | Reverb: medium-long tail. Compression: light, preserve dynamics |
| **Percussion** | Brushed snare or no drums in verse. Light kick enters in chorus only |
| **Dynamics** | REQUIRED — verse quiet, chorus 8-12dB louder. Flat dynamics = rejected take |
| **Outro** | Piano out, fade natural — no hard stop |

**SUNO style prompt additions for ballad:**
```
cinematic ballad, piano and cello, [X]bpm, minor key, reverb vocals, 
emotional, building dynamics, orchestral swell in chorus, vinyl warmth
```
Remove from style prompt: "upbeat", "energetic", "trap beat", "EDM", "synth lead"

---

## Scriptwriter Parameters

| Parameter | Ballad setting |
|---|---|
| **Scene pacing** | Slow — prefer 8-14s per scene. Scenes shorter than 6s feel rushed |
| **Framing emphasis** | Close-up and medium-close preferred. Wide shots only for establishing or emotional-peak moments |
| **Camera motion** | Static or very slow push-in. No whip pans, no handheld shake |
| **Cut rhythm** | On emotional beat, NOT on drum hit (ballads have no strong drum hits in verse) |
| **Location weight** | Interior intimate locations weighted 60%. Exterior nature 30%. Urban 10% |
| **Emotion arc** | Must build over the song — start reserved, reach peak at bridge, resolve (not necessarily happily) |
| **Action density** | Low — characters observe, remember, move slowly. No action sequences |
| **Dialogue** | NONE in video — all emotion through action, glances, environment |

**Preferred scene types for ballad:**
- Character alone in a room: looking out a window, sitting at a table
- Memory/flashback insert: same character but different time period (lighting change signals flashback)
- Environmental detail: rain on glass, hands around a cup, unopened door
- Two characters in same space but not touching (emotional distance conveyed visually)
- Natural landscapes: fog, winter trees, empty platform, dark road

---

## Image Generator Parameters

| Parameter | Ballad setting |
|---|---|
| **Color temperature** | Cool to neutral. Warm only in memory/flashback scenes |
| **Saturation** | 40-60% saturation — muted, desaturated palette |
| **Lighting** | Soft, diffused. Window light preferred. No direct sunlight, no harsh shadows |
| **Lens feel** | 35mm full-frame feel. Slight vignette acceptable |
| **Depth of field** | Shallow — subject sharp, background blurred |
| **Time of day** | Dusk, dawn, overcast day, or interior night preferred |
| **Weather** | Rain, fog, overcast, or diffuse snow — no clear blue sky |
| **Model selection** | `gpt_image_2` for standard scenes; `text2image_soul_v2` for character close-ups with Soul ID |
| **Aspect ratio** | `16:9` always |
| **Resolution** | `2k` always |

**Ballad prompt additions:**
```
cinematic, melancholic mood, soft diffused light, muted color palette, 
desaturated, shallow depth of field, 35mm film aesthetic, quiet atmosphere
```

**Ballad prompt exclusions:** (add these as negative guidance when possible)
- "bright", "vibrant", "sunny", "smiling", "colorful", "high saturation", "action", "dynamic"

---

## Video Editor Parameters

| Parameter | Ballad setting |
|---|---|
| **Default cut length** | 10-14s per scene |
| **Minimum cut length** | 5s (never shorter for ballad) |
| **Transition type** | Cross-dissolve (1-2s) preferred for verse. Hard cut acceptable for chorus peak moments |
| **Fade to black** | Use at verse-to-chorus transitions where emotional weight is highest |
| **Match cut** | Use for flashback entries and exits (match eyeline or hand position) |
| **Text overlay** | NONE during main video. Credits only at outro |
| **Lyric display** | Only if explicitly in the project brief — ballad fans prefer to hear, not read |
| **Slow motion** | Acceptable for key emotional moments (50% speed). Do not use slow-mo as a default |

---

## Color Grade Parameters

| Parameter | Ballad setting |
|---|---|
| **Preset** | `cinematic_teal_orange` (default cinematic grade) |
| **Shadows** | Lift to dark grey — avoid pure black |
| **Highlights** | Bring down — clipped whites feel harsh |
| **Saturation** | Master sat -15 to -25 from neutral |
| **Skin tones** | Protect — color grade must not push skin into orange or grey |
| **LUT intensity** | 65-75% — do not apply LUT at 100% (too heavy) |
| **Reference image** | `Brand/color_palette.json` → `cinematic_teal_orange` preset |

---

## YouTube / Distribution Parameters

| Parameter | Ballad setting |
|---|---|
| **Title tone** | Emotional, poetic — include a feeling word (e.g., "Потерянная", "Холодный") |
| **Description opening** | Start with the emotional hook of the lyrics (one translated phrase), then standard description |
| **Tags** | Include: "cinematic ballad", "[language] ballad", "emotional music video", genre + language combo |
| **Thumbnail feel** | One character, facing slightly away or looking down. Dark background, accent text |
| **Upload time** | Follow YouTube recommendations from `Brand/company_learnings.md [YOUTUBE]` |
| **Category** | Music |
| **Age restriction** | None (ballads rarely trigger restriction) |
| **Playlist** | Add to "Ballads" playlist if it exists; create if first ballad |
