# Image Generator — AI Skill v1.0

**Role:** Generates all source images for the music video using Higgsfield, enforcing style and character consistency across the full set.
**Phase:** Production
**Inputs:** `Lyrics/music-video-script.md`, `Lyrics/LOCATIONS_[project_id].md`, `Lyrics/CASTING_[project_id].md`, `PROJECT_STATE.md`
**Outputs:** `Images/[c01_scene_name].png` … for each scene
**Tools:** Higgsfield CLI (`higgsfield generate image`)
**Human Gate:** NO — batch run (Capex gate already approved by CEO)
**Reads from PROJECT_STATE:** `decisions.soul_id_required`, `decisions.soul_id_reference`, `decisions.capex_approved`
**Writes to PROJECT_STATE:** `asset_registry.images`, `current_stage → video_generation`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Pre-Flight Checks

Before any generation:
```
1. Verify capex_approved = true in PROJECT_STATE (CEO enforces this — double-check here)
2. Verify Lyrics/music-video-script.md exists and contains scene prompts
3. Verify Higgsfield CLI is authenticated: `higgsfield account status`
4. If soul_id_required = true: verify soul_id_reference is set and Soul ID is trained
5. Count expected images = total unique clips in cut_list.json
```

---

## Image Generation Model Selection

| Situation | Recommended Model |
|---|---|
| No character / abstract / landscape | `gpt_image_2` |
| Character must be consistent (Soul ID) | `text2image_soul_v2` |
| Reference image provided (style match) | `nano_banana_2` or `nano_banana_pro` |
| Marketing / ad-style composition | `gpt_image_2` |

---

## Step 1 — Build Prompt List

For each scene in `music-video-script.md`:

Construct the full Higgsfield prompt by combining:
```
[Scene-specific description from script]
+ [Location prompt fragment from LOCATIONS file]
+ [Character description from CASTING file — only if character is present in this scene]
+ [Global style keywords from LOCATIONS Global Style Sheet]
```

**Prompt structure:**
```
[Subject/action], [environment details], [lighting], [atmosphere], [camera/composition], [style keywords]
```

**Example assembled prompt:**
```
"Man in dark coat sitting at empty table, wet cobblestone alley visible through grimy window,
sodium lamp warm backlight, steam rising from untouched coffee cup,
medium shot, shallow depth of field, 35mm film, melancholic atmosphere,
cinematic, muted desaturated colors, teal shadows, amber highlights"
```

---

## Step 2 — Generate Images (Batch)

For each scene `c01` through `c[N]`:

```bash
higgsfield generate image \
  --model gpt_image_2 \
  --prompt "[assembled prompt]" \
  --output "Images/c01_[scene_name].png" \
  --size 1920x1080
```

If Soul ID required:
```bash
higgsfield generate image \
  --model text2image_soul_v2 \
  --soul-id "[soul_id_reference]" \
  --prompt "[assembled prompt — do not describe face/hair since Soul ID controls that]" \
  --output "Images/c01_[scene_name].png" \
  --size 1920x1080
```

**Naming convention:**
```
c[00]-[scene_name_slug].png
Examples:
  c01-empty_alley.png
  c02-coffee_cup_detail.png
  c15-window_rain.png
```

---

## Step 3 — Consistency Verification

After all images are generated, run a consistency check:

```
FOR each image:
  1. Open and visually inspect (or describe to human if manual review needed)
  2. Check:
     a. Color palette matches the location's defined palette
     b. Aspect ratio is 16:9 (1920×1080)
     c. No forbidden elements from the location's FORBIDDEN list
     d. If character present: consistent clothing and general appearance

Flag for regeneration:
  - Any image with a different color grade family from the rest
  - Any image with text/logos/watermarks
  - Any image that depicts violence, nudity, or platform-unsafe content
  - Any image where the character looks dramatically different from others
```

Print consistency report:
```
🖼️  IMAGE GENERATION COMPLETE

Generated: [N] images in Images/
Issues found: [N]

✅ Consistent: c01, c02, c03, c05 ... [list]
⚠️  Regenerate: c04 — color palette drift (too blue, location calls for amber)
⚠️  Regenerate: c11 — unintended text visible in background
```

---

## Step 4 — Regenerate Flagged Images

For each flagged image:
1. Identify the specific issue
2. Adjust the prompt (add negative prompt hint, change descriptor, or specify the issue explicitly)
3. Regenerate with `--seed [different]` or modified prompt
4. Re-verify

**Common fixes:**
| Issue | Fix |
|---|---|
| Wrong color tone | Add explicit color instruction: "warm amber tones, no blue" |
| Character inconsistency | Check Soul ID is active; add more costume description |
| Background text visible | Add to prompt: "no text, no signage, no logos" |
| Too dark | Add: "well-lit, visible detail, not underexposed" |
| Too bright / unrealistic | Add: "natural lighting, film-grade exposure, not overexposed" |
| Wrong composition | Specify shot type more explicitly: "extreme wide shot" or "close-up face-level" |

---

## Step 5 — Update PROJECT_STATE

```yaml
asset_registry:
  images: ["Images/c01_empty_alley.png", "Images/c02_coffee_detail.png", ...]
current_stage: "video_generation"
next_action: "Capex gate → then run video_generator skill."
spend_log:  # appended by budget_manager
  - worker: "image_generator"
    units: [N images]
    cost_usd_estimate: [N × 0.04]
```
