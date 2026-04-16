# Envelopes

A lightweight, local-first envelope budgeting app built on zero-based budgeting principles. No backend, no accounts, no setup — just open and start budgeting.

---

## Overview

Envelope budgeting is a method where every dollar is assigned a purpose. This app lets you create spending envelopes, fund them from your available cash (To Be Budgeted), and track expenses against each one.

It runs entirely in the browser and stores all data in `localStorage`. Nothing leaves your device.

---

## How to Use

1. **Create an envelope** — Click "+ New Envelope", give it a name and a color. No budget required at creation.
2. **Add income** — Click "+ Income" to record cash coming in. This increases your "To Be Budgeted" (TBB) balance.
3. **Assign funds to an envelope** — While TBB is positive, click "+ Add Funds" on any envelope card to allocate money to it.
4. **Transfer funds between envelopes** — Use the "⇄ Move" button on any card to shift budget from one envelope to another.
5. **Record expenses** — Click "+ Expense", select an envelope, and enter the amount. This reduces that envelope's remaining balance.
6. **View envelope transactions** — Click on an envelope card to filter the transaction list to that envelope only. Click again to deselect.
7. **Reorder envelopes** — Drag and drop cards to rearrange them. Order is persisted.
8. **Reset all data** — Click "Reset Data" in the footer to wipe all envelopes and transactions and start fresh.

---

## Features

- Create envelopes with a name and color — no predefined budget required
- Zero-based budgeting: assign every dollar of income to an envelope
- "To Be Budgeted" counter keeps track of unassigned cash
- Add income and expense transactions
- Move funds between envelopes without touching real transaction history
- Drag-and-drop envelope reordering
- Click-to-filter transaction list by envelope
- Export data as JSON / Import from a previous export
- Reset all data with a single click
- No backend — all data persists in browser `localStorage`

---

## Current Limitations

- Data is stored only in the current browser — no sync, no cloud backup
- Clearing browser data or switching browsers will lose all data (use Export to back up)
- No recurring or scheduled budgets
- No multi-user or multi-device support
- No authentication

---

## Tech Notes

- Single HTML file — no build step, no install
- Vanilla JavaScript, no frameworks
- Tailwind CSS via CDN
- Persistence via `localStorage` (keys: `envelopes`, `transactions`)
- All monetary values stored as integer cents to avoid floating-point issues

---

## Future Improvements

- Scheduled / recurring budget allocations
- Multi-device sync or cloud backup
- Improved mobile experience
- Budget history by month
- Bulk import from CSV

## Development Approach

See [AI_COLLABORATION.md](./AI_COLLABORATION.md) for how this project is being built using ChatGPT and Claude Code.