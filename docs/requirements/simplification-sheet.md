# Requirements — Monthly Pay Allocation ("Simplification")

> **White-room specification.** This document describes the *intended behaviour*
> of the household budgeting model currently kept in the `Simplification` tab of
> `Finances.xlsx`. It is written so the model can be re-implemented from scratch
> without copying the spreadsheet's formulas (several of which are broken — see
> §9). Where the spreadsheet's behaviour is buggy or ambiguous, the requirement
> states the *intent* and flags the open question rather than reproducing the
> defect.

---

## 1. Purpose

Each month the household receives two salaries on (or near) the 20th. The tool's
job is to take the total income for a **pay period** and split it into a fixed
set of **pots** (bank accounts / sinking funds), so the user knows exactly how
much to move where. Whatever is left after every pot is funded is the month's
**savings**.

The model also scales spending to the *actual length of the pay period*, because
the period is not a calendar month — it runs pay-date to pay-date and varies
between 28 and 34 days depending on weekends.

The output the user cares about is a single number — **how much can we save this
month** — plus the per-pot transfer amounts that produce it.

---

## 2. Glossary

| Term | Meaning |
|------|---------|
| **Pay date** | The day salary actually lands. Nominally the 20th, adjusted for the employer's weekend rule **and** Monzo's pay-a-day-early scheme (see §4). |
| **Pay period / budget month** | The window from the day after the *previous* pay date up to and including the *current* pay date. This is the period all spending is scaled to. |
| **Pot** | A destination for money: an account or sinking fund (Bills, Main/spending, Moo money, Lumpy, Holidays, Savings). |
| **Lumpy** | Large, irregular, predictable annual expenses, smoothed into a monthly set-aside (annual total ÷ 12). |
| **Holidays** | Planned trips, each smoothed into a monthly set-aside (trip cost ÷ number of months to save for it). |
| **Little Moo / Big Moo** | The two earners in the household. |

---

## 3. Actors & inputs

There is a single user (the household). All inputs below are **user-editable**;
everything else is derived.

### 3.1 Period selection
- **Pay month** — the month whose pay date defines the current period (e.g. "May"). Drives all date math.

### 3.2 Income (per period)
- **Little Moo salary** — fixed monthly amount (current: £3,050).
- **Big Moo salary** — fixed monthly amount (current: £8,835).

### 3.3 Outgoings — bills (per period)
Bills are funded into the **Bills pot**, from which the direct debits are paid.
The list must be editable (add/remove/rename rows). A bill is one of two kinds:

- **Flat** — a fixed amount that debits every month; these empty the pot cleanly.
- **Smoothed** — billed irregularly (e.g. 10 months a year, or quarterly) but
  funded at **annual ÷ 12** every month, so the pot accrues a float in the
  off-months and pays the lump when due. See §3.3.1.

This list was reconciled against 3 months of Monzo statements (Mar–May 2026);
see §12 for the full reconciliation. Corrected contents:

#### Flat monthly bills (empty the pot each month)

| Bill | £/mo | | Bill | £/mo |
|------|-----:|---|------|-----:|
| Mortgage | 2,748 | | Sky | 45 |
| NatWest *(ex-Sainsbury's CC)* | 80 | | NUS | 28 |
| Barclays CC | 125 | | Life insurance | 22 |
| Energy | 308 | | National Trust | 14 |
| ISA | 500 | | Ailsa | 10 |
| Swim | 107 | | NHS Prescription | 11.45 |
| Kids savings | 200 | | DVLA | 17 |
| Water | 67 | | | |

**Flat bills subtotal = £4,282.45/period.**

#### Smoothed bills (annual ÷ 12 — pot carries a float, see §3.3.1)

| Bill | Annual £ | £/mo | Billed |
|------|---------:|-----:|--------|
| Council Tax | 3,500 *(est — confirm actual annual)* | 291.67 | 10 installments/yr |
| TV License | 179.48 | 14.96 | quarterly (~£44.87) |

**Smoothed subtotal = £306.63/period.**

> **Total Bills pot funding = £4,589.08/period** (was £4,761 — see changes below).

**Changes from the original sheet (reason the pot never emptied):**
- **Sainsbury's CC → NatWest** — same debit, NatWest took the card over (~£80). Kept.
- **Barclays CC £125** — kept as-is (variable repayment; do not smooth).
- **NUS £28** — kept; the *payment* will be adjusted to match the £28 budget.
- **Sky £33 → £45** — was under-budgeted (two Sky DDs / price rise).
- **Council Tax & TV License** — moved to *smoothed* (annual ÷ 12), §3.3.1.
- **Removed: Peloton (£45), Dog food (£60), Pet insurance (£34)** — these are
  **card payments from the Main account**, never debited from the Bills pot. They
  belong to Main/spending (§3.4), not bills. Leaving them in the bills list was
  the bulk of the ~£195/mo that piled up in the pot.

### 3.3.1 Smoothed-bill behaviour (sinking float)
A smoothed bill is funded at `annual ÷ 12` every period but debits only in the
months it is actually billed. Consequently the Bills pot is **deliberately not
empty** at month-end — it holds a sinking-fund float for these bills that
**nets to zero across a full year**. This residue is *saved-up money toward the
next bill, not surplus to be swept out*. Only flat bills are expected to empty
the pot monthly.

### 3.4 Spending rates
- **Daily spend** — total day-to-day outflow per day (current: £100/day). **This
  is an all-in figure that already includes cleaner and commute costs**, so those
  are carved *out* of the daily total rather than added on top — see §6 C3.
- **Cleaner rate** — cost per cleaner visit, charged once a week (current: £52/week).
- **Commute rate** — cost per commute day, one commute day a week (current: £60/day).

### 3.5 Lumpy items (annual, smoothed ÷12)
Editable list of `{name, annual £}`:

| Item | Annual £ |
|------|---------:|
| Send | 2,322 |
| Blue Spider | 850 |
| Multisports | 540 |
| Julie's Swim | 510 |
| Glasses | 600 |
| Car | 1,000 |
| Garden bin | 75 |

Annual total **£5,897 → £491.42/period** (÷12).

### 3.6 Holiday items (smoothed ÷ months-to-save)
Editable list of `{name, total £, months to save}`:

| Trip | Total £ | Months | £/mo |
|------|--------:|-------:|-----:|
| Spain | 800 | 3 | 266.67 |
| Valencia Airbnb | 800 | 3 | 266.67 |
| Cotswolds | 500 | 5 | 100.00 |

Holiday set-aside **= £633.33/period.**

### 3.7 Manual pot overrides
- **Moo money** — fixed transfer (current: £1,100, described as "Moo bonus plus insurance").
- **Bonus London** — manual **extra commute days** added to the auto-counted commute days (current: 0). Each bonus day is charged at the commute rate.

---

## 4. Pay-period / date logic

### 4.1 Actual pay-landing date

A **working day** is Monday–Friday excluding **England & Wales bank holidays**
(bank-holiday shifting confirmed by the user). The nominal pay date is the
**20th**, adjusted in **two stages**:

1. **Employer rule** — salary is due on the 20th. If the 20th is **not a working
   day**, the employer pays on the **preceding working day**.
   - e.g. Sun 20th → Fri 18th; Sat 20th → Fri 19th; if the resulting Friday is a
     bank holiday, step back again to Thursday.
2. **Monzo early-pay** — the bank deposits funds on the **working day immediately
   before** the employer's pay-in date.

Equivalently: **actual landing date = the working day immediately before the
first working day on-or-before the 20th.**

Ignoring bank holidays, the actual landing date by the weekday of the 20th
(WEEKDAY: Sun=1 … Sat=7):

| 20th falls on | Employer pays | Monzo lands (= actual) |
|---------------|---------------|------------------------|
| Sunday | Fri 18th | **Thu 17th** |
| Monday | Mon 20th | Sun 19th → **Fri 17th** |
| Tuesday | Tue 20th | **Mon 19th** |
| Wednesday | Wed 20th | **Tue 19th** |
| Thursday | Thu 20th | **Wed 19th** |
| Friday | Fri 20th | **Thu 19th** |
| Saturday | Fri 19th | **Thu 18th** |

> The "weekend" test therefore flags the 20th when it is a **Saturday, Sunday,
> *or Monday*** — Monday is included precisely because Monzo's day-before then
> falls on a Sunday and rolls back to Friday. This is correct, not a bug.

> This date model is driven solely by **Big Moo's** salary. Little Moo is paid on
> a different schedule, but that is irrelevant here: Big Moo's income funds every
> pot except the savings residual, so only Big Moo's pay date defines the period.

### 4.2 Period boundaries

1. **This pay date** = actual landing date for the 20th of the selected pay month (§4.1).
2. **Previous pay date** = actual landing date for the 20th of the prior month (December of the prior year if the selected month is January).
3. **Start date** = previous pay landing date **+ 1 day**.
4. **End date** = this pay landing date.
5. **Days in period** = End date − Start date (the spreadsheet's `DAYS` count, i.e. exclusive of one endpoint). Current example: 18 Apr 2026 → 19 May 2026 = **31**.

---

## 5. Weekday counters → variable-cost pots

The model counts how many times a given weekday occurs within the pay period.
These counts drive two **dedicated pots** (confirmed with the user):

- **Cleaning days** — count of the cleaner's weekday (Thursday in the source
  sheet) in the period. Once a week → one count per week.
  → **Cleaner pot = cleaning-day count × cleaner rate (£52).**
- **Commute days** — count of the commute weekday (Wednesday in the source sheet)
  in the period, **plus** the manual **Bonus London** extra days (§3.7).
  → **Commute pot = (commute-day count + bonus days) × commute rate (£60).**

The specific weekday for each should be configurable; only the *number of
occurrences in the period* is financially material.

> The old "× 2" multiplier on commute days is dropped — commute is now once a
> week at £60/day. The "Mondays in period" counter is no longer used.

---

## 6. Derived calculations

| # | Output | Definition | Current value |
|---|--------|-----------|--------------:|
| C1 | **Total In** | Little Moo salary + Big Moo salary | £11,885 |
| C2 | **Bills pot** | Σ flat bills + Σ (smoothed bill annual ÷ 12) | £4,589.08 |
| C3 | **Cleaner pot** | Cleaning-day count × cleaner rate | £208 |
| C4 | **Commute pot** | (Commute-day count + bonus) × commute rate | £240 |
| C5 | **Main (spending) pot** | (Daily spend × Days in period) − Cleaner pot − Commute pot | £2,652 |
| C6 | **Moo money pot** | Manual input | £1,100 |
| C7 | **Lumpy pot** | (Σ lumpy annual) ÷ 12 | £491.42 |
| C8 | **Holidays pot** | Σ (trip total ÷ months to save) | £633.33 |
| C9 | **Savings** | Total In − (Bills + Cleaner + Commute + Main + Moo + Lumpy + Holidays) | £1,971.17 |

**Cleaner and Commute are carved out of the daily total, not added on top** (C5):
because the £100/day already funds them, total day-to-day spend stays at
`daily × days` (£3,100 = Main £2,652 + Cleaner £208 + Commute £240) and the
savings residual is unchanged by the split.

`Savings` is the residual and is the headline figure the user reads off
("Total Saving") — i.e. what's left after every pot is funded.

> Correcting the bills list (§3.3) cut the Bills pot from £4,761 to £4,589.08,
> raising recognised savings by ~£172/mo. That ~£172 was previously piling up
> unspent in the Bills pot — it was never waste, just **savings the sheet failed
> to recognise**.

---

## 7. Outputs

A **pot allocation table** showing the transfer amount for each pot, the total
allocated, and the residual savings:

```
Total In ............ 11,885.00
  Bills ............. 4,589.08
  Cleaner ........... 208.00
  Commute ........... 240.00
  Main / spending ... 2,652.00
  Moo money ......... 1,100.00
  Lumpy ............. 491.42
  Holidays .......... 633.33
  ---------------------------
  Savings ........... 1,971.17
```

Plus contextual readouts: pay-period start/end dates, number of days in the
period, and the cleaning-day / commute-day counts.

---

## 8. Functional requirements (summary)

- **FR1** Compute the pay period (start, end, day count) from the selected pay month per §4, using a working-day calendar that excludes weekends **and England & Wales bank holidays**.
- **FR2** Sum the two salaries into Total In (§3.2).
- **FR3** Maintain an editable list of fixed bills and sum them into the Bills pot (§3.3).
- **FR4** Count cleaning days and commute days in the period; compute the Cleaner pot and Commute pot (incl. Bonus London) per §5.
- **FR5** Compute the Main/spending pot as `(daily rate × days) − Cleaner − Commute`, so cleaner/commute are carved out of the daily total rather than double-counted (§6 C5).
- **FR6** Maintain editable Lumpy items; set aside annual total ÷ 12 (§3.5).
- **FR7** Maintain editable Holiday items; set aside Σ(total ÷ months). Editing a trip's amount/months (e.g. after booking) must immediately change its monthly contribution (§3.6, Q5).
- **FR8** Accept manual Moo money and Bonus London inputs (§3.7).
- **FR9** Compute Savings as the residual of Total In minus all funded pots (§6 C9).
- **FR10** Present the pot allocation table and period readouts (§7). All amounts in GBP; no special rounding required — display the residual (Q3).

---

## 9. Known defects in the source sheet (must NOT be carried over)

1. **`#REF!` in weekday counters** (`B4`, `B5`, `B7`) — formulas reference deleted
   cells (`SEQUENCE(..., #REF!, ...)`) and silently return 0. The rebuild
   re-implements these cleanly as the Cleaner/Commute counts (§5).
2. **`H12` "Monthly Spends"** has a typo `(B7 * (K27 + K27))` — `K27` is added to
   itself (should have been `K27 + K28`). Superseded by the explicit Cleaner and
   Commute pots, so the cell is dropped.
3. **Dead / disconnected cells:** "Weekly" (`H11`), "Monthly Spends" (`H12`),
   `B43`, and the scratch block `K40:K68` were never referenced by the pot
   calculations — half-built variable-spend modelling now formalised as the
   Cleaner/Commute pots (§5). Drop the leftovers.
4. **`H37` is labelled "Holidays total"** but actually holds the *monthly*
   set-aside (each item is pre-divided). Rename for clarity in the rebuild.

---

## 10. Resolved decisions

- **Q1 — ✅ Resolved.** Pay-date rule per §4.1: employer pays the preceding
  working day for a non-working 20th; Monzo deposits one working day earlier.
  **Bank holidays do shift the landing date** — the working-day calendar
  excludes England & Wales bank holidays.
- **Q2 — ✅ Resolved.** Cleaner and Commute become their own pots: Cleaner £52
  per weekly visit, Commute £60 per commute day (once a week) + Bonus London
  extra days. They are carved out of the daily total (§5, §6 C3–C5).
- **Q3 — ✅ Resolved.** No special rounding logic required; the user reconciles
  manually. The tool just needs to display the post-everything **savings
  residual** (and the per-pot amounts).
- **Q4 — ✅ Resolved.** Salaries are fixed; **no** variable/bonus-month or
  one-off income line is needed.
- **Q5 — ✅ Resolved.** The tool is a **single-month snapshot** — it does **not**
  track balances or how much is held where. Sinking-fund inputs (Lumpy, Holidays)
  must simply be **editable**, so that e.g. booking a holiday lets the user raise
  that fund's monthly contribution.

- **Q6 (bills reconciliation) — ✅ Resolved.** Bills list corrected against 3
  months of statements (§3.3, §12). Smoothed-bill approach (Option B) adopted for
  Council Tax & TV License; three card-paid items removed from bills.

### Remaining minor items
- **R1** — Confirm the bank-holiday calendar source/region (assumed England &
  Wales) and behaviour if a pay date and its day-before are both around a
  multi-day bank-holiday weekend (the working-day roll-back handles this, but
  worth a sanity check against a real Easter/Christmas example).
- **R2** — Provide the **actual annual Council Tax** (or installment × count); the
  £3,500 used for the £291.67/mo smoothed figure is an estimate from the Mar–May
  installments (~£338–354).

---

## 11. Acceptance fixture (May 2026)

Given the §3 inputs and pay month **May** (period 18 Apr 2026 → 19 May 2026, 31 days;
4 cleaning days, 4 commute days, Bonus London 0):

| Quantity | Expected |
|----------|---------:|
| Days in period | 31 |
| Cleaning days / Commute days | 4 / 4 |
| Total In | 11,885.00 |
| Bills pot | 4,589.08 |
| Cleaner pot | 208.00 |
| Commute pot | 240.00 |
| Main / spending pot | 2,652.00 |
| Moo money pot | 1,100.00 |
| Lumpy pot | 491.42 |
| Holidays pot | 633.33 |
| **Savings (residual)** | **1,971.17** |

A correct re-implementation fed the §3 inputs must reproduce this table. The Bills
pot uses the reconciled list (§3.3); savings rose from £1,799.25 to £1,971.17 once
the card-paid items were removed from bills. (Council Tax £3,500 annual is an
estimate — confirming the real annual will shift Bills and Savings slightly.)

---

## 12. Bills reconciliation (Mar–May 2026 Monzo statements)

The original sheet funded the Bills pot at £4,761/mo but only ~£4,566/mo actually
debited it, leaving ~£195/mo to pile up. Mechanism: Monzo's Bills pot auto-covers
each direct debit with an equal transfer out of the pot; the pot is refilled in a
lump on payday, so it empties only if funding matches the actual debits.

Root causes and fixes:

| Item | Original | Finding | Fix |
|------|---------:|---------|-----|
| Sainsbury's CC | 80 | Card taken over by **NatWest**; the ~£80 NatWest DD *is* this line | Rename → NatWest, keep £80 |
| Peloton | 45 | **Card payment from Main**, never a pot DD | Remove from bills → Main spend |
| Dog food | 60 | Tails.com **card payment from Main** | Remove from bills → Main spend |
| Pet insurance | 34 | No matching pot DD found | Remove from bills |
| Sky | 33 | Actual ~£44–49 (two DDs / price rise) | Raise to £45 |
| NHS Prescription | — | £11.45/mo DD paid from pot, unbudgeted | Add £11.45 |
| Barclays CC | 125 | Variable repayment | Keep £125 (do not smooth) |
| NUS | 28 | DD is £24 | Keep £28; adjust the *payment* to match |
| Council Tax | 351 | 10 installments/yr → £0 in 2 months | **Smooth**: annual ÷ 12 ≈ £291.67 |
| TV License | 12 | Quarterly ~£44.87 | **Smooth**: annual ÷ 12 ≈ £14.96 |

Net effect: Bills pot funding £4,761 → **£4,589.08/mo**; the freed ~£172/mo is
recognised as savings. The remaining month-to-month residue in the pot is the
intended sinking float for the smoothed bills (§3.3.1), which nets to zero yearly.

---

## 13. Bills pot target balance (end-of-month)

The tool must show a **target pot balance** for the end of each month, so the user
can check the actual Bills pot against it.

**Principle:**
- **Flat bills** (§3.3) are funded and paid the same month → they contribute **£0**
  to the target. The pot should clear them to zero every month.
- **Smoothed bills** (§3.3.1) are funded at `annual ÷ 12` but paid in bursts → they
  leave a **sinking float** in the pot. **The end-of-month target = the sum of the
  smoothed-bill floats.**

**Float rule (per smoothed bill).** Starting from its trough (the month-end right
after its final payment of the cycle), the float moves each month by:
- `+ monthly set-aside` in a non-billing month, and
- `− (bill amount − monthly set-aside)` in a billing month.

Over a full year it returns to the trough (nets to zero). Per current bills:

| Smoothed bill | Set-aside/mo | Builds (non-bill mo) | Draws down (bill mo) | Float range |
|---------------|-------------:|---------------------:|---------------------:|-------------|
| Council Tax | 291.67 | +291.67 | −58.33 | £0 → ~£583 |
| TV License | 14.96 | +14.96 | −29.91 | £0 → ~£30 |

**Target table (illustrative — confirm billing calendar, see below).**
Assumes Council Tax £0 in **Feb & Mar** and TV License paid **Jan/Apr/Jul/Oct**:

| Month-end | Council Tax | TV License | **Target pot** |
|-----------|------------:|-----------:|---------------:|
| Jan | 0.00 | 0.00 | **0.00** |
| Feb | 291.67 | 14.96 | **306.63** |
| Mar | 583.33 | 29.91 | **613.24** |
| Apr | 525.00 | 0.00 | **525.00** |
| May | 466.67 | 14.96 | **481.63** |
| Jun | 408.33 | 29.91 | **438.24** |
| Jul | 350.00 | 0.00 | **350.00** |
| Aug | 291.67 | 14.96 | **306.63** |
| Sep | 233.33 | 29.91 | **263.24** |
| Oct | 175.00 | 0.00 | **175.00** |
| Nov | 116.67 | 14.96 | **131.63** |
| Dec | 58.33 | 29.91 | **88.24** |

**How to read it:**
- The pot is **not** meant to be empty most months — only at the Council Tax trough
  (~end of Jan here). Peak is ~£613 (end Mar).
- **Pot ≈ target** → funding is correct. **Pot > target** (by more than a few £) →
  over-funding (a card-paid item crept back into bills, or a bill shrank). **Pot <
  target** → under-funded; a smoothed bill will be short when it lands.
- **First-year seeding:** in steady state the float is already built up. When first
  switching to smoothing you must seed the pot up to that month's target (e.g.
  ~£525 if you start in April), otherwise an early Council Tax installment overdraws
  it. After one full cycle it self-sustains.

> **To finalise the exact table I need three inputs (also R2):**
> 1. Actual **annual Council Tax** (or installment × count) — £3,500 is an estimate.
> 2. Which **two months Council Tax is £0** — the Mar 2026 statement shows a payment,
>    so the Feb/Mar assumption above is probably wrong for your council.
> 3. Which **months the TV License** quarterly payment lands.
