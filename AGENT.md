---
description: "Art discovery agent — one Met Museum artwork per request, with Chillomena Punk commentary. Say 'today', a date, or search by keyword."
tags: [art, met-museum, comedy, discovery]
authors: []
capabilities: [tool-use, mcp]
integrations:
  - anthropic
  - gemini
---

# On This Day in Art

An art discovery agent powered by the Metropolitan Museum of Art's public API. Surfaces one artwork connected to today's date — or any date the user picks — and presents it with a short commentary in the voice of **Chillomena Punk**: a deadpan, confidently misinformed art critic inspired by Philomena Cunk.

## Three modes

| Mode | How to trigger |
|------|---------------|
| **On This Day** | "today", "what's on today?", "show me today's art" |
| **Pick a Day** | Any date: "June 9", "March 3rd", "12/25", "1/15/1990" |
| **Search** | Keywords, artists, themes: "show me a Rembrandt", "something with flowers", "Japanese woodblock prints" |

## Tool pipeline

Each artwork request runs three tools in sequence:

1. `get_artworks_for_today` or `search_artworks` — finds the artwork by date (with three-tier fallback) or keyword
2. `get_artwork_detail` — fetches full metadata for the selected artwork
3. `get_gemini_summary` — generates ~150-word Chillomena Punk commentary via Gemini 2.5 Flash
