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
| **Pay date** | The day salary lands: nominally the 20th of the month, rolled back to the nearest preceding working day if the 20th is a weekend. |
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

### 3.3 Outgoings — fixed bills (per period)
A flat list of named bills paid from the Bills pot, each with a monthly amount.
The list must be editable (add/remove/rename rows). Current contents:

| Bill | £/mo | | Bill | £/mo |
|------|-----:|---|------|-----:|
| Mortgage | 2,748 | | Sky | 33 |
| Sainsbury's CC | 80 | | NUS | 28 |
| Barclays CC | 125 | | Life insurance | 22 |
| Energy | 308 | | National Trust | 14 |
| Council Tax | 351 | | Peloton | 45 |
| ISA | 500 | | Pet insurance | 34 |
| Swim | 107 | | TV License | 12 |
| Kids savings | 200 | | Ailsa | 10 |
| DVLA | 17 | | Guildford city | (blank) |
| Water | 67 | | Dog food | 60 |

**Total fixed bills = £4,761/period.**

### 3.4 Spending rates
- **Daily spend** — discretionary spend per day (current: £100/day).
- *(Weekly / Monthly spend rows exist in the sheet but do not feed the pots — see §9.4.)*

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
- **Bonus London** — manual adjustment to commute-day count (current: 0).

---

## 4. Pay-period / date logic

The period boundaries are computed from the selected pay month:

1. **This pay date** = the 20th of the selected pay month (in the current year).
2. **Previous pay date** = the 20th of the prior month (December of the prior year if the selected month is January).
3. **Weekend roll-back**: if a pay date falls on a weekend, salary lands on the
   nearest **preceding working day**. The current period:
   - **Start date** = (previous pay date, weekend-adjusted) **+ 1 day**.
   - **End date** = (this pay date, weekend-adjusted).
4. **Days in period** = End date − Start date (inclusive count). Current example: 18 Apr 2026 → 19 May 2026 = **31 days**.

> **Open question (Q1):** The spreadsheet's weekend test treats the 20th as
> "weekend-affected" when it is a **Saturday, Sunday, *or Monday*** and the
> roll-back arithmetic lands on Thursday in some cases rather than Friday. The
> *intended* rule appears to be simply "roll back to the Friday before a
> Sat/Sun pay date." The new implementation should implement the clean intent
> and confirm the exact rule with the user. Treat WEEKDAY as Sunday=1 … Saturday=7.

---

## 5. Weekday counters (for period-scaled spending)

The model counts occurrences of specific weekdays within the pay period, intended
to drive variable costs:

- **Cleaning days** — count of Thursdays in the period.
- **London (commute) days** — count of Wednesdays in the period × 2, plus a manual **Bonus London** adjustment.
- **Mondays in period** — count of Mondays.

> **Open question (Q2):** In the current sheet these three counters are **broken**
> (`#REF!`, evaluating to 0) and the cells they would feed (per-day extra cost
> `K27`/`K28`) are empty, so they have **no effect on the pots today**. The new
> implementation must decide whether variable per-weekday costs (cleaner fee per
> Thursday, commute cost per London day, etc.) should feed the spending pot, and
> if so at what rate. Until specified, the spending pot is daily-rate-only (§6).

---

## 6. Derived calculations

| # | Output | Definition | Current value |
|---|--------|-----------|--------------:|
| C1 | **Total In** | Little Moo salary + Big Moo salary | £11,885 |
| C2 | **Bills pot** | Σ fixed bills | £4,761 |
| C3 | **Main (spending) pot** | Daily spend × Days in period | £3,100 |
| C4 | **Moo money pot** | Manual input | £1,100 |
| C5 | **Lumpy pot** | (Σ lumpy annual) ÷ 12 | £491.42 |
| C6 | **Holidays pot** | Σ (trip total ÷ months to save) | £633.33 |
| C7 | **Savings** | Total In − (Bills + Main + Moo + Lumpy + Holidays) | £1,799.25 |

`Savings` is the residual and is the headline figure ("Total Saving").

---

## 7. Outputs

A **pot allocation table** showing the transfer amount for each pot, the total
allocated, and the residual savings:

```
Total In ............ 11,885.00
  Bills ............. 4,761.00
  Main / spending ... 3,100.00
  Moo money ......... 1,100.00
  Lumpy ............. 491.42
  Holidays .......... 633.33
  ---------------------------
  Savings ........... 1,799.25
```

Plus contextual readouts: pay-period start/end dates and number of days in the period.

---

## 8. Functional requirements (summary)

- **FR1** Compute the pay period (start, end, day count) from the selected pay month per §4.
- **FR2** Sum the two salaries into Total In (§3.2).
- **FR3** Maintain an editable list of fixed bills and sum them into the Bills pot (§3.3).
- **FR4** Compute the Main/spending pot as daily rate × days in period (§6 C3).
- **FR5** Maintain editable Lumpy items; set aside annual total ÷ 12 (§3.5).
- **FR6** Maintain editable Holiday items; set aside Σ(total ÷ months) (§3.6).
- **FR7** Accept manual Moo money and Bonus London inputs (§3.7).
- **FR8** Compute Savings as the residual of Total In minus all funded pots (§6 C7).
- **FR9** Present the pot allocation table and period readouts (§7).
- **FR10** All amounts in GBP. Define and apply a consistent rounding rule (see Q3).

---

## 9. Known defects in the source sheet (must NOT be carried over)

1. **`#REF!` in weekday counters** (`B4`, `B5`, `B7`) — formulas reference deleted
   cells (`SEQUENCE(..., #REF!, ...)`) and silently return 0. Re-implement cleanly per §5.
2. **Weekend roll-back arithmetic** is inconsistent (§4 Q1) — implement the intent.
3. **`H12` "Monthly Spends"** mixes a typo `(B7 * (K27 + K27))` — note `K27` is
   added to itself (should almost certainly be `K27 + K28`). This cell is not
   wired to any pot, so the bug is currently latent.
4. **Dead / disconnected cells:** "Weekly" (`H11`), "Monthly Spends" (`H12`),
   `B43`, and the scratch block `K40:K68` are not referenced by the pot
   calculations. Decide whether they represent intended-but-unfinished features
   (variable spend modelling) or can be dropped.
5. **`H37` is labelled "Holidays total"** but actually holds the *monthly*
   set-aside (each item is pre-divided). Rename for clarity in the rebuild.

---

## 10. Open questions for the user

- **Q1** — Exact pay-date weekend rule: roll back to the Friday before a Sat/Sun 20th? What about when the 20th is a bank holiday?
- **Q2** — Should variable per-weekday costs (cleaning / London commute / Mondays) feed the spending pot, and at what £ rates? If yes, this replaces or augments §6 C3.
- **Q3** — Rounding: round each pot to the penny, or to whole pounds? Should Savings absorb the rounding remainder so pots always reconcile to Total In?
- **Q4** — Are salaries ever variable (bonus months), or always fixed? Should the model support a one-off income line?
- **Q5** — Should funded sinking funds (Lumpy, Holidays) track a running balance across months, or is this a single-month snapshot only?

---

## 11. Acceptance fixture (May 2026)

Given the §3 inputs and pay month **May** (period 18 Apr 2026 → 19 May 2026, 31 days):

| Quantity | Expected |
|----------|---------:|
| Days in period | 31 |
| Total In | 11,885.00 |
| Bills pot | 4,761.00 |
| Main / spending pot | 3,100.00 |
| Moo money pot | 1,100.00 |
| Lumpy pot | 491.42 |
| Holidays pot | 633.33 |
| **Savings (residual)** | **1,799.25** |

A correct re-implementation fed the §3 inputs must reproduce this table.
