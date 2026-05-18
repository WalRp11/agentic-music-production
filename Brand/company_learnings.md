# Company Learnings — WARPMUZ
> **Memory type:** Long-term semantic memory | Maintained by: Retrospective skill + Innovation Scout
> **Last updated:** 2026-05-17 | **Projects completed:** 0 (in progress: Разные Пути)
> **Last intelligence scan:** 2026-05-17 (scope: FULL — initial seed) | **Next scan due:** 2026-06-17
>
> ## HOW TO READ THIS FILE (for CEO / workers)
>
> NEVER load this entire file into context. Instead:
> 1. Read the INDEX below (≈15 lines, ~200 tokens)
> 2. Identify the relevant section tag(s) for your current task
> 3. Read ONLY those section(s)
>
> ## INDEX — Section Tags
> [SUNO]       — SUNO generation tips, style prompt patterns, take selection criteria
> [HIGGSFIELD] — Image and video generation best practices, model selection, prompt patterns
> [SOUL_ID]    — Soul ID training, variant selection, usage rules
> [YOUTUBE]    — Upload settings, SEO patterns, algorithm behavior, CTR/retention benchmarks
> [LEGAL]      — Licensing, TOS, platform policies, rights confirmations
> [PRODUCTION] — Workflow lessons, cross-skill consistency rules, error patterns
> [ANALYTICS]  — Performance benchmarks, audience behavior, content patterns
> [BRAND]      — Visual identity rules, thumbnail performance, channel consistency
> [TOOLS]      — CLI commands, ffmpeg patterns, tool version notes

---

## [SUNO]

> Soul of the SUNO generation strategy — what works, what doesn't.

```
[2026-05-17] For Russian ballad: add "vinyl warmth texture" to style prompt — improves analog feel vs. digital harshness.
[2026-05-17] For non-English languages: order 4 takes minimum — accent/pronunciation varies significantly between takes.
[2026-05-17] Style prompt length: keep under 80 words. SUNO truncates or ignores prompts that are too verbose.
[2026-05-17] BPM: always specify explicitly (e.g., "62bpm"). "slow" is vague; SUNO interprets it differently by genre.
[2026-05-17] soul-2 variant: use soul-2 for text2image_soul_v2 model. soul-cinematic is incompatible with text2image_soul_v2.
```

---

## [HIGGSFIELD]

> Image and video generation patterns.

```
[2026-05-17] Model: gpt_image_2 is the standard for storyboard images. Use soul_cinematic for Soul ID close-ups.
[2026-05-17] Location coherence: always use --image [reference_photo] for all scenes in the same interior location. Without it, the same "room" will look different in every scene.
[2026-05-18] Chain reference limitation: the --image reference anchors only what IS visible in that image. Details NOT in the reference (e.g., a specific door type, a window macro) are generated from model priors and are often wrong. For each unique architectural element, pre-generate and approve a dedicated reference image before generating scenes that feature it.
[2026-05-18] Sliding doors cannot be reliably prompted: ICE train sliding plug doors (horizontal sliding, full-height glass panel) are consistently generated as hinged glass doors or wrong styles by text2image_soul_v2. A dedicated reference image of the correct door must be pre-generated and approved first.
[2026-05-17] Resolution: always 2k for storyboard images. 1k is acceptable only for quick test iterations.
[2026-05-17] Aspect ratio: always 16:9 for standard music video output.
[2026-05-17] Soul HEX color anchoring: include the exact hex color of key wardrobe/environment elements in the prompt when coherence matters (e.g., "ICE 3 train seat color #D4C5A9").
[2026-05-17] Popcorn multi-frame: use for scenes requiring matched eyeline/pose between the man and woman in the same frame.
[2026-05-17] Camera motion in video: default to static or very slow motion. Faster motion looks bad when cut together. Never mix zoom + pan in same clip.
```

---

## [SOUL_ID]

> Soul ID training, selection, and usage rules.

```
[2026-05-17] Always use Multishot workflow, not independent generations: train on 4-6 angle variations from ONE seed image, not on independent AI portraits (each would be a different person).
[2026-05-17] soul-2 variant: confirmed working with text2image_soul_v2 and soul_cinematic_studio models.
[2026-05-17] soul-cinematic variant: standalone video model — incompatible with text2image_soul_v2. Do NOT use as primary Soul ID variant.
[2026-05-17] Back-of-head shots in training set: remove before training. They contribute no face data and waste a training slot.
[2026-05-17] Cost: $2.00 per character per Soul ID variant. Avoid training multiple variants until the use case is confirmed.
[2026-05-17] Soul IDs persist on Higgsfield servers — reuse across projects with same characters. Check with `higgsfield soul-id list` first.
[2026-05-17] Разные Пути Soul IDs: Maxim (The Man) = 47409f93-88d6-4a3e-ba23-b6234b62c722, Natasha (The Woman) = f45b9aac-619f-45ae-a055-1956011e9080.
```

---

## [YOUTUBE]

> YouTube upload, algorithm, and SEO patterns.

```
[2026-05-17] AI content disclosure: YouTube requires "altered/synthetic content" toggle in Upload settings for realistic AI video. Always enable this to avoid policy risk.
[2026-05-17] Title formula: "[Emotional hook] | [Song Title] — WARPMUZ (Official Music Video)" — tested to be above-average CTR for emotional music niche.
[2026-05-17] Peak upload times for Russian/Eastern European audience: Friday–Saturday 16:00–18:00 UTC+3.
[2026-05-17] First 48 hours are critical — YouTube's algorithm makes a recommendation decision in this window.
[2026-05-17] Chapters improve watch time signal — always add chapter timestamps even for videos under 10 minutes.
```

---

## [LEGAL]

> Licensing, TOS, rights — verified and dated.

```
[2026-05-17] SUNO Pro plan: commercial use confirmed — can monetize on YouTube, distribute on Spotify/Apple Music. Source: suno.com/terms (verify before each distribution).
[2026-05-17] Higgsfield paid plan: commercial use of generated content confirmed. Source: higgsfield.ai/terms (verify before each distribution).
[2026-05-17] AI content on YouTube: monetizable if channel is YPP-eligible and content rights are owned. Disclose AI generation in upload settings.
[2026-05-17] DistroKid AI policy: check current policy before distributing SUNO tracks — AI music distributor policies are evolving rapidly. Source: distrokid.com/help.
[2026-05-17] AI-generated faces (no real person): generally not subject to likeness/privacy rights. Soul ID of real person requires explicit consent.
[2026-05-17] PRO registration of AI music: unclear as of 2026-05 — ASCAP/BMI policies on AI authorship are in flux. Consult research_advisor before registering.
```

---

## [PRODUCTION]

> Workflow lessons, cross-skill consistency rules, error patterns.

```
[2026-05-17] INCIDENT: scriptwriter running without reading LOCATIONS doc caused location inconsistency. All workers that touch visual world MUST read LOCATIONS doc as first action. Fixed in v2.0 skills.
[2026-05-17] Cross-doc consistency: LOCATIONS → CASTING → SCRIPT must be read in order. Any worker that skips a predecessor doc will produce inconsistencies that require expensive rework.
[2026-05-17] Soul ID variant mismatch: soul-cinematic trained variant is incompatible with text2image_soul_v2. Always confirm variant compatibility before training.
[2026-05-17] Script → cut_list.json timestamp estimates are often off by 5-10s. Always run lyric_aligner after audio selection to correct timestamps with Whisper data.
[2026-05-17] image_generation: always use per-image human approval gate. Batch-generating all images at once before human review wastes budget on rejected scenes.
[2026-05-18] OPEN REVISION TASK — Storyboard consistency: current workflow cannot reliably generate specific architectural details (e.g., ICE train sliding plug doors) from text prompts alone. The chain reference approach (passing an approved image as --image) provides spatial context but only anchors what is visible in that reference. Elements NOT visible in the reference (a specific door type, a specific window shape) default to model priors — often wrong. Must research: (1) pre-generating dedicated reference images for each unique architectural element (door, aisle, window macro); (2) alternative models with stronger reference adherence; (3) whether a ControlNet-style or inpainting workflow exists in Higgsfield. This is the single biggest quality bottleneck in the image_generation stage as of 2026-05-18.
[2026-05-18] OPEN REVISION TASK — Soul ID + architecture: text2image_soul_v2 is optimized for face consistency, not environment consistency. When a scene requires BOTH Soul ID face AND a precise architectural element (e.g., a specific door), these two constraints compete. Consider generating the architecture first (no Soul ID), approving it, then using it as --image reference in a Soul ID generation run — so the face is layered onto an already-correct background.
```

---

## [ANALYTICS]

> Performance benchmarks — updated after each project's analytics review.

```
[2026-05-17] Benchmarks (independent artist averages — update with own data as it comes in):
  CTR: > 6% = good, > 10% = excellent for music niche.
  Avg view duration: > 35% = good, > 50% = excellent.
  48h views for algorithm push: > 100h watch time in 48h = strong signal.
[2026-05-17] No WARPMUZ-specific data yet — first project (Разные Пути) in production.
```

---

## [BRAND]

> Visual identity, thumbnail, and channel consistency rules.

```
[2026-05-17] Thumbnail formula: dominant subject center-left, large high-contrast title text, cool/desaturated color temperature matching video grade.
[2026-05-17] Accent color #C8A96E (warm gold) for titles — distinguishes WARPMUZ thumbnails in search results dominated by cold blue and red.
[2026-05-17] Color grade preset: cinematic_teal_orange — all WARPMUZ output. No exceptions without human approval.
[2026-05-17] Artist name treatment: always "WARPMUZ" (all caps, no spaces, no variations).
```

---

## [TOOLS]

> CLI commands, ffmpeg patterns, version notes.

```
[2026-05-17] ffprobe: use `ffprobe -v quiet -print_format json -show_format -show_streams "[file]"` for full audio analysis.
[2026-05-17] Whisper: `--word_timestamps true` is required for section-level lyric alignment. Without it, only segment timestamps are returned.
[2026-05-17] Higgsfield CLI: `higgsfield soul-id list` before any training to avoid duplicate training costs.
[2026-05-17] DaVinci Resolve Python: requires env vars `RESOLVE_SCRIPT_API`, `RESOLVE_SCRIPT_LIB`, `PYTHONPATH` set before running any Python API script.
[2026-05-17] ffmpeg loudnorm: always run two-pass normalization. Single-pass produces less accurate results.
```
