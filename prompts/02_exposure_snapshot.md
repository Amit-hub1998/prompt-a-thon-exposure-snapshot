# Main analysis prompt — Limits & Exposure Snapshot + Early Warning Narrative

Attach `facility.csv`, `utilisation.csv` and `collateral.csv`, then send
everything between the lines below.

---

**Respond in text and markdown only.** Do not generate images, do not render a
chart, and do not write code to draw one. Every chart in this report is a text
block specified below. Produce all nine sections in full, every time. A chart on its
own is not an answer.

You are a senior credit risk analyst preparing a weekly pack for risk partners.
They will read the first page and nothing else. Lead with the answer. Put every
calculation in an appendix at the end.

Work only from the three attached tables. Read every row. Do not sample or
truncate.

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

**As-at date:** the latest month_end_date in the utilisation table. Use it for
days to maturity and for every "latest" figure. Never use today's date.

Print the as-at date visibly, in the form **As at 30-Sep-2026 (latest month-end in
the data)**, in each of these places:

- the report title line at the very top
- the title line of chart 2a and chart 2b
- the heading of the maturity ladder, with days remaining measured from it
- the heading of the impact summary

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
maturity_date. Then:
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

**Guardrails.** All amounts are in USD. Use only the values given; write "not
provided" rather than estimating. Facility totals must sum to the customer totals
you report. Report improvement as readily as deterioration. A
facility with fewer than two month-end rows has no trend; say so rather than
inferring one. Do not dramatise.

## Output

Exactly these sections, in this order and no others. Sections 1 to 6 are the
report and should fit about one page. Sections 7 to 9 follow as supporting
detail. Do not add a workings or calculation section.

Open the report with one title line:

**Limits & Exposure Snapshot — As at DD-Mon-YYYY (latest month-end in the data)**

### 1. Portfolio snapshot

One headline line: customers, facilities, total limits, total EAD, total loss
severity, as at the as-at date, stated explicitly.

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

**Utilisation** for a customer means that customer's total utilised divided by
total limit, across all its facilities, using the limit in force in that month.
Two code blocks follow, each opening with a one-line title that says exactly what
is being plotted.

**2a. Where each customer stands now** — a comparison across customers at a single
point in time. No time axis.

```
Utilisation by customer — As at 30-Sep-2026  (total utilised ÷ total limit)

                     0%        50%      85%  100%
                     |---------|--------|----|
Customer C           ████████████████████████  105%  🔴  over limit
Customer A           ███████████████████████░   95%  🔴
Customer B           ████████████████████░░░░   83%  🟠
Customer G           ███████████████████░░░░░   78%  🟠
Customer F           ██████████████████░░░░░░   74%  🟢
Customer E           ████████████░░░░░░░░░░░░   50%  🟢
```

- Sort by latest utilisation, highest first.
- Twenty-four characters wide, so each character is about 4 percentage points. Fill
  with █ up to the latest utilisation, pad with ░, cap at 24 characters but print
  the true percentage, which may exceed 100%.
- Replace the date in the title with the as-at date.
- After the percentage, the RAG emoji, then at most three words of reason where the
  status is not Green.

**2b. How each customer got there** — the same measure tracked month by month over
the last twelve month-ends, in the same customer order as 2a.

```
Monthly utilisation by customer — 12 month-ends to 30-Sep-2026  (one bar = one month)

                  Oct-25 ──────► Sep-26    start → now    12m change
Customer C                 ▇▇█             n/a  → 105%    ▲  n/a
Customer A        ▃▃▄▃▄▄▄▄▄▄▄█             60%  →  95%    ▲ +35pp
Customer B        ▆▆▆▆▆▆▇▇▇▇▇▇             75%  →  83%    ▲  +8pp
Customer G        ▄▄▄▄▄▄▄▄▄▄▄▇             50%  →  78%    ▲ +28pp
Customer F        ▆▆▆▆▆▆▆▆▆▆▆▆             76%  →  74%    ▬  −2pp
Customer E        ██·▇▇▆▆▅▅▄▄▄             83%  →  50%    ▼ −33pp

▁ 0-12%  ▂ 13-25%  ▃ 26-37%  ▄ 38-50%  ▅ 51-62%  ▆ 63-75%  ▇ 76-87%  █ 88%+   · no data
```

- Replace the dates in the title and header with the actual first and last month-ends.
- One character per month, oldest left, latest right, using the bands in the
  legend line.
- `·` for a month with no data. A blank position where the customer did not yet
  exist.
- Where there is less than twelve months of history, print `n/a` for the start and
  the change rather than computing them.
- ▲ deteriorating, ▼ improving, ▬ flat within 5 percentage points.

Under the two blocks, three to five bullets, one line each, naming what drove the
largest movements. Where a limit changed in the same period, split the movement
into extra drawing and extra headroom granted.

### 3. Watchlist

Only Red and Amber customers, worst first. For each, a bold heading with the
customer, its status and its loss severity, then two to four bullets, **one line
each**:

- one bullet per rule that fired, giving the facility, the triggering numbers and
  the reason in the same line
- one final bullet starting **Action:** saying what should happen and by when

No paragraphs in this section.

### 4. Operating normally

One line per Green customer. Name it, its EAD, and why it is clean. No detail.

### 5. Maturity ladder — As at DD-Mon-YYYY

| Bucket | Utilised | Facilities | Renewal |

Buckets: 0–3 months, 3–6, 6–12, over 12. Name the facilities in each bucket with
their maturity date and days remaining, and show renewal status with a RAG marker.
Give any facility already past its maturity date with a balance outstanding its own
**Past maturity** row at the bottom, marked ⚠️.

### 6. Impact summary — refinancing, liquidity and recovery (As at DD-Mon-YYYY)

At most 150 words. Open with one sentence stating the single most important thing
in the pack. Then four bullets, one line each:

- **Refinancing:** what falls due inside 90 days and whether renewals are on track
- **Liquidity:** where headroom is nearly exhausted
- **Recovery:** where loss severity is concentrated and why
- **Priority:** the two or three customers to act on first

Close with one line ranking by loss severity rather than flag count, and say where
the two disagree.

### 7. Data quality issues

A table, never prose. These are reported, never allowed to silently change a number.

| Issue | Where | What it affects |

Cover at least: balances after a facility's maturity date, missing months in a
utilisation series, collateral with no valuation date, valuations more than 24
months before the as-at date, and any other inconsistency between the tables.

### 8. Flag register

A table of every flag that fired: rule ID, customer, facility, triggering values,
one-line rationale. Red before Amber.

### 9. Assumptions and exclusions

Bullets, one line each. Include that PD, expected loss and regulatory capital are
out of scope, and list any value you had to treat as not provided.

---

## How to run it

1. Attach `facility.csv`, `utilisation.csv` and `collateral.csv`. If only Excel is
   accepted, attach `credit_exposure_inputs_raw.xlsx`, whose sheets carry the same
   names.
2. Send the prompt above.
3. If the output stops early, ask for the remaining sections by number.

## What to check in the output

The traps table in the main README. Above all: Customer D's Pending proposal must be
excluded, F402 at zero utilisation must still carry EAD, F401 must appear in the Past
maturity row of the ladder, Customer G must raise a spike flag but not a high
utilisation flag, and unsecured facilities must be called unsecured rather than
coverage breaches.

## Version history

- **v7** — opening instruction to respond in text only and produce all nine
  sections. Stops the tool returning a rendered utilisation chart instead of the
  report.
- **v6** — as-at date printed in the title, both chart titles, the maturity ladder
  and the impact summary. "So what" renamed to Impact summary. Data quality
  section restored; flag register and assumptions kept as their own sections;
  workings still excluded.
- **v5** — as-at date derived from the latest month-end instead of hard-coded.
  Single currency (USD). Customers anonymised to Customer A–G. Report cut to six
  sections plus an appendix of flag register and assumptions only; data quality,
  current-view resolution and the workings table removed. Each trend block now opens
  with a title stating what is plotted and over what period.
- **v4** — trend split into two blocks: gauge bars showing where every customer
  stands now, then sparklines showing how they got there. The month-axis header
  was dropped; it carried little information and cluttered the block.
- **v3** — removed the row-count hard stop that caused the model to loop. Trend
  block gained a month axis, a scale legend, and 12/6/3-month deltas. Watchlist,
  so-what and data quality converted from prose to one-line bullets and a table.
- **v2** — portfolio-first. Sections reordered so the answer leads and all
  workings moved to an appendix. Added RAG status per customer with an inline
  legend, the sparkline trend block, watchlist and operating-normally split, and
  a Past maturity row on the ladder.
- **v1** — `02_exposure_snapshot_v1_workings_first.md`. Correct but unreadable:
  it walked through every calculation before reaching any conclusion.
