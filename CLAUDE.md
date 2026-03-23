# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build step or server required. Open the file directly:

```bash
start martial-arts-scheduler.html        # Windows — opens in default browser
```

Or double-click the file in Explorer. It is a fully self-contained single HTML file.

## Deploying / Sharing

Because there are no dependencies, the file can be hosted on any static file host (GitHub Pages, Netlify, etc.) by simply serving `martial-arts-scheduler.html` as-is.

## Git Workflow

**Commit and push after every piece of work — no exceptions.** This ensures we never lose progress and can always roll back to a working state.

Remote: `https://github.com/emijoreyes64/dragon-dojo-scheduler`

```bash
git add martial-arts-scheduler.html
git commit -m "short imperative description of what changed"
git push
```

### When to commit

- After completing any feature or improvement, even small ones
- After fixing a bug
- Before starting a new task (so the current working state is saved)
- Any time a feature is "working" — don't wait until everything is perfect

### Commit message format

Use short imperative phrases that describe *what changed and why*, not *what you did*:

```
Add student enrollment count to class detail modal
Fix resize handle snapping on fractional pixel positions
Change week view scroll height to account for sticky header
```

Avoid vague messages like `update`, `fix stuff`, or `changes`.

## Architecture

Everything lives in one file (`martial-arts-scheduler.html`) in three ordered sections: **CSS → HTML → JavaScript**.

### Data model

All classes are stored in a single mutable `let classes = [...]` array. Each object has:

```js
{
  id, name, discipline, instructor, level,
  start,       // "HH:MM" 24-hour string
  duration,    // minutes (integer)
  capacity, room, days,   // days: array of DOW integers 0–6
  notes
}
```

`visibleDisc` (a `Set`) controls which disciplines are shown. `currentView`, `selectedDate`, and `miniDate` are the only other pieces of global state.

### Render pipeline

`render()` is the single entry point — it calls `renderMini()` then delegates to `renderWeek()`, `renderMonth()`, or `renderDay()` based on `currentView`. Every state change (drag-drop, form save, filter toggle) ends with a `render()` call to fully rebuild the DOM.

### Week view layout

`renderWeek()` builds a flex-based grid: a fixed header row + a scrollable body containing a time-label column (`wk-time-col`) and seven `wk-day-col` divs (one per day). Class blocks are `position:absolute` children of their `wk-day-col`. Vertical position and height are calculated from `HOUR_START`, `CELL_H` (52 px/hour), and `minsToPx()` / `pxToMins()`.

### Drag and drop

Three module-level variables hold drag state: `drag`, `dragGhost`, `dragTooltip`.

- **Move**: `startMoveDrag()` → `onMouseMove()` → `onMouseUp()`. On drop, the class's `start` is updated; if the target column has a different DOW, that DOW is swapped in/out of `cls.days`.
- **Resize**: `startResizeDrag()` → same `onMouseMove` / `onMouseUp`. Only `cls.duration` changes.
- Both snap to 15-minute intervals via `snapMins(m, 15)`.
- A single click (no movement, `drag.hasMoved === false`) falls through to `openDetail()`.

### Time helpers

All time math uses plain minute integers. Key helpers: `timeToMins(t)`, `minsToTime(m)`, `addMins(t, m)`, `snapMins(m, snap)`. Times are stored as `"HH:MM"` strings and displayed via `fmtTime()` (converts to 12-hour AM/PM).

### Modals

Two overlays: `form-overlay` (add/edit) and `detail-overlay` (read-only detail + edit shortcut). Both are shown/hidden by toggling the `.open` class. `editingId` tracks which class is being edited; `null` means a new class is being created.

## Key Constants

| Constant | Value | Meaning |
|---|---|---|
| `HOUR_START` | 6 | First visible hour in week view |
| `HOUR_END` | 23 | Last visible hour in week view |
| `CELL_H` | 52 | Pixels per hour in week grid |

To extend visible hours, change `HOUR_START` / `HOUR_END`; the layout scales automatically.

## Adding a New Discipline

1. Add an entry to `DISC_LABELS` and `DISC_COLORS`.
2. Add a matching `<option>` in the `#f-discipline` select (HTML).
3. Add CSS classes `.cb-<key>` (block color) in the discipline colors section.
