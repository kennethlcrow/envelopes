# Build Notes — Envelopes

## Why I Built This

I wanted a simple envelope budgeting tool for personal use — something I could actually use daily in retirement without dealing with subscriptions, accounts, or apps that overcomplicate a straightforward method. Most budgeting apps do too much. This one does what I need and nothing else.

---

## Goals

- Lightweight and fast
- Local-first — no backend, no server, no accounts
- No authentication or account management
- Simple enough for everyday, non-technical use
- Aligned with zero-based budgeting principles

---

## Core Decisions

### Local Storage Instead of a Backend

All data lives in the browser's `localStorage`. No API calls, no database, no login. This keeps the app private, fast, and completely dependency-free. The trade-off is that data doesn't sync across devices — that's an acceptable limitation for personal use.

### No Budget Required at Envelope Creation

Envelopes are created with just a name and a color. Funding happens afterward through the "Assign Money" step. This aligns better with zero-based budgeting — you shouldn't need to know exact allocations before you can define a category. It also removes friction: create the envelope now, decide how much to fund it when income arrives.

### Integer-Based Money Handling

All monetary values are stored as integer cents (e.g., `$12.50` → `1250`). This eliminates floating-point precision issues that would otherwise accumulate across additions and subtractions. Conversion to dollars happens only at display time, in a single `formatMoney()` function.

---

## Challenges

### Drag-and-Drop Breaking Click Interactions

After reordering envelopes via drag-and-drop, all envelope cards would become unclickable for the rest of the session. The root cause was an `isDragging` flag that was set on `dragstart` but not reliably reset — the post-drag click that fires after `dragend` was being suppressed, but the flag was never cleared afterward.

Fixed by consuming and resetting the flag inside the click guard itself (`if (isDragging) { isDragging = false; return; }`), and increasing the `dragend` reset timeout to 50 ms to ensure the post-drag click arrives while the flag is still `true`. This way the click is suppressed once, the flag clears, and all subsequent clicks work normally.

---

## Current State

The app currently supports:

- Creating envelopes (name + color)
- Adding income, which increases the To Be Budgeted (TBB) balance
- Assigning funds from TBB to individual envelopes
- Moving funds between envelopes (budget reallocation, not a transaction)
- Recording expenses against envelopes
- Viewing transactions filtered by envelope
- Drag-and-drop reordering of envelopes (persisted)
- Exporting data as JSON and importing from a previous export
- Resetting all data with a single button

---

## Next Steps

- Recurring / scheduled budget allocations
- Monthly tracking and history view
- Data backup or optional sync
- Mobile UX improvements
