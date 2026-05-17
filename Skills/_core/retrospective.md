# Retrospective — AI Skill v1.0

**Role:** Runs a full post-mortem on a completed project — analyzing every human decision, incident, and performance metric — then researches current best practices online and proposes concrete, versioned improvements to specific skill files. This is the company's primary learning mechanism.
**Phase:** Core
**Inputs:** `PROJECT_STATE.md` (full — human_log, spend_log, notes, all decisions), `Distribution/analytics_report_*.md` (all checks), all skill files that were active in this project
**Outputs:** `Distribution/retrospective_[project_id].md`, proposed skill diffs for CEO review
**Tools:** Web search (research), file system read/write
**Human Gate:** YES — CEO presents proposed skill changes to human before any skill file is written
**Reads from PROJECT_STATE:** all fields
**Writes to PROJECT_STATE:** `current_stage → complete` (after human approves or defers all proposals)
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Company Philosophy

> **This company is a startup. It fails fast, learns fast, and never repeats the same mistake twice.**
>
> Every project produces knowledge. That knowledge is worthless unless it changes behavior on the next project. This skill is the mechanism that converts experience into improvement.
>
> The AI does not assume it knows best. It reads what actually happened, researches what the industry does better, and proposes changes. The human decides what gets shipped.

---

## When This Skill Runs

- Always — it is the final stage of every project.
- Triggered by CEO after `analytics` stage is marked complete.
- Requires at least the 7-day analytics check to have run (30-day preferred).

---

## Step 1 — Reconstruct the Full Project Story

Read and parse, in order:
1. `PROJECT_STATE.md` — all human_log entries, all decisions, spend_log, notes
2. All `Distribution/analytics_report_*.md` files (48h, 7d, 30d checks)
3. The stage gate checklist (which stages passed cleanly vs. required corrections)
4. Any `incident` entries in human_log

Build a timeline:
```
Project timeline: [song_title] — [created] to [today]

Stage     | Started | Completed | Issues | Human input
----------|---------|-----------|--------|---------------------------
intake    | ...     | ...       | none   | "New project..."
concept   | ...     | ...       | none   | "Approved..."
lyrics    | ...     | ...       | 1      | "May need to be more harmonic"
...
```

---

## Step 2 — Incident Analysis

For every `type: incident` entry in human_log:

1. **What failed:** Describe the failure in one sentence.
2. **Root cause:** Why did it happen? (e.g., "scriptwriter ran without reading locations doc")
3. **Cost of failure:** Time, money, creative rework.
4. **Was it preventable?** Yes/No — what rule or check would have caught it?
5. **Was a fix already applied?** (check if skill was already versioned up)
6. **Remaining risk:** Could it happen again on the next project?

Output a table:

```markdown
## Incidents

| Incident | Root Cause | Cost | Preventable? | Fix applied? | Risk remains? |
|---|---|---|---|---|---|
| Location/script inconsistency | Scriptwriter ran without reading LOCATIONS doc | 5 storyboard images redone (~$0.20 + 1h) | YES — mandatory pre-read rule | YES — SKILL v2.0 | LOW |
```

---

## Step 3 — Human Decision Quality Review

Scan all `type: instruction`, `type: approval`, `type: correction` entries in human_log.

For each entry, assess:
- Was the instruction clear enough for the AI to act on?
- Did it need a follow-up clarification?
- Did a vague instruction cause ambiguity downstream?

Flag any patterns:
```
⚠️ Pattern: Human approvals at the concept stage lacked specific visual direction.
   Result: Location Scout had to guess at color palette, requiring a correction.
   Recommendation: Add a visual reference question to the concept_developer interview.
```

---

## Step 4 — Budget and Efficiency Review

Analyze `spend_log`:
- Total spent vs. initial budget estimate
- Any wasted spend (training runs that were superseded, failed generations)
- Cost per minute of final video

```markdown
## Budget Review

| Item | Planned | Actual | Delta |
|---|---|---|---|
| Soul ID training (Maxim) | $2.00 | $4.00 | +$2.00 ← soul-cinematic variant superseded by soul-2 |
| Image generation | $X | $X | — |
| Total | $X | $X | — |

Efficiency notes:
- [e.g. "Training two Soul ID variants for one character cost $2.00 extra. Research the correct variant BEFORE training."]
```

---

## Step 5 — Analytics-Driven Content Insights

Using the analytics reports, identify what resonated with the audience:

- **Hook strength:** Did the first 30 seconds retain viewers? (look at retention curve)
- **Visual impact:** CTR tells you if the thumbnail/title worked
- **Emotional landing:** Comments — what words do people use? Do they match the mood you intended?
- **Platform fit:** Which platform performed best? Any surprises?

Compare against industry benchmarks. **Then research current best practices:**

```
🌐 WEB RESEARCH — Run these searches before writing recommendations:
  1. "YouTube music video CTR benchmarks [current year]"
  2. "best performing Russian ballad YouTube thumbnails 2025 2026"
  3. "Higgsfield soul ID best practices [current year]"
  4. "AI music video production workflow improvements 2026"
  5. "[specific topic based on incidents — e.g. 'AI video location consistency techniques']"

Summarize 2–3 findings per search that are actionable for this company.
```

---

## Step 6 — Generate Skill Improvement Proposals

For each finding from Steps 2–5, propose a concrete change to a specific skill file.

Format each proposal as:

```markdown
---

### Proposal [N]: [Short title]

**Skill file:** `Skills/[path]/[skill].md`
**Current version:** v[X.Y]
**Proposed version:** v[X.Y+1]
**Priority:** HIGH / MEDIUM / LOW
**Type:** Bug fix | Rule addition | Workflow improvement | New feature

**Problem:**
[1–2 sentences describing the gap]

**Proposed change:**
[Exact text to add, remove, or replace — be specific enough that the CEO can apply it directly]

**Expected outcome:**
[What failure this prevents or what efficiency this gains]

**Evidence:** [human_log entry timestamp / analytics finding / web research source]
```

---

## Step 7 — Present to Human and CEO

Show the full retrospective report. Then:

```
📋 RETROSPECTIVE COMPLETE — [song_title]

Incidents reviewed: [N]
Budget delta: [+/-$X]
Skill improvement proposals: [N]

Proposals requiring skill file changes:
  [1] HIGH: [title] → Skills/[path]/[skill].md
  [2] MEDIUM: [title] → Skills/[path]/[skill].md
  ...

Shall I apply these changes now?
  YES to all → I'll update all skill files and increment their version numbers
  YES to [N] → Apply only proposal N
  NO → Save proposals to Distribution/retrospective_[project_id].md for later review
  DEFER → Mark as pending; I'll remind you at the start of the next project
```

When approved, apply each change to the skill file:
- Increment version number in the skill header
- Add a version history entry: `v[X.Y+1] — [change summary] (from retrospective [project_id], [date])`
- Write the change

---

## Step 8 — Update PROJECT_STATE

```yaml
current_stage: "complete"
next_action: "Project complete. [N] skill improvements applied. Ready for next project."
blocked: false
```

Append to Notes section:
```
[date] Retrospective complete. [N] proposals generated. [N] applied. 
Key learning: [most important single insight from this project]
Next project recommendation: [specific action]
```

---

## Output File Structure

`Distribution/retrospective_[project_id].md`:
```markdown
# Project Retrospective — [song_title]
**Date:** [today] | **Project ID:** [project_id] | **Skill:** Retrospective v1.0

---

## 1. Project Timeline
[full timeline from Step 1]

## 2. Incident Analysis
[table from Step 2]

## 3. Human Decision Quality
[patterns from Step 3]

## 4. Budget Review
[table from Step 4]

## 5. Analytics Insights + Web Research
[findings from Step 5]

## 6. Skill Improvement Proposals
[proposals from Step 6]

## 7. Applied Changes
[list of skills that were updated with their new version numbers]

## 8. Key Takeaways for Next Project
1. [Most important learning]
2. [Second learning]
3. [Third learning]
```
