# Prompt-A-Thon submission content

Slide-ready text for the five submission slides. Each block maps to one box on the
template. Text in `[square brackets]` is a placeholder to fill with your own
results before submitting.

---

## Slide 1 — Problem Understanding

**Use case:** AI-Enabled Limits and Utilisation Monitoring for Regulatory Reporting

### Business Problem

Limits, utilisation and collateral sit in separate systems. Consolidating,
validating and interpreting them for regulatory and risk reporting is manual and
slow. The result: limit breaches identified late, inconsistent reporting between
teams, data quality issues found at review stage rather than at source, and long
review cycles.

### Who is impacted

- Regulatory reporting, credit risk and portfolio management teams
- Relationship managers, credit officers, risk data and technology teams
- Compliance, finance, internal audit and senior risk oversight

### Description

Today, users collect approved limits, utilisation and collateral from multiple
sources, reconcile inconsistencies by hand, and investigate exceptions one at a
time. This use case consolidates the three datasets in a single prompt, calculates
utilisation and exposure, applies explicit early warning rules, flags data quality
issues, and produces a one-page summary a risk partner can read in two minutes.

**What good looks like:** one consistent view per customer, breaches and maturities
visible without manual stitching, and every flag traceable to a named rule and the
numbers that triggered it.

---

## Slide 2 — AI Approach & Prompting Strategy

### Tool used: GitHub Copilot in VS Code

- Designed and refined the input data model (facility, utilisation, collateral)
- Drafted, tested and iterated the analysis prompt across seven versions
- Compared output quality across the available models
  `[list the models you tested, e.g. model A, model B]`
- Used for structuring, analysis, drafting and refining, all in one workspace

### AI Assist: Grader agent

A second prompt acts as a **reverse grader**. It takes the generated report and an
independently built answer key, and scores the output on numerical accuracy,
correct flags, missed traps and format. Appropriate because it turns "the output
looks right" into a repeatable, scored check. `[mark "Not used" if not built]`

### Why this tool mix

Copilot gave access to several models in one place, so the same prompt could be
tested across them and the most reliable chosen. The grader separates generation
from checking, which is the simplest way to catch errors a single prompt cannot see
in its own output.

### Description

**Workflow:** resolve current view → calculate exposure → apply rules → classify
RAG → summarise → recommend actions.

**Prompting pattern:**

- **Role:** senior credit risk analyst writing for risk partners who read one page
- **Input schema:** three tables, every column defined in the prompt
- **Instructions:** a fixed calculation order, so numbers are produced before any
  narrative
- **Output schema:** nine fixed sections, with a template for each

**Rules versus LLM:** thresholds are deterministic and written into the prompt as a
seven-rule table (utilisation above 85%, excess over limit, arrears, maturity
within 90 days without renewal, and others). The model does the joins, arithmetic
and narrative. It never decides what counts as a breach.

**Context provided:** synthetic tables attached as files; EAD, LGD and loss
severity formulas stated inline; a RAG legend printed on the report itself.

**Reducing hallucination:**

- Use only the supplied data; write "not provided" rather than estimate
- Never invent a rule; apply thresholds strictly, with values on a threshold not
  breaching
- Out-of-scope items (PD, expected loss, capital) named explicitly so the model
  does not fabricate them
- As-at date taken from the data, never from the model's idea of today
- Totals must reconcile from facility to customer level
- Every flag traceable in a flag register to rule, facility and triggering values

---

## Slide 3 — Solution / Output

### What is produced

A one-page limits and exposure snapshot, plus three supporting sections:

1. Portfolio snapshot with RAG status per customer
2. Utilisation trend: where each customer stands now, and how it got there
3. Watchlist of Red and Amber customers with actions
4. Customers operating normally
5. Maturity ladder, including facilities past maturity
6. Impact summary: refinancing, liquidity and recovery
7. Data quality issues
8. Flag register
9. Assumptions and exclusions

`[add a screenshot of sections 1 and 2]`

### Example / evidence

Customer G's utilisation jumps 28 percentage points in one month but ends at 78%,
just under the 85% threshold. The report raises a spike warning and correctly does
**not** raise a high-utilisation breach, and says why. A facility at zero
utilisation is still shown carrying exposure, because its committed undrawn limit
could be drawn at any time.

### How is it better

| Before | After |
|---|---|
| Data stitched manually from three sources | One prompt, three attached tables |
| Breaches found at review | Breaches flagged with rule and numbers |
| Inconsistent formats between analysts | Fixed nine-section output every run |
| Data issues discovered late | Data quality listed on the report itself |
| Hours per review cycle | Minutes, plus human review |

### Description

**Format:** markdown report. Tables for the snapshot, ladder, data quality and
flag register; text charts for trend; one-line bullets elsewhere.

**Mandatory fields:** as-at date on every section; RAG status per customer; rule ID
and triggering values for every flag.

**Variants:** sections 1 to 6 are the short version for risk partners; sections 7 to
9 are the full version for reviewers and audit.

**Enables:** prioritising which customers to call this week, starting renewals
before maturity, and escalating breaches early.

**Deliberately does not:** approve or change limits, make credit decisions, assign
ratings, or estimate probability of default or expected loss. It recommends
actions; people decide.

---

## Slide 4 — Validation, Controls & Responsible Use

### Human review

`[name / role]` and `[colleague names / roles]` reviewed each version. Checked:
every number against the answer key, every flag against the rule table, that no
flag fired on a value exactly on a threshold, and that the narrative matched the
numbers.

### Accuracy measures

- Answer key built independently of the prompt, for every facility and customer
- Fourteen planted edge cases in the data, each with a known correct answer
- Each run scored on: figures matching the answer key, flags correct (none missed,
  none false), and edge cases passed
- Repeated runs to test consistency

Results: `[X of 14 edge cases passed, Y% of figures matched, over N runs]`

### Responsible AI use

- Human in the loop: the report recommends, a person decides
- Explainable: every flag names its rule and the numbers behind it
- No speculation: missing values reported as "not provided", never estimated
- Scope stated on the report, so nobody mistakes severity for probability

### Data & confidentiality

- Fully synthetic dataset, created for this exercise
- Customer names are placeholders (Customer A to G)
- No customer-identifiable, restricted or production data used at any stage
- Permitted tools only

### Description

Accuracy is managed by fixing the rules in the prompt and checking the output
against an independent answer key. Human review sits at two points: after each
prompt revision, and before any report is acted on. Exceptions are routed to a
data quality section rather than silently absorbed. Every figure is traceable
through the flag register to its source facility. Errors found in review were fed
back as prompt changes, tracked across seven versions in the repository.

---

## Slide 5 — Business Value & Reusability

### Business value

- Faster turnaround: consolidation and first-pass analysis in minutes
  `[replace with your own timing: e.g. manual X hours vs Y minutes]`
- Earlier breach detection, before review rather than during it
- Consistent output regardless of who runs it
- Data quality surfaced on every run, not discovered downstream

### Usefulness

Output accuracy is checked against a known answer key on every version. Risk
partners get the answer on page one; reviewers get the flag register and data
quality detail behind it. `[add your accuracy result here]`

### Reusability

- Any portfolio, product line, region or team with limits, utilisation and
  collateral data
- Weekly or monthly cadence without changes
- Rules, thresholds, haircuts and CCFs live in the data and prompt, so they can be
  changed without rewriting the approach

### Description

**Impact:** less manual consolidation, fewer reporting inconsistencies, earlier
escalation of breaches and maturities, and an auditable trail from each flag to its
source.

**Reuse:** the same pattern applies to other portfolios, products and regions.

**What changes per use:** the input tables and the rule thresholds.

**What stays:** the method, the output structure and the validation approach.

**To scale:** a governed prompt template, an agreed rule table owned by risk, the
grader for ongoing accuracy checks, and short training for users on reading the
report and challenging its output.

This is a repeatable capability, not a one-off demo.
