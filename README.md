<a id="readme-top"></a>

[![Contributors][contributors-shield]][contributors-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]

<br />
<div align="center">
  <h1 align="center">The Data Scientist's Handbook</h1>
  <p align="center">
    A community-maintained reference covering <strong>every type of data</strong> a data scientist might encounter, <strong>where it comes from</strong>, <strong>how to clean it</strong>, <strong>how to visualize it</strong>, and <strong>how it shows up across industries</strong>.
    <br /><br />
    <a href="#how-to-use-this-handbook"><strong>Start reading »</strong></a>
    <br /><br />
    <a href="#quickstart">Run the notebooks</a>
    &middot;
    <a href="https://github.com/rafsunsheikh/data-scientists-handbook/issues">Report an Error</a>
    &middot;
    <a href="CONTRIBUTING.md">Contribute</a>
  </p>
</div>

<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#about-the-handbook">About The Handbook</a></li>
    <li><a href="#status">Status</a></li>
    <li><a href="#how-to-use-this-handbook">How to use this handbook</a></li>
    <li><a href="#repository-layout">Repository layout</a></li>
    <li><a href="#quickstart">Quickstart</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
  </ol>
</details>

## About The Handbook

This handbook is organized around five questions that recur on every data project:

1. **What kind of data is this?** → [`data_types/`](data_types/)
2. **Where does it come from?** → [`data_sources/`](data_sources/)
3. **What industries produce it, and what shape is it in there?** → [`industries/`](industries/)
4. **What does it need before it's analyzable?** → [`data_cleaning/`](data_cleaning/)
5. **How do I show what's in it?** → [`data_visualization/`](data_visualization/)

Each chapter is a self-contained markdown file. Where Python is helpful, you'll find a companion notebook in [`notebooks/`](notebooks/).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Status

All five sections now have chapters written in depth:

| Section | What it covers | Status |
|---------|----------------|--------|
| [`data_types/`](data_types/) | Numerical, categorical, text, time series, image, audio, geospatial, graph and multimodal data | Comprehensive |
| [`data_sources/`](data_sources/) | Databases, APIs, files, web scraping, streaming, sensors and IoT, surveys, public datasets and synthetic data | Comprehensive |
| [`industries/`](industries/) | All 15 industries, from healthcare and finance to agriculture and cybersecurity | Comprehensive |
| [`data_cleaning/`](data_cleaning/) | Missing values, outliers, duplicates, parsing, and cleaning for each data type | Comprehensive |
| [`data_visualization/`](data_visualization/) | A chart catalog organized by purpose and data type, plus dashboards and storytelling | Comprehensive |

The handbook is still growing. Corrections, extra examples and new notebooks are always welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) if you'd like to help.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## How to use this handbook

**As a reference.** Browse to the chapter that matches your current problem. Each chapter has a TL;DR at the top and decision tables in the middle so you can find the answer to "which method should I use?" quickly.

**As a learning path.** New to data science? Read in order: data types → data sources → data cleaning → data visualization → industries. By the end you'll have a working mental model of the whole pipeline.

**As a checklist.** Starting a new project? Use the chapter intros as a checklist. "Do I know what type each column is? Do I know its source? Do I have a cleaning plan? Do I have a viz plan?"

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Repository layout

```
data-scientists-handbook/
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
│   ├── _chart_catalog_reference.md
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
│   ├── README.md              # notebook index
│   ├── cleaning/
│   └── visualization/
│
└── references/                # bibliography and external links
    └── README.md
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Quickstart

```bash
git clone https://github.com/rafsunsheikh/data-scientists-handbook.git
cd data-scientists-handbook
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

Then open any notebook under `notebooks/`. There are four cleaning notebooks (inspection and profiling, missing values, outliers, text cleaning) and three visualization notebooks (distributions, relationships, trends). See [`notebooks/README.md`](notebooks/README.md) for the full list.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Contributing

Contributions are welcome: new notebooks, worked visualization examples, corrections, citations and translations. Read [CONTRIBUTING.md](CONTRIBUTING.md) for the style guide and the pull request checklist.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## License

MIT. See [LICENSE](LICENSE).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
[contributors-shield]: https://img.shields.io/github/contributors/rafsunsheikh/data-scientists-handbook.svg?style=for-the-badge
[contributors-url]: https://github.com/rafsunsheikh/data-scientists-handbook/graphs/contributors
[stars-shield]: https://img.shields.io/github/stars/rafsunsheikh/data-scientists-handbook.svg?style=for-the-badge
[stars-url]: https://github.com/rafsunsheikh/data-scientists-handbook/stargazers
[issues-shield]: https://img.shields.io/github/issues/rafsunsheikh/data-scientists-handbook.svg?style=for-the-badge
[issues-url]: https://github.com/rafsunsheikh/data-scientists-handbook/issues
[license-shield]: https://img.shields.io/github/license/rafsunsheikh/data-scientists-handbook.svg?style=for-the-badge
[license-url]: https://github.com/rafsunsheikh/data-scientists-handbook/blob/main/LICENSE
