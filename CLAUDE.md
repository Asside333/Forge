# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FORGE is a French-language PWA (Progressive Web App) for commitment tracking and self-discipline. It helps users create and follow through on engagements ("what you said you'd do"), track time spent, log resistance to temptation, and review history.

## Architecture

**Single-file app** — all HTML, CSS, and JavaScript live in `index.html`. There is no build system, no framework, no npm. To "run" the app, open `index.html` in a browser or serve it with any static file server.

`sw.js` is a minimal service worker handling offline caching and push notification clicks.

### Screen System

The app uses a single-page screen-switching pattern. Only one `.screen` div is `.active` at a time:

- `#screenMain` — main view: either empty state (no active engagement) or engagement view with chrono
- `#screenForm` — form to create a new engagement
- `#screenRegistre` — history log with stats
- `#screenPhrases` — manage anchor phrases

### Overlay Stack (z-index layers)

Overlays appear above screens and are shown/hidden via the `.visible` class:

| Overlay | z-index | Trigger |
|---|---|---|
| `#morningOverlay` | 300 | First open of the day |
| `#nightOverlay` | 250 | After 23h with no active engagement |
| `#journalOverlay` | 150 | After manually stopping chrono |
| `#minimalOverlay` | 200 | "JUSTE ÇA" button |
| `#anchorOverlay` | 100 | 5-second countdown before chrono starts |
| `#checkinOverlay` | 100 | Daily check-in |
| `#confirmOverlay` | 100 | Verdict confirmation |

### State & Persistence

All state is stored in `localStorage`:

- `forge_data` → `{ current: Engagement|null, history: Engagement[], checkins: {} }`
- `forge_phrases` → `string[]` (anchor phrases shown before chrono)
- `forge_theme` → `'dark'|'light'`
- `forge_last_open` → ISO date string (for morning anchor detection)
- `forge_night_override` → stored in `sessionStorage`

An **Engagement** object contains: `text`, `minimal` (optional threshold), `deadline` (optional), `createdAt`, `totalSeconds`, `resistances[]`, `journal[]`, `status` (set at verdict: `'done'|'not-done'|'replaced'`).

### Core Flow

1. User creates an engagement via the form (`showForm()` → `createEngagement()`)
2. On main screen: start chrono → 5s anchor overlay with random phrase → `startChrono()`
3. Stop chrono → session seconds added to `current.totalSeconds` → journal overlay
4. Verdict buttons (FAIT / PAS FAIT / REMPLACÉ) → `confirmVerdict()` → moves `current` to `history`

### Design System

CSS custom properties are defined on `:root` (dark) and `body.light` (light mode):

- `--accent: #ff4a1c` (orange-red)
- `--done: #22c55e`, `--replaced: #eab308`
- Fonts: `JetBrains Mono` (mono labels/buttons), `Space Grotesk` (body/inputs)
- Max width: 480px, mobile-first
