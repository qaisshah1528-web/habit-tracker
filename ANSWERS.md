# ANSWERS.md

---

## 1. How to run

No build step, no dependencies, no install.

```bash
open index.html
```

Or serve it locally to avoid `file://` quirks in some browsers:

```bash
npx serve .
# → http://localhost:3000
```

Or:

```bash
python3 -m http.server 8080
# → http://localhost:8080
```

The only external resources are Google Fonts (loaded over CDN). Everything else is self-contained in `index.html`.

---

## 2. Stack & design choices

**Why vanilla HTML/CSS/JS?**

A habit tracker is a CRUD app with local state and no server. A React/Vue setup would add a build pipeline, a `node_modules` folder, and a bundler for what amounts to ~400 lines of logic. Vanilla JS handles it cleanly — one file, zero dependencies, instant load, nothing to configure. The constraint also forces deliberate decisions: every render is explicit, every event listener is intentional, there's nowhere to hide behind framework magic.

**Design decision 1: The grid uses a fixed column for today, with a full-column background wash**

Today's column gets a faint violet background that spans every row of the grid — not just the header cell. I implemented this as a wrapper `div` with `today-col-bg` applied per cell, creating a vertical stripe that runs the full height of the habit list.

The alternative (just coloring the header) fails the "glance test": when you have 10 habits, the header is off-screen and you've lost your orientation. The column wash means no matter where you're looking in the grid, you know which column is today. The today checkmark also uses violet (the column accent color) instead of green, so checked-today cells read distinctly from checked-past cells. This matters: "I did it today" should feel different from "I did it last Wednesday."

**Design decision 2: Streak badges use a three-state color system, not a gradient**

`streak === 0` → muted gray pill. `streak 1–6` → amber. `streak ≥ 7` → green. I chose hard thresholds over a smooth gradient because thresholds create legible categories. At a glance, green means "this one is working," amber means "building," gray means "not started or broken." A gradient would require you to judge relative position; the three-state system is instantly readable across 15 habits.

The amber/green also deliberately echo traffic light semantics (effort → reward) without being literal — I avoided red for "zero streak" because the absence of a streak isn't a failure state worth flagging anxiously every morning.

**Week starts on Monday**

Monday is the conventional week start for calendars in most of the world outside North America, and more importantly, it front-loads the "work week" visually — the grid reads left-to-right as Mon → Sun, matching how most people mentally schedule habits ("I want to do this on weekdays"). A Sunday start buries Saturday/Sunday together at the end, which works for an American calendar but breaks the mental model for habit tracking.

**Streak counting: up to today, or yesterday if today is unchecked**

The streak counts up to and including today *if* today is checked. If today is not yet checked, it counts from yesterday backward. Rationale: showing a broken streak first thing in the morning (before you've had a chance to check today's habit) is punishing and inaccurate. You haven't *broken* anything yet — you just haven't done it yet. This avoids the anxiety of seeing "0" every morning before your morning run. The streak collapses to 0 only if yesterday was also unchecked.

---

## 3. Responsive & accessibility

**360px phone vs 1440px laptop**

The grid has `min-width: 580px` to preserve column legibility — seven 40px check cells plus the habit name column cannot meaningfully compress below that without the cells becoming untappable. On narrow screens, the grid wraps in a scrollable container (`overflow-x: auto`), so the full week is accessible with a horizontal swipe. The header, stats bar, and add row all reflow naturally with flexbox wrapping.

At 1440px, the app is capped at `max-width: 900px` and centered. The habit name column stretches with `minmax(120px, 1fr)` so it uses the available space without letting check cells drift too far apart.

The stats bar uses `flex-wrap: wrap` and at 400px collapses to a 2×2 grid via a media query.

**Accessibility — what I handled**

Keyboard navigation: all interactive elements (check buttons, nav buttons, add button, example pills, action buttons) are focusable and respond to Enter/Space. Check buttons explicitly handle `keydown` for Space (which browsers don't always fire as a `click` on non-native button elements). Tab order follows visual reading order.

ARIA labels: every check button has an `aria-label` that reads out the habit name and the date in full ("Check Exercise for Monday, 2 June"). The grid uses `role="table"`, `role="row"`, `role="columnheader"`, and `role="cell"` so screen readers can announce the habit/day intersection. Streak badges are wrapped in a cell with an `aria-label` that reads "Current streak: N days."

Focus management: after adding a habit, focus returns to the add input. After a rename commits (blur or Enter), focus returns naturally to the document.

Color contrast: all text-on-background combinations meet WCAG AA. The dark theme uses `#f0f0f2` on `#0e0e10` for body text (contrast > 12:1). The amber streak badge uses `#fbbf24` on `#3a2a0a`, which passes AA at the 0.7rem badge size.

**What I knowingly skipped**

Live region announcements for check state changes (`aria-live`). When you toggle a checkmark, a screen reader user currently gets no announcement that the state changed — they'd need to re-focus the button and read its `aria-pressed` attribute. Adding `aria-live="polite"` to a status region and injecting "Exercise checked for Monday" on each toggle would fix this. I skipped it because it adds meaningful complexity (you need to debounce announcements when someone rapidly checks/unchecks) and the `aria-pressed` attribute already communicates state to any screen reader that respects it.

---

## 4. AI usage

**What I used AI for:**

1. **Initial scaffold:** I asked Claude to generate a starting point for the grid layout — specifically the CSS grid template with `repeat(7, 40px)` for the day columns. It gave me a grid with fixed pixel columns for everything including the habit name. I changed the name column to `minmax(120px, 1fr)` so it stretches to use available space on wide screens rather than staying fixed-width, which would leave a lot of dead space on desktop.

2. **Streak algorithm:** I asked Claude to write a `calcStreak` function. The initial version always checked from today backward without the "if today is unchecked, start from yesterday" branch. I added that branch myself after thinking through the UX — the AI's version would show a broken streak every morning before you'd checked anything, which is the wrong behavior.

3. **Color palette iteration:** I described the aesthetic direction ("dark, editorial, not purple-gradients-on-white") and asked for a palette suggestion. Claude gave me a monochromatic dark scheme with a single green accent. I added the violet/indigo accent (`#7c6ef5`) specifically for the today column because I wanted the "where am I now" indicator to be a distinct hue from the "success" green — two different semantic meanings should use two different colors. I also added the amber for mid-range streaks, which wasn't in Claude's original suggestion.

4. **ARIA markup:** I asked Claude to add ARIA roles to the grid HTML. The initial output used `aria-label` on the table div but didn't go deeper. I extended it to add `role="rowheader"` to the habit name cell and `aria-label` to the streak cell so screen readers would have complete row context.

---

## 5. Honest gap

**The weakest part is the rename UX.**

Right now, renaming is triggered by clicking a small ✎ icon that only appears on hover. On touch devices, hover doesn't exist, so there's no affordance for renaming at all — you'd never know it was possible. The icon-only trigger is also a discoverability problem on desktop: new users won't know to hover.

With another day, I'd replace this with a tap-to-edit interaction: clicking the habit name itself activates an inline input, styled as an underlined text field within the existing row. This is the pattern used by linear.app and Notion for inline renaming. I'd also add a long-press gesture on mobile as a secondary path. The current approach works but it's the kind of thing a thorough user test would surface immediately.
