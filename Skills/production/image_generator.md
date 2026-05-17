# Image Generator — AI Skill v1.2

**Role:** Generates all source images for the music video using Higgsfield, enforcing strict visual coherence across all scenes that share the same location.
**Phase:** Production
**Inputs:** `Lyrics/music-video-script.md`, `Lyrics/LOCATIONS_[project_id].md`, `Lyrics/CASTING_[project_id].md`, `PROJECT_STATE.md`
**Outputs:** `Images/storyboard/[c01_scene_name].png` … for each scene
**Tools:** Higgsfield CLI (`higgsfield generate create`), Higgsfield Web UI (for Soul HEX panel and Popcorn multi-frame)
**Human Gate:** YES — human must approve EACH image individually before the next image is generated. Never generate the next scene without explicit approval of the current one.
**Reads from PROJECT_STATE:** `decisions.soul_id_required`, `decisions.soul_id_reference_man`, `decisions.soul_id_reference_woman`, `decisions.capex_approved`
**Writes to PROJECT_STATE:** `asset_registry.images`, `current_stage → video_generation`
**Version History:**
  v1.0 — Initial release (2026-05-17)
  v1.1 — Added location coherence rules, reference image workflow, storyboard-first gate (2026-05-17)
  v1.2 — Added Style Lock Phrase enforcement, Soul HEX colour anchoring, Soul Cinema model for close-ups, Popcorn multi-frame workflow (2026-05-17)

---

## CRITICAL RULE: Location Coherence

> **All scenes set in the same location MUST look like the same physical space.**
>
> This is non-negotiable. If the man is on a German ICE train in c03, every subsequent interior scene (c04, c05, c06, c07…) must show the EXACT SAME train interior — same seat color, same wall color, same lighting fixtures, same window shape, same door design.
>
> Failure: showing an ICE interior in one scene and a Soviet-era train interior in the next destroys the story's spatial logic and will require full regeneration.

### How to enforce coherence

**Step 1 — Identify location groups.**
Before generating anything, group all scenes by location:
```
Group A: Night Platform (exterior) → c01, c26, c27, c28
Group B: ICE Train Interior → c03, c04, c05, c06, c07, c08, c10, c13, c14, c16, c20, c21, c22, c23, c25
Group C: Train Window Close-up → c02, c09, c12, c17, c19, c29
Group D: Night City Through Glass → c11, c15, c18, c24
```

**Step 2 — Find or generate a base reference image for each location.**

For architecturally specific interiors (train cars, stations, rooms):
- Search Wikimedia Commons or free stock for a real photograph of the exact space type
- Download and save to `Images/[location_name]_reference.jpg`
- This is the structural ground truth — seat count, window shape, door style, wall color

For exterior or abstract locations:
- Generate one establishing shot first (e.g., c01 for the platform)
- Human approves it
- Use that approved image as `--image` reference for all subsequent scenes in the same location

**Step 3 — Generate the first scene of each location group using the reference image.**
```bash
higgsfield generate create gpt_image_2 \
  --prompt "[scene description]" \
  --image "[path to reference photo]" \
  --aspect_ratio 16:9 --resolution 2k --wait
```
Pass the reference via `--image`. Higgsfield uses it as a compositional anchor while applying the prompt's lighting/atmosphere.

**Step 4 — Human approves the base image.**
Do NOT generate any other scenes in that location group until the base image is approved. The base image becomes the visual contract for all scenes in that group.

**Step 5 — Use the approved base image as reference for all subsequent scenes in the same group.**
```bash
# For a no-character scene in the same interior:
higgsfield generate create gpt_image_2 \
  --prompt "[scene-specific description]" \
  --image "[path to approved base image]" \
  --aspect_ratio 16:9 --resolution 2k --wait

# For a Soul ID character scene in the same interior:
higgsfield generate create text2image_soul_v2 \
  --soul-id "[soul_id]" \
  --prompt "[scene-specific description]" \
  --image "[path to approved base image]" \
  --aspect_ratio 16:9 --quality 2k --wait
```

**Step 6 — Visual review before saving.**
Before saving any generated image, verify:
- Same seat/furniture design as the base image
- Same window shape and placement
- Same door style (ICE sliding glass door ≠ old compartment door with oval windows)
- No missing or malformed furniture (check seat count per row matches reference)
- No architectural errors (diagonal window frames, impossible geometry)

If the image fails any check: regenerate with a more explicit structural description or add `--image [reference]` again.

---

## Pre-Flight Checks

Before any generation:
```
1. capex_approved = true in PROJECT_STATE
2. Lyrics/music-video-script.md exists and contains scene prompts
3. Higgsfield CLI authenticated: higgsfield account status
4. If soul_id_required = true: soul_id_reference_man and soul_id_reference_woman are set
5. Real reference photos downloaded for architecturally specific locations
6. Location groups mapped (see above)
```

---

## Model Selection

| Situation | Model | Quality flags |
|---|---|---|
| No character / abstract / atmosphere / window close-up | `imagegen_2_0` (GPT Image 2 Unlimited) | `--quality high --resolution 4k` |
| Character with Soul ID, medium/wide shot | `text2image_soul_v2` | `--quality 2k` |
| Character with Soul ID + location reference image | `text2image_soul_v2` with `--image [reference]` | `--quality 2k` |
| **Character close-up / emotional beat / the smile** | **`soul_cinema_studio`** | `--quality 2k` |
| Any scene needing structural reference | `imagegen_2_0` with `--image [reference]` | `--quality high --resolution 4k` |

**Unlimited model:** Use `imagegen_2_0` (not `gpt_image_2`) for all non-Soul-ID scenes. This maps to the "GPT Image 2 Unlimited" tier in the Higgsfield web UI and does not consume credits on the Ultra plan.

**`soul_cinema_studio` — when to use:** Close-up portraits, intimate emotional shots, and any scene the Creative Designer flagged as `narrative_beat: emotional_peak` or `narrative_beat: turning_point`. Soul Cinema produces cinematic-grade grain, shallow depth of field, and richer spontaneous texture. It is NOT for wide or environmental shots.

**Note on `text2image_soul_v2` + `--image`:** Pass the location base image via `--image` to anchor the environment. The Soul ID controls the face; the `--image` controls the space. Both work together.

---

## Soul ID Usage

Always use `--soul-2` trained Soul IDs with `text2image_soul_v2`:
- Man: `--soul-id [soul_id_reference_man from PROJECT_STATE]`
- Woman: `--soul-id [soul_id_reference_woman from PROJECT_STATE]`
- Do NOT use `--soul-cinematic` trained IDs with `text2image_soul_v2` — they are incompatible

When both characters appear in a scene:
- Generate two versions (one anchored to each Soul ID) and composite, OR
- Use the dominant character's Soul ID and describe the other character by appearance (they appear in soft focus/distance anyway)

---

## Storyboard Generation Order

Generate in this order to minimize regeneration:

1. **Platform exterior base** (c01) → approve → use as reference for c26, c27, c28
2. **ICE interior base** (c03 corrected) → approve → use as reference for ALL other interior scenes
3. **Window close-up base** (c02) → approve → use as reference for c09, c12, c17, c19, c29
4. **Abstract city-through-glass** (c11) → approve → use as reference for c15, c18, c24
5. Then generate all remaining scenes in cut-list order, passing the approved base for their location group

---

## Style Lock Phrase (mandatory — read from LOCATIONS doc)

Every single prompt MUST begin with the project-wide Style Lock Phrase defined in `LOCATIONS_[project_id].md`. This phrase is identical for every scene; it is the visual fingerprint of the film.

```
[STYLE LOCK PHRASE] + [Subject + action] + [Location details] + [Lighting] + [Camera spec] + [Clone guard if multiple characters]
```

Example mandatory prompt structure:
```
cinematic, 35mm film grain, shallow depth of field, desaturated warm Russian winter palette,
teal shadows amber highlights, ARRI ALEXA aesthetic, slight film halation on highlights,
[then scene-specific content]
```

**Never omit the Style Lock Phrase.** A prompt without it will produce inconsistent visual style that cannot be recovered without full regeneration.

---

## Soul HEX Colour Anchoring

Each location group has a Soul HEX palette defined in `LOCATIONS_[project_id].md`. Apply it in two ways:

**CLI method (embed in prompt text):**
```bash
higgsfield generate create imagegen_2_0 \
  --prompt "[STYLE LOCK PHRASE], [scene description], colour palette #1A1A2E #E8A87C #2C3E50 #F0E6D3" \
  --aspect_ratio 16:9 --resolution 4k --wait
```

**Web UI method (more precise):** After generating via CLI, open the image in Higgsfield web UI → Soul HEX panel → paste the hex codes for that location group → re-generate with locked palette. This is the preferred method for the first image of each location group.

Document all Soul HEX codes per location group in `PROJECT_STATE.md` under `soul_hex_palettes`.

---

## Prompt Assembly

Combine in this exact order:
```
1. [STYLE LOCK PHRASE verbatim from LOCATIONS doc]
2. [Subject + action]
3. [Location details from LOCATIONS doc]
4. [Lighting spec from LOCATIONS doc]
5. [Camera spec from script]
6. [Clone guard phrase if 2+ characters: "one person only visible in foreground" etc.]
```

Do NOT describe facial features when using Soul ID. Describe clothing, pose, expression, position only.

**Soul HEX:** Embed the location group's hex codes at the end of step 3 (location details).

---

## Popcorn Multi-Frame Workflow (for location groups with 3+ scenes)

For any location group with three or more scenes, use **Popcorn** in the Higgsfield web UI after the base image is approved. Popcorn generates multiple frames with scene-aware continuity — it tracks characters and environment across frames, preserving spatial consistency far better than sequential single-image generation.

**How to use Popcorn:**
1. Open Higgsfield web UI → Popcorn tool
2. Set the approved base image as the first reference (up to 4 reference inputs per frame)
3. Write a separate prompt for each frame (each scene in the location group)
4. Use Auto Mode for exploratory passes; Manual Mode when exact shot composition matters
5. Export approved frames to `Images/storyboard/` with correct naming convention

**Popcorn is the preferred tool** for all ICE train interior scenes (Group B: 15 scenes). Using sequential single-image generation for that many scenes in the same space will produce drift. Popcorn's continuity engine prevents it.

---

## Human Approval Gate

After every scene image is generated:
1. Show image to human
2. Wait for explicit approval, correction, or rejection
3. Only after approval: proceed to next scene
4. Keep a running approval log in PROJECT_STATE human_log

**Do not batch-generate and batch-approve.** Show each scene as soon as it's ready.

---

## Naming Convention

```
Images/storyboard/c[NN]_[scene_name_slug].png

Examples:
  c01_rain_platform_establishing.png
  c04_the_man_waiting.png
  c22_the_smile.png
```

---

## Update PROJECT_STATE on completion

```yaml
asset_registry:
  images: ["Images/storyboard/c01_rain_platform_establishing.png", ...]
current_stage: "video_generation"
next_action: "Human approves all storyboard images → run video_generator skill."
spend_log:
  - worker: "image_generator"
    units: 29
    cost_usd_estimate: ~1.20
```
