# Data Types

> **TL;DR** — Every column, file, or stream you ever analyze fits into one (or a combination) of a small number of fundamental data types. The type determines what statistics make sense, what cleaning is required, and what visualizations are valid. Get the type wrong and everything downstream is wrong.

## A working taxonomy

Data types can be classified along several axes. We use the following pragmatic taxonomy throughout the handbook:

| Axis | Values |
|---|---|
| **Measurement scale** (Stevens, 1946 [^stevens]) | Nominal, Ordinal, Interval, Ratio |
| **Mathematical structure** | Scalar, Vector, Tensor, Sequence, Graph |
| **Storage modality** | Tabular, Document, Image, Audio, Video, Geospatial, Stream |
| **Cardinality** | Univariate, Multivariate, High-dimensional |

A single dataset typically mixes several. A customer record might be tabular (modality) with a mix of numerical-ratio (income), categorical-nominal (country), ordinal (NPS score), and text (free-form review) columns, plus a geospatial point (home location) and a graph (referral network).

## Index

| Chapter | Covers |
|---|---|
| [numerical.md](numerical.md) | Continuous, discrete, ratio vs. interval, units |
| [categorical.md](categorical.md) | Nominal, ordinal, binary, high-cardinality |
| [text.md](text.md) | Strings, documents, conversations, code |
| [time_series.md](time_series.md) | Regular, irregular, multivariate, panel, event |
| [image.md](image.md) | Raster, vector, multispectral, medical (DICOM) |
| [audio.md](audio.md) | Waveform, spectrogram, MIDI, speech |
| [geospatial.md](geospatial.md) | Points, lines, polygons, rasters, trajectories |
| [graph.md](graph.md) | Directed, undirected, weighted, multigraph, temporal |
| [multimodal.md](multimodal.md) | Combinations: image+text, video, sensor fusion |

## Choosing the right type

Use this decision tree before you do anything else with a column:

```
Is the value a number that supports arithmetic (mean, ratio)?
├── Yes → does 0 mean "absence"?
│         ├── Yes → ratio (e.g., revenue, count, distance)
│         └── No  → interval (e.g., temperature in °C, calendar year)
└── No  → is there a natural order?
          ├── Yes → ordinal (e.g., satisfaction 1–5, education level)
          └── No  → nominal (e.g., country, product SKU)
                    └── Is the cardinality very large (>1000)?
                          └── treat as high-cardinality categorical
                              or as text/identifier
```

For non-tabular data (image, audio, geospatial, graph, text) the chapter for that modality goes deeper than this tree.

## Why this matters

| Decision | Driven by data type |
|---|---|
| What summary statistic is meaningful | mean is invalid on nominal data |
| What missing-value strategy is valid | mean-imputation is invalid on categorical |
| What encoding to use before ML | one-hot vs. ordinal vs. embedding |
| What chart type is honest | bar vs. histogram vs. line vs. heatmap |
| What distance metric to use | Euclidean vs. Hamming vs. Jaccard vs. DTW |
| What model class is appropriate | regression vs. classification vs. CNN vs. sequence model |

Mismatched type and method is the single most common source of silent errors in data analysis [^wickham-tidy].

[^stevens]: Stevens, S. S. (1946). *On the Theory of Scales of Measurement.* Science, 103(2684), 677–680.
[^wickham-tidy]: Wickham, H. (2014). *Tidy Data.* Journal of Statistical Software, 59(10).
