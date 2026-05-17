# Company Setup (Bootstrap) — AI Skill v1.0

**Role:** One-time setup skill that scaffolds the entire Agentic Music Company folder structure on any new machine, installs all worker skill files, and creates Brand and Project template placeholders.
**Phase:** Core
**Inputs:** Target root folder path provided by human
**Outputs:** Complete company folder structure, all 28 skill files copied, brand scaffold, project template
**Tools:** File system (mkdir, copy), shell (PowerShell or bash)
**Human Gate:** YES — human confirms root path before any writes
**Reads from PROJECT_STATE:** N/A (bootstrap — no project yet)
**Writes to PROJECT_STATE:** N/A
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Purpose

Run this skill exactly once when setting up the company on a new machine or new drive. It creates the canonical folder tree, copies all worker skills from the source Skills library, and creates placeholder files for the brand.

After this skill completes, the company is ready to receive its first project.

---

## Step 1 — Confirm Root Path

Ask the human:
```
Where should the music company root folder be created?
Example: C:\Music Company\  or  ~/MusicCompany/

Type the full path:
```

Then confirm: *"I will create the company folder structure at [path]. This will create new folders but will NOT delete anything existing. Confirm? (yes/no)"*

---

## Step 2 — Create Folder Structure

Create the following directories (skip if already exist):

```
[ROOT]/
├── Skills/
│   ├── _core/
│   ├── development/
│   ├── pre_production/
│   ├── production/
│   ├── post_production/
│   └── distribution/
├── Brand/
│   └── assets/
└── Projects/
    └── _template/
        ├── Sound/
        ├── Lyrics/
        ├── Images/
        ├── Clips/
        ├── Cuts/
        ├── Assembly/
        ├── Distribution/
        └── Marketing/
            ├── Thumbnail/
            ├── Trailer/
            └── Social_Clips/
```

**PowerShell:**
```powershell
$root = "[ROOT]"
$dirs = @(
    "Skills\_core", "Skills\development", "Skills\pre_production",
    "Skills\production", "Skills\post_production", "Skills\distribution",
    "Brand\assets",
    "Projects\_template\Sound", "Projects\_template\Lyrics",
    "Projects\_template\Images", "Projects\_template\Clips",
    "Projects\_template\Cuts", "Projects\_template\Assembly",
    "Projects\_template\Distribution",
    "Projects\_template\Marketing\Thumbnail",
    "Projects\_template\Marketing\Trailer",
    "Projects\_template\Marketing\Social_Clips"
)
foreach ($d in $dirs) { New-Item -ItemType Directory -Path "$root\$d" -Force | Out-Null }
```

**Bash:**
```bash
ROOT="[ROOT]"
mkdir -p "$ROOT/Skills/_core" "$ROOT/Skills/development" \
  "$ROOT/Skills/pre_production" "$ROOT/Skills/production" \
  "$ROOT/Skills/post_production" "$ROOT/Skills/distribution" \
  "$ROOT/Brand/assets" \
  "$ROOT/Projects/_template/Sound" "$ROOT/Projects/_template/Lyrics" \
  "$ROOT/Projects/_template/Images" "$ROOT/Projects/_template/Clips" \
  "$ROOT/Projects/_template/Cuts" "$ROOT/Projects/_template/Assembly" \
  "$ROOT/Projects/_template/Distribution" \
  "$ROOT/Projects/_template/Marketing/Thumbnail" \
  "$ROOT/Projects/_template/Marketing/Trailer" \
  "$ROOT/Projects/_template/Marketing/Social_Clips"
```

---

## Step 3 — Copy Skill Files

Copy every skill file from the source Skills library to the target. If the source library is this repository, the source paths are relative to this file's location.

### Complete Skill File Manifest

| Destination | File |
|---|---|
| `Skills/_core/` | `ceo_orchestrator.md` |
| `Skills/_core/` | `project_intake.md` |
| `Skills/_core/` | `company_setup.md` |
| `Skills/_core/` | `budget_manager.md` |
| `Skills/_core/` | `project_memory_template.md` |
| `Skills/development/` | `concept_developer.md` |
| `Skills/pre_production/` | `lyricist.md` |
| `Skills/pre_production/` | `music_director.md` |
| `Skills/pre_production/` | `casting_director.md` |
| `Skills/pre_production/` | `location_scout.md` |
| `Skills/pre_production/` | `scriptwriter.md` |
| `Skills/production/` | `audio_engineer.md` |
| `Skills/production/` | `image_generator.md` |
| `Skills/production/` | `video_generator.md` |
| `Skills/production/` | `soul_identity.md` |
| `Skills/post_production/` | `video_editor.md` |
| `Skills/post_production/` | `color_grader.md` |
| `Skills/post_production/` | `sound_designer.md` |
| `Skills/post_production/` | `motion_designer.md` |
| `Skills/post_production/` | `graphic_designer.md` |
| `Skills/distribution/` | `export_specialist.md` |
| `Skills/distribution/` | `youtube_uploader.md` |
| `Skills/distribution/` | `music_distributor.md` |
| `Skills/distribution/` | `social_media_manager.md` |
| `Skills/distribution/` | `analytics_reporter.md` |
| `Brand/` | `brand_guardian.md` |

---

## Step 4 — Create Brand Scaffold

Create these placeholder files (skip if already exist):

**`Brand/color_palette.json`:**
```json
{
  "_note": "Edit this file to define your channel's color identity. Used by graphic_designer and motion_designer workers.",
  "primary": "#000000",
  "secondary": "#FFFFFF",
  "accent": "#FF0000",
  "background": "#111111",
  "text_on_dark": "#FFFFFF",
  "text_on_light": "#111111",
  "font_primary": "TBD",
  "font_secondary": "TBD"
}
```

**`Brand/channel_brief.md`:**
```markdown
# Channel Brief
> Fill in this file to define the identity of your YouTube channel.
> Read by the brand_guardian, graphic_designer, and youtube_uploader workers.

## Channel Name
[TBD]

## Genre
[TBD — remember: one genre per channel for best audience retention]

## Artist Name / Persona
[TBD]

## Channel Description (YouTube About)
[TBD — max 1000 characters]

## Visual Style
[Describe the look: e.g. "cinematic noir", "warm retro", "dark electronic"]

## Target Audience
[e.g. "25-40 year olds, fans of cinematic pop, Lana Del Rey, Hans Zimmer"]

## Upload Cadence
[e.g. "one video per month"]
```

---

## Step 5 — Verification

After all files are created, run this check and report to human:

```
✅ COMPANY SETUP COMPLETE

📂 Root: [ROOT]
   Skills/   — [count] skill files installed
   Brand/    — brand_guardian.md, color_palette.json, channel_brief.md
   Projects/ — _template folder ready

🚀 Next step:
   To start your first project, run the CEO Orchestrator skill:
   → Open Skills/_core/ceo_orchestrator.md
   → Tell it: "Start a new project" or "Import existing project from [path]"
```

---

## Updating Skills

When a skill file is updated (improved by experience), update it in the `Skills/` library. Existing `Projects/` folders are not affected — they reference skill versions at the time of their run. 

To force a project to use an updated skill: add an entry to `human_log` in the project's `PROJECT_STATE.md`:
```yaml
- timestamp: "[now]"
  type: instruction
  stage: "[stage to re-run]"
  message: "Re-run this stage with updated skill file."
```
Then set that stage back to `⬜ pending` in the gate checklist and run the CEO orchestrator.
