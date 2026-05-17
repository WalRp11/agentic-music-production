# Location Scout — AI Skill v2.0

**Role:** Finds real visual references for each location via stock image search, gets user approval, checks licensing, and hands off to Higgsfield using either the approved image directly or a reverse-engineered visual profile.
**Phase:** Pre-Production
**Inputs:** `Lyrics/CONCEPT_[project_id].md`, `Lyrics/CASTING_[project_id].md`, `Brand/color_palette.json`
**Outputs:**
  - `Lyrics/LOCATIONS_[project_id].md` — visual world document with location dossiers
  - `Images/storyboard/[location_slug]/candidates.md` — search results per location
  - `Images/storyboard/[location_slug]/approved.md` — approved reference + license status
  - `Images/storyboard/[location_slug]/visual_profile.json` — extracted visual DNA (restricted-license path only)
  - `Images/storyboard/[location_slug]/generated.md` — Higgsfield output URL + prompt used
**Tools:** `fetch_webpage` (stock image search), `view_image` (visual profile extraction on downloaded files), `create_file` (storyboard folder records)
**Human Gate:** YES — user approves each location reference image before proceeding
**Reads from PROJECT_STATE:** `genre`, `mood`, `decisions.color_grade_style`
**Writes to PROJECT_STATE:** `decisions.color_grade_style`, `current_stage → script`
**Version History:**
  - v1.0 — 2026-05-17: Text-only location dossiers, no tools, no human gate
  - v2.0 — 2026-05-17: Stock image research, approval loop, licensing check, Higgsfield integration

---

## Purpose

In a real film production, the location scout finds physical places. Here, the scout finds real stock images that capture the exact visual atmosphere of each scene — then either uses them as direct references in Higgsfield or reverse-engineers their look into a structured visual profile that drives generation without infringing on any license.

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

## Step 3 — Stock Image Research

For each location defined in Step 2, search stock image websites to find real reference images that match the written dossier.

### Build the Search Query

Combine fields from the dossier into a concise search string:
```
[architecture_style] + [space_type] + [condition] + [lighting keyword]
```
Examples:
- `"brutalist interior abandoned corridor natural light"`
- `"rain-soaked narrow alley night neon reflections"`
- `"soviet apartment block exterior winter fog"`

Hyphenate multi-word phrases in URL slugs: `rain-soaked-alley-night`.

### Search Sites — in Priority Order

| Site | Search URL Pattern | License Tier |
|------|--------------------|--------------|
| Unsplash | `https://unsplash.com/s/photos/[query-slug]` | Free — Unsplash License ✓ |
| Pexels | `https://www.pexels.com/search/[query-slug]/` | Free — Pexels License ✓ |
| Pixabay | `https://pixabay.com/images/search/[query-slug]/` | Free — CC0 ✓ |
| Adobe Stock | `https://stock.adobe.com/search?k=[query]` | Paid subscription ⚠ |
| Shutterstock | `https://www.shutterstock.com/search/[query]` | Paid subscription ⚠ |

Start with Unsplash, Pexels, and Pixabay. Only search paid sites if the free tier returns no usable matches.

### Procedure

```
FOR each location in dossiers:
  1. Use fetch_webpage on the search URL
  2. Extract 3–5 image URLs + titles that best match the dossier
  3. Write results to Images/storyboard/[location_slug]/candidates.md:
     - Image title + author
     - Direct image URL
     - Source site
     - One sentence: why this image matches the dossier
  4. Present candidates to user (Step 4)
```

**Slug rule:** derive `[location_slug]` from the location's short name, lowercased, spaces → underscores. E.g. "Rainy Alley" → `rainy_alley`.

---

## Step 4 — User Approval Loop

**HUMAN GATE — do not proceed without explicit user approval.**

Present the candidates clearly:

> "Here are **[N] candidates** for **[Location Name]**:
>
> 1. [Title] — [one-line description of why it fits] → [URL]
> 2. [Title] — [one-line description] → [URL]
> 3. ...
>
> Does any of these match your vision? If not, tell me what's wrong (lighting? scale? architecture? color?) and I'll refine the search."

### If the user rejects all candidates

1. Ask one focused question: "What specifically is missing or wrong?"
2. Update the search query based on the answer
3. Try different search terms or a different site
4. Repeat from Step 3 — loop until approval

### If the user approves one image

1. Note which image was approved (URL, title, source site)
2. Save to `Images/storyboard/[location_slug]/approved.md`:
   ```markdown
   # Approved Reference — [Location Name]
   Title: [title]
   Author: [author if available]
   Source: [Unsplash / Pexels / Pixabay / Adobe Stock / Shutterstock / Other]
   URL: [direct image URL]
   Approved: [date]
   ```
3. Proceed to Step 5

---

## Step 5 — Licensing Check

Determine whether the approved image can be passed as a direct `--image` reference to Higgsfield without license risk.

### Assessment Rules

| Source | Higgsfield Reference Use | Rationale |
|--------|--------------------------|-----------|
| Unsplash | ✓ **Reference OK** | Unsplash License — free for commercial creative use |
| Pexels | ✓ **Reference OK** | Pexels License — free for commercial creative use |
| Pixabay | ✓ **Reference OK** | CC0 — no restrictions |
| Adobe Stock | ✗ **RESTRICTED** | Standard license does not cover AI input/reference use |
| Shutterstock | ✗ **RESTRICTED** | Standard license does not cover AI input/reference use |
| Getty Images | ✗ **RESTRICTED** | Editorial license only — not for AI reference use |
| Unknown / Other | ⚠ **ASK USER** | Cannot determine; present the ToS URL and ask |

### Outcome routing

- **Reference OK** → proceed to **Step 6A** (direct reference)
- **RESTRICTED** → inform the user, then proceed to **Step 6B** (JSON profile path)
- **Unknown** → present the source's ToS page to the user and ask them to confirm before choosing a path

Notification to user:
- **OK:** "This image is from [Source] under a free license — I can use it directly as a visual reference in Higgsfield."
- **Restricted:** "This image is from [Source] and its standard license doesn't cover use as an AI reference input. I'll reverse-engineer its visual profile instead and recreate the look without touching the original."

---

## Step 6A — Direct Reference Path (License OK)

Pass the approved image directly to Higgsfield's Soul Location model, which is purpose-built for environments and location shots.

1. Use the image URL (or download the file locally first if the URL requires auth)
2. Compose the command from the dossier's **Higgsfield Prompt Fragment**:

```bash
# Verify the exact model ID first
higgsfield model list --json | jq '.[] | select(.name | test("soul.location"; "i"))'

# Generate with direct image reference
higgsfield generate create soul_location \
  --prompt "[Higgsfield Prompt Fragment from dossier]" \
  --image "[image_url_or_local_path]" \
  --aspect_ratio 16:9 \
  --wait
```

3. Save the result to `Images/storyboard/[location_slug]/generated.md`:
   ```markdown
   # Generated Location — [Location Name]
   Model: soul_location
   Path: direct reference
   Reference image: [URL]
   Prompt: [exact prompt used]
   Output URL: [Higgsfield result URL]
   ```
4. Update the dossier's **Higgsfield Prompt Fragment** with the final prompt used.

---

## Step 6B — JSON Profile Path (License Restricted)

When the approved image cannot be used as a direct reference, reverse-engineer its visual identity into a structured JSON profile and use that to drive generation.

### Sub-step 6B.1 — Download the Image

Save the reference image locally to `Images/storyboard/[location_slug]/reference_restricted.[ext]`.
Do not commit or redistribute this file — it is a temporary analysis artifact only.

### Sub-step 6B.2 — Extract Visual Profile

Use `view_image` on the downloaded file and extract the following JSON structure. Be precise — every field will be used in prompt construction:

```json
{
  "visual_profile": {
    "space_type": "interior | exterior | abstract",
    "architecture_style": "e.g. Soviet brutalist, Haussmann apartment, industrial warehouse, Art Deco ballroom",
    "era": "e.g. 1970s Soviet, timeless, post-industrial, contemporary",
    "condition": "pristine | worn | decayed | abandoned | active",
    "scale": "intimate | medium | vast",
    "color_palette": {
      "dominant": "#hex",
      "secondary": "#hex",
      "accent": "#hex",
      "overall_tone": "warm | cool | neutral | high-contrast | desaturated"
    },
    "lighting": {
      "source": "natural | artificial | mixed",
      "direction": "overhead | side | back | frontal | multiple",
      "quality": "hard | soft | diffuse | dappled",
      "color_temperature": "e.g. 3200K warm tungsten | 5600K daylight | 4200K fluorescent",
      "shadow_depth": "deep | medium | shallow",
      "notable_features": "e.g. god rays, window-bar shadow patterns, candle flicker, neon spill"
    },
    "surfaces": ["e.g. cracked plaster", "wet concrete", "rusted corrugated metal", "water-stained ceiling"],
    "atmosphere": "e.g. melancholic, oppressive, serene, electric, liminal",
    "camera_perspective": "e.g. corridor wide shot, low angle looking up, eye-level mid-shot, over-shoulder",
    "depth_and_geometry": "e.g. strong linear perspective, flat symmetry, layered receding planes, compressed telephoto",
    "mood_keywords": ["isolation", "decay", "tension"],
    "forbidden_elements": ["people", "cars", "logos", "snow", "modern tech"]
  }
}
```

Save to `Images/storyboard/[location_slug]/visual_profile.json`.

### Sub-step 6B.3 — Assemble the Higgsfield Prompt

Convert the JSON into a Soul Location prompt using this assembly template:

```
[space_type], [architecture_style], [condition], [lighting.direction] [lighting.source] light,
[lighting.color_temperature], [surfaces — comma-joined], [camera_perspective],
[depth_and_geometry], cinematic, [mood_keywords — comma-joined],
[era] aesthetic, no people, architectural photography
```

**Weak example (DO NOT DO):** `"a dark sad abandoned place, cinematic"`

**Strong example:**
> `"interior Soviet brutalist corridor, abandoned and decayed, side natural light through barred windows, 4500K cool daylight, cracked plaster walls and wet concrete floor, corridor wide shot, strong linear perspective with receding vanishing point, cinematic, isolation and melancholy, 1970s aesthetic, no people, architectural photography"`

The assembled prompt replaces the dossier's placeholder `Higgsfield Prompt Fragment`.

### Sub-step 6B.4 — Generate

```bash
# Verify the exact model ID first
higgsfield model list --json | jq '.[] | select(.name | test("soul.location"; "i"))'

higgsfield generate create soul_location \
  --prompt "[assembled prompt]" \
  --aspect_ratio 16:9 \
  --wait
```

Save to `Images/storyboard/[location_slug]/generated.md`:
```markdown
# Generated Location — [Location Name]
Model: soul_location
Path: JSON profile (restricted source — [Source name])
Reference image: [URL — kept local only, not distributed]
Visual profile: Images/storyboard/[location_slug]/visual_profile.json
Prompt: [exact prompt used]
Output URL: [Higgsfield result URL]
```

Delete the locally-downloaded reference file after generation is confirmed.

---

## Step 7 — Define the Global Visual Style

After all location dossiers are approved and generated, write a **Global Style Sheet** — applies to every image and video generated for this project:

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

## Step 8 — Update PROJECT_STATE

```yaml
decisions:
  color_grade_style: "[extracted from Global Visual Style Sheet — 1 sentence summary]"
current_stage: "script"
next_action: "Run scriptwriter skill."
asset_registry.lyrics.locations: "Lyrics/LOCATIONS_[project_id].md"
asset_registry.images.storyboard: "Images/storyboard/"
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
