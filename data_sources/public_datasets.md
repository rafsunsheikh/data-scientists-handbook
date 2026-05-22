# Public Datasets

> **TL;DR** — Public datasets are the backbone of data science education, benchmarking, and research. They range from government open data to academic research datasets to community-curated collections. The key challenges are licensing (many public datasets are NOT free to use commercially), reproducibility (datasets disappear or change versions), and data quality (public datasets often have undocumented cleaning issues). Always document the source, version, and license of every dataset you use. Prefer datasets with clear documentation (data dictionary, codebook, usage examples) and stable URLs or DOIs.

## 1. General catalogs and portals

### 1.1 Kaggle Datasets

https://www.kaggle.com/datasets

- **Size:** 100,000+ datasets.
- **Types:** Everything from toy datasets to proprietary data shared by companies.
- **Licensing:** Varies — check each dataset. Many are MIT, CC-BY, or custom.
- **Features:** Notebooks (Jupyter), discussions, leaderboards.
- **Best for:** Beginners, competitions, quick exploration.

### 1.2 Hugging Face Datasets

https://huggingface.co/datasets

- **Size:** 300,000+ datasets.
- **Types:** Primarily NLP, but growing to vision, audio, multimodal, tabular.
- **Licensing:** Each dataset has a license tag (CC-BY, Apache, MIT, custom).
- **Features:** `datasets` library for streaming, caching, and preprocessing.
- **Best for:** ML model training, benchmarking, NLP research.

```python
from datasets import load_dataset
dataset = load_dataset("glue", "mrpc")  # loads MRPC from GLUE benchmark
```

### 1.3 Google Dataset Search

https://datasetsearch.research.google.com/

- **Aggregator:** Searches across publishers, repositories, and websites.
- **Not a repository:** Links to the original source.
- **Best for:** Finding datasets when you don't know where they're hosted.

### 1.4 AWS Open Data

https://registry.opendata.aws/

- **Hosted on S3:** Free to access (standard S3 request costs apply).
- **Types:** Genomics, satellite imagery, government, scientific.
- **Features:** EC2 Launch Templates for large datasets.
- **Best for:** Cloud-native analysis, large-scale datasets.

### 1.5 Data.gov (US Government)

https://data.gov/

- **Size:** 300,000+ datasets from US federal agencies.
- **Types:** Agriculture, climate, education, energy, finance, geography, health.
- **License:** Most are CC0 or CC-BY (public domain or attribution).
- **Formats:** CSV, JSON, XML, KML, PDF.

### 1.6 EU Open Data Portal

https://data.europa.eu/

- **EU institutions and agencies.**
- **Types:** Statistics, geography, funding, publications.
- **License:** European Union Public License (EUPL) or CC-BY.

### 1.7 Zenodo

https://zenodo.org/

- **General-purpose:** Research outputs including datasets.
- **DOI assignment:** Every upload gets a DOI for citation.
- **Integration:** Connected to CERN, European Commission.
- **Best for:** Academic datasets, supplementary materials for papers.

### 1.8 ICPSR

https://www.icpsr.umich.edu/

- **Social sciences:** Sociology, political science, psychology, economics.
- **Access:** Many datasets free; some require membership (University of Michigan).
- **Best for:** Academic research in social sciences.

## 2. Government and official statistics

### 2.1 US agencies

| Agency | Datasets | Topics |
|---|---|---|
| **Census Bureau** | American Community Survey, Decennial Census, Population Estimates | Demographics, housing, economy |
| **BLS (Bureau of Labor Statistics)** | CES, CPI, OES, JOLTS | Employment, inflation, wages |
| **BEA (Bureau of Economic Analysis)** | GDP, NIPA, Input-Output | National accounts, trade |
| **CDC / NCHS** | NHANES, BRFSS, WONDER, Mortality | Health, nutrition, mortality |
| **NIH** | dbGaP, GEO, GDC | Genomics, biomedical |
| **NOAA** | GHCN, NDBC, NOMAD | Weather, climate, oceans |
| **NASA** | EOSDIS, NASA Open Data, Exoplanet Archive | Earth science, astronomy |
| **EPA** | Air Quality System, Envirofacts | Environment, pollution |
| **FRED (Federal Reserve)** | Economic data | Finance, economics |
| **BLS** | Occupational Employment Statistics | Wages, employment by occupation |

### 2.2 International organizations

| Organization | Datasets | Topics |
|---|---|---|
| **World Bank** | World Development Indicators | Development, poverty, economics |
| **IMF** | World Economic Outlook, Financial Access | Finance, trade, fiscal |
| **OECD** | OECD Data | Education, health, economy, environment |
| **Eurostat** | EU statistics | European Union |
| **UN Data** | UN Statistics Division | Global, all topics |
| **WHO** | Global Health Observatory | Health, disease, mortality |
| **FAO** | FAOSTAT | Agriculture, food, fisheries |
| **UNESCO** | Education, science, culture | Education, research |

### 2.3 Key US datasets

| Dataset | Agency | Description |
|---|---|---|
| **American Community Survey (ACS)** | Census | Annual US demographics (sampled) |
| **Decennial Census** | Census | Full count every 10 years |
| **NHANES** | CDC | Health and nutrition exams + interviews |
| **BRFSS** | CDC | Behavioral health phone survey |
| **GSS** | NORC / Harvard | General Social Survey (attitudes) |
| **PSID** | Michigan | Panel Study of Income Dynamics |
| **CPS (Current Population Survey)** | BLS / Census | Monthly employment report basis |
| **JOLTS** | BLS | Job openings and labor turnover |
| **CPI** | BLS | Consumer Price Index |
| **GDP (NIPA)** | BEA | Gross Domestic Product |

## 3. Health and medical

| Dataset | Description | Access |
|---|---|---|
| **MIMIC-IV / MIMIC-IV-ED** | ICU data from Beth Israel (100k+ admissions) | Required certification (CITI training) |
| **eICU Collaborative Research Database** | Multi-hospital ICU data (200k+ admissions) | Required certification |
| **NHANES** | National Health and Nutrition Examination Survey | Public |
| **BRFSS** | Behavioral Risk Factor Surveillance System | Public |
| **SEER** | Cancer incidence and survival (NCI) | Public |
| **UK Biobank** | 500k participants, genetic + health data | Application required |
| **dbGaP** | Genotype + phenotype studies | Application required |
| **GEO (Gene Expression Omnibus)** | Gene expression profiles | Public |
| **ICD-10 / ICD-11** | International disease classification | Public (WHO) |
| **RxNorm** | US drug nomenclature | Public (NLM) |
| **Medicare Provider Utilization Data** | CMS claims data | Public (aggregated) |
| **Flatiron Health** | Real-world oncology data | Application required |

## 4. Geospatial

| Dataset | Description | License |
|---|---|---|
| **OpenStreetMap (OSM)** | Crowdsourced world map | Open Database License (ODbL) |
| **Natural Earth** | Public domain map data | Public domain |
| **GADM** | Administrative boundaries | Creative Commons |
| **SEDAC (CIESIN)** | Gridded population data | Custom |
| **OpenStreetMap Extracts** | Full OSM data dumps | ODbL |
| **US Census TIGER** | US boundaries, roads | Public domain |
| **EU Copernicus Open Access** | European geographic data | Custom |
| **ESRI World Imagery** | Satellite imagery | Commercial (free trial) |

### OSM data access

```python
import osmnx as ox

# Download street network for a place
G = ox.graph_from_place("Piedmont, California, USA", network_type="drive")

# Download points of interest
pois = ox.points_from_place("Piedmont, California, USA", tags={"amenity": True})
```

## 5. Climate and Earth science

| Dataset | Source | Description |
|---|---|---|
| **ERA5** | Copernicus / ECMWF | Reanalysis climate data (hourly, 0.25°) |
| **MERRA-2** | NASA | Modern-era climate reanalysis |
| **NOAA GHCN** | NOAA | Global historical climatology network |
| **NOAA NDBC** | NOAA | Marine weather station data |
| **CMIP6** | WGI | Climate model projections |
| **MODIS** | NASA | Satellite land/ocean/atmosphere |
| **Landsat** | USGS / NASA | Satellite imagery (30m resolution) |
| **Sentinel** | ESA | Satellite imagery (10m resolution) |
| **NASA Power** | NASA | Solar, meteorological data |
| **WorldClim** | Research gateways | Global climate surfaces (1 km) |

### ERA5 access with Python

```python
import cdsapi

c = cdsapi.Client()
c.retrieve(
    "reanalysis-era5-single-levels",
    {
        "variable": ["2m_temperature", "surface_solar_radiation_downwards"],
        "year": "2023",
        "month": "01",
        "day": "01",
        "time": "12:00",
        "format": "netcdf",
    },
    "era5.nc"
)
```

## 6. Computer vision

| Dataset | Size | Description | License |
|---|---|---|---|
| **ImageNet** | 14M images (1k classes) | Image classification benchmark | Research |
| **COCO** | 330k images | Object detection, segmentation, captions | Creative Commons |
| **Open Images** | 9M images (19k classes) | Detection, segmentation, relations | Creative Commons |
| **LAION-5B** | 5.8B image-text pairs | Web-scale multimodal | Custom |
| **CIFAR-10/100** | 60k / 600k images | Small-scale classification | Public |
| **MNIST** | 70k digits | Handwritten digit classification | Public |
| **SVHN** | 1M house numbers | Real-world digit recognition | Research |
| **PASCAL VOC** | 10k images | Object detection, segmentation | Public |
| **ADE20K** | 25k images | Scene parsing (150 classes) | Academic |
| **DIV2K** | 800 images | Image super-resolution benchmark | Creative Commons |
| **FFHQ** | 70k faces | High-quality face images | Research |

## 7. Natural language processing

| Dataset | Size | Description | License |
|---|---|---|---|
| **Common Crawl** | 100+ TB web text | Web crawl data | Custom |
| **Wikipedia dumps** | ~60 GB XML | All language editions | CC-BY-SA |
| **Project Gutenberg** | 70k books | Public domain books | Public domain |
| **OSCAR** | 30+ TB text | Cleaned Common Crawl subset | Custom |
| **The Pile** | 825 GB | Diverse text (20 subsets) | Custom |
| **GLUE / SuperGLUE** | Multiple | NLP benchmark suites | Varies |
| **SQuAD** | 300k Q&A | Reading comprehension | Apache 2.0 |
| **CommonVoice** | 800+ hours speech | Voice recognition (Mozilla) | CC-BY |
| **CNN/DailyMail** | 300k articles | Summarization | Research |
| **PubMed Abstracts** | 30M+ | Biomedical abstracts | Public |
| **arXiv** | 2M+ papers | Physics, CS, math, more | arXiv non-exclusive |

### Hugging Face datasets

```python
from datasets import load_dataset

# Load a dataset from Hugging Face
dataset = load_dataset("wikitext", "wikitext-2-raw-v1")
print(dataset["train"][0])

# Stream a large dataset (doesn't load everything into memory)
dataset = load_dataset("c4", "en", streaming=True)
for batch in dataset["train"].batch(10):
    process(batch)
```

## 8. Audio

| Dataset | Size | Description | License |
|---|---|---|---|
| **AudioSet** | 200k+ clips (2s) | Audio events with labels (YouTube) | Creative Commons |
| **LibriSpeech** | 1,000+ hours | English speech (Audiobook) | Custom |
| **VoxCeleb** | 1M+ utterances | Speaker recognition (celebrities) | Academic |
| **ESPS** | 10k+ utterances | Speech enhancement | MIT |
| **FSD50K** | 50k audio clips | Environmental sound classification | Creative Commons |
| **UrbanSound8K** | 8,732 clips | Urban sound classification | Custom |
| **GTZAN** | 1,000 clips | Music genre classification | Public |
| **MuscleSound** | 1,200 clips | Medical auscultation | Creative Commons |
| **ESC-50** | 2,000 clips | Environmental sound classification | Custom |
| **DCASE** | Various | Audio benchmarks for challenges | Varies |

## 9. Domain-specific datasets

### 9.1 Finance

| Dataset | Description | Source |
|---|---|---|
| **CRSP** | Center for Research in Security Prices | Stanford (subscription) |
| **Compustat** | Financial statements of US companies | Standard & Poor's (subscription) |
| **TAQ (Trade and Quote)** | Tick-level US stock data | WRDS (subscription) |
| **FRED** | US economic indicators | Federal Reserve St. Louis |
| **OECD Financial Statistics** | International finance | OECD |
| **Kaggle stock datasets** | Various (AAPL, GOOGL, etc.) | Kaggle |

### 9.2 Social media

| Dataset | Description | Access |
|---|---|---|
| **Twitter / X Academic** | Tweet data (research) | Application required |
| **Reddit (Pushshift archive)** | Reddit posts and comments | Archived (some available) |
| **Stack Exchange** | Q&A across 180+ sites | Monthly dumps |
| **GitHub Archive** | GitHub events (since 2015) | BigQuery (free) |
| **Hacker News** | Posts and comments | BigQuery (official) |
| **Yelp Dataset** | Reviews, businesses, users | Application |
| **Amazon Reviews** | Product reviews (multi-category) | UCI / Kaggle |

### 9.3 Science and research

| Dataset | Description | Source |
|---|---|---|
| **arXiv** | Preprints (physics, CS, math, bio) | arXiv |
| **PubMed** | Biomedical literature | NCBI |
| **Crossref** | DOI metadata for scholarly articles | Crossref |
| **OpenAlex** | Scholarly work, authors, venues | OpenAlex |
| **PDB (Protein Data Bank)** | 3D protein structures | RCSB |
| **ChEMBL** | Bioactive molecules with drug data | EMBL-EBI |
| **KEGG** | Pathway database | KEGG / Tokyo |
| **TCGA** | Cancer genomics (20k+ samples) | NCI / NCBI |

## 10. Licensing and legal considerations

### 10.1 Common licenses

| License | Commercial use | Attribution | Share alike | Changes |
|---|---|---|---|---|
| **CC0** | Yes | No | No | No |
| **CC-BY** | Yes | Yes | No | Yes |
| **CC-BY-SA** | Yes | Yes | Yes | Yes (must share alike) |
| **CC-BY-NC** | No (non-commercial) | Yes | No | Yes |
| **ODbL** | Yes | Yes | Yes (database) | Yes (share alike) |
| **MIT** | Yes | Yes | No | Yes |
| **Apache 2.0** | Yes | Yes | No | Yes |
| **Custom / Research** | Varies | Varies | Varies | Varies |

### 10.2 Key rules

1. **Always read the license.** "Public" does not mean "free to use."
2. **Check for derivative use restrictions.** Some licenses prohibit training ML models.
3. **Attribution requirements.** Many datasets require citing the original authors.
4. **Data use agreements.** Some datasets require signing a DUA before access.
5. **Redistribution restrictions.** Some datasets can be used but not shared with third parties.
6. **Privacy restrictions.** Datasets with personal data may have usage limitations.

### 10.3 Citation best practices

```bibtex
@dataset{author_year_dataset_name,
  author = {Author, A. and Author, B.},
  title = {Dataset Name},
  year = {2024},
  publisher = {Publisher},
  url = {https://doi.org/10.xxxx/xxxxx},
  note = {Version 1.0}
}
```

Always include: authors, title, year, publisher, URL/DOI, and version number.

## 11. Reproducibility

### 11.1 Version your datasets

Datasets change. URLs break. Sources update or disappear.

**Best practices:**

- Download and store a local copy.
- Record the download date and source URL.
- Use versioned datasets (most public datasets have version numbers).
- Use DOIs when available (persistent identifiers).
- Hash the downloaded file (SHA-256) to detect changes.

### 11.2 Data provenance tracking

```python
import hashlib
from datetime import datetime

def record_provenance(url, local_path, name):
    with open(local_path, "rb") as f:
        sha256 = hashlib.sha256(f.read()).hexdigest()
    return {
        "name": name,
        "url": url,
        "downloaded": datetime.utcnow().isoformat(),
        "sha256": sha256,
        "license": "CC-BY-4.0",  # document this
    }
```

### 11.3 Tools for dataset management

| Tool | Purpose |
|---|---|
| **DVC** | Version data files with Git |
| **LakeFS** | Git-like branching for data |
| **Zenodo** | DOI assignment for datasets |
| **Figshare** | Dataset publication and citation |
| **Dryad** | Scientific data repository |
| **pandas** + `hashlib` | Local file hashing |

## 12. References

- Wilkinson, M. et al. *The FAIR Guiding Principles for scientific data management and stewardship*. Scientific Data (2016).
- Borman, G. *Open Data in Education*. Springer (2015).
- Hugging Face Datasets Documentation — https://huggingface.co/docs/datasets/
- Kaggle Datasets — https://www.kaggle.com/datasets
- Google Dataset Search — https://datasetsearch.research.google.com/
- AWS Open Data — https://registry.opendata.aws/
- Data.gov — https://data.gov/
- ICPSR — https://www.icpsr.umich.edu/
