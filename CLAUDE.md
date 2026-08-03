# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**SITREP — Briefing de Mercado** is a zero-dependency Progressive Web App (PWA) that generates AI-powered market intelligence briefings in Portuguese. The entire application lives in a single `index.html` file with all CSS and JavaScript inlined. There is no build step, no package manager, and no framework.

## Development

To run locally, serve the directory with any static file server:

```bash
python3 -m http.server 8080
# or
npx serve .
```

Then open `http://localhost:8080`. The app requires an Anthropic API key (entered in-browser and saved to `localStorage` as `sitrep_key`).

There are no tests, no linter, and no CI pipeline. Changes to `index.html` take effect on page reload.

## Architecture

### Single-file structure

Everything — styles, markup, and script — is in `index.html`. Supporting files:

| File | Purpose |
|---|---|
| `sw.js` | Service worker: cache-first for app shell, network-only for `api.anthropic.com` |
| `manifest.json` | PWA manifest for installability |
| `icons/` | App icons (192px and 512px) — note: `manifest.json` and `sw.js` reference `icons/icon-192.png` and `icons/icon-512.png` under a subdirectory |

### How the app works

The `CATEGORIES` array (defined at the top of the `<script>` block) drives the entire UI. Each category object has:
- `id` — used as DOM IDs (`card-{id}`, `body-{id}`)
- `label` — displayed as the card header
- `prompt` — the exact Claude prompt sent to the API; each instructs Claude to return **only a raw JSON array** with no markdown fences

On "generate", `runOne(id)` calls the Anthropic Messages API directly from the browser using the `anthropic-dangerous-direct-browser-access: true` header. The model is `claude-sonnet-4-6` with the `web_search_20250305` built-in tool enabled, so Claude performs live web searches before answering.

The response parsing strips any accidental markdown fences and `JSON.parse()`s the text blocks from `data.content`.

### Item schema

Each card renders a list of items following this shape returned by Claude:

```json
{
  "title": "string",
  "description": "string (1-2 sentences)",
  "tag": "VIRAL | EMERGENTE | SAZONAL | POR EXPLORAR | NICHO | CRESCIMENTO | DECLÍNIO | FINANCIAMENTO",
  "score": 1-5,
  "source_name": "string",
  "source_url": "URL string"
}
```

The `score` field drives the ▮▮▮▯▯ signal bars rendered by `bars()`.

## Design Conventions

- **Language**: UI strings and Claude prompts are in **Portuguese (pt-PT)**
- **Aesthetic**: military terminal — dark amber-on-black, IBM Plex Mono for all labels/buttons/code, IBM Plex Sans for body text
- **Color tokens** (defined as CSS custom properties on `:root`):
  - `--amber` / `--amber-dim` — primary accent
  - `--teal` — tags and signal bar highlights
  - `--danger` — error states
  - `--line` — borders and inactive bars
  - `--surface` / `--surface-2` — card backgrounds
- Cards are a simple 2-column CSS Grid, collapsing to 1 column below 720px

## Key Constraints

- **No external JS** — no CDN scripts; the CSP-compatible design avoids external dependencies entirely
- **API key in localStorage** — stored under `sitrep_key`; the key is never sent anywhere except `api.anthropic.com`
- **Service worker versioning** — when updating `APP_SHELL` files, bump `CACHE_NAME` in `sw.js` (currently `sitrep-v2`) so clients get fresh assets
- **Prompt format is strict** — each category prompt explicitly instructs "APENAS um array JSON, sem texto antes ou depois, sem markdown". If you modify prompts, preserve this instruction or the JSON parsing will break
