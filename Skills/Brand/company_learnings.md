# Company Learnings — WARPMUZ AI Studio
> **Format:** INDEX at top (≤ 25 lines). Tagged sections below. CEO loads INDEX first, then only relevant tagged sections (max 2 per run, ~600 tokens total).
> **Updated by:** retrospective skill after each project, or by CEO immediately on incident.
> **Last updated:** 2026-05-18 | Project: Разные Пути

---

## INDEX

| Tag | Topic | Section |
|-----|-------|---------|
| [CONSISTENCY] | Cross-document consistency — location scout / scriptwriter sync | §1 |
| [HIGGSFIELD] | Higgsfield generation — model selection, reference images, known failures | §2 |
| [SOUL_ID] | Soul ID training and usage — best practices | §3 |
| [PIPELINE] | Pipeline sequencing — known gaps and proposals | §4 |

**CEO loading rules:**
- `locations`, `script`, `creative_design`, `casting` → load [CONSISTENCY]
- `image_generation`, `video_generation` → load [HIGGSFIELD] + [SOUL_ID]
- `soul_identity` → load [SOUL_ID]
- Session start / planning → load [PIPELINE]

---

## §1 — [CONSISTENCY]

**Incident (2026-05-17 · Разные Пути · pre-generation audit):**
Location Scout v1.0 and Script Writer v1.0 worked independently and produced conflicting location specs — generic commuter train with fluorescent blue-white lighting (Location Scout) vs. German ICE 3 with warm amber LED lighting (Script Writer). The Casting Doc inherited the wrong spec from Location Scout. All three documents were mutually inconsistent before image generation began. Five storyboard images were pre-generated from the ICE spec before the conflict was caught.

**Rules learned:**
1. `scriptwriter` MUST read `LOCATIONS_*.md` and `CASTING_*.md` as its FIRST step before writing any scene content. If it changes any location detail it must update those documents in the same run.
2. `location_scout` output becomes the canonical location spec. All downstream workers (scriptwriter, casting_director, creative_designer) inherit from it and must cite it.
3. CEO must run a cross-document consistency check (LOCATIONS vs SCRIPT vs CASTING) before invoking `image_generation`. If any conflict is found, block and resolve before generating a single image.

**Fix applied:** scriptwriter v2.0 and location_scout v2.0 created with mandatory cross-doc read as their first step.

---

## §2 — [HIGGSFIELD]

### Model selection (confirmed working)

| Situation | Model | Flags |
|-----------|-------|-------|
| No character / environment / atmosphere | `imagegen_2_0` (GPT Image 2 Unlimited) | `--quality high --resolution 4k` |
| Character with Soul ID, medium/wide shot | `text2image_soul_v2` + `--image [location_ref]` | `--quality 2k` |
| Character close-up / emotional peak | `soul_cinema_studio` | `--quality 2k` |
| 3+ scenes in same interior | Popcorn multi-frame (web UI) | — |
| Structurally specific interior (trains, rooms) | `imagegen_2_0` + `--image [reference_photo]` | `--quality high --resolution 4k` |

**`imagegen_2_0` is free on the Ultra plan.** Always prefer it over `gpt_image_2` for non-Soul-ID scenes.

### Reference image workflow

- Always pass `--image [reference_photo]` for architecturally specific locations.
- Source real photographs from Wikimedia Commons before generation — never rely on text-only prompts for specific interiors.
- Approved base image for each location group = visual contract for all scenes in that group.
- Use Popcorn multi-frame (web UI) for any location group with 3+ scenes — it preserves spatial continuity far better than sequential single-image generation.

### Architectural element failures — KNOWN ISSUE

**Incident (2026-05-18 · Разные Пути · c05):**
`text2image_soul_v2` and `imagegen_2_0` cannot reliably generate ICE train sliding plug doors from text prompts alone. The model defaults to hinged glass doors or swinging doors regardless of prompt precision. The c05 scene was rejected twice before the root cause was identified.

**Rule learned — Architectural Element Pre-Catalog:**
Before generating any scene that contains a specific architectural sub-element (door type, window shape, fixture, hardware), source or generate a dedicated reference photo of that EXACT element FIRST:
1. Search Wikimedia Commons or free stock for a real photo of the specific element (e.g., "ICE 3 plug sliding door interior").
2. Save to `Images/[element_name]_reference.jpg`.
3. Pass via `--image` as the primary reference (or alongside the location base image if the location is already locked).
4. Do NOT retry the same text-only prompt more than once — model defaults do not change without a new visual anchor.
5. If the model still fails with a reference image, try GPT Image 2 web UI with the reference photo as an edit input, or treat the problematic element as an overlay composited in post.

**Apply this protocol to:** any scene where the script or storyboard specifies a non-generic architectural detail.

---

## §3 — [SOUL_ID]

**Confirmed Soul IDs (Разные Пути):**
- Maxim (man): soul-2 variant, `47409f93-88d6-4a3e-ba23-b6234b62c722`, trained 2026-05-17
- Gentle Breeze Smile (woman): soul-2 variant, `32ce04ce-4980-4b9b-b642-6239c2c3a740`, trained 2026-05-17 — replaces Natasha for c05 onward

**Rules:**
- Always use `--soul-2` trained IDs with `text2image_soul_v2`. Do NOT mix soul-cinematic IDs with the soul_v2 model — they are incompatible.
- Pass location base image via `--image` when using Soul ID in an interior scene — the Soul ID controls the face; `--image` controls the space. Both work together.
- Do NOT describe facial features in prompts when using Soul ID. Describe clothing, pose, expression, position only.
- When both characters appear in one scene: generate two versions (one per Soul ID) and composite, OR use the dominant character's Soul ID and describe the secondary character by appearance at soft-focus distance.

---

## §4 — [PIPELINE]

### Known gap: no mid-project retrospective

**Current state:** The retrospective skill only runs at stage 23 (project end). Incidents logged mid-project (e.g., c05 door failure on 2026-05-18) do not update the affected skill until the very end of the project.

**Proposed fix:** CEO should trigger a mini-retrospective immediately on any `human_log` entry with `type: incident` — update the affected skill file before resuming. No formal implementation yet; track in next retrospective.

### Known gap: parallelizable stages not implemented

Independent stages that could run concurrently (proposal, not yet implemented):
- Pre-production: `casting` + `locations` (fully independent)
- Post-production: `color_grade` + `sound_design` + `motion_design` (fully independent)

### Known gap: no formal `revision` stage

When a worker fails and triggers a skill update, the only mechanism is `blocked: true` + text note. No formal stage or exit criteria exist. This leaves the project in an ambiguous state (see Разные Пути 2026-05-18 block).
