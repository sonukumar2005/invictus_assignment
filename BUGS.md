# Bugs found

Add one section per issue. Bug 1 is filled in to show the format — fix it, then write what you changed. Copy the blank template for the rest.

Keep this file in the repo and **commit it** with your fixes.

---

## Bug 1

**How to reproduce:** Open the app. The expense list says “Newest first”. The first row is Wine (7 Mar). Board game (15 Mar) is further down.

**What is wrong:** The list is showing oldest expenses first. Newest should be at the top.

**What I changed:** Two bugs, both in the sorting path.

1. `dateValue()` in `src/lib/format.js` just returned the raw date string instead of converting it to a comparable value. Subtracting two ISO date strings (e.g. `"2026-03-12" - "2026-03-10"`) coerces them with `Number()`, which returns `NaN` for a string like `"2026-03-12"`. A comparator that always returns `NaN` is treated as `0` by `.sort()`, so the array never actually got reordered — it just stayed in whatever order it started in. Fixed by parsing the date into a timestamp: `new Date(date).getTime()`.
2. In `src/components/ExpenseList.jsx`, the comparator was `dateValue(a.date) - dateValue(b.date)`, which sorts ascending (oldest first) even once dates parse correctly. Changed it to `dateValue(b.date) - dateValue(a.date)` for descending (newest first), matching the "Newest first" label.

---

## Bug 2

**How to reproduce:** Get a group into a state where one person's debt exactly equals another person's credit — e.g. Alice and Bob each owe $50, Carlos and Diya are each owed $50 (balances: `{Alice: -50, Bob: -50, Carlos: 50, Diya: 50}`). Open the "Settle up" panel.

**What is wrong:** The panel shows "Everyone is settled" (or drops that pair silently) even though money is still owed. In `suggestSettlements()` (`src/lib/settle.js`), the loop has three branches for comparing a debtor's amount to a creditor's amount: greater-than, less-than, and equal. The greater-than and less-than branches both push a transfer before advancing; the equal branch (`d.amount === c.amount`) only advances `i` and `j` — it never records the transfer. So whenever a debt and a credit match exactly, that payment simply vanishes from the suggestions instead of being listed.

**What I changed:** Added the missing `transfers.push(...)` in the equal branch, same as the other two branches, before incrementing `i` and `j`. Verified with `{1: -50, 2: -50, 3: 50, 4: 50}` — before the fix `suggestSettlements` returned `[]`; after the fix it correctly returns two $50 transfers (1→3, 2→4).

---
