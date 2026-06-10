# on-this-day-in-art

An art discovery agent that surfaces one Met Museum artwork per request — by today's date, a user-specified date, or keyword search — with Chillomena Punk commentary generated via Gemini.

## Key files

| File | Purpose |
|------|---------|
| `agent/index.ts` | Entry point. Configures the Mastra `Agent` (Claude), spawns the MCP server subprocess via `MCPClient`, sets up memory and observability, and calls `serve()` to connect to Astropods. |
| `mcp/index.ts` | MCP server (stdio transport). Wraps the Met Museum public API and Gemini API as five tools. Spawned as a Bun subprocess by `agent/index.ts` — it is **not** a network service. |
| `astropods.yml` | Astropods platform config (build, interfaces). |
| `AGENT.md` | Astropods marketplace spec (frontmatter + public description). Not the same as this file. |
| `Dockerfile` | Two-stage Bun build. Both `agent/` and `mcp/` are copied into the runtime image. |
| `.env` | Local secrets. Never committed. |

## MCP tools (exposed by `mcp/index.ts`)

| Tool | What it does |
|------|-------------|
| `get_artworks_for_today` | Date search with three-tier fallback: exact date → month → full collection (`*`). Returns `search_tier` (1/2/3) so the agent can tell the user which tier was used. |
| `get_artwork_detail` | Fetches full metadata for one artwork by `objectID`. |
| `get_departments` | Lists all Met departments with IDs. |
| `search_artworks` | Keyword/artist/theme search. Fetches `limit * 5` candidates internally to survive transient Met API failures. |
| `get_gemini_summary` | Calls Gemini 2.5 Flash to generate ~150-word Chillomena Punk commentary for a given artwork. |

## Environment variables

| Variable | Where it comes from | Used by |
|----------|---------------------|---------|
| `ANTHROPIC_API_KEY` | `ast project configure` / Astropods Anthropic integration | Mastra / Claude LLM |
| `GEMINI_API_KEY` | `ast project configure` / Astropods secrets | MCP server (`mcp/index.ts`) |
| `GRPC_SERVER_ADDR` | Injected automatically by Astropods at runtime | `@astropods/adapter-mastra` |

**Important:** The MCP server subprocess inherits env vars from the agent process via `{ ...process.env, GEMINI_API_KEY: ... }` in `MCPClient`. If `GEMINI_API_KEY` is missing, the MCP server starts but `get_gemini_summary` silently fails.

## Running locally

```bash
bun install
# Add ANTHROPIC_API_KEY and GEMINI_API_KEY to .env, then:
ast dev
# Playground at http://localhost:3100
```

## Architecture

```
User message
     │
     ▼
Astropods Platform (Web Chat UI + Messaging)
     │
     ▼
agent/index.ts  (Mastra Agent — Claude claude-sonnet-4-5)
     │  stdio subprocess
     ▼
mcp/index.ts  (MCP Server — @modelcontextprotocol/sdk)
     ├─ Met Museum API  (public, no auth)
     └─ Gemini API      (GEMINI_API_KEY)
```

## Deployment

Deployed via Astropods using the GitHub integration (push to `main` triggers a build). The `ast blueprint push` CLI builder currently has infrastructure issues — use the GitHub flow instead.

