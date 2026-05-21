# Notebooks

Runnable Python examples that accompany the handbook chapters. Every notebook is designed to run top-to-bottom on a fresh kernel, using only `requirements.txt` and the small CSVs in `sample_data/`.

## Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r ../requirements.txt
jupyter lab
```

## Cleaning

| Notebook | Companion chapter |
|---|---|
| [cleaning/01_inspection_and_profiling.ipynb](cleaning/01_inspection_and_profiling.ipynb) | [../data_cleaning/README.md](../data_cleaning/README.md) |
| [cleaning/02_missing_values.ipynb](cleaning/02_missing_values.ipynb) | [../data_cleaning/missing_values.md](../data_cleaning/missing_values.md) |
| [cleaning/03_outliers.ipynb](cleaning/03_outliers.ipynb) | [../data_cleaning/outliers.md](../data_cleaning/outliers.md) |
| [cleaning/04_text_cleaning.ipynb](cleaning/04_text_cleaning.ipynb) | [../data_cleaning/text_cleaning.md](../data_cleaning/text_cleaning.md) |

## Visualization

| Notebook | Companion chapter |
|---|---|
| [visualization/01_distributions.ipynb](visualization/01_distributions.ipynb) | [../data_visualization/distributions.md](../data_visualization/distributions.md) |
| [visualization/02_relationships.ipynb](visualization/02_relationships.ipynb) | [../data_visualization/relationships.md](../data_visualization/relationships.md) |
| [visualization/03_trends.ipynb](visualization/03_trends.ipynb) | [../data_visualization/trends.md](../data_visualization/trends.md) |

Each notebook generates its own synthetic data, so no external downloads are needed.
