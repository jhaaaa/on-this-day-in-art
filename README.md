# on-this-day-in-art

An art discovery agent that surfaces one Metropolitan Museum of Art artwork connected to today's date, any date you choose, or search keywords — with commentary by **Chillomena Punk**, a deadpan and confidently misinformed art critic inspired by Philomena Cunk.

## Quick start

```bash
# Install dependencies
bun install

# Configure environment variables (run once)
ast project configure
# → set ANTHROPIC_API_KEY and GEMINI_API_KEY when prompted

# Start the agent locally
ast dev
```

The playground is available at `localhost:3100` during dev. Try:
- `"today"` — surfaces an artwork connected to today's date
- `"June 9"` or `"12/25"` — surfaces an artwork for any date you pick
- `"Show me flowers"` or `"How about a Rembrandt"` - surfaces an artwork for any keywords you enter

## Project structure

```
on-this-day-in-art/
├── agent/
│   └── index.ts          # Agent entry point (Mastra + MCPClient)
├── mcp/
│   └── index.ts          # MCP server: Met Museum API + Gemini tools
├── astropods.yml         # Agent specification
├── Dockerfile            # Agent container
├── .env                  # Local environment variables (not committed)
└── package.json
```

## How it works

```
User message
     │
     ▼
agent/index.ts  ─── MCPClient (stdio) ──►  mcp/index.ts
  (Claude)                                  │
     │                                      ├─ get_artworks_for_today  →  Met API (3-tier fallback)
     │                                      ├─ get_artwork_detail      →  Met API
     |                                      ├─ search_artworks         →  Met API search by keyword
     │                                      └─ get_gemini_summary      →  Gemini 2.5 Flash
     │
     ▼
Chillomena Punk commentary + artwork card
```

### Three-tier date fallback

The Met's API searches text metadata, not a calendar. To handle dates with sparse results:

1. **Tier 1** — search for `"June 9"` (exact date string)
2. **Tier 2** — if no results, search for `"June"` (month only)
3. **Tier 3** — if still nothing, search `"*"` (full collection)

## Configuration

| Variable | Source | Description |
|----------|--------|-------------|
| `ANTHROPIC_API_KEY` | `ast project configure` | Required — powers agent reasoning, tool orchestration, and response formatting |
| `GEMINI_API_KEY` | `ast project configure` | Required — powers Chillomena Punk commentary |

The MCP server subprocess inherits `GEMINI_API_KEY` from the agent process at runtime.

## MCP tools

| Tool | What it does |
|------|-------------|
| `get_artworks_for_today` | Date search with three-tier fallback |
| `get_artwork_detail` | Full metadata for a single artwork |
| `search_artworks` | Keyword search across the collection |
| `get_gemini_summary` | Generate Chillomena Punk commentary via Gemini |

## Related

- Website version: [onthisdayin.art](https://onthisdayin.art) / [otdia GitHub repo](https://github.com/jhaaaa/otdia)
