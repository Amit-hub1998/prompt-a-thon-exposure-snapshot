# Prompts

| File | Purpose | Status |
|---|---|---|
| `02_exposure_snapshot.md` | The main analysis prompt. Attach the three CSVs and send it. | v2, current |
| `02_exposure_snapshot_v1_workings_first.md` | First draft. Kept to show why the output was restructured. | Superseded |
| `01_generate_data.md` | Prompt that regenerates the three input tables from scratch. | To do |
| `03_eval_rubric.md` | Scoring rubric and answer key. | To do |

## Design note

v1 asked for the workings before the narrative. That is right for an auditor and
wrong for a risk partner, who reads the first page and stops. v2 inverts it: the
portfolio table and RAG status lead, the trend and watchlist follow, and every
calculation moves to an appendix.

## Optional: Mermaid trend chart

If the target tool renders Mermaid, swap section 2b for a real line chart. Worth
one test message before the event: paste the block below and see whether it draws
axes or shows as raw code.

```
xychart-beta
    title "Customer utilisation %"
    x-axis [Oct, Nov, Dec, Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep]
    y-axis "Utilisation %" 0 --> 110
    line [60, 61, 63, 61, 64, 63, 65, 67, 61, 59, 63, 95]
```

If it renders, keep the gauge bars from 2a above it and replace the sparklines.
If it does not, keep v4 as written.
