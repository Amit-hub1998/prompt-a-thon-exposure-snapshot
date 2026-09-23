# Five-minute pitch

Spoken script, roughly 650 words. Timings in brackets. Fill the placeholders
before presenting.

---

## 1. The problem (45 seconds)

A risk partner covering a portfolio has to answer one question every week: which
clients need my attention now?

Answering it today means opening three systems, exporting limits, utilisation and
collateral, reconciling them by hand, and only then working out who is close to
their ceiling and what falls due next quarter. By the time that is done, a breach
may already be a week old, and two analysts doing the same job will format it two
different ways.

We wanted one consistent view per customer, produced in minutes, where every
warning is traceable to a rule and the numbers behind it.

## 2. Our AI approach (75 seconds)

We used **a single prompt**, not a chain. Three files attached, one report out.

That was a deliberate choice. Chaining would have meant passing intermediate
numbers between steps, and every hand-off is a chance for a figure to change
quietly. One prompt keeps the arithmetic in one place, and makes the whole thing
reproducible by anyone who has the file.

Getting there was iterative. The prompt went through **eight versions**, and the
repository keeps the first one alongside the last, because the gap between them is
the real story:

- Version 1 was correct and unreadable. It walked through every calculation before
  reaching a conclusion. A risk partner would never have got to the answer.
- So we inverted it. The answer leads, and the workings move to the back.
- Later versions fixed things we only discovered by running it: it looped on a
  self-check, it once returned a chart and nothing else, and its trend table did
  not line up on screen.

The important design decision: **thresholds are not left to the model**. Seven
rules with fixed thresholds are written into the prompt. The model does the joins,
the arithmetic and the narrative. It never decides what counts as a breach.

## 3. Our solution and output (90 seconds)

The output is a one-page pack.

- **Page one** is a portfolio table: every customer with a red, amber or green
  status, its exposure, its loss severity and a twelve-word headline. Below it, two
  charts. One shows where every customer stands now against the 85% threshold. The
  other shows how each got there over twelve months.
- **Then** the watchlist, with an action line per customer, the customers operating
  normally in one line each, the maturity ladder, and an impact summary.
- **Behind that**, for reviewers: data quality issues, a flag register tracing every
  warning to its rule, and the assumptions.

Two things it does that matter more than they look.

It ranks by **loss severity, not by flag count**, and says so where the two
disagree. A customer with three flags and a small secured exposure is not the
problem; a customer with one flag and an unsecured facility drawn to the ceiling
is.

And it is honest about scope. We have no rating data, so we cannot compute
probability of default. Rather than let the model invent one, the prompt excludes
it explicitly and reports severity, which answers how bad a default would be, not
how likely it is.

## 4. Validation and control (60 seconds)

We built the test data deliberately. **Fourteen traps** are planted in it, each one
a plausible way to be wrong:

- A pending credit proposal that was never approved, with a higher limit on it.
- A committed facility drawn to zero, which still carries exposure.
- A facility past its maturity date still carrying a balance.
- A utilisation spike that stops just under the threshold and must not raise a
  breach.

We built an answer key independently and scored each run against it.
`[X of 14 passed, over N runs]`

Limitations we know about: it is severity only, the thresholds and haircuts are
illustrative rather than policy, and the data is fully synthetic with anonymised
names. A person still decides; the report recommends.

## 5. Business value and reusability (45 seconds)

The value is not that AI wrote a report. It is that the same question gets the same
answer every week, in minutes, with the reasoning attached.

Reuse costs nothing structural. Swap the three tables and it works on another
portfolio, product or region. Change a threshold or a haircut and the numbers move
with it, because those live in the data and the prompt, not in code.

One prompt, three files, one page. The complexity is in the rules, not in the
prompting.
