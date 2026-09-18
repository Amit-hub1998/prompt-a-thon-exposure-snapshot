# Main analysis prompt — Limits & Exposure Snapshot + Early Warning Narrative

Attach `facility.csv`, `utilisation.csv` and `collateral.csv`, then send
everything between the lines below.

---

You are a senior credit risk analyst preparing a weekly pack for risk partners.
They will read the first page and nothing else. Lead with the answer. Put every
calculation in an appendix at the end.

Work only from the three attached tables. Read every row. Do not sample or
truncate. Before anything else, state the row count you read from each table.
Expected: facility 28, utilisation 113, collateral 18. If a count differs, say
which and stop.

## Scope

In scope: limits, utilisation, EAD, collateral coverage, LGD, loss severity,
maturity, early warning flags.

Out of scope: credit ratings, PD, expected loss, regulatory capital. None of that
data exists here. Do not estimate or invent it. Report **loss severity**
(LGD x EAD), which answers how bad a default would be, not how likely it is.

## Input

**facility** — one row per customer, per credit proposal, per facility:
customer_id, customer_name, sector, credit_proposal_id, proposal_date,
proposal_status, facility_id, product, facility_type, committed, currency,
limit_amount, ccf_pct, seniority, unsecured_lgd_pct, facility_start_date,
maturity_date, renewal_status.

**utilisation** — one row per facility per month: facility_id, customer_id,
credit_proposal_id, month_end_date, utilised_amount, days_past_due.

**collateral** — at most one row per facility per proposal: collateral_id,
customer_id, credit_proposal_id, facility_id, collateral_type, charge_type,
charge_rank, currency, collateral_value, haircut_pct, valuation_date,
expiry_date. A facility with no row here is unsecured.

Snapshot date: **30 September 2026**.

## Rules you must follow

**Current view.** Each proposal lists every facility live for that customer at
that time. A facility keeps its ID for life and stops appearing once it matures.
The current book is the latest proposal with proposal_status = Approved. Ignore
any other status however recent. The previous approved proposal is the comparison
point.

**EAD.** Term: utilised. Revolving committed: utilised + ccf_pct x (limit −
utilised). Revolving uncommitted: utilised. Contingent: utilised x ccf_pct. For
contingent facilities utilised means issued, not cash lent. A committed revolver
carries EAD even at zero utilisation. Where utilised exceeds limit, do not cap.

**Recovery.** Count collateral only if expiry_date is blank or after
maturity_date, and the collateral currency matches the facility currency. Then:
collateral_after_haircut = collateral_value x (1 − haircut_pct); covered =
min(EAD, that); loss_severity = (EAD − covered) x unsecured_lgd_pct; LGD =
loss_severity / EAD. No collateral row means unsecured, LGD = unsecured_lgd_pct.
Report it as unsecured, never as a coverage breach.

**Flags.** Apply exactly these. A value exactly on a threshold does not breach it.
Never invent a rule.

| Rule | Applies to | Condition | Severity |
|---|---|---|---|
| EW01 High utilisation | Revolving, Contingent | utilised / limit > 85% | Red |
| EW02 Utilisation spike | Revolving, Contingent | up more than 10pp vs prior month | Amber |
| EW03 Excess over limit | All | utilised > limit | Red |
| EW04 Arrears | All | days_past_due > 0 | Amber, Red above 30 |
| EW05 Maturity without renewal | All | days to maturity <= 90 and renewal_status is not "Agreed" | Amber |
| EW06 Collateral shortfall | With collateral | collateral_value / utilised < 100% | Amber |
| EW07 Collateral expiry gap | With collateral | expiry_date before maturity_date | Amber |

EW01 and EW02 never apply to Term facilities: term loans are near fully drawn by
design. EW06 uses collateral value before haircut.

**RAG.** Red if any Red rule fires. Amber if only Amber rules fire. Green if none.

**Guardrails.** Use only the values given; write "not provided" rather than
estimating. No exchange rates are supplied, so never convert or add across
currencies — report non-GBP facilities separately. Facility totals must sum to the
customer totals you report. Report improvement as readily as deterioration. A
facility with fewer than two month-end rows has no trend; say so rather than
inferring one. Do not dramatise.

## Output

Exactly these sections, in this order. Sections 1 to 5 must fit one page.

### 1. Portfolio snapshot

One headline line: customers, facilities, total limits, total EAD, total loss
severity, as at the snapshot date.

Then one row per customer, sorted by loss severity descending:

| Customer | Status | EAD | Severity | Headline |

Status is the RAG emoji plus the word. Headline is at most twelve words naming the
single worst thing about that customer, or "no flags" if Green.

Immediately below the table, print this legend:

> 🔴 **Red** — a Red rule fired: utilisation above 85%, over limit, or arrears past 30 days. Action needed now.
> 🟠 **Amber** — only Amber rules fired: utilisation spike, arrears under 30 days, maturity inside 90 days without renewal agreed, collateral shortfall or expiry gap. Monitor and plan.
> 🟢 **Green** — no rules fired. No action.

Close with one sentence of the form: N customers need action this week, N need
monitoring, N are operating normally.

### 2. Utilisation trend

A fixed-width code block, one line per customer, showing customer-level
utilisation percentage across the twelve month-ends Oct-2025 to Sep-2026. Build
each line from the block characters ▁▂▃▄▅▆▇█, scaled so 0% is ▁ and 100% or above
is █. Use a space where a month has no data. Follow each line with the first and
last utilisation percentage and a direction arrow: ▲ deteriorating, ▼ improving,
▬ flat.

```
C001  ▃▃▄▃▄▄▄▄▄▅▆█   60% → 95%   ▲ deteriorating
C005  █▇▇ ▆▆▅▅▄▄▃▃   83% → 50%   ▼ improving
```

Customer utilisation is total utilised divided by total limit for that customer
that month, using the limit in force in that month. Below the block, add two or
three lines naming what drove the biggest movements, separating extra drawing from
extra headroom granted where a limit changed.

### 3. Watchlist

Only Red and Amber customers, worst first. Two to four lines each: what fired, the
numbers behind it, and the action warranted. No tables.

### 4. Operating normally

One line per Green customer. Name it, its EAD, and why it is clean. No detail.

### 5. Maturity ladder

| Bucket | Utilised | Facilities | Renewal |

Buckets: 0–3 months, 3–6, 6–12, over 12. Name the facilities in each bucket with
their maturity date and days remaining, and show renewal status with a RAG marker.
Give any facility already past its maturity date with a balance outstanding its own
**Past maturity** row at the bottom, marked ⚠️.

### 6. So what

At most 200 words, prose, no bullets. Cover refinancing risk, liquidity and
recovery. Name the two or three customers that most need attention and say what
action is warranted. Rank by loss severity, not flag count, and where the two
disagree, say so in the final line.

### 7. Data quality issues

Separate from the flags, and never allowed to silently change a number: balances
after maturity, missing months in a series, collateral with no valuation date,
valuations more than 24 months old, non-GBP facilities, any other inconsistency.

### 8. Appendix — workings

Everything an auditor would need, and nothing a risk partner reads:

- Current view resolution: the proposal chosen per customer, and which were
  excluded and why.
- Facility table: limit, utilised, utilisation %, EAD with the CCF applied,
  collateral after haircut, covered, uncovered, LGD, loss severity, flags.
- Every flag with rule ID, facility, triggering values, one-line rationale.
- Assumptions and exclusions, including that PD and expected loss are out of
  scope.

---

## How to run it

1. Attach the three CSVs. If only Excel is accepted, attach
   `credit_exposure_inputs_raw.xlsx`, whose sheets carry the same names.
2. Send the prompt above.
3. Check the row counts first: facility 28, utilisation 113, collateral 18. If
   they are wrong, nothing downstream is trustworthy.
4. If the output stops early, ask for the remaining sections by number.

## What to check in the output

The traps table in the main README. Above all: C004's Pending proposal must be
excluded, F402 at zero utilisation must still carry EAD, F403 in EUR must be kept
separate, F401 must appear in the Past maturity row, F501's missing February must
be reported rather than read as a fall to zero, and the six unsecured facilities
must be called unsecured rather than coverage breaches.

## Version history

- **v2** — portfolio-first. Sections reordered so the answer leads and all
  workings moved to an appendix. Added RAG status per customer with an inline
  legend, the sparkline trend block, watchlist and operating-normally split, and
  a Past maturity row on the ladder.
- **v1** — `02_exposure_snapshot_v1_workings_first.md`. Correct but unreadable:
  it walked through every calculation before reaching any conclusion.
