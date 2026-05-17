# Location Scout — AI Skill v1.0

**Role:** Defines the visual world of the music video — locations, environments, color palettes, and lighting moods — as a text-based mood board that feeds directly into the image_generator and scriptwriter skills.
**Phase:** Pre-Production
**Inputs:** `Lyrics/CONCEPT_[project_id].md`, `Lyrics/CASTING_[project_id].md`, `Brand/color_palette.json`
**Outputs:** `Lyrics/LOCATIONS_[project_id].md` — visual world document
**Tools:** None (pure creative reasoning)
**Human Gate:** NO — auto-generated from concept
**Reads from PROJECT_STATE:** `genre`, `mood`, `decisions.color_grade_style`
**Writes to PROJECT_STATE:** `decisions.color_grade_style`, `current_stage → script`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Purpose

In a real film production, the location scout finds physical places. Here, the "locations" are fully defined in text and used as the ground truth for AI image generation prompts. Every scene in the music video traces back to one of the locations defined in this document.

The quality of image generation is directly proportional to the specificity of this document.

---

## Step 1 — Map Emotional Arc to Environments

Read the concept brief's emotional arc (Beginning / Middle / End) and map each phase to a distinct visual environment.

```
FOR each act in emotional arc:
  Define:
    1. Location type (interior/exterior, urban/nature/abstract)
    2. Time of day / lighting condition
    3. Season / weather
    4. State of the space (pristine/decayed/active/abandoned)
    5. Camera-feel (intimate close-up space vs. vast empty space)
```

**Rule: Each act should have a visually distinct environment.** Do not repeat the same location across all three acts unless the point is that the character is trapped in the same place while everything changes around them.

---

## Step 2 — Define Location Dossiers

For each location (typically 2–5 for a 3–4 minute video), write a dossier in this format:

```markdown
## Location [N]: [Short Name]
Used in: [Act / Section(s) — e.g. "Verse 1, Chorus 1"]
Type: [Interior / Exterior / Abstract]

### Environment Description
[3–5 sentences. Be precise. Describe what a camera would see:
architecture, surfaces, objects present, signs of use or abandonment.
Example: "A narrow urban alley at 3am. Wet cobblestones reflecting neon from an unseen sign.
A single overflowing trash can. Fire escape ladders ascending out of frame. No people."]

### Lighting
[Natural / Artificial / Mixed]
Quality: [e.g. "single overhead sodium streetlamp creating harsh top-light with deep shadows"]
Color temperature: [e.g. "3200K warm orange, with blue moonlight from above"]

### Color Palette
Primary: [hex or color name]
Secondary: [hex or color name]
Accent: [hex or color name]
Avoid: [any colors that clash with brand palette or mood]

### Emotional Function
[What this environment communicates emotionally.
Example: "The alley represents isolation in a city that is full of people but offers no connection."]

### Higgsfield Prompt Fragment
[The exact phrase to append to every image/video prompt when shooting this location.
Example: "narrow wet cobblestone alley at night, sodium streetlight, rain-slicked ground,
neon reflection, film noir atmosphere, cinematic, 35mm"]

### Forbidden Elements
[Things that must NOT appear in any image generated in this location:
Example: "no cars, no smartphones, no brand logos, no snow, no crowds"]
```

---

## Step 3 — Define the Global Visual Style

After all location dossiers, write a **Global Style Sheet** — applies to every image and video generated for this project:

```markdown
## Global Visual Style Sheet

### Color Grade Direction
[The overall color treatment. Used by color_grader skill.
Example: "Muted, desaturated palette. Teal-green in shadows, warm amber in highlights.
No pure blacks — lifted blacks for a cinematic grade. Think 'Manchester by the Sea' or 'A Ghost Story'."]

### Lens / Camera Feel
[e.g. "35mm film equivalent. Shallow depth of field. Static or very slow push-ins only.
No handheld shake. No drone shots. No wide-angle distortion."]

### Lighting Philosophy
[e.g. "Always motivated light — every light source in the image should be visible
or implied by the environment. No unmotivated fill light."]

### Forbidden Visual Elements (global)
[Things that must never appear in ANY image for this project:
- Comic or cartoonish elements
- Modern logos, smartphones, social media interfaces
- Anything that dates the video to a specific year
- Bright saturated primary colors
- Text overlays (motion_designer handles all text)]

### Era / Timelessness
[e.g. "Timeless — could be 1990s or today. No identifiable modern tech."]

### Aspect Ratio
[16:9 for all standard shots | 9:16 variants for social_media_manager]
```

---

## Step 4 — Update PROJECT_STATE

```yaml
decisions:
  color_grade_style: "[extracted from Global Visual Style Sheet — 1 sentence summary]"
current_stage: "script"
next_action: "Run scriptwriter skill."
asset_registry.lyrics.locations: "Lyrics/LOCATIONS_[project_id].md"
```

---

## Prompt Fragment Quality Standard

Every Higgsfield prompt fragment must:
1. Be **30–80 words** — specific enough to guide the model, not so long it contradicts itself
2. Include: **subject**, **environment**, **lighting**, **atmosphere**, **style keyword** (cinematic / noir / editorial / etc.)
3. End with a **technical reference**: film stock, director style, or genre keyword
4. NOT include: character names, abstract emotion labels, or relative terms like "beautiful" or "sad"

**Example of a WEAK fragment:** `"a dark sad place where someone feels lonely, cinematic"`
**Example of a STRONG fragment:** `"empty rain-soaked alley at night, single overhead sodium lamp, deep shadows, wet cobblestones reflecting amber light, film noir, 35mm, high contrast, no people, architectural detail"`
