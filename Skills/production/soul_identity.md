# Soul Identity — AI Skill v1.1

**Role:** Guides the human to build a consistent AI character using Higgsfield Multishot, then trains a Soul ID from the resulting reference photos for use across all image and video generation.
**Phase:** Production (optional — activates only when casting_director sets soul_id_required = true)
**Inputs:** `Brand/assets/soul_id_[character]/` folder (populated by human), `PROJECT_STATE.md`
**Outputs:** Soul ID reference string (stored in `decisions.soul_id_reference`), `Brand/assets/soul_identity_guide.md`
**Tools:** Higgsfield CLI (`higgsfield soul-id create`, `higgsfield soul-id list`)
**Human Gate:** YES — the human MUST build the character themselves in Higgsfield. The AI cannot reliably generate consistent faces across multiple independent image generations.
**Reads from PROJECT_STATE:** `decisions.soul_id_required`, `decisions.soul_id_reference_man`, `decisions.soul_id_reference_woman`
**Writes to PROJECT_STATE:** `decisions.soul_id_reference_man`, `decisions.soul_id_reference_woman`
**Version History:**
  v1.0 — Initial release (2026-05-17)
  v1.1 — Rewrote character-building procedure to require human Multishot workflow (2026-05-17)

---

## Critical Principle: Why the Human Must Build the Character

> **Do NOT generate 5 independent AI portrait images and use them directly as Soul ID training data.**
>
> Each independent AI generation produces a *different person*. The faces will be inconsistent — different bone structure, different features — even if the prompt is identical. Soul ID trained on inconsistent faces produces unreliable identity matching.
>
> The correct workflow is: generate ONE seed image → human uses Higgsfield Multishot to create consistent angle variations from that one face → human selects best shots → train Soul ID.

---

## When This Skill Runs

Only when `decisions.soul_id_required = true` AND (`soul_id_reference_man` is empty OR `soul_id_reference_woman` is empty for characters that appear in the casting brief).
The CEO orchestrator inserts this skill between the `casting` and `image_generation` stages.

If both `decisions.soul_id_reference_man` and `decisions.soul_id_reference_woman` are already set (Soul IDs previously trained), skip to Step 5 (usage guide).
If only one is set, run only for the missing character.

---

## Step 1 — Generate One Seed Portrait (AI Task)

Generate a single high-quality seed portrait using the character description from the casting brief. Use `gpt_image_2`, 2k, 1:1.

```bash
higgsfield generate create gpt_image_2 \
  --prompt "[character description from casting brief — age, hair, expression, clothing, lighting]" \
  --aspect_ratio 1:1 --resolution 2k --wait
```

Download the result. Show it to the human.

Ask:
```
🎭 SEED PORTRAIT — Character: [character name]

Here is the AI-generated seed image for [character].
Does this face feel right for the role?

  YES → I will save this as the seed and you will create Multishot variations in Higgsfield
  NO  → Describe what to change (age, hair color, expression, etc.) and I'll regenerate
  PARTIAL → Tell me what to adjust
```

Save approved seed as: `Brand/assets/soul_id_[character]/seed.png`

---

## Step 2 — Human Builds the Character (Human Task — REQUIRED)

This step CANNOT be automated. The human must do this in the Higgsfield web app.

Tell the human:

```
🎬 YOUR TURN — Build the character in Higgsfield

The AI cannot generate consistent faces across multiple shots.
You need to use Higgsfield's Multishot feature to create angle variations
from the seed image — these will all be the same person.

Steps:
  1. Open app.higgsfield.ai in your browser
  2. Upload the seed image: Brand/assets/soul_id_[character]/seed.png
  3. Use the Multishot function to generate 4–8 angle variations
     (front, three-quarter, side profile, downward gaze, etc.)
  4. Upscale the best 3–5 shots using Higgsfield's upscale function
  5. Download the upscaled images
  6. Place them in: Brand/assets/soul_id_[character]/
     Name them: [character]_ref_01.png, [character]_ref_02.png, etc.
  7. Delete any shots that show the BACK of the head (no face data = wasted training slot)
  8. Tell me: "Soul ID photos for [character] are ready"

Tips for best Soul ID results:
  • Include: front-facing, side profile, three-quarter angle
  • 4–6 images is sufficient — more is not always better
  • All must show the face clearly
  • Avoid back-of-head shots — they contribute no face data to training
```

STOP. Wait for the human to complete this step and confirm.

---

## Step 3 — Review and Clean Up (AI Task)

When human confirms photos are ready:

1. List all files in `Brand/assets/soul_id_[character]/`
2. Show thumbnail previews to the human
3. Check: are all images showing the same face? Is any image a back-of-head shot?

If a back-of-head or non-face image is found:
```
⚠️ man_ref_XX.png appears to be a back-of-head shot.
This won't contribute face data to Soul ID training.
Recommend removing it. Shall I delete it?
```

4. Remove the seed.png from the training set (it was just the starting point)
5. Confirm final list with the human before proceeding

---

## Step 4 — Train the Soul ID (AI Task)

```bash
# List existing Soul IDs first (avoid duplicates)
higgsfield soul-id list

# Train new Soul ID — use --soul-cinematic for video projects
higgsfield soul-id create \
  --name "[CharacterName]" \
  --soul-cinematic \
  --image "Brand/assets/soul_id_[character]/[character]_ref_01.png" \
  --image "Brand/assets/soul_id_[character]/[character]_ref_02.png" \
  --image "Brand/assets/soul_id_[character]/[character]_ref_03.png" \
  --image "Brand/assets/soul_id_[character]/[character]_ref_04.png"

# Wait for training
higgsfield soul-id wait [returned_id]
```

Training typically takes 5–15 minutes. Cost: ~50 credits / ~$2.00 per character (one-time).

---

## Step 5 — Human Confirmation

Generate a test image with the new Soul ID:

```bash
higgsfield generate create soul_cinematic \
  --soul-id "[returned_soul_id]" \
  --prompt "portrait, neutral background, soft studio lighting, looking at camera, cinematic, 35mm" \
  --quality 2k --wait
```

Present to human:
```
✅ SOUL ID TRAINED — Test image generated

Character: [character name]
Soul ID: [soul_id_string]

Does this look like the same person from your reference photos?
  YES → Soul ID confirmed — proceeding to image generation
  NO  → Please provide better reference photos or we can retrain
```

---

## Step 6 — Write Soul Identity Guide

Write `Brand/assets/soul_identity_guide.md`:

```markdown
# Soul Identity Usage Guide

## Characters

### [Character Name]
Soul ID: [soul_id_string]
Variant: soul-cinematic
Training date: [date]
Reference photos: Brand/assets/soul_id_[character]/
Cost: $2.00 (one-time)

## How to Use in Image Generation
  --soul-id [soul_id_string]
  Model: soul_cinematic (for stills and video)

## Prompt Rules with Soul ID
✅ DO describe: clothing, pose, expression, setting, lighting, camera angle
❌ DON'T describe: facial features (nose, eye color, etc.) — Soul ID handles this

## Consistent Wardrobe (from casting brief)
  [Wardrobe details from casting brief]

## Scenes Where Soul ID is NOT Used
Abstract or location-only scenes (no character present) → use gpt_image_2
```

---

## Step 7 — Update PROJECT_STATE

```yaml
decisions:
  soul_id_reference_man: "[soul_id_string]"    # set this if character is male
  soul_id_reference_woman: "[soul_id_string]"  # set this if character is female
  # If only one character: set the relevant field and leave the other empty
spend_log:
  - worker: "soul_identity"
    action: "Soul ID training — [character name]"
    units: 50
    cost_usd_estimate: 2.00
```

---

## Reusing Soul ID Across Projects

The Soul ID is stored on Higgsfield's servers. On future projects:
1. Set `decisions.soul_id_reference` to the existing Soul ID string in `PROJECT_STATE.md`
2. Skip training
3. Image generator uses it directly

```bash
higgsfield soul-id list   # find existing Soul IDs
```
