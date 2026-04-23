# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Open Brain (OB1) is a persistent AI memory system — one database (Supabase + pgvector), one MCP protocol, any AI client. This repo contains the extensions, recipes, schemas, dashboards, integrations, primitives, and skills that the community builds on top of the core Open Brain setup. It is a **contribution monorepo**, not a single application — each contribution lives in its own subfolder and is shipped independently.

**License:** FSL-1.1-MIT. No commercial derivative works.

## Repo Structure

```
extensions/     Curated, ordered learning path (6 builds). Curated — do NOT add without maintainer approval.
primitives/     Reusable concept guides (must be referenced by 2+ extensions). Curated.
recipes/        Standalone capability builds. Open for community contributions.
schemas/        Database table extensions. Open.
dashboards/     Frontend templates (Vercel/Netlify). Open.
integrations/   MCP extensions, webhooks, capture sources. Open.
skills/         Reusable AI client skills and prompt packs. Open.
server/         Reference remote MCP server (Supabase Edge Function, Deno + Hono + @modelcontextprotocol/sdk). Canonical pattern.
docs/           Setup guides, FAQ, companion prompts.
resources/      Official companion files and packaged exports.
```

Every category (except `docs/`, `resources/`, `server/`) has a `_template/` subfolder — use it as the starting point for new contributions.

Each contribution lives in its own subfolder (e.g., `recipes/my-thing/`) and must contain `README.md` + `metadata.json`. Skills additionally require at least one plain-text skill artifact (`SKILL.md`, `*.skill.md`, or `*-skill.md`).

## How Contributions Are Gated

Every PR is automatically reviewed by `.github/workflows/ob1-gate.yml` before any human review. The gate enforces 15 machine-readable rules (folder structure, required files, metadata schema, SQL safety, no secrets, PR title prefix, remote-MCP pattern, etc.). When generating or editing contributions, match these rules or the PR will fail:

- **Metadata schema** — `metadata.json` is validated against `.github/metadata.schema.json`. Required: `name`, `description`, `category`, `author.name`, `version` (semver regex `^\d+\.\d+\.\d+$`), `requires.open_brain: true`, `tags` (≥1), `difficulty` (`beginner|intermediate|advanced`), `estimated_time`.
- **Scope check** — all changed files must live within a single contribution folder (or `docs/`, `.github/`). Don't mix contributions in one PR.
- **Internal links** — relative links in READMEs must resolve.
- **Tool audit link** — extensions and integrations must link to `docs/05-tool-audit.md`.
- **Docs-only PRs** (title prefix `[docs]` or no contribution folders touched) skip contribution checks.

When Claude is generating a new contribution, start by copying the relevant `_template/` directory — it already satisfies the gate.

## Reference MCP Implementation

`server/index.ts` is the canonical remote-MCP server pattern. It is a **Supabase Edge Function** (Deno runtime, `server/deno.json` pins imports: `@modelcontextprotocol/sdk`, `@hono/mcp`, `hono`, `zod`, `@supabase/supabase-js`). When building new MCP extensions, mirror this shape. Do NOT introduce a local Node.js MCP server, `StdioServerTransport`, or `claude_desktop_config.json` — the gate rejects them.

Connection flow: Claude Desktop / client → Settings → Connectors → Add custom connector → paste Edge Function URL. Auth is a bearer `MCP_ACCESS_KEY` set as a Supabase secret.

## Guard Rails

- **Never modify the core `thoughts` table structure.** Adding columns is fine; altering or dropping existing ones is not. The gate scans SQL files for this.
- **No credentials, API keys, or secrets** in any file. Use environment variables and document what the user must set.
- **No binary blobs over 1 MB.** No `.exe`, `.dmg`, `.zip`, `.tar.gz`.
- **No `DROP TABLE`, `DROP DATABASE`, `TRUNCATE`, or unqualified `DELETE FROM`** in SQL files.
- **MCP servers must be remote** (Supabase Edge Functions). See `server/index.ts` and `docs/01-getting-started.md` Step 7.
- **Tables created by extensions must GRANT to `service_role`** — Supabase no longer auto-grants CRUD on new projects.

## PR Standards

- **Title format:** `[category] Short description` — e.g., `[recipes] Email history import via Gmail API`. Category must be one of `recipes|schemas|dashboards|integrations|skills|primitives|extensions|docs`.
- **Branch convention:** `contrib/<github-username>/<short-description>`.
- **Commit prefixes:** `[category]` matching the contribution type.
- PR description must confirm you tested on your own Open Brain instance.

## Commands

This repo has no build step — contributions are shipped as source (SQL, README, skill text, Edge Function code). The only tooling that runs here:

- **Markdown lint** (local check before opening PR): `markdownlint-cli2 --config .github/.markdownlint.jsonc '**/*.md'`. The `.markdownlint.jsonc` config is deliberately permissive — only real problems are caught.
- **Metadata schema validation** (what CI runs): `check-jsonschema --schemafile .github/metadata.schema.json <dir>/metadata.json`.
- **Edge Function** in `server/` is run via Supabase's Deno runtime in production; there is no local dev command committed here.

**Ignore `main.py`, `pyproject.toml`, `uv.lock`, `.python-version`, `.venv/`** — these are a local scratchpad artifact, not part of the project. Don't add Python code here or reference them in contributions.

## Key Files

- `CONTRIBUTING.md` — Source of truth for contribution rules, metadata format, README required sections, and the review process. Read this before generating a contribution.
- `.github/workflows/ob1-gate.yml` — The automated PR gate (the 15 rules above).
- `.github/metadata.schema.json` — JSON Schema validated by the gate.
- `.github/PULL_REQUEST_TEMPLATE.md` — PR description template.
- `docs/01-getting-started.md` — Full Open Brain setup (referenced by many contributions; use as the canonical remote-MCP pattern).
- `docs/05-tool-audit.md` — MCP tool audit guide; extensions/integrations must link to it.
- `<category>/_template/` — Starter scaffold for new contributions in each category.
- `LICENSE.md` — FSL-1.1-MIT terms.
