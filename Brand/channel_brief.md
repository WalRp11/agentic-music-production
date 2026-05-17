# Channel Brief — WARPMUZ
> **Version:** 1.0 | Created: 2026-05-17
> This is the single source of truth for all channels operated by this company.
> All workers that produce branded output (motion_designer, graphic_designer, youtube_uploader) read this file.
> Update this file whenever a channel strategy changes — never update individual project files.

---

## Channels

### WARPMUZ
```yaml
channel_name: "WARPMUZ"
youtube_url: "https://www.youtube.com/@WARPMUZ"
status: active

genre_lock: "ballad"
allowed_genres:
  - ballad
  - cinematic_ballad
  - folk
  # DO NOT produce pop, EDM, hip-hop, or trap on this channel.
  # If a new song concept doesn't fit these genres, the CEO must create a new channel.

mood_range:
  - melancholic
  - cinematic
  - intimate
  - hopeful
  - bittersweet

language_primary: "Russian"
languages_allowed:
  - Russian
  - Ukrainian
  - English (only if concept strongly calls for it)

target_audience:
  age_range: "25–50"
  listener_archetype: "Listeners of cinematic emotional music — think Hans Zimmer, Ludovico Einaudi, Russian romance, slow-burn ballads"
  platforms: "YouTube (primary), Spotify, Apple Music"
  timezone_peak: "UTC+3 (Moscow)"

upload_schedule:
  cadence: "1 video per month (target)"
  best_day: "Friday or Saturday"
  best_time: "16:00–18:00 local time"
```

---

## Visual Identity

```yaml
visual_identity:
  # Color palette — used by motion_designer and graphic_designer
  primary: "#1A1A2E"        # Deep navy — backgrounds, dark areas
  secondary: "#16213E"      # Dark blue — secondary elements
  accent: "#C8A96E"         # Warm gold — titles, key highlights
  highlight: "#E8D5B7"      # Cream — text on dark backgrounds
  text_light: "#FFFFFF"     # Pure white — main title text
  text_dark: "#111111"      # Near-black — text on light backgrounds

  color_grade_preset: "cinematic_teal_orange"
  # Matches: desaturated shadows with teal tint, amber/orange highlights, lifted blacks

  thumbnail_font: "Arial Bold"
  thumbnail_font_size_title: 80   # at 1920x1080 scale
  thumbnail_font_size_artist: 48
  thumbnail_layout: >
    Dominant subject center-left.
    Song title large, top-right or bottom-left, high contrast against dark background.
    Artist name smaller, below title.
    Color temperature: cool/desaturated — consistent with video grade.
    No busy elements, no borders, no watermarks.

  title_card_style: >
    Black or very dark background.
    Song title in accent gold (#C8A96E), centered, 80pt.
    Artist name in cream (#E8D5B7), centered, 48pt, below title.
    Fade in 1.5s, hold 3s, fade out 1s.

  outro_style: >
    Same dark background.
    "WARPMUZ" centered, accent gold.
    YouTube subscribe prompt below in cream.
    Last 5 seconds of every video.
```

---

## Artist Identity

```yaml
artist:
  name: "WARPMUZ"
  name_treatment: "WARPMUZ"    # Always ALL CAPS, no punctuation, no variations
  bio_short: "Cinematic ballads in Russian and beyond."
  website: ""
  social:
    youtube: "https://www.youtube.com/@WARPMUZ"
    instagram: ""
    tiktok: ""
    spotify: ""

  # For YouTube descriptions — boilerplate to append
  description_footer: |
    ─────────────────────────────
    🎵 WARPMUZ — Cinematic Ballads
    📺 Subscribe: https://www.youtube.com/@WARPMUZ
    ─────────────────────────────
```

---

## SEO Strategy

```yaml
seo:
  title_formula: "[Emotional hook] | [Song Title] — WARPMUZ (Official Music Video)"
  primary_keywords:
    - "Russian ballad"
    - "cinematic music"
    - "emotional music"
    - "piano ballad"
    - "sad Russian song"
  secondary_keywords:
    - "WARPMUZ"
    - "official music video"
    - "Russian music 2026"
    - "melancholic piano"
  avoid_keywords:
    - "cover" (unless it is actually a cover)
    - clickbait emotional language ("you will cry", "most emotional ever")
```

---

## Channel Rules (enforced by brand_guardian.md)

1. **One genre per channel.** If a concept doesn't fit `ballad / cinematic_ballad / folk`, it goes to a new channel.
2. **Artist name is always WARPMUZ** — never "Warp Muz", "warpmuz", "WARP MUZ".
3. **Color grade is always cinematic_teal_orange** for this channel. No exceptions without CEO + human approval.
4. **Thumbnail formula is fixed.** Changes require CEO + human approval and retroactive update of this file.
5. **All lyrics and metadata must be platform-safe.** No explicit content on this channel.

---

## Future Channels (Placeholder)

```yaml
# Add new channel blocks here as the company grows.
# Example:
# channel_name: "WARPMUZ EDM"
# genre_lock: "edm"
# youtube_url: ""
# status: planned
```
