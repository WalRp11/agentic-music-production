# Casting Director — AI Skill v1.0

**Role:** Defines characters and visual identities for the music video — decides whether a recurring AI character (Soul ID) is needed, defines voice/performance style, and sources or specifies reference imagery.
**Phase:** Pre-Production
**Inputs:** `Lyrics/CONCEPT_[project_id].md`, `PROJECT_STATE.md`
**Outputs:** `Lyrics/CASTING_[project_id].md` — character dossiers and Soul ID decision
**Tools:** Higgsfield CLI (Soul ID path only), file system
**Human Gate:** NO — auto-generated; CEO routes to soul_identity skill if Soul ID flagged as required
**Reads from PROJECT_STATE:** `genre`, `mood`, `decisions.soul_id_required`, `decisions.soul_id_reference_man`, `decisions.soul_id_reference_woman`
**Writes to PROJECT_STATE:** `decisions.soul_id_required`, `decisions.soul_id_reference_man`, `decisions.soul_id_reference_woman`, `current_stage → locations`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Purpose

The casting brief answers: *who is in this video?* This determines whether we need a consistent human face (Soul ID), abstract/faceless visuals, or a combination. This decision has direct cost implications (Soul ID training costs ~$2.00) and pipeline implications (soul_identity skill inserts itself before image_generation if flagged).

---

## Step 1 — Read the Concept Brief

Load `Lyrics/CONCEPT_[project_id].md`. Find:
- Protagonist description (is there a visible face/person?)
- Emotional arc (does the protagonist appear throughout, or is the video purely visual/abstract?)
- Visual style (cinematic narrative vs. abstract vs. lyric video)

---

## Step 2 — Soul ID Decision

Apply this decision tree:

```
Does the concept require a consistent human face or performer to appear in multiple scenes?
  YES → Does the human have a specific person/face they want to use?
    YES → Soul ID required. Set soul_id_required = true. Get reference photo from human.
    NO  → Can the video work with different stylized faces per scene (no continuity)?
            YES → No Soul ID. Use character description only.
            NO  → Soul ID required. Human must provide a reference photo.
  NO  → No Soul ID. Abstract / location-driven visuals only.
```

**Ask the human only if the decision tree is ambiguous:**
```
The concept mentions [protagonist type]. For the music video:
Option A: Use a consistent AI character (same face throughout) — requires Soul ID training (~$2.00, one-time)
Option B: Abstract/atmospheric visuals — no consistent face, more artistic freedom, no extra cost
Option C: Stylized different characters per scene — varied looks, no continuity

Which do you prefer?
```

---

## Step 3 — Write Casting Brief

Output `Lyrics/CASTING_[project_id].md`:

```markdown
# Casting Brief — [Song Title]

## Soul ID Decision
Required: [YES / NO]
Reference: [path to reference photo OR "not provided" OR "N/A"]
Reasoning: [why this decision was made]
Cost impact: [$2.00 one-time training / $0]

---

## Character Dossiers

### Character 1: [Name or "The Protagonist"]
Role in story: [e.g. "The person looking back at a lost relationship"]
Age appearance: [e.g. "late 20s to mid 30s"]
Visual description: [e.g. "Dark hair, angular features, tired eyes, stubble. Dressed in worn dark coat."]
Emotional state across arc:
  - Beginning: [e.g. "Quietly reflective, still, controlled"]
  - Middle: [e.g. "Visibly strained, isolated"]
  - End: [e.g. "At peace, slightly distant"]
Screen presence: [Prominent / Background / Implied / None]
Higgsfield prompt fragment: "[copy-paste fragment for image_generator to append to scene prompts]"

### Character 2: [if applicable]
[same structure]

---

## Voice / Vocal Style Notes
(Fed to audio_engineer for take selection criteria)
Preferred vocal character: [e.g. "Husky, controlled, not operatic — think Nick Cave, quiet intensity"]
Language accent consideration: [e.g. "Neutral American English preferred" or "Native [language] accent"]
Backing vocals: [YES — [description] / NO]

---

## Visual Identity Notes
(Fed to image_generator for style consistency)
Wardrobe palette: [e.g. "Desaturated earth tones — charcoal, olive, dark rust. No bright colors."]
Hair & makeup direction: [e.g. "Natural, no heavy makeup — real, lived-in look"]
Era / setting: [e.g. "Contemporary, slightly timeless — no visible logos or dated elements"]

---

## Reference Images
[List any reference photos, films, or artists provided by human, or "none provided"]
```

---

## Step 4 — Update PROJECT_STATE

```yaml
decisions:
  soul_id_required: [true/false]
  soul_id_reference_man: "[Higgsfield Soul ID string for male character, or empty]"
  soul_id_reference_woman: "[Higgsfield Soul ID string for female character, or empty]"
  # Note: leave empty if Soul ID is not yet trained — soul_identity skill will fill this in
current_stage: "locations"
next_action: "Run location_scout skill."
```

If `soul_id_required = true` and both references are empty:
```yaml
blocked: true
blocked_reason: "Soul ID required but not yet trained. CEO will insert soul_identity skill before image_generation."
next_action: "Proceed to locations stage. Soul ID training will run after audio_production."
```
