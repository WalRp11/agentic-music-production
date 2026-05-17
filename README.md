# Agentic Music Production

A complete AI music company as a set of agent skill files. One person operates as creative director; AI agents (Claude Code / OpenAI Codex CLI) run the full pipeline — from concept to YouTube upload — with human approval gates at key decisions.

---

## What This Is

26 markdown skill files that turn an AI agent runtime into a structured music production company. The company covers:

**Development → Pre-Production → Production → Post-Production → Distribution**

Each skill file is a "worker" the CEO orchestrator assigns tasks to. The human approves creative decisions and budget gates; the agents do all the generation, editing, and file work.

**AI tools used:** SUNO (music), Higgsfield CLI (images + video), FFmpeg (assembly + export)

---

## Install

### Claude Code CLI

**1. Install the CEO Orchestrator as a global slash command (one-time):**

```powershell
# Windows PowerShell
if (!(Test-Path "$env:USERPROFILE\.claude\commands")) {
    New-Item -ItemType Directory -Path "$env:USERPROFILE\.claude\commands" -Force
}
Copy-Item "Skills\_core\ceo_orchestrator.md" "$env:USERPROFILE\.claude\commands\ceo_orchestrator.md" -Force
```

```bash
# macOS / Linux
mkdir -p ~/.claude/commands
cp Skills/_core/ceo_orchestrator.md ~/.claude/commands/ceo_orchestrator.md
```

**Optional — install all skills as global slash commands:**

```powershell
# Windows PowerShell
Get-ChildItem "Skills" -Recurse -Filter "*.md" | ForEach-Object {
    Copy-Item $_.FullName "$env:USERPROFILE\.claude\commands\$($_.Name)" -Force
}
```

```bash
# macOS / Linux
find Skills -name "*.md" -exec cp {} ~/.claude/commands/ \;
```

---

### Bootstrap Your Machine (one-time)

Always `cd` to your company root first — skills use relative paths to load each other.

```bash
cd "C:\Users\ruppw\Agentic Music Production"   # Windows
# cd ~/Agentic\ Music\ Production               # macOS / Linux
claude
```

Then inside the Claude Code session, use `@` to load the skill as context:

```
@Skills/_core/company_setup.md Run this skill. Root folder: C:\Users\ruppw\Agentic Music Production
```

This creates `Brand/`, `Projects/_template/`, and all subfolders.

---

### Day-to-Day Usage

```bash
cd "C:\Users\ruppw\Agentic Music Production"
claude
```

Then inside the session:

```
/ceo_orchestrator
```

The CEO will read your project state and route to the correct worker automatically.

> **Note:** `/command-name` invokes installed slash commands from `~/.claude/commands/`. `@path/to/file.md` attaches any file as context inline. There is no `/add-file` command.

---

## Skill Map

### Core (5 skills)
| File | Purpose |
|---|---|
| `_core/ceo_orchestrator.md` | Master coordinator — routes tasks, enforces stage gates |
| `_core/project_intake.md` | Entry point — scans folder, interviews human, creates PROJECT_STATE |
| `_core/project_memory_template.md` | Template for PROJECT_STATE.md (copied per project) |
| `_core/company_setup.md` | One-time bootstrap — creates all folders and copies skill files |
| `_core/budget_manager.md` | Tracks API spend, fires capex gate before batch generation |

### Development (1 skill)
| File | Purpose |
|---|---|
| `development/concept_developer.md` | Story brief, emotional arc, mood board, budget estimate |

### Pre-Production (5 skills)
| File | Purpose |
|---|---|
| `pre_production/lyricist.md` | Universal genre-agnostic lyric writer (ballad/pop/hip-hop/EDM/folk) |
| `pre_production/music_director.md` | SUNO brief — 3 prompt variations + submission instructions |
| `pre_production/casting_director.md` | Character dossiers, Soul ID decision |
| `pre_production/location_scout.md` | Visual world — locations, palettes, lighting |
| `pre_production/scriptwriter.md` | Scene-by-scene script + machine-readable cut list JSON |

### Production (4 skills)
| File | Purpose |
|---|---|
| `production/audio_engineer.md` | Take selection, master file processing |
| `production/image_generator.md` | Batch Higgsfield image generation |
| `production/video_generator.md` | Per-image Higgsfield video clip generation |
| `production/soul_identity.md` | Optional — trains Higgsfield Soul ID from portrait photo |

### Post-Production (5 skills)
| File | Purpose |
|---|---|
| `post_production/video_editor.md` | FFmpeg clip assembly |
| `post_production/color_grader.md` | Cinematic color grade (5 presets) |
| `post_production/sound_designer.md` | Loudness normalization to platform standards |
| `post_production/motion_designer.md` | Animated title cards + outro |
| `post_production/graphic_designer.md` | YouTube thumbnail (with human approval gate) |

### Distribution (5 skills)
| File | Purpose |
|---|---|
| `distribution/export_specialist.md` | Export all platform formats (YouTube, WAV, Reels, Shorts) |
| `distribution/youtube_uploader.md` | SEO metadata package — human uploads to YouTube Studio |
| `distribution/music_distributor.md` | Aggregator submission (DistroKid / TuneCore / Amuse) |
| `distribution/social_media_manager.md` | Promotional clips + posting schedule |
| `distribution/analytics_reporter.md` | 48h/7d/30d performance review, feedback to CEO |

### Brand (1 skill + config)
| File | Purpose |
|---|---|
| `Brand/brand_guardian.md` | Visual identity rules, one-genre-per-channel policy |
| `Brand/color_palette.json` | Per-channel colors, fonts, grade presets |

---

## Pipeline Overview

```
project_intake (new or existing project)
    ↓
concept_developer → [HUMAN GATE: approve concept]
    ↓
lyricist → music_director → casting_director → location_scout → scriptwriter
    ↓ [HUMAN GATE: approve script + cost estimate]
budget_manager (capex gate)
    ↓
audio_engineer (SUNO takes) → [HUMAN GATE: pick take]
    ↓
[soul_identity if needed] → image_generator → video_generator
    ↓
video_editor → color_grader → sound_designer → motion_designer
    ↓
graphic_designer → [HUMAN GATE: approve thumbnail]
    ↓
export_specialist → youtube_uploader → [HUMAN GATE: approve metadata]
    ↓
music_distributor → [HUMAN GATE: human uploads to distributor]
    ↓
social_media_manager → analytics_reporter
```

---

## PROJECT_STATE.md

Every project has one `PROJECT_STATE.md` — a machine-readable progress file the CEO reads and updates at each stage. Created by `project_intake.md` from the template in `_core/project_memory_template.md`.

Key fields: `current_stage`, `intake_complete`, `decisions{}` (all creative decisions), `asset_registry{}` (paths to all generated files), `spend_log{}` (API cost tracking), `human_log[]` (all human approvals with timestamps).

---

## Brand Setup

After your first project, fill in:
- `Brand/color_palette.json` — your channel's colors, font, and color grade preset
- `Brand/channel_brief.md` — one entry per YouTube channel with its genre focus

**One genre per channel** is enforced by the CEO. YouTube's algorithm learns your channel's niche — mixing genres resets that signal.

---

## Requirements

| Tool | Purpose | Install |
|---|---|---|
| FFmpeg | Video assembly, color grade, export, text overlays | [ffmpeg.org](https://ffmpeg.org) |
| Higgsfield CLI | Image + video generation | `pip install higgsfield` |
| SUNO | Music generation | [suno.com](https://suno.com) — web UI |
| Claude Code or Codex CLI | Agent runtime | [claude.ai/code](https://claude.ai/code) or [github.com/openai/codex](https://github.com/openai/codex) |

---

## Version

v1.0 — 2026-05-17 — Initial release. 26 skill files. Full pipeline from concept to analytics.
