# Text Visualization

> **TL;DR** — Use word-frequency bars (not word clouds) for top terms. Use embedding scatter plots (t-SNE/UMAP on document or token embeddings) to see semantic clusters. Use heatmaps for similarity matrices. Use small-multiples to compare topics or sentiments across groups.

## Pick a chart

| Goal | Chart |
|---|---|
| Most frequent words / n-grams | **Sorted bar chart** (not word cloud) |
| Vocabulary distribution / Zipf | **Log-log frequency plot** |
| Document length distribution | **Histogram, ECDF** |
| Semantic similarity between docs | **Cosine similarity heatmap** |
| Cluster structure of docs | **t-SNE / UMAP** of embeddings |
| Topic structure | **Topic-word bars per topic**; pyLDAvis |
| Sentiment over time | **Line chart** of mean sentiment per period |
| Word usage over time | **Multi-line chart** of top n-grams |
| Differences between corpora | **Word-shift graph** (Dodds et al.) |
| Attention / explanation | **Token-level heatmap** colored by attention or attribution |
| Search results / RAG retrieval | **Side-by-side document panels with highlighted spans** |

## Word frequency

```python
from collections import Counter
counts = Counter(tokens).most_common(30)
words, n = zip(*counts)
plt.barh(words, n)
plt.gca().invert_yaxis()
```

Always sort. Top-N is more useful than a "wall of words". For comparison between corpora, prefer:

- **Log-odds ratio with informative prior** (Monroe et al., 2008) — handles rare words better than raw frequency.
- **TF-IDF** ranking per group.

## Why not word clouds

Word clouds are a popular vis-cliché, but:

- Area is hard to compare.
- Orientation makes some words easier to read than others.
- They emphasize aesthetic over precision.

A horizontal bar chart of top 20 words conveys the same information faster and more honestly. If you must produce one (because a stakeholder insists), at least keep word orientation horizontal and use color sparingly.

## Embedding visualization

Embed documents (or sentences, or tokens) and project to 2-D:

```python
from sentence_transformers import SentenceTransformer
from umap import UMAP

model = SentenceTransformer('all-MiniLM-L6-v2')
emb = model.encode(docs)
xy = UMAP(n_components=2, random_state=0).fit_transform(emb)

plt.scatter(xy[:, 0], xy[:, 1], c=labels, s=4, alpha=0.7, cmap='tab20')
```

Interpret with care: t-SNE / UMAP distort distances. Trust *clusters*, not gaps between them.

## Topic models

For LDA, NMF, BERTopic:

- **Top-N words per topic** as a horizontal bar chart, one panel per topic.
- **Topic proportions over time** — line chart.
- **pyLDAvis** — interactive web view of LDA topics in PCA space with per-topic word relevance sliders.

For BERTopic specifically, its built-in `visualize_topics`, `visualize_barchart`, `visualize_heatmap`, and `visualize_documents` cover most needs.

## Token-level attention / attribution

For explaining a transformer's prediction:

- Highlight tokens by attention weight or integrated-gradients attribution.
- Plot in-text with `displaCy` or HTML span backgrounds colored on a sequential scale.
- Make sure to subtract / normalize baselines so the visualization actually reflects the model's reasoning, not just word frequency.

## Similarity matrix

```python
from sklearn.metrics.pairwise import cosine_similarity
S = cosine_similarity(emb)
sns.heatmap(S, cmap='magma', square=True)
```

Reorder rows/columns by hierarchical clustering (`sns.clustermap`) to bring similar documents together.

## Word-shift graph

Compares two corpora (or two time periods) and explains *which* words drive the difference in sentiment / frequency. Library: `shifterator`.

## Sentiment over time

Bin documents by period, compute mean sentiment (model-based, not lexicon-based when possible), plot as line with confidence band.

```python
df.groupby(pd.Grouper(key='ts', freq='W'))['sentiment'].mean().plot()
```

## Pitfalls

1. **Word clouds when a bar chart is better.**
2. **Raw counts** when log-odds or TF-IDF is more informative.
3. **Reading t-SNE / UMAP distances literally.**
4. **Plotting attention as if it were the model's explanation** — attention ≠ attribution.
5. **Comparing corpora of very different sizes** without normalization.
6. **Ignoring stop words** in frequency plots — top 10 = "the, of, and, ..."
