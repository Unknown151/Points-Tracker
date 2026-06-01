# Battle Points Tracker — Build Spec

## Overview

A generic, mobile-first tabletop points tracker as a single `index.html` file.
No build tools, no dependencies, no backend. Works offline. Easy to self-host or share.

---

## Feature Requirements

### Point Trackers (left column)

Three counters, stacked vertically:

| Tracker | Default Label | Cap |
|---|---|---|
| Primary | Primary Objective Points | Configurable (default 50) |
| Secondary | Secondary Objective Points | None |
| Bonus | Bonus Points | None |

- Each tracker is a **card** with a full-width label header (gold border, dark background)
- Each card has stacked buttons on both sides: `−5` (small, top) / `−` (large, bottom) on the left, `+5` (small, top) / `+` (large, bottom) on the right
- Value displayed large and centred inside the card
- Primary cap is **enforced across all rounds combined** (not per-round) — e.g. if 30 scored in rounds 1–2, max 20 remaining for rounds 3–5
- Labels are **editable** — tapping the label lets you rename it (stored in localStorage)
- Primary cap is **editable** — tapping the `/50` in the Total header lets you change it

### Total Header (top of left column)

A card above the three trackers showing two numbers side by side:

- **Left column:** "Primary" — sum of primary points across all rounds (with `/[cap]` indicator)
- **Right column:** "All Points" — absolute total of primary + secondary + bonus across all rounds
- Separated by a vertical divider

### Round Sidebar (right column)

- Header: "ROUND"
- 5 rows, one per round (1–5)
- Each row shows the round number and the **round total** (primary + secondary for that round; `—` if untouched)
- Tapping a row switches to that round — **snapshots primary + secondary** on switch, **bonus flows freely** (not round-locked)
- Active round highlighted in gold
- Reset button at the bottom (↺) — confirms before wiping all data

### Persistence

- All state saved to `localStorage` on every change
- Restored on page load (survives refresh)
- Key: `bpt_state` (battle points tracker)

### Settings Panel

Accessible via a ⚙ icon in the top-right corner. A simple modal or slide-down panel with:

- **Primary cap** — number input, default 50
- **Tracker labels** — three text inputs to rename "Primary Objective Points", "Secondary Objective Points", "Bonus Points"
- **Number of rounds** — 3, 4, or 5 (default 5)
- Changes apply immediately and persist to localStorage

---

## Visual Design

### Colour palette (dark theme, CSS custom properties)

```css
--bg:        #0d0d0f   /* page background */
--bg2:       #141417   /* card backgrounds */
--bg3:       #1a1a1f   /* inset surfaces */
--border:    #2a2a32   /* subtle borders */
--text:      #e8e6e0   /* primary text */
--text2:     #b8b4aa   /* secondary text */
--text3:     #706c62   /* muted/label text */
--gold:      #d4a84b   /* primary accent */
--gold-bright: #e8c060 /* headings */
--gold-dark: #8a6820   /* borders */
--border-gold: rgba(212,168,75,.3) /* gold borders */
```

### Fonts (Google Fonts, load via `<link>`)

- `Orbitron` — headings, labels, numbers
- `Share Tech Mono` — monospace values

### Layout

```
┌─────────────────────────────────────────────┐
│  BATTLE POINTS TRACKER            [⚙] [☽]  │  ← sticky header
├────────────────────────┬────────────────────┤
│  TOTAL                 │  ROUND             │
│  [Primary | All Pts]   │  1   —             │
│                        │  2   —             │
│  PRIMARY OBJ POINTS    │  3   —  (active)   │
│  [-5][-] [val] [+][+5] │  4   —             │
│                        │  5   —             │
│  SECONDARY OBJ POINTS  │                    │
│  [-5][-] [val] [+][+5] │  [↺ reset]         │
│                        │                    │
│  BONUS POINTS          │                    │
│  [-5][-] [val] [+][+5] │                    │
└────────────────────────┴────────────────────┘
│  Battle Points Tracker · github.com/...     │  ← footer
```

- Left column: `flex: 1`, right sidebar: `width: 72px`, gap: `.75rem`
- Layout: `display: flex; align-items: stretch` so sidebar matches left column height
- Sidebar round rows: `flex: 1` so they fill available height evenly
- Full viewport height feel on mobile

### Buttons

- `pts-tracker-btn` (±1): `42×42px`, rounded, border, large font
- `pts-tracker-btn-5` (±5): `42×20px`, rounded, smaller font, muted colour — stacked above the ±1 button

### Responsive

- Mobile-first (base styles for ~390px width)
- At `min-width: 768px`: centre content, `max-width: 600px`
- Tab nav scrolls horizontally on small screens (`overflow-x: auto; scrollbar-width: none`)

---

## File Structure

```
/
├── index.html   ← entire app (HTML + embedded CSS + embedded JS)
└── README.md
```

No other files needed.

---

## JS Architecture

All state in module-level variables:

```js
let primaryPts   = 0;
let secondaryPts = 0;
let bonusPts     = 0;
let currentRound = 1;
const roundStates = {};   // { [round]: { primary, secondary } }

// Settings (editable)
let primaryCap     = 50;
let labelPrimary   = 'Primary Objective Points';
let labelSecondary = 'Secondary Objective Points';
let labelBonus     = 'Bonus Points';
let totalRounds    = 5;
```

Key functions:

| Function | Purpose |
|---|---|
| `selectRound(n)` | Snapshot current round, load saved state for `n` |
| `changePts(type, delta)` | Increment/decrement a counter; enforce primary cap |
| `renderAll()` | Update all DOM values and round sidebar |
| `saveState()` | Serialize all state to `localStorage` |
| `loadState()` | Restore from `localStorage` on page load |
| `resetState()` | Confirm then wipe everything |
| `openSettings()` / `closeSettings()` | Toggle settings panel |
| `applySettings()` | Read settings form, update vars, save, re-render |
| `totalPrimaryAcrossRounds(excludeRound)` | Sum primary in all saved rounds except current |

---

## Notes for Implementation

- All CSS and JS **embedded in `index.html`** — no external files except Google Fonts
- `localStorage` key is `bpt_state` — store a single JSON blob:
  ```json
  {
    "primaryPts": 10, "secondaryPts": 5, "bonusPts": 2,
    "currentRound": 2,
    "roundStates": { "1": { "primary": 10, "secondary": 5 } },
    "primaryCap": 50,
    "labelPrimary": "Primary Objective Points",
    "labelSecondary": "Secondary Objective Points",
    "labelBonus": "Bonus Points",
    "totalRounds": 5
  }
  ```
- Confirmation before reset: `confirm('Reset all points and rounds?')`
- Settings changes take effect immediately (no save button needed; just close)
- Editable labels: clicking the `.pts-tracker-label` turns it into an `<input>` inline, blurring saves it
- The `/[cap]` under the Primary total in the header should update when cap changes in settings

