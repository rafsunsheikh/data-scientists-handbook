# Numerical Data

> **TL;DR** — Numerical data is any value that supports meaningful arithmetic. The key sub-distinctions are *continuous vs. discrete* (does the value live on the real line or on the integers?) and *ratio vs. interval* (does zero mean absence?). Get these wrong and you'll compute summary statistics that don't mean what you think they mean.

## 1. Sub-types

### 1.1 Continuous (real-valued)
- **Examples:** height (cm), revenue ($), latency (ms), pH, temperature.
- **Storage:** `float32`, `float64`, `Decimal` (for money).
- **Watch out for:** floating-point precision, units, scale (log-normal data is common in business metrics).

### 1.2 Discrete (integer-valued)
- **Examples:** count of purchases, number of children, dice roll.
- **Storage:** `int8` … `int64`, unsigned variants.
- **Watch out for:** sentinel values like `-1` or `999` used to encode "unknown" — these will silently corrupt your means and histograms.

### 1.3 Ratio vs. Interval (Stevens' scales [^stevens])
| | Ratio | Interval |
|---|---|---|
| Has true zero | Yes | No |
| Multiplication / ratios valid | Yes ("twice as much") | No |
| Examples | mass, length, count, revenue | °C, °F, calendar year, IQ score |
| Valid statistics | mean, geometric mean, CV, ratios | mean, SD; **not** ratios |

You can compute the mean of temperatures in °C, but you cannot say "20°C is twice as warm as 10°C." On the Kelvin scale (ratio) you can.

### 1.4 Bounded vs. unbounded
- **Bounded:** percentages (0–100), probabilities (0–1), pixel intensities (0–255).
- **Unbounded:** revenue, age, distance.

Bounded data often needs a transformation (logit, beta) before linear models.

### 1.5 Signed vs. unsigned
- A negative revenue might be a refund — or a parsing bug. Always confirm.

## 2. Distributions you'll meet

| Distribution | Where it shows up | How to spot it |
|---|---|---|
| Normal | Measurement errors, sums of independent factors | Symmetric bell, mean ≈ median |
| Log-normal | Income, file sizes, time durations, gene expression | Right-skewed; symmetric on log scale |
| Poisson | Counts of rare events per unit time | Mean ≈ variance, integer-valued |
| Negative binomial | Over-dispersed counts (variance > mean) | Like Poisson but with a heavy tail |
| Power-law / Pareto | Website traffic, wealth, city sizes | Linear on log-log axes |
| Bimodal / mixture | Mixed populations (e.g., adult vs. child heights) | Two visible peaks in histogram |
| Heavy-tailed (Cauchy, Student-t) | Financial returns | Mean unreliable; outliers dominate |

Always plot the distribution before reporting a mean. (See [`data_visualization/distributions.md`](../data_visualization/distributions.md).)

## 3. Storage and precision pitfalls

- **Money:** never store as `float`. Use `Decimal` or store cents as `int64`. `0.1 + 0.2 != 0.3` in IEEE 754.
- **Large counts:** `int32` overflows at ~2.1 billion. Pageview counters and Unix timestamps in seconds have both hit this in production.
- **Mixed `int` and `NaN`:** pandas before 2.0 forced an integer column to `float64` if it contained `NaN`. Use the nullable `Int64` dtype.
- **Unit mismatches:** the most expensive bug in this class crashed the Mars Climate Orbiter (pounds-force vs. newtons) [^nasa-mco]. Always store units in the column name or schema.

## 4. Cleaning checklist

See [`data_cleaning/`](../data_cleaning/) for full treatment. For numerical columns specifically:

- [ ] Check the dtype is what you expect (no `object` columns hiding strings).
- [ ] Check for sentinel values (`-1`, `999`, `9999-12-31`).
- [ ] Plot the distribution. Log-transform if right-skewed.
- [ ] Identify outliers — but don't drop them reflexively (see [`data_cleaning/outliers.md`](../data_cleaning/outliers.md)).
- [ ] Confirm units (and that they are consistent across rows / sources).
- [ ] Decide on a scaling strategy (z-score, min-max, robust) appropriate for your model.

## 5. Visualizations

| Question | Chart |
|---|---|
| What does the distribution look like? | Histogram, KDE, ECDF, violin |
| Are there outliers? | Box plot, ECDF on log scale |
| How do two numerical variables relate? | Scatter, hexbin, 2D KDE |
| How does a numerical variable change over time? | Line chart |
| How does a numerical variable differ across groups? | Box plot, strip plot, violin by group |

See [`data_visualization/`](../data_visualization/) for full chart catalog.

## 6. Common pitfalls

1. **Reporting a mean for skewed data.** Use median + IQR for income, latency, file sizes.
2. **Treating IDs as numerical.** A `customer_id` of 42 is not "twice" customer 21.
3. **Ignoring units.** "Distance: 5" — meters? miles? km? light-years?
4. **Discretizing continuous data prematurely.** Bucketing age into "young/old" before EDA throws away information.
5. **Comparing means across groups of wildly different sizes** without a confidence interval.

[^stevens]: Stevens, S. S. (1946). *On the Theory of Scales of Measurement.* Science, 103(2684), 677–680.
[^nasa-mco]: NASA. (1999). *Mars Climate Orbiter Mishap Investigation Board Phase I Report.*
