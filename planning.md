# Envelope Budgeting App — Implementation Plan

---

## 1. App Overview

**What it does:**
A single-page, offline-capable budgeting app using **zero-based budgeting**: every dollar you receive gets assigned to an envelope. The goal is always "To Be Budgeted = $0" — meaning no unassigned money is sitting idle, and you haven't assigned more than you have. Income transactions are the source of all money. Envelopes are where that money gets planned. Expenses draw from both your cash and an envelope's monthly allocation.

**Core design principles:**
- Zero dependencies beyond Tailwind CDN — no build step, no Node, no server
- Readable code first — every function will be short, named clearly, and commented for a learning reader
- Data lives in the browser — `localStorage` is the only database
- Portable — the whole app is one file you can email to yourself or open from a USB drive
- Three clearly labeled top-level numbers: **Total Cash**, **Total Assigned**, **To Be Budgeted**

**Key tradeoffs of a single HTML file + localStorage:**

| Tradeoff | Implication |
|---|---|
| No server | Data never syncs between devices — intentional for v1 |
| localStorage ~5MB limit | Fine for years of personal transactions (JSON is small) |
| No auth | Anyone with browser access can see the data — acceptable for personal use |
| Single file | Easy to share/version; all HTML and JS in one place is fine at this scale |
| localStorage can be cleared | Export/import is the backup strategy — must be obvious in the UI |

---

## 2. Data Model

**Two core entities:** `Envelope` and `Transaction`. Transactions are the single source of truth for all money movement. Envelopes hold only budget targets — they do not hold money.

### Envelope
```json
{
  "id": "env_1713200000000",
  "name": "Groceries",
  "monthlyBudget": 400,
  "color": "#4f46e5",
  "createdAt": "2026-04-16T00:00:00.000Z"
}
```

| Field | Type | Purpose |
|---|---|---|
| `id` | string | Unique, timestamp-based — no UUID library needed |
| `name` | string | Display name ("Groceries", "Rent", "Savings", etc.) |
| `monthlyBudget` | number | Dollars assigned to this envelope per month |
| `color` | string | Hex color for visual identification |
| `createdAt` | ISO string | For display/sorting |

> **Note on Savings:** Savings is just a regular envelope with a `monthlyBudget`. There is no special savings type. Setting aside $200/month for savings means creating a "Savings" envelope with `monthlyBudget: 200` and never adding expense transactions to it.

### Transaction
```json
{
  "id": "txn_1713200000001",
  "type": "expense",
  "envelopeId": "env_1713200000000",
  "amount": 52.47,
  "description": "Trader Joe's",
  "date": "2026-04-15",
  "createdAt": "2026-04-16T00:00:00.000Z"
}
```
```json
{
  "id": "txn_1713200000002",
  "type": "income",
  "envelopeId": null,
  "amount": 3000.00,
  "description": "April Paycheck",
  "date": "2026-04-01",
  "createdAt": "2026-04-16T00:00:00.000Z"
}
```

| Field | Type | Purpose |
|---|---|---|
| `id` | string | Unique identifier |
| `type` | `"income"` or `"expense"` | Determines sign and whether an envelope is required |
| `envelopeId` | string or `null` | Required for expenses; always `null` for income |
| `amount` | number | Always a positive number — `type` determines direction |
| `description` | string | Free text note |
| `date` | string (YYYY-MM-DD) | The actual transaction date (user-entered) |
| `createdAt` | ISO string | When the record was created |

**Relationships:**
- One Envelope → many expense Transactions
- Income Transactions belong to no envelope (`envelopeId: null`)
- Deleting an envelope cascades to delete all its expense transactions
- Deleting an income transaction reduces Total Cash but touches no envelope

---

## 3. Calculations — The Four Numbers

This is the core of the zero-based budgeting model. The top of the app always shows three summary numbers. Each envelope card shows a fourth. All four are calculated live on every render from raw data — nothing is stored.

---

### The key design rule

> **Expense transactions affect Total Cash and envelope spending only. They never affect Total Assigned or To Be Budgeted.**

TBB is a budgeting signal, not a cash signal. It answers the question: *"Of all the money I've ever received, how much is still unassigned to an envelope?"* Spending money you've already assigned doesn't change that answer.

---

### Number 1 — Total Cash

**"How much money do I actually have right now?"**

```
totalCash = sum(amount for ALL transactions where type = "income")
          − sum(amount for ALL transactions where type = "expense")
```

| Changes when | Direction |
|---|---|
| Income transaction added/deleted | + / − |
| Expense transaction added/deleted | − / + |
| Envelope budget changed | No change |

- Spans all time — never filtered by month
- This is the real-world number. It reflects your actual bank-like balance.

---

### Number 2 — Total Assigned

**"How much have I committed to spending across all envelopes per month?"**

```
totalAssigned = sum(monthlyBudget for ALL envelopes)
```

| Changes when | Direction |
|---|---|
| Envelope created or budget increased | + |
| Envelope deleted or budget decreased | − |
| Any transaction added/deleted | **No change** |

- Not filtered by month — envelope budgets are standing monthly targets
- This number represents your spending plan, not your actual spending

---

### Number 3 — To Be Budgeted (TBB)

**"Of all the money I've received, how much hasn't been assigned to an envelope yet?"**

```
totalIncome   = sum(amount for ALL transactions where type = "income")
totalAssigned = sum(monthlyBudget for ALL envelopes)

toBeBudgeted  = totalIncome − totalAssigned
```

| Changes when | Direction |
|---|---|
| Income transaction added | + |
| Income transaction deleted | − |
| Envelope created or budget increased | − |
| Envelope deleted or budget decreased | + |
| **Expense transaction added/deleted** | **No change** |

- **TBB > $0:** Unassigned money — put it in an envelope
- **TBB = $0:** Every dollar is assigned — goal achieved
- **TBB < $0:** Over-budgeted — you've assigned more than you've received

Notice: TBB uses `totalIncome` (gross inflow), not `totalCash` (net balance). This is the fix. Spending reduces `totalCash` but does not reduce `totalIncome`, so TBB is unaffected by expenses.

---

### Number 4 — Envelope Remaining (per envelope, per month)

**"How much of this envelope's budget is left to spend this month?"**

```
envelopeRemaining = envelope.monthlyBudget
                  − sum(amount for transactions where type    = "expense"
                                                  AND envelopeId = envelope.id
                                                  AND date within selected month)
```

| Changes when | Direction |
|---|---|
| Expense added to this envelope this month | − |
| Expense deleted from this envelope this month | + |
| Envelope's `monthlyBudget` changed | Recalculates |
| Month navigated | Recalculates for new month |
| Income added | **No change** |

- Filtered by the currently viewed month
- Resets each month (spending history from prior months is ignored)
- Goes negative if you overspend the budget (allowed, shown in red)
- This is a spending progress bar — not a cash balance

---

### Summary: what affects what

| Action | Total Cash | Total Assigned | TBB | Envelope Remaining |
|---|---|---|---|---|
| Add income | + | — | + | — |
| Delete income | − | — | − | — |
| Add expense | − | — | **—** | − |
| Delete expense | + | — | **—** | + |
| Create envelope / raise budget | — | + | − | recalculates |
| Delete envelope / lower budget | — | − | + | recalculates |
| **Move money between envelopes** | **—** | **—** | **—** | **recalculates both** |

Moving money is a budget reallocation — it touches only two `monthlyBudget` values. Nothing else changes.

---

### A worked example

**Step 1 — Receive paycheck:**
- Add income: `$3,000`
- Total Cash = `$3,000` | Total Income = `$3,000` | Total Assigned = `$0` | **TBB = $3,000**

**Step 2 — Assign money to envelopes:**
- Create: Rent `$1,200`, Groceries `$400`, Savings `$300`, Utilities `$150`, Dining `$200`, Gas `$100`, Fun `$150`, Misc `$500`
- Total Assigned = `$3,000` | **TBB = $3,000 − $3,000 = $0** ← done

**Step 3 — Spend money:**
- Add expense: `$52.47` → Groceries
- Total Cash = `$2,947.53` ← decreased
- Total Assigned = `$3,000` ← unchanged
- **TBB = $3,000 − $3,000 = $0** ← unchanged ✓
- Groceries remaining = `$400 − $52.47 = $347.53` ← decreased

**Step 4 — Receive more income mid-month:**
- Add income: `$500` freelance payment
- Total Cash = `$3,447.53` | Total Income = `$3,500`
- **TBB = $3,500 − $3,000 = $500** ← assign this to an envelope

**Step 5 — Over-budget scenario:**
- Create a new envelope: Subscriptions `$800`
- Total Assigned = `$3,800` | **TBB = $3,500 − $3,800 = −$300** ← warning shown
- Resolution: reduce Subscriptions to `$500` → TBB = `$0`

---

## 4. Moving Money Between Envelopes

### What it is

Moving money is a **budgeting action**, not a transaction. It does not record any real-world payment. It simply reallocates your existing plan — lowering one envelope's `monthlyBudget` and raising another's by the same amount. No transaction object is created or modified.

### What it changes

```
envelopeA.monthlyBudget -= amount
envelopeB.monthlyBudget += amount
```

That is the entire operation. Two fields on two objects in the `envelopes` array.

### What it does NOT change

| Value | Changed? | Why |
|---|---|---|
| Total Cash | No | No money moved in or out of the real world |
| Total Assigned | No | One budget goes down by exactly as much as the other goes up — net zero |
| TBB | No | TBB = totalIncome − totalAssigned; totalAssigned didn't change |
| Transaction list | No | This is not a transaction |

### What it does change

- `envelopeA.monthlyBudget` — decreases
- `envelopeB.monthlyBudget` — increases
- **Envelope Remaining for both** — recalculates immediately on re-render

### Validation

| Rule | Reason |
|---|---|
| From and To envelopes must be different | Moving to the same envelope is a no-op |
| Amount must be > 0 | Zero moves are pointless |
| Amount must be ≤ from-envelope's `monthlyBudget` | Prevent an envelope budget going below $0 |

The third rule keeps things clean: a budget can't go negative. If you need to pull more than an envelope has budgeted, reduce its budget to $0 and leave the rest in TBB.

### Suggested UI

A simple modal triggered by a **"Move Money"** button in the envelope card overflow menu (or a small arrow icon between cards). The form has exactly three fields:

```
┌─────────────────────────────────────┐
│  Move Money                         │
│                                     │
│  From:  [Dining Out        ▾]       │
│  To:    [Groceries         ▾]       │
│  Amount: [$  _________ ]            │
│                                     │
│  [Cancel]            [Move Money]   │
└─────────────────────────────────────┘
```

- The "From" dropdown pre-selects the envelope whose card the user clicked
- The "To" dropdown excludes the selected "From" envelope
- On submit: validate → update both `monthlyBudget` values → save to localStorage → re-render → close modal
- No confirmation dialog needed — the action is immediately visible and easily reversed

### Worked example

Current state: Dining Out budget = `$200`, Groceries budget = `$400`, TBB = `$0`

You overspent on groceries this month and want to shift $50 from Dining Out:

- Move `$50` from Dining Out → Groceries
- Dining Out `monthlyBudget`: `$200 → $150`
- Groceries `monthlyBudget`: `$400 → $450`
- Total Assigned: `$3,000` ← unchanged (150 + 450 = 200 + 400)
- TBB: `$0` ← unchanged
- Total Cash: `$2,947.53` ← unchanged

---

## 6. Over-Budgeting Detection

Over-budgeting means **Total Assigned > Total Income** — you've planned to spend more than you've ever received. TBB is negative.

**How it's surfaced in the UI:**
- The **To Be Budgeted** number turns **red** when negative
- A warning label appears: *"Over-budgeted — reduce envelope totals"*
- No hard block — the user can still add envelopes and transactions
- The warning disappears the moment TBB reaches $0 or above

**What causes it:**
- Creating envelopes with budgets that exceed total income received
- Increasing an envelope's budget beyond remaining unassigned income
- Importing data where envelope budgets exceed total income

**What does NOT cause it:**
- Spending money (expenses never affect TBB)
- Navigating to a different month

**What fixes it:**
- Add more income transactions
- Reduce one or more envelope `monthlyBudget` values
- Delete envelopes that are no longer needed

---

## 7. localStorage Design

**Keys:**
| Key | Contents |
|---|---|
| `envelopes` | JSON array of all Envelope objects |
| `transactions` | JSON array of all Transaction objects (income and expense) |
| `settings` | Small object: `{ currency, startOfMonth }` |

**Example stored transactions:**
```json
[
  { "id": "txn_001", "type": "income",  "envelopeId": null,               "amount": 3000.00, "description": "April Paycheck", "date": "2026-04-01", "createdAt": "..." },
  { "id": "txn_002", "type": "expense", "envelopeId": "env_1713200000000", "amount": 52.47,  "description": "Trader Joe's",   "date": "2026-04-15", "createdAt": "..." },
  { "id": "txn_003", "type": "expense", "envelopeId": "env_1713200000001", "amount": 35.00,  "description": "Dinner out",     "date": "2026-04-14", "createdAt": "..." }
]
```

**Read/write pattern:**
```
load()  → JSON.parse(localStorage.getItem(key)) || []
save()  → localStorage.setItem(key, JSON.stringify(data))
```

**Export:**
- Bundle `{ envelopes, transactions, settings }` into one JSON object
- Trigger a browser download via a temporary `<a>` element with a `data:` URL
- Filename: `budget-export-2026-04-16.json`

**Import:**
- File input (`<input type="file" accept=".json">`)
- Validate: both `envelopes[]` and `transactions[]` arrays must exist; each transaction must have a valid `type`
- Confirmation prompt before overwriting: *"This will replace all current data. Continue?"*
- On confirm, write to localStorage and re-render

---

## 8. UI Structure

**Single-page layout (no routing needed):**

```
┌───────────────────────────────────────────────────┐
│  HEADER: App name + month navigator                │
├───────────────────────────────────────────────────┤
│  ZBB SUMMARY BAR (always visible, not month-      │
│  filtered):                                        │
│                                                    │
│  Total Cash: $3,000   Assigned: $3,000   TBB: $0  │
│                       ← sum of budgets →  ← diff  │
│                       [green/red based on TBB]     │
├──────────────────┬────────────────────────────────┤
│  ENVELOPE CARDS  │  TRANSACTION PANEL             │
│  (left column)   │  (right column)                │
│                  │                                 │
│  [+ New Envelope]│  [+ Add Expense]               │
│                  │  [+ Add Income]                 │
│  Card per        │                                 │
│  envelope:       │  Filtered list —               │
│  name, budget,   │  click envelope to filter,     │
│  spent this mo,  │  or view all                   │
│  progress bar,   │  (income shown in "All" only)  │
│  remaining       │                                 │
│  [⇄ Move Money] │                                 │
├──────────────────┴────────────────────────────────┤
│  FOOTER: Export JSON | Import JSON                │
└───────────────────────────────────────────────────┘
```

**ZBB Summary Bar color logic:**
- TBB = $0: green — every dollar is assigned
- TBB > $0: yellow/amber — unassigned income waiting to be placed
- TBB < $0: red — over-budgeted (assigned more than received), action required

Note: TBB does not change as you spend during the month. A TBB of $0 at the start of the month stays $0 after groceries, rent, and gas are paid.

**Forms:**
- **Add/Edit Envelope:** name, monthly budget, color
- **Add Expense Transaction:** envelope (dropdown), amount, description, date
- **Add Income Transaction:** amount, description, date (no envelope field)
- **Move Money:** from-envelope (dropdown), to-envelope (dropdown), amount
- **Delete confirmations:** `window.confirm()` — no custom modal in v1

**Key controls:**
- Month navigator: `< April 2026 >` — filters envelope spending and the transaction list
- ZBB Summary Bar: always shows all-time cash values, unaffected by month navigation
- Envelope card click: filters the transaction panel to that envelope
- "All" view: shows all transactions for the selected month, including income

---

## 9. Application Behavior

**Main user flows:**
1. First visit → empty state, prompt to add income to get started
2. Add income → Total Cash increases; TBB increases; no envelope is affected
3. Create envelope with budget → Total Assigned increases; TBB decreases
4. Repeat until TBB = $0 (every dollar assigned)
5. Add expense → Total Cash decreases; envelope remaining decreases; **TBB unchanged**
6. Move money between envelopes → two `monthlyBudget` values change; **Total Cash, Total Assigned, and TBB all unchanged**
7. Navigate month → envelope spending recalculates for that month; top-level numbers unchanged
8. Export → JSON file downloads
9. Import → confirmation → data replaced → page re-renders

**Monthly budget behavior:**
- Each month, envelope spending resets — progress bars start empty again
- Budgets (`monthlyBudget`) are standing monthly targets, not per-month records
- Total Cash never resets — it accumulates across all time
- TBB is most meaningful when new income arrives and needs assigning

**Over-budgeting:**
- TBB goes negative only when Total Assigned > Total Income
- Surfaced immediately in the ZBB bar (red + warning label)
- No hard block — user resolves by reducing budgets or adding income
- Spending during the month never triggers this warning

**Overspending an envelope (spending > budget this month):**
- Allowed — no hard block
- Progress bar goes red; card shows "Over budget" label
- Does not affect TBB — only Total Cash and envelope remaining are reduced

**Validation rules:**
| Field | Rule |
|---|---|
| Envelope name | Required, max 40 chars, must be unique |
| Monthly budget | Required, number > 0 |
| Transaction amount | Required, number > 0 |
| Transaction date | Required, valid YYYY-MM-DD |
| Expense envelope | Required — must select one |
| Income envelope | Must be `null` — no envelope field shown |
| Import file | Must be valid JSON with `envelopes[]` and `transactions[]`; each transaction must have a valid `type` |

---

## 10. Implementation Phases

**Phase 1 — Core (smallest usable version):**
- HTML skeleton with Tailwind
- localStorage read/write helpers
- Create/delete envelopes
- Add/delete expense and income transactions
- Total Cash, Total Assigned, TBB calculations
- Envelope balance calculation (month-filtered)
- ZBB summary bar with color coding
- Move money between envelopes (modal with from/to/amount)
- Basic responsive layout

**Phase 2 — Navigation & Polish:**
- Month navigator (`< >` arrows)
- Progress bar per envelope (green → yellow → red)
- Empty states (no envelopes, no transactions)
- Form validation with inline error messages
- Over-budget and over-assigned warnings

**Phase 3 — Data Portability:**
- Export to JSON
- Import from JSON with validation and confirmation

**Phase 4 — Edit & UX:**
- Edit existing envelopes (rename, change budget/color)
- Edit existing transactions
- Transaction list sorting (newest first)

**Phase 5 — Comments & Learning Layer:**
- Add thorough inline comments throughout
- Explain every function, every data structure, every DOM operation

---

## 11. Risks and Simplifications

| Risk | Simplification |
|---|---|
| Users expect TBB to decrease as they spend | Label TBB clearly: "Unassigned Income" — it only moves when income or budgets change |
| Users may try to use Move Money to fix overspending | Move Money changes the plan, not the history — overspending still shows red on the card |
| Move Money could set a budget below $0 | Validate: amount must be ≤ from-envelope's current `monthlyBudget` |
| Month boundaries get tricky | Use `YYYY-MM` string comparison — no Date arithmetic needed |
| Over-budgeted warning feels alarming for normal use | Only show it when TBB < 0; normal spending drift is expected and not flagged |
| Editing envelopes breaks transaction links | Only edit name/budget/color — never the `id` |
| Cascading deletes are easy to forget | One `deleteEnvelope(id)` function handles both envelope and its transactions |
| `amount` as float causes rounding errors | Round all amounts to 2 decimal places on input and display |
| Income transactions accidentally attached to envelopes | Enforce `envelopeId = null` for income at save time, not just in the UI |
| Import could corrupt data | Validate `type` field is `"income"` or `"expense"` on every transaction before accepting |
| Large transaction lists get slow | No concern at personal scale — skip virtualization entirely |
| Responsive on mobile | Tailwind's `sm:` breakpoints handle this with minimal effort |

---

## Recommended v1 Scope

Phases 1 + 2 + 3 combined. This gives you:

- Add income transactions that build Total Cash
- Create and delete envelopes with monthly budgets
- ZBB summary bar: Total Cash / Total Assigned / To Be Budgeted (with color)
- Add and delete expense transactions assigned to envelopes
- Move money between envelopes (budgeting action, not a transaction)
- Envelope balances with progress bars (month-filtered)
- Over-budgeting warning when TBB < $0
- Month navigation
- Export and import JSON

**Not in v1:** edit existing records, accounts, rollover, multi-currency.

---

## Recommended File Structure (v1)

```
env-budgeting/
├── index.html        ← the entire app (HTML + CSS classes + JS)
└── CLAUDE.md         ← project notes
```

The JS will be organized into logical comment sections:

```
// === DATA LAYER ===     (load/save, id generation)
// === STATE ===          (in-memory app state object)
// === CALCULATIONS ===   (totalCash, totalIncome, totalAssigned, toBeBudgeted, envelopeBalance)
// === RENDER ===         (functions that build DOM)
// === EVENT HANDLERS === (form submissions, clicks)
// === INIT ===           (startup, wire up events)
```

---

## Decisions to Make Before Coding

1. **Month navigation default:** Should the app open on the current month always, or remember the last-viewed month across sessions? *(Recommendation: always open on current month — simpler, and stale months are confusing)*

2. **Negative balance behavior:** When an envelope goes over budget this month, should the card show a warning and turn red, or hard-block the transaction? *(Recommendation: warn + red, never block)*

3. **Transaction deletion:** Should deleting an envelope also silently delete all its transactions, or confirm with the user how many transactions will be removed? *(Recommendation: explicit confirmation listing the count)*

4. **Income display in transaction list:** Should income transactions appear in the "All" view but be hidden when a specific envelope is selected? *(Recommendation: yes — income is global, not envelope-specific)*

5. **Color selection UX:** Full `<input type="color">` wheel picker, or a fixed palette of ~8 preset colors? *(Recommendation: preset palette — more consistent, faster to pick, easier to code)*
