# Habit Tracker

A single-page weekly habit tracker. Add habits, tick them off each day, track streaks, and navigate between weeks. All data persists in `localStorage`.

---

## How to run

**No build step. No dependencies. Open one file.**

```bash
# Option 1 — Just open it
open index.html          # macOS
start index.html         # Windows
xdg-open index.html      # Linux

# Option 2 — Serve locally (avoids any browser file:// quirks)
npx serve .
# then open http://localhost:3000

# Option 3 — Python one-liner
python3 -m http.server 8080
# then open http://localhost:8080
```

That's it. No `npm install`, no build, no framework tooling.

---

## Stack

Vanilla HTML + CSS + JavaScript. Single `index.html` file, no external dependencies except two Google Fonts (`DM Serif Display`, `DM Mono`, `Outfit`).

See `ANSWERS.md` for a full explanation of the stack choice and design decisions.

---

## Features

- Add, rename, and delete habits
- Weekly grid (Mon–Sun) with toggleable checkmarks
- Today's column is always highlighted
- Streak counter per habit (consecutive days up to and including today, or yesterday if today is unchecked)
- Week navigation: previous, next, "This week" shortcut
- Past weeks show their historical data; future days are disabled
- Stats bar: today's progress, weekly completion %, best streak, habit count
- Empty state with example habit pills to click
- Inline rename (click the ✎ icon, or just double-click the name row area)
- Full keyboard navigation (Tab, Enter/Space to toggle checks)
- Persists across page reloads via `localStorage`

---

## File structure

```
habit-tracker/
├── index.html    ← the entire app
├── README.md     ← this file
└── ANSWERS.md    ← assessment answers
```
