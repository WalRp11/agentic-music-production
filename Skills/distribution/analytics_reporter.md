# Analytics Reporter — AI Skill v1.0

**Role:** Reviews YouTube and Spotify performance metrics after release, generates a structured report, and feeds insights back to the CEO for the next project.
**Phase:** Distribution
**Inputs:** Human-provided metrics from YouTube Studio and Spotify for Artists, `PROJECT_STATE.md`
**Outputs:** `Distribution/analytics_report_[date].md`, updated `PROJECT_STATE` notes, insights for next concept
**Tools:** None (metrics provided by human — no API access in v1)
**Human Gate:** NO — report generation is automatic from provided data; insights go to CEO
**Reads from PROJECT_STATE:** `song_title`, `artist_name`, `target_channel`, `decisions.youtube_publish_date`
**Writes to PROJECT_STATE:** `current_stage → complete` (after analytics review)
**Version History:** v1.0 — Initial release (2026-05-17)

---

## When to Run This Skill

| Check | When |
|---|---|
| First check | 48 hours after YouTube publish |
| Second check | 7 days after publish |
| Third check | 30 days after publish |

The 48h check is most important — YouTube's algorithm decides in the first 24–48 hours whether to recommend the video.

---

## Step 1 — Request Metrics from Human

Ask the human to pull these numbers from YouTube Studio and Spotify for Artists:

```
📊 ANALYTICS REQUEST — [song_title] — [check number] check

Please go to YouTube Studio → Content → [video] → Analytics and provide:

YOUTUBE (last [48h / 7d / 30d]):
  1. Total views: ____
  2. Watch time (hours): ____
  3. Average view duration: ____  (and as % of total length)
  4. Click-through rate (CTR): ____  (impressions → views %)
  5. Impressions: ____
  6. Unique viewers: ____
  7. Subscriber gained from this video: ____
  8. Top traffic source: ____ (e.g. YouTube search, Suggested, Browse)
  9. Audience retention graph: Note any big drop-off points (timestamps)
  10. Top comments (copy 3–5): ____

SPOTIFY (if distributed — check after 7 days):
  11. Streams: ____
  12. Listeners: ____
  13. Saves: ____
  14. Playlist adds: ____
  15. Top city: ____
```

---

## Step 2 — Analyze the Data

Apply these benchmarks (independent artist averages — adjust over time based on your own history):

### CTR (Click-Through Rate)
| CTR | Assessment |
|---|---|
| > 10% | Excellent — thumbnail/title working very well |
| 6–10% | Good — above average |
| 3–6% | Average — room for improvement |
| < 3% | Poor — thumbnail or title needs work |

### Average View Duration
| % of video watched | Assessment |
|---|---|
| > 50% | Excellent — strong content retention |
| 35–50% | Good |
| 20–35% | Average — review drop-off points |
| < 20% | Poor — intro problem; first 30s not engaging |

### Watch Time (Hours) for Algorithm Boost
| Hours in 48h | Assessment |
|---|---|
| > 100h | Strong signal — algorithm may push it |
| 10–100h | Moderate signal |
| < 10h | Weak signal — algorithm unlikely to recommend |

---

## Step 3 — Generate Analytics Report

Output `Distribution/analytics_report_[YYYY-MM-DD].md`:

```markdown
# Analytics Report — [Song Title]
**Check:** [48h / 7d / 30d] | **Date:** [today] | **Published:** [publish_date]

---

## Performance Summary
| Metric | Value | Benchmark | Assessment |
|---|---|---|---|
| Views | [N] | — | — |
| Watch time | [Xh] | — | — |
| Avg view duration | [X%] | 35–50% | ✅/⚠️/❌ |
| CTR | [X%] | 6–10% | ✅/⚠️/❌ |
| Subscribers gained | [N] | — | — |
| Top traffic source | [source] | — | — |

---

## Retention Analysis
Drop-off points noted:
- [timestamp]: [what's happening at this moment in the video]
- [timestamp]: [possible reason: transition, slow section, etc.]

Retention Assessment: [strong / acceptable / needs work]

---

## Audience Response
Selected comments:
- "[comment 1]"
- "[comment 2]"
- "[comment 3]"

Comment sentiment: [positive / mixed / negative]

---

## Spotify Performance (7d+ checks only)
| Metric | Value |
|---|---|
| Streams | [N] |
| Saves | [N] (save rate: [saves/streams %]) |
| Playlist adds | [N] |
| Top city | [city] |

---

## Key Findings

### What worked:
1. [Finding 1 — e.g. "High CTR suggests thumbnail and title are effective"]
2. [Finding 2]

### What to improve:
1. [Finding 1 — e.g. "Drop-off at 1:45 — the bridge transition may be too slow"]
2. [Finding 2]

---

## Recommendations for Next Project

From this performance data, the following adjustments are suggested for the next song/video:

**Concept:** [e.g. "Top traffic from YouTube Search for 'sad piano music' — consider a concept that matches this search intent"]
**Thumbnail:** [e.g. "CTR was above 7% — current thumbnail formula is working, keep it"]
**Video pacing:** [e.g. "Drop at 1:45 — next video should avoid slow bridge sections; keep momentum throughout"]
**Release timing:** [e.g. "Saturday 15:00 UTC performed well — repeat this timing"]
**Genre/mood:** [e.g. "Comments mention 'beautiful melody' — lean further into orchestral elements"]
```

---

## Step 4 — Feed Back to CEO

Append recommendations to `PROJECT_STATE.md` notes section. Flag any insights that should influence the next project's concept development:

```yaml
# In PROJECT_STATE.md — Notes section:
Analytics findings [date]:
  - CTR: [X%] — [assessment]
  - Avg view duration: [X%] — [assessment]  
  - Key learning: "[most important actionable insight]"
  - Apply to next project: "[specific recommendation]"
```

---

## Step 5 — Mark Project Complete

```yaml
current_stage: "complete"
next_action: "Project complete. Start new project with CEO orchestrator. Apply analytics learnings from this project's notes."
blocked: false
```

Print final summary:
```
🎉 PROJECT COMPLETE — [song_title]

Pipeline stages: 20/20 ✅
Analytics: [X] views in [N] days | CTR: [X%] | Avg view: [X%]
Total production cost estimate: $[total_usd_estimate]

Key learning for next project:
[top recommendation from analytics]

Ready to start next project. Run CEO orchestrator with your next concept.
```
