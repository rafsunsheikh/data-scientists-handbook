# The Data Scientist's Handbook

> A community-maintained reference covering **every type of data** a data scientist might encounter, **where it comes from**, **how to clean it**, **how to visualize it**, and **how it shows up across industries**.

This handbook is organized around five questions that recur on every data project:

1. **What kind of data is this?** → [`data_types/`](data_types/)
2. **Where does it come from?** → [`data_sources/`](data_sources/)
3. **What industries produce it, and what shape is it in there?** → [`industries/`](industries/)
4. **What does it need before it's analyzable?** → [`data_cleaning/`](data_cleaning/)
5. **How do I show what's in it?** → [`data_visualization/`](data_visualization/)

Each chapter is a self-contained markdown file. Where Python is helpful, you'll find a companion notebook in [`notebooks/`](notebooks/).

---

## Status

This is an early version of the handbook. The following sections are written in depth:

- [`data_types/`](data_types/) — comprehensive
- [`data_cleaning/`](data_cleaning/) — comprehensive
- [`data_visualization/`](data_visualization/) — comprehensive
- [`industries/`](industries/) — **5 of 15** filled in depth (Healthcare, Finance, Retail/E-commerce, Manufacturing, Marketing); rest are stubs

The following sections are **stubbed and looking for contributors**:

- [`data_sources/`](data_sources/) — topic outlines only
- [`industries/`](industries/) — 10 remaining stubs (Telecom, Energy, Transportation, Media, Government, Agriculture, Education, Insurance, Real Estate, Cybersecurity)

See [CONTRIBUTING.md](CONTRIBUTING.md) if you'd like to help fill them in.

---

## How to use this handbook

**As a reference.** Browse to the chapter that matches your current problem. Each chapter has a TL;DR at the top and decision tables in the middle so you can find the answer to "which method should I use?" quickly.

**As a learning path.** New to data science? Read in order: data types → data sources → data cleaning → data visualization → industries. By the end you'll have a working mental model of the whole pipeline.

**As a checklist.** Starting a new project? Use the chapter intros as a checklist. "Do I know what type each column is? Do I know its source? Do I have a cleaning plan? Do I have a viz plan?"

---

## Repository layout

```
data_scientists_handbook/
├── README.md                  # you are here
├── CONTRIBUTING.md            # how to contribute
├── LICENSE                    # MIT
├── requirements.txt           # Python dependencies for notebooks
│
├── data_types/                # what kind of data exists
│   ├── README.md              # taxonomy + index
│   ├── numerical.md
│   ├── categorical.md
│   ├── text.md
│   ├── time_series.md
│   ├── image.md
│   ├── audio.md
│   ├── geospatial.md
│   ├── graph.md
│   └── multimodal.md
│
├── data_sources/              # where data comes from
│   ├── README.md
│   ├── databases.md
│   ├── apis.md
│   ├── files.md
│   ├── web_scraping.md
│   ├── streaming.md
│   ├── sensors_iot.md
│   ├── surveys.md
│   ├── public_datasets.md
│   └── synthetic.md
│
├── industries/                # industry-specific data shapes
│   ├── README.md
│   ├── healthcare.md
│   ├── finance.md
│   ├── retail_ecommerce.md
│   ├── manufacturing.md
│   ├── telecom.md
│   ├── energy_utilities.md
│   ├── transportation_logistics.md
│   ├── media_entertainment.md
│   ├── government_public_sector.md
│   ├── agriculture.md
│   ├── education.md
│   ├── insurance.md
│   ├── real_estate.md
│   ├── marketing_advertising.md
│   └── cybersecurity.md
│
├── data_cleaning/             # cleaning per data type
│   ├── README.md
│   ├── missing_values.md
│   ├── outliers.md
│   ├── duplicates.md
│   ├── inconsistent_categories.md
│   ├── data_types_and_parsing.md
│   ├── text_cleaning.md
│   ├── time_series_cleaning.md
│   ├── image_cleaning.md
│   ├── audio_cleaning.md
│   ├── geospatial_cleaning.md
│   └── feature_scaling_encoding.md
│
├── data_visualization/        # viz catalog
│   ├── README.md              # picker: data type × purpose → chart
│   ├── distributions.md
│   ├── comparisons.md
│   ├── relationships.md
│   ├── compositions.md
│   ├── trends.md
│   ├── geospatial_viz.md
│   ├── network_viz.md
│   ├── text_viz.md
│   ├── image_viz.md
│   ├── multivariate.md
│   └── dashboards_and_storytelling.md
│
├── notebooks/                 # runnable Python examples
│   ├── cleaning/
│   └── visualization/
│
└── references/                # bibliography and external links
    └── README.md
```

---

## Quickstart

```bash
git clone https://github.com/<your-username>/data_scientists_handbook.git
cd data_scientists_handbook
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

Then open any notebook under `notebooks/`.

---

## License

MIT — see [LICENSE](LICENSE).
