# Dashboards and Data Storytelling

> **TL;DR** — A *dashboard* surfaces the metrics that someone monitors. A *story* leads the reader through a sequence of insights to a conclusion. They are different objects with different design rules. Most "dashboards" should be stories, and most "stories" should be shorter.

## Dashboards

### Principles

1. **One question per dashboard.** "What is the health of our acquisition funnel?" — not "everything about the company."
2. **The most important metric goes top-left.** Eyes start there.
3. **Big numbers (KPIs) at the top, trends below, breakdowns at the bottom.** Inverted pyramid.
4. **Show change, not absolute.** "↑ 12% vs. last week" is more actionable than "1,432 today".
5. **Use the same time axis on every chart.** Don't make readers re-anchor.
6. **One color per metric.** Consistency lets the eye lock on.
7. **Annotate known events** (deploys, promotions, holidays) on the time axis.
8. **Default to a "compare to" period** — DoD, WoW, YoY.
9. **No more than ~6 charts on one screen.** If you need more, break into tabs.
10. **Make filters visible.** Hidden filters cause "why does the number disagree with mine?" confusion.

### KPI tile

A big number, a small "vs. previous" indicator (% and arrow), a tiny trend sparkline. Three pieces of information in a few square inches.

### Layouts that work

- **Executive summary** — 4 KPI tiles + 1 trend chart + 1 breakdown.
- **Funnel** — large funnel chart left, conversion rates and counts right.
- **Operations** — current state at top (gauge / status), event log below.
- **Cohort** — heatmap (rows = cohort, columns = period age).

### Tools

| Tool | Strengths |
|---|---|
| **Looker / Tableau / PowerBI / Mode** | Enterprise BI, governance |
| **Metabase / Superset** | Open-source, SQL-first |
| **Streamlit / Dash / Panel / Shiny** | Custom dashboards in Python / R |
| **Observable / D3** | Highly custom, web |
| **Grafana** | Time-series, ops monitoring |

## Data storytelling

A story has a *thesis* and a *path*. A dashboard is a *map*.

### The shape

1. **Hook** — a single chart, no caveats yet, that makes the reader want to know more. "Conversion dropped 20% last quarter."
2. **Setup** — context. "Conversion has been stable at 4% for 8 quarters. Then it dropped to 3.2%."
3. **Investigation** — successive charts isolating the cause. Each chart answers one question and raises the next.
4. **Resolution** — the cause is identified. "It's not traffic mix; it's checkout latency on mobile."
5. **Implication** — what should happen. "Recommend fixing X by Q3."

### Per-chart rules

- **Title = conclusion.** Not "Sales by month," but "Sales fell in Q3, recovering in Q4."
- **Annotate the point.** Add a label or callout to the bar / line / region you want the reader to notice.
- **De-emphasize the rest.** Use color for what matters; gray for context.
- **One thing per chart.** If you're tempted to add a second axis, split into two charts.

### Format

For a written/slide story, alternate text and charts:

> *Sales fell 20% in Q3.* [chart 1] *The drop was concentrated in mobile.* [chart 2] *Mobile checkout latency had increased to 8s.* [chart 3]

For a talk: one chart per slide; one sentence per chart.

### Examples worth studying

- The New York Times graphics desk.
- Reuters Graphics.
- Bloomberg Graphics.
- Pudding.cool (long-form data essays).
- Information is Beautiful.

## Pitfalls

1. **Dashboards as compromise.** A dashboard meant to "answer everything" answers nothing.
2. **Charts without titles.** Readers don't know what they're looking at.
3. **Adding charts to look thorough.** Each extra chart subtracts attention.
4. **Decorative elements** (logos, 3-D, gradients) that occupy ink without information.
5. **Telling the wrong story.** Confirmation bias is easy with charts — show the chart that would falsify your claim and explain why it doesn't.

## References

- Tufte. *The Visual Display of Quantitative Information.*
- Few. *Information Dashboard Design.*
- Cairo. *The Truthful Art.*
- Knaflic. *Storytelling with Data.*
- Wilke. *Fundamentals of Data Visualization* (free online).
