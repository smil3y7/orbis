# Orbis — Technical Guide

This is developer/reference documentation for Orbis. For how to *use* the app, open the in-app Help panel (`H` key) — this file covers implementation details instead.

## Tech Stack

- Vanilla JS · HTML5 Canvas — no build step, no framework
- [sql.js](https://sql.js.org/) 1.6 (SQLite compiled to WASM), loaded from cdnjs
- Fonts: Inter · JetBrains Mono (Google Fonts)
- Single HTML file — no server required, runs entirely in the browser
- Database format: SQLite (`.sqlite`)

## Database Schema

Orbis reads dream data from a user-supplied `.sqlite` file (expected tables: `SleepCycle`, `Dreams`, `Location`) and layers its own tables on top, created automatically on first load:

| Table | Purpose |
|---|---|
| `Atlas_Nodes` | Locations placed on the map (name, type, position, stability, parent group, search terms, notes) |
| `Atlas_Warps` | Connections between two locations (one-way or two-way) |
| `Atlas_NodeDreams` | Manually pinned links between a specific dream and a location |

## Color Scheme

| Swatch | Hex | Meaning |
|---|---|---|
| 🟦 | `#66fcf1` | Accent / Personal location |
| 🟥 | `#ff4d4d` | Warp |
| 🟨 | `#f7d794` | Archetype |
| 🟪 | `#a29bfe` | Dream Sign |
| 🟡 | `#ffd700` | Home |
| ⬛ | `#050506` | Background |

## Localization

All UI strings live in `lang/sl.js` and `lang/en.js` (`window.LANG_SL` / `window.LANG_EN`), each a single flat object of `'key': 'value'` pairs. Both files must define the exact same set of keys — the app has no built-in fallback dictionary, so a key missing from a lang file will render as its raw key name (e.g. `mv.set_home`) instead of readable text.

A handful of keys contain inline HTML (`<kbd>`, `<span>` tags for shortcut/indicator rows) and are applied via `data-i18n-html` instead of `data-i18n`; everything else is plain text.
