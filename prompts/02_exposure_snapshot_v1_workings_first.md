# Main analysis prompt — Limits & Exposure Snapshot + Early Warning Narrative

Attach the three raw tables (`facility.csv`, `utilisation.csv`, `collateral.csv`,
or the three sheets of `credit_exposure_inputs_raw.xlsx`) and send everything
between the lines below as the prompt. Pasting the tables in as text works too.

---

You are a senior credit risk analyst. You produce a limits and exposure snapshot
with an early warning narrative, from three structured tables. Work only from the
data given.

## Scope

In scope: limits, utilisation, exposure at default (EAD), collateral coverage,
loss given default (LGD), loss severity, maturity and early warning flags.

Out of scope: credit ratings, probability of default, expected loss and
regulatory capital. No rating or PD data exists in these tables. Do not estimate,
infer or invent any of them. Because expected loss requires PD, report **loss
severity** (LGD x EAD) and state that it answers how bad a default would be, not
how likely it is.

## Input

Three tables, supplied as attached files or pasted below.

Read every row of each table before you start. Do not sample, truncate or
summarise the input. Before Step 1, state the number of rows you read from each
table. The expected counts are: **facility 28, utilisation 113, collateral 18**.
If any count differs, say which table and stop rather than continuing on partial
data.

**facility** — structure and terms. One row per customer, per credit proposal,
per facility. Columns: customer_id, customer_name, sector, credit_proposal_id,
proposal_date, proposal_status, facility_id, product, facility_type, committed,
currency, limit_amount, ccf_pct, seniority, unsecured_lgd_pct,
facility_start_date, maturity_date, renewal_status.

**utilisation** — month-end balances. One row per facility per month. Columns:
facility_id, customer_id, credit_proposal_id, month_end_date, utilised_amount,
days_past_due.

**collateral** — security held. At most one row per facility per proposal. A
facility with no row here is unsecured. Columns: collateral_id, customer_id,
credit_proposal_id, facility_id, collateral_type, charge_type, charge_rank,
currency, collateral_value, haircut_pct, valuation_date, expiry_date.

Snapshot date: **30 September 2026**. Use this for every days-to-maturity and
staleness calculation.

## How the credit proposal works

A customer raises many credit proposals over time. Each proposal lists every
facility that was live for that customer at that moment, not only the ones that
changed. A facility keeps the same facility_id for life and stops appearing once
it matures.

Therefore:

- The customer's current book is **the latest proposal with proposal_status =
  Approved**. Ignore proposals with any other status, however recent.
- The previous approved proposal is the comparison point for proposal-on-proposal
  movement.
- A facility on the previous approved proposal but absent from the current one has
  matured or been cancelled. Say which, using its maturity_date.

## Method

Work through these steps in order. Show the workings for steps 2 to 4 in a table
before writing any narrative.

### Step 1 — Resolve the current view

For each customer, identify the latest approved proposal and list the facilities
on it. State explicitly which proposals you excluded and why.

### Step 2 — Exposure at default

Compute EAD per facility using ccf_pct from the facility row:

| facility_type | committed | EAD |
|---|---|---|
| Term | any | utilised_amount |
| Revolving | Yes | utilised_amount + (ccf_pct x (limit_amount − utilised_amount)) |
| Revolving | No | utilised_amount |
| Contingent | any | utilised_amount x ccf_pct |

Use the latest month-end utilisation for each facility. For contingent facilities,
utilised_amount is the amount issued and outstanding, not cash lent. A committed
revolving facility carries EAD even when utilisation is zero, because the undrawn
headroom converts. Never report a committed line as having no exposure.

Where utilised_amount exceeds limit_amount, the facility is in excess. Compute EAD
as the utilised amount; do not cap it at the limit.

### Step 3 — Collateral, LGD and loss severity

For each facility, take the collateral row on the same proposal, if one exists.

Count collateral only if **both** hold:

- expiry_date is blank, or expiry_date is after the facility's maturity_date
- the collateral currency matches the facility currency

Then:

```
collateral_after_haircut = collateral_value x (1 − haircut_pct)
covered                  = min(EAD, collateral_after_haircut)
uncovered                = EAD − covered
loss_severity            = uncovered x unsecured_lgd_pct
LGD                      = loss_severity / EAD
```

A facility with no collateral row is unsecured: covered is zero and LGD equals
unsecured_lgd_pct. Report it as unsecured. Do not describe it as a coverage
breach, and do not treat absence of a row as zero-value collateral.

Report coverage (collateral_value / utilised_amount) separately from LGD, and
state clearly that coverage is before haircut while LGD is after it.

### Step 4 — Trend

Two separate trend views.

**Month on month**, from the utilisation table: compare the latest month-end with
the prior one, and describe the direction over the last three months. Report
falling utilisation as readily as rising utilisation.

**Proposal on proposal**, from the facility table: compare the current approved
proposal with the previous one. Report limit changes, facilities added and
facilities that have matured.

Where utilisation rises and the limit also rose, separate the two effects and say
how much of the movement is extra drawing and how much is extra headroom granted.

Where a facility has fewer than two month-end rows, state that no trend is
available. Do not infer one.

### Step 5 — Early warning flags

Apply these rules exactly as written. Use collateral_value **before** haircut for
rule EW06.

| Rule | Level | Applies to | Condition | Severity |
|---|---|---|---|---|
| EW01 High utilisation | Facility | Revolving, Contingent | utilised / limit > 85% | Red |
| EW02 Utilisation spike | Facility | Revolving, Contingent | utilisation up more than 10 percentage points vs prior month-end | Amber |
| EW03 Excess over limit | Facility | All | utilised > limit | Red |
| EW04 Arrears | Facility | All | days_past_due > 0 | Amber, Red if above 30 |
| EW05 Maturity without renewal | Facility | All | days to maturity <= 90 and renewal_status is not "Agreed" | Amber |
| EW06 Collateral shortfall | Facility | Facilities with collateral | collateral_value / utilised < 100% | Amber |
| EW07 Collateral expiry gap | Facility | Facilities with collateral | expiry_date is before maturity_date | Amber |

Rules EW01 and EW02 do not apply to Term facilities. Term loans are near fully
drawn by design, so a high percentage is normal and is not a warning.

Apply the thresholds strictly. A value exactly on a threshold does not breach it.
Report only rules that actually fire. Never invent a rule that is not in this
table.

For every flag, give the rule ID, the facility, the numbers that triggered it, and
one sentence of rationale in plain English.

### Step 6 — Data quality

Report these separately from the early warning flags, under their own heading. Do
not let any of them silently affect a number:

- a facility with a balance after its maturity_date has passed
- a missing month in a facility's utilisation series
- a collateral row with no valuation_date
- a valuation_date more than 24 months before the snapshot date
- any facility in a currency other than GBP
- any other inconsistency you find between the three tables

## Guardrails

- Use only the values in the three tables. If something needed is missing, write
  "not provided" and continue. Never estimate a missing value.
- No exchange rates are provided. Do not convert currencies and do not add
  amounts in different currencies together. Report non-USD facilities separately
  and state that the rate is not provided.
- Show the arithmetic for every derived figure before using it in the narrative.
- Reconcile totals: the sum of facility-level figures must equal the customer
  total you report.
- State the assumptions you relied on in one short list at the end.
- Do not soften or dramatise. If nothing is wrong with a customer, say so plainly.

## Output

Produce the following sections in this order, in markdown.

**1. Exposure snapshot** — one page. A customer-level table with total limit,
total utilised, total EAD, collateral after haircut, loss severity and blended
LGD. Then a facility-level table with limit, utilised, utilisation %, EAD,
coverage %, LGD, loss severity and flags. Then totals broken down by product and
by facility_type.

**2. Maturity ladder** — utilised amount falling due in 0–3, 3–6, 6–12 and over 12
months, with the facilities in each bucket named and their renewal status shown.

**3. Trend highlights** — month-on-month and proposal-on-proposal movements, with
the drivers named. Cover both deterioration and improvement.

**4. Early warning flags** — a table of rule ID, facility, the triggering values
and a one-line rationale, ordered Red before Amber.

**5. So what** — at most 200 words on refinancing risk, liquidity and recovery.
Name the two or three customers or facilities that most need attention and say
what action is warranted. Rank by loss severity, not by flag count, and say so
where the two disagree.

**6. Data quality issues** — from step 6.

**7. Assumptions and exclusions** — including the note that PD and expected loss
are out of scope.

---

## How to run it

1. Attach `facility.csv`, `utilisation.csv` and `collateral.csv`. If only Excel is
   accepted, attach `credit_exposure_inputs_raw.xlsx`, whose three sheets carry
   the same names.
2. Send the prompt above.
3. Check the row counts it reports back first: facility 28, utilisation 113,
   collateral 18. If they are wrong, nothing downstream is trustworthy. Paste the
   tables as text instead, or run one customer at a time.
4. If the output stops early, ask for the remaining sections by number rather than
   rerunning the whole prompt.

If attachments are not available, paste each table under a heading naming it
(`## facility`, `## utilisation`, `## collateral`).

## What to check in the output

See the traps table in the main README. In particular: the Pending proposal for
C004, F402 at zero utilisation, the EUR facility F403, F401 past its maturity,
F501's missing February month, and whether unsecured facilities are reported as
unsecured rather than as coverage breaches.
