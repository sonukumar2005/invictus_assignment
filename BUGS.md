# Bugs found

Add one section per issue. Bug 1 is filled in to show the format — fix it, then write what you changed. Copy the blank template for the rest.

Keep this file in the repo and **commit it** with your fixes.

---

## Bug 1

**How to reproduce:** Open the app. The expense list says “Newest first”. The first row is Wine (7 Mar). Board game (15 Mar) is further down.

**What is wrong:** The list is showing oldest expenses first. Newest should be at the top.

**What I changed:** Spent a bit digging on this one bc it looked like a one-line fix but was actually two bugs stacked on top of each other lol.

First thing — `dateValue()` in format.js wasn't doing anything useful, it just handed back the same string you gave it. So when the sort tried `a.date - b.date`, it was really doing something like `"2026-03-12" - "2026-03-10"`, and JS has no idea what to do with that so it just returns `NaN`. And if your sort function always returns `NaN`, `.sort()` basically gives up and leaves the array however it found it. So the list was never being sorted, period — it just kinda looked sorted by coincidence based on the order the data was in. Fixed that by actually converting the date to a real timestamp with `new Date(date).getTime()`.

Second thing, once dates were parsing right, the comparison itself was backwards — it was doing `a - b` which sorts oldest first. Swapped it to `b - a` so newest shows up first, which is what the label already promised.

---


## Bug 2

**How to reproduce:** Get a group into a state where one person's debt exactly equals another person's credit — e.g. Alice and Bob each owe $50, Carlos and Diya are each owed $50 (balances: `{Alice: -50, Bob: -50, Carlos: 50, Diya: 50}`). Open the "Settle up" panel.

**What is wrong:** It tells you "Everyone is settled" which is just... not true, people still owe money. Traced it back to `settle.js` — the loop matching people who owe money to people who are owed money has three cases: debtor owes more than creditor's owed, creditor's owed more than debtor owes, or they're the exact same amount. First two cases both add the payment to the list properly. The "exact same amount" case does nothing, it just skips to the next pair without recording anything. So any time the numbers line up perfectly, that payment just quietly vanishes and nobody gets told to pay it.

**What I changed:** Added the same `transfers.push(...)` that the other two branches had, just missing from this one. Tested with `{1: -50, 2: -50, 3: 50, 4: 50}` — before the fix it returned nothing (empty array, which makes no sense since people owe $100 total), after the fix it correctly shows the two $50 payments that need to happen.

---
