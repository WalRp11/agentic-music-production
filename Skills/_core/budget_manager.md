# Budget Manager — AI Skill v1.0

**Role:** Tracks API credit consumption and USD cost estimates per worker per project run; surfaces the Capex gate before any costly batch API call.
**Phase:** Core
**Inputs:** Worker name, action performed, units consumed
**Outputs:** Updated `spend_log` in `PROJECT_STATE.md`; Capex gate reports
**Tools:** File system read/write
**Human Gate:** YES — Capex gate before image_generation, video_generation, audio_production batch runs
**Reads from PROJECT_STATE:** `spend_log`, `decisions.capex_approved`, `decisions.capex_budget_usd`
**Writes to PROJECT_STATE:** `spend_log`, `decisions.capex_approved`
**Version History:** v1.0 — Initial release (2026-05-17)

---

## Credit Cost Reference Table

> Update these rates as platform pricing changes. Estimates only — verify on your platform dashboard.

### SUNO
| Action | Units | Est. USD |
|---|---|---|
| Generate 1 song (2 takes) | 10 credits | $0.10 |
| Generate 1 song (full, 2 takes + extend) | 20 credits | $0.20 |
| Monthly Pro plan (unlimited) | — | $10/mo |

### Higgsfield CLI
| Action | Units | Est. USD |
|---|---|---|
| Generate 1 image (GPT Image 2) | 1 credit | $0.04 |
| Generate 1 video clip (10s, standard) | 3 credits | $0.12 |
| Generate 1 video clip (10s, HD) | 5 credits | $0.20 |
| Soul ID training | 50 credits | $2.00 |

### Whisper (OpenAI API)
| Action | Units | Est. USD |
|---|---|---|
| Transcribe 1 song (~3-4 min) | — | $0.02 |

---

## Capex Gate Procedure

This gate fires automatically before any worker that involves batch API calls. The CEO orchestrator calls this skill before delegating to those workers.

### Trigger stages: `audio_production`, `image_generation`, `video_generation`, `soul_identity`

**Step 1 — Calculate estimated cost:**

For `audio_production`:
```
Takes planned: [from music_director decisions]
Cost = takes × $0.10
```

For `image_generation`:
```
Images planned: [from scriptwriter cut_list]
Cost = images × $0.04
```

For `video_generation`:
```
Clips planned: [from scriptwriter cut_list]
Cost = clips × $0.12 (standard) or clips × $0.20 (HD)
```

For `soul_identity` (first time only):
```
Cost = $2.00 (one-time training)
```

**Step 2 — Report to human:**
```
💰 CAPEX GATE — Approval Required

Upcoming batch job: [worker name]
Estimated units:    [X images / Y clips / Z takes]
Estimated cost:     $[amount]
Current spend:      $[total_usd_estimate to date]
Remaining budget:   $[capex_budget_usd - total_usd_estimate]

Do you approve this run? (yes/no)
If yes, any changes to the plan (e.g. fewer clips)?
```

**Step 3 — Record decision:**
- If approved: set `capex_approved: true` in `decisions{}`, append to `human_log`, continue.
- If not approved: set `blocked: true`, set `blocked_reason: "Capex not approved for [worker]"`, stop.
- Reset `capex_approved: false` after each batch run (next batch needs its own approval).

---

## Spend Logging

After every worker run that makes API calls, append to `spend_log`:

```yaml
- timestamp: "[ISO datetime]"
  worker: "[worker skill name]"
  action: "[e.g. 'Generated 25 images via Higgsfield']"
  units: [count]
  cost_usd_estimate: [calculated from table above]
```

Also update `total_usd_estimate`:
```
total_usd_estimate = SUM of all cost_usd_estimate entries
```

---

## Budget Report

When asked "show me the budget" or at the end of each session, output:

```
💰 BUDGET REPORT — [project_id]

Total spent (est.):   $[total_usd_estimate]
Approved budget:      $[capex_budget_usd]
Remaining:            $[remaining]

Breakdown by worker:
  audio_engineer:      $[sum]  — [X SUNO takes]
  image_generator:     $[sum]  — [X images]
  video_generator:     $[sum]  — [X clips]
  soul_identity:       $[sum]  — [training + X renders]
  whisper (intake):    $[sum]  — [X transcriptions]

⚠️  Over budget: [yes/no]
```

---

## Budget Exceeded Warning

```
IF total_usd_estimate >= capex_budget_usd × 0.80:
    WARN human: "⚠️ You have used 80% of your approved budget ($[total]/$[budget])."

IF total_usd_estimate >= capex_budget_usd:
    BLOCK all further API batch runs
    SET blocked = true
    SET blocked_reason = "Budget limit reached. Human must approve additional Capex."
```
