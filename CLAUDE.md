# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Taskmaster** is a single-file, static web application — a game master interface for running social party games with scoring, challenges, and card-based mechanics. The entire application lives in `index.html` (~6,400 lines) with no build system, no dependencies, and no external tooling.

To run: open `index.html` directly in a browser. There are no build, lint, or test commands.

## File Layout (within `index.html`)

| Lines | Content |
|-------|---------|
| 1–2257 | HTML structure + embedded CSS |
| 2258–2590 | JS constants: `PLAYER_COLORS`, `EMOJIS`, deck arrays (`DRINKS`, `NOMS`, `SOCIALS`, `HOTSEATS`, `CHALLENGES`, `CURSES`, `GAMES`, `ITEMS`), hotspot definitions |
| 2591–3623 | Utility functions, core helpers |
| 3624–5500 | Game logic, rendering functions, modal dialogs |
  5500–6250 | Admin panel, deck/hotspot editors, special game modes |
| 6250–6389 | Event listeners and init (`loadFromStorage()` → `renderEverything()`) |

## Global State

Three primary state containers:

```js
players[]        // Teams: { id, name, members[], color, score, thirst, tokPos }
maps[]           // Board zones: { id, name, imageData, hotspots[], annotations[], playerTokPos, npcTokPos }
sessionState     // Runtime: { activeEffects, sessionLog, annotations, playerConditions,
                 //            deckState, currentRule, timers, effectTemplates,
                 //            initiativeIndex, notes }
```

Supporting globals: `activeMapId`, `sidebarTab` (`"session"` | `"admin"`), `adminMode` (`"run"` | `"edit"`), `playerDisplay` (team-safe mode), `actionHistory[]`, `redoHistory[]`.

## Core Systems

**Rendering** — `renderEverything()` is the master re-render. Individual `render*()` functions handle players, tokens, admin panel, card decks, and spotlight overlays. Call `renderEverything()` after any state change that affects multiple UI regions.

**Persistence** — State is saved to `localStorage` (up to ~3 MB). Use `scheduleSave()` (debounced) after mutations, never `saveToStorage()` directly in hot paths. JSON export/import via `exportSave()` / `importSave()`.

**Undo/Redo** — Call `pushHistory(description)` before any destructive state mutation. `undo()` / `redo()` replay from `actionHistory[]` / `redoHistory[]`.

**Card Decks** — Eight named decks keyed by string (`DRINKS`, `NOMS`, etc.). `drawFromDeck(key)` handles animation and state; `rebuildDeckState(key)` reshuffles. Deck runtime state lives in `sessionState.deckState`.

**Hotspots** — Clickable zones on the map canvas. Each hotspot has a `type` (e.g. `dice`, `boss`, `shrine`, `jukebox`) dispatched through `handleHotspot(h)`, which opens the appropriate modal.

**Modals** — 15+ specialized modal openers (`openTaskCardModal()`, `openBossBattleModal()`, `openWheelModal()`, etc.). All modals share a common overlay pattern.

## UI Layout

```
.layout
├── .mapCol        Canvas-based interactive board (left ~65%)
└── .sidebar       Leaderboard + admin panel (right ~35%)
```

The sidebar has two tabs: `"session"` (player-facing) and `"admin"` (game master). Admin has sub-modes `"run"` and `"edit"`.

## Key Patterns

- All DOM references are cached in the `els` object at startup — do not query the DOM repeatedly.
- CSS uses custom properties (`--gold`, `--dark`, `--border`, `--text`) defined at `:root`; prefer these over hardcoded colours.
- The app scales with `clamp()` and repositions the sidebar on narrow viewports — avoid fixed pixel sizes in new UI.
- `Image.png` is the default board background; custom boards are stored as base64 data URLs in `maps[].imageData`.
