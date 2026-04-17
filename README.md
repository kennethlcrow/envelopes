# Envelopes

A lightweight, local-first budgeting app built around one idea:

👉 **Give every dollar a job — and know exactly what happened to it.**

No accounts. No syncing. No subscriptions.  
Just a clear, predictable way to manage your money.

---

## Why this exists

Most budgeting apps try to do everything.

They connect to banks, auto-categorize transactions, and pile on features until you spend more time managing the app than your money.

This project takes the opposite approach:

- No automation you don’t control  
- No hidden rules  
- No guessing  

Just a simple system that makes your financial decisions visible.

---

## Who this is for

This is for people who:

- Want full control over their money
- Don’t trust bank integrations or aggregators
- Prefer clarity over automation
- Like envelope budgeting, but not the complexity of existing tools

If you’ve ever thought:

> “Why is budgeting software so complicated?”

You’ll probably like this.

---

## Overview

Envelope budgeting is a method where every dollar is assigned a purpose.

This app lets you:

- Create envelopes (categories)
- Assign income to those envelopes
- Track spending against them
- Adjust your plan as life changes

Everything runs entirely in your browser.  
Nothing leaves your device.

---

## How it works

1. **Add income**  
   Record money coming in. This increases your “To Be Budgeted” (TBB).

2. **Assign money to envelopes**  
   Give every dollar a job using “+ Add Funds”.

3. **Spend from envelopes**  
   Record expenses against a category.

4. **Adjust your plan**  
   Move money between envelopes as needed.

---

## Key features

### Core budgeting

- Zero-based budgeting (ZBB)
- Envelope-based money allocation
- “To Be Budgeted” (TBB) tracking
- Move funds between envelopes (no fake transactions)

### Clarity & visibility

- Real-time summary:
  - Total Cash (all-time reality)
  - Total Assigned (your plan)
  - To Be Budgeted
- Filtered summary:
  - Income / Expenses / Net (current month, last month, or all time)

### Tracking & history

- Transaction history (income + expenses)
- Budget activity log:
  - Assignments
  - Removals
  - Transfers between envelopes

### Data control

- Runs entirely in the browser (no backend)
- Data stored in `localStorage`
- Export to JSON (full backup)
- Import from JSON
- Export to CSV (filtered data for analysis)

### Usability

- Drag-and-drop envelope reordering
- Click-to-filter transactions by envelope
- Simple UI — no setup required

---

## Why this is different

Most apps try to automate budgeting.

This one makes it explicit.

You decide:

- where money goes
- what gets priority
- how to adjust when things change

Nothing is hidden. Nothing is inferred.

It’s simple on purpose.

---

## Running the app

1. Download `index.html`
2. Open it in your browser

That’s it.

No install. No build. No dependencies.

---

## Data storage

All data is stored locally in your browser using `localStorage`.

Important:

- Clearing browser data will erase everything
- Use **Export JSON** to create backups
- Use **Import JSON** to restore your data

---

## Current limitations

- No cloud sync or multi-device support
- Data is tied to a single browser
- No recurring or scheduled budgets (yet)
- No authentication

---

## Future improvements

- Overspending indicators
- Per-envelope breakdown views
- Budget history by month
- Optional sync (without compromising simplicity)

---

## Development approach

This project is being built using a combination of:

- ChatGPT (architecture, design, reasoning)
- Claude Code (implementation)

See [AI_COLLABORATION.md](./AI_COLLABORATION.md) for details.

---

## License

GNU General Public License v3.0