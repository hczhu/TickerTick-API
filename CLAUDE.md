# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Documentation-only repository: it is the **public API contract** for the TickerTick stock-news API served at `https://api.tickertick.com`. There is **no application code, build, lint, or test** here — the deliverable is `README.md` (rendered on GitHub as the API docs) plus the curated source lists in `docs/`. The backend that actually implements and serves this API lives in a **separate repository, `TickerTick-backend-private`**; editing files here changes documentation, not API behavior.

## The two endpoints (the contract)

- `GET /feed?q=<query>&n=<1-200>&last=<story_id>` — latest matching stories, reverse-chronological. `last` paginates by an existing story's `id` (a 64-bit int passed as a string). Story `time` is epoch **milliseconds** (JS `Date.now()` semantics). Rate limit: **10 req/min per IP** (enforced; >5 requests in any 30s window → 30s block).
- `GET /tickers?p=<prefix>&n=<count>` — ticker / company-name autocomplete.

## The query language (the core artifact)

A context-free grammar; the full grammar and operator/term tables are in `README.md`:
- Operators: `(and ...)`, `(or ...)`, `(diff query1 query2)` (matches `query1` but not `query2`).
- Terms: `z:TICKER` (narrow), `tt:TICKER` / `TT:TICKER` (broad), `s:DOMAIN` (no dots/slashes, e.g. `s:wsj`), `E:entity` (lowercase, spaces → `_`), `T:story_type`.
- `T:` story types: curated, earning, market, sec, sec_fin, trade, ugc, analysis, industry.

`docs/Retrieve-most-important-news.md` documents **advanced terms that are NOT in the README's tables** — notably `tag:_best_` and story types like `T:fin_news`. The README term/type tables are not exhaustive; treat `docs/` as authoritative for those.

## docs/ source lists ↔ story types

The `docs/*.txt` files are curated lists of source-website domains that back the `T:` story-type terms:
- `top-news-sources.txt` → backs `T:curated` and `T:analysis`
- `industry-publications.txt` → backs `T:industry`
- `ugc-sources.txt` → backs `T:ugc` (user-generated content: Reddit, forums)
- `analysis-sources.txt` → analyst / Substack / blog sources

These are the documented source-of-truth lists. The live service uses its own copies in the backend repo, so changing a list here does not by itself change what the API returns.

## External-link fragility

`README.md` and the live product deep-link to specific paths and anchors on the **`master`** branch (e.g. `docs/top-news-sources.txt`, the `#story-types` heading). Renaming `docs/` files, changing section headings, or restructuring the README **breaks those external links**. Preserve file names and heading anchors when editing.

## Skill packaging

`skills/tickertick/SKILL.md` is the **single source of truth** for the skill. There is deliberately no copy at the repo root.

The repo is simultaneously a marketplace for two agents, which read different manifests, so both install from the one file:

| Agent | Marketplace manifest | Plugin manifest |
|---|---|---|
| Claude Code | `.claude-plugin/marketplace.json` | `.claude-plugin/plugin.json` |
| Codex | `.agents/plugins/marketplace.json` | `.codex-plugin/plugin.json` (`"skills": "./skills/"`) |

Both set the plugin source to the repo root (`./`), so `skills/<name>/SKILL.md` is the layout both expect.

**Do not add a symlink for a second copy.** Claude Code reads the plugin in place, so a symlink works there, but Codex *copies* the plugin tree into `~/.codex/plugins/cache/` and silently drops symlinks — the skill directory arrives empty and the skill disappears with no error. A real file at that one path is the only layout both agents handle.

The README's manual-install `curl` URLs point at `master/skills/tickertick/SKILL.md`; update them together if the file ever moves again.

## Related
- Backend implementation (scrapers, indexer, NLP via spaCy `en_core_web_trf`, Wikidata entity expansion): `TickerTick-backend-private`.
- Historical bulk news-data dumps are published on this repo's GitHub **Releases** page.
- Official Python wrapper: `pytickertick` (`pip install pytickertick`).
