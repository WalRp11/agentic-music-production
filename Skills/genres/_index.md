# Genre Module Index

> **Role:** Genre module routing table. CEO reads ONLY this file at concept stage to determine which genre skill file to load.
> **Load rule:** Find the genre tag that matches `genre` in PROJECT_STATE.md → load `Skills/genres/[genre].md`
> **Add a new genre:** Create `Skills/genres/[genre].md` and add one line here.

---

| genre tag | Aliases | Description | Skill file |
|---|---|---|---|
| `ballad` | cinematic_ballad, emotional_ballad, folk_ballad | Slow emotional songs, piano/strings, Russian/European vocal style. 55-75 BPM. High lyric syllable fidelity. Long cuts, intimate framing, melancholic visuals. | `Skills/genres/ballad.md` |
| `pop` | pop_radio, modern_pop, indie_pop | Hook-driven, upbeat, 100-130 BPM. Syllable count flexible. Fast cuts, high energy visuals, bright color. | `Skills/genres/pop.md` |
| `folk` | acoustic_folk, singer_songwriter | Acoustic storytelling, loose rhyme, 75-95 BPM. Intimate, nature or rural settings. | `Skills/genres/folk.md` |
| `hip_hop` | rap, trap, boom_bap | Beat-driven bars, strict syllable count, 80-140 BPM. Urban visuals, dynamic cuts. | (not yet created) |
| `edm` | dance, electronic, house | Hook + drop structure, melody-light lyrics, 120-145 BPM. Abstract visuals, VFX-heavy. | (not yet created) |
| `rock` | indie_rock, alternative | Guitar-driven, moderate syllable flexibility, 110-140 BPM. Performance-style shooting preferred. | (not yet created) |
