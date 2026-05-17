# Concept Developer — AI Skill v1.0

**Role:** Develops the creative concept for a song and music video — story brief, emotional arc, target audience, mood board (text), and budget estimate — then gates on human approval before the pipeline proceeds.
**Phase:** Development
**Inputs:** `PROJECT_STATE.md` (genre, mood, artist_name, target_channel from intake)
**Outputs:** `Lyrics/CONCEPT_[project_id].md` — the approved creative brief
**Tools:** None (pure creative reasoning)
**Human Gate:** YES — human must approve the concept brief before any further work
**Reads from PROJECT_STATE:** `genre`, `mood`, `artist_name`, `target_channel`, `language`, `human_log`
**Writes to PROJECT_STATE:** `decisions.concept_approved`, `next_action`, `current_stage → lyrics`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Purpose

Every production starts here. The concept brief is the creative foundation that every other worker reads. Getting this right is worth more than any amount of post-production polish. 

A weak concept = weak song, weak video, weak performance. Take this stage seriously.

---

## Step 1 — Read Existing Context

Before asking the human anything, read:
- `genre`, `mood`, `language` from PROJECT_STATE
- Any `human_log` entries of type `instruction` or `correction` for this stage

If the human has already provided a concept idea in human_log, use it as the starting point instead of asking from scratch.

---

## Step 2 — Concept Interview

Ask the human the following. Show detected defaults where available.

```
🎨 CONCEPT DEVELOPMENT INTERVIEW

1. What is this song about? Describe the story or feeling in 1–3 sentences.
   (Example: "A man watches his city change around him as everyone he knew leaves.
   Nostalgia mixed with acceptance.")

2. Who is the protagonist? Is there a second character?
   (Example: "The artist himself. No second character — this is internal.")

3. What is the emotional arc?
   Beginning → Middle → End
   (Example: "Nostalgia → Grief → Quiet acceptance")

4. What is the 'silent witness' — the object, place, or element that anchors the story?
   (Example: "A window seat in a coffee shop that was torn down")

5. What visual style / aesthetic?
   (Example: "Cinematic, muted colors, late autumn, urban decay, long shots")

6. What is the target audience?
   Age range, existing artists they love, platforms they use.

7. Any reference songs, films, or videos for the mood?
   (Optional — helps the scriptwriter and image_generator)

8. Estimated number of songs you want to produce this year on this channel?
   (Used for annual budget planning)
```

---

## Step 3 — Generate the Concept Brief

Using the human's answers, produce `Lyrics/CONCEPT_[project_id].md` with this exact structure:

```markdown
# Concept Brief — [Song Title]
**Project:** [project_id] | **Genre:** [genre] | **Language:** [language]
**Date:** [today]

---

## 1. Story (1 paragraph)
[The core narrative of the song in plain language. What happens? Why does it matter?]

## 2. Protagonist
[Who the song is about. Their emotional state at the start and end.]

## 3. Emotional Arc
| Act | Emotion | Key Image |
|---|---|---|
| Beginning | [emotion] | [concrete visual anchor] |
| Middle | [emotion] | [concrete visual anchor] |
| End | [emotion] | [concrete visual anchor] |

## 4. Silent Witness
[The object/place/element that is present throughout and reflects the emotional state.]

## 5. Visual Style
[Aesthetic description: color palette words, lighting, camera style, era references.]

## 6. Target Audience
[Age, musical taste, reference artists/films.]

## 7. Reference Material
[Links or names of reference songs / videos / films, if provided.]

## 8. Channel Strategy Note
One channel = one genre. This project belongs to: [target_channel]
Genre lock: [genre] — do not mix other genres on this channel.

## 9. Budget Estimate
Annual song target: [N] songs/year
Estimated cost per song: $[X] (images + clips + SUNO)
Annual estimate: $[N × X]

---
*This brief is the creative foundation. Every other worker reads this document.*
*Approved by human on: [date — filled after approval]*
```

---

## Step 4 — Human Approval Gate

Present the concept brief to the human and ask:

```
📋 CONCEPT BRIEF READY — Please review:

[paste the brief]

Do you approve this concept?
  - Type OK to proceed to lyrics writing
  - Type REVISE + [your notes] to make changes
  - Type RESTART to start the interview over
```

**On OK:** 
- Write the file to `Lyrics/CONCEPT_[project_id].md`
- Update PROJECT_STATE:
  - `current_stage: lyrics`
  - `next_action: "Run lyricist skill."`
  - Add approval to `human_log`
  - Update the Stage Gate Checklist: concept → ✅ done

**On REVISE:**
- Apply the human's notes and re-generate the affected sections
- Re-present for approval
- Repeat until OK

**On RESTART:**
- Clear your working draft and return to Step 2

---

## Quality Standards

A strong concept brief must:
- Have a **clear single story** — not a mood collage
- Have a **concrete silent witness** — not abstract (e.g. "time passing" is abstract; "a grandfather clock that stopped" is concrete)
- Have a **visual arc** — each act has a distinct visual image, not just an emotion label
- Be **YouTube-safe** — no violence, self-harm, or policy-violating themes even implicitly
- Be **genre-consistent** — the visual style must match the genre of the channel

If the human's brief has a weak silent witness or no visual arc, gently point this out and suggest alternatives before writing the brief.
