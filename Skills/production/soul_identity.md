# Soul Identity — AI Skill v1.0

**Role:** Trains a Higgsfield Soul ID from a human-provided portrait photo, and provides the usage rules for maintaining visual character consistency across all image and video generation.
**Phase:** Production (optional — activates only when casting_director sets soul_id_required = true)
**Inputs:** Reference portrait photo (in `Brand/assets/`), `PROJECT_STATE.md`
**Outputs:** Soul ID reference string (stored in `decisions.soul_id_reference`), `Brand/assets/soul_identity_guide.md`
**Tools:** Higgsfield CLI (`higgsfield soul train`, `higgsfield soul list`)
**Human Gate:** YES — human provides the photo and confirms the trained Soul ID looks correct
**Reads from PROJECT_STATE:** `decisions.soul_id_required`, `decisions.soul_id_reference`
**Writes to PROJECT_STATE:** `decisions.soul_id_reference`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## When This Skill Runs

Only when `decisions.soul_id_required = true` AND `decisions.soul_id_reference` is empty.
The CEO orchestrator inserts this skill between the `casting` and `image_generation` stages.

If `decisions.soul_id_reference` is already set (Soul ID previously trained), skip training and go directly to Step 4 (usage guide).

---

## Step 1 — Get the Reference Photo

Check `Brand/assets/` for a portrait photo. If not found:

```
⚠️ SOUL ID SETUP REQUIRED

The casting brief requires a consistent AI character throughout this video.
I need a portrait photo to train the Soul ID.

Please provide:
  1. A clear frontal portrait photo of the person
  2. File: JPEG or PNG, minimum 512×512 pixels
  3. Requirements: single face visible, neutral expression preferred,
     good lighting (no harsh shadows), no sunglasses
  4. Place the file at: Brand/assets/soul_reference.jpg

Type READY when the file is in place.
```

---

## Step 2 — Train the Soul ID

```bash
# Check available Soul IDs first
higgsfield soul list

# Train new Soul ID
higgsfield soul train \
  --image "Brand/assets/soul_reference.jpg" \
  --name "[artist_name]_soul" \
  --output-id
```

Wait for training to complete (typically 2–5 minutes). Record the returned Soul ID string.

Log the cost: Soul ID training = ~50 credits / ~$2.00 (one-time).

---

## Step 3 — Human Confirmation

Generate a test image with the new Soul ID to verify quality:

```bash
higgsfield generate image \
  --model text2image_soul_v2 \
  --soul-id "[returned_soul_id]" \
  --prompt "portrait, neutral background, soft studio lighting, looking at camera, cinematic" \
  --output "Brand/assets/soul_test.png" \
  --size 1024x1024
```

Present to human:
```
✅ SOUL ID TRAINED — Test image generated

Soul ID: [soul_id_string]
Test image: Brand/assets/soul_test.png

Does this look correct?
  - YES → Soul ID will be used for all character images in this project
  - NO → Please provide a better reference photo and we will retrain
  - PARTIAL → Tell me what's wrong (lighting, angle, etc.) and I'll try a different test prompt
```

---

## Step 4 — Soul ID Usage Guide

Write `Brand/assets/soul_identity_guide.md` for the image_generator and video_generator skills:

```markdown
# Soul Identity Usage Guide

## Soul ID Reference
ID: [soul_id_string]
Trained from: Brand/assets/soul_reference.jpg
Training date: [date]
Cost: $2.00 (one-time)

## How to Use in Image Generation
Always add these flags when the character must appear:
  --model text2image_soul_v2
  --soul-id [soul_id_string]

## What the Soul ID Controls
The Soul ID controls: facial features, facial proportions, skin tone.
You still control: clothing, hairstyle, expression, pose, lighting, background.

## Prompt Writing Rules with Soul ID
✅ DO describe: clothing, pose, expression, setting, lighting, camera angle
❌ DON'T describe: facial features (nose shape, eye color, etc.) — Soul ID handles this
❌ DON'T use: another person's name or describe someone else's face

## Consistent Look Across Scenes
For this project, the character should always appear in:
  Wardrobe: [from casting brief — e.g. "dark coat, no visible brand logos"]
  Hairstyle: [from casting brief]
  Consistent age appearance: [from casting brief]

## Scenes Where Soul ID is NOT Used
Abstract or location-only scenes (no character present) should use gpt_image_2 instead.
The image_generator skill handles this routing automatically based on the scene's casting flag.
```

---

## Step 5 — Update PROJECT_STATE

```yaml
decisions:
  soul_id_reference: "[soul_id_string]"
spend_log:
  - worker: "soul_identity"
    action: "Soul ID training"
    units: 50
    cost_usd_estimate: 2.00
```

The CEO orchestrator then routes to `image_generation` as normal. The `image_generator` skill reads `soul_id_reference` and applies it to scenes flagged as having the character present.

---

## Reusing Soul ID Across Projects

The Soul ID is stored on Higgsfield's servers and tied to your account. On future projects:
1. Set `decisions.soul_id_reference` to the same Soul ID string in the new `PROJECT_STATE.md`
2. Skip Soul ID training
3. The image_generator will use the existing Soul ID directly

**To find existing Soul IDs:**
```bash
higgsfield soul list
```

The cost is one-time per training. Reuse is free.
