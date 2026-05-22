# Files as Data Sources

> **TL;DR** — Files are the universal data interchange format, but the format you choose (CSV, Parquet, JSON, etc.) determines how much data you can store, how fast you can read it, whether you can filter on the server side, and how well schema changes are handled. The single most impactful decision for a data scientist working with files is: use Parquet for analytical workloads, JSON/JSONL for nested/hierarchical data, CSV only for small simple exports, and always be aware of encoding, partitioning, and compression. At scale, files live on cloud object storage (S3, GCS, Azure Blob) and are accessed via unified interfaces like `fsspec`.

## 1. File formats

### 1.1 CSV / TSV

**What it is:** Comma-separated (or tab-separated) values, one row per line, optional header row.

**Parameters:**

- **Delimiter:** Comma (CSV), tab (TSV), pipe (`|`), semicolon (`;`). Semicolons common in European locales where comma is a decimal separator.
- **Encoding:** UTF-8 (preferred), Latin-1, Windows-1252, GBK. Wrong encoding → garbled text or silent data corruption (e.g., `â€™` instead of `'`).
- **Quoting:** Fields containing the delimiter or newlines are quoted (typically with `"`). Escaped quotes are doubled (`""`).
- **Header:** Optional. Some CSVs have no header — you need a separate schema file.
- **Line endings:** `\r\n` (Windows), `\n` (Unix), `\r` (old Mac).parsers should handle all.
- **Trailing newline:** Some CSVs end with a blank line → extra row of NaNs.

**Performance:**

- Read speed: Slow (full row parse, no column pruning).
- Write speed: Slow (no compression, string encoding on write).
- File size: Largest of common formats (no compression, no columnar storage).

**When to use:**

- Small data exchanges (< 100 MB).
- Human-readable data sharing.
- Input for tools that only accept CSV.

**When NOT to use:**

- Large datasets (slow to read, no type info).
- Data with nested structures.
- Repeated read/write cycles (type conversion loss).

**Python:**

```python
import pandas as pd
df = pd.read_csv("data.csv", encoding="utf-8", sep=",",
                 dtype={"id": str}, parse_dates=["created_at"])
# Common pitfalls:
# - dtype="object" for everything if not specified
# - numbers parsed as strings if they contain commas
# - dates parsed as strings unless format is specified
```

### 1.2 JSON / JSONL

**What it is:** JavaScript Object Notation — text-based, hierarchical, key-value pairs, arrays. JSONL (JSON Lines) is one JSON object per line.

**Parameters:**

- **Structure:** Objects (`{}`), arrays (`[]`), strings, numbers, booleans, `null`.
- **Nesting:** Arbitrary depth. Arrays of objects, objects within arrays, etc.
- **JSONL vs. JSON:** JSONL is line-delimited, streamable, appendable. Regular JSON is a single document — loading the whole file into memory.
- **Streaming:** `ijson` or `orjson` for streaming parse of large JSON files without loading everything into memory.
- **Encoding:** UTF-8 only.

**Performance:**

- Read speed: Moderate (text parsing, no column pruning).
- Write speed: Moderate (text serialization).
- File size: Larger than Parquet (no compression by default, redundant key names).
- JSONL read speed: Good (line-by-line, parallelizable).

**When to use:**

- Nested / hierarchical data (user profiles, API responses, logs).
- Event streams (one event per line → JSONL).
- Schema-less data (fields vary per record).

**When NOT to use:**

- Numerical analytics (no columnar storage, no type info).
- Data with many repeated string values (no dictionary encoding).
- Very large datasets where you only need a subset of fields (can't do predicate pushdown).

**Python:**

```python
import json
import pandas as pd

# Nested JSON → flat DataFrame
with open("data.json") as f:
    data = json.load(f)
df = pd.json_normalize(data, sep="_")

# JSONL — line by line
import ijson
with open("large.json", "rb") as f:
    parser = ijson.items(f, "item")
    for item in parser:
        process(item)

# JSONL → pandas (fast)
df = pd.read_json("data.jsonl", lines=True)
```

### 1.3 Parquet

**What it is:** Columnar storage format with schema, compression, and encoding — designed for analytical workloads.

**Parameters:**

- **Columnar:** Each column stored contiguously. Reading 3 columns from a 100-column file reads only those 3 columns.
- **Schema:** Embedded in file footer. Types preserved (int64, string, timestamp, nested structs, arrays).
- **Compression:** Per-column. Snappy (default, fast), GZIP (better ratio), ZSTD (best ratio), LZ4.
- **Encoding:** Dictionary encoding (replace repeated values with IDs), RLE (run-length), delta encoding (numbers, dates).
- **Predicate pushdown:** Filters applied at read time — only matching rows decoded.
- **Partition pruning:** File paths encode partition columns (`year=2024/month=01/`). Only read matching directories.
- **Row groups:** Data partitioned into row groups (typically 128 MB). Each group has its own min/max statistics per column — enables skipping entire groups.

**Performance:**

- Read speed: Very fast for selective queries (column pruning + predicate pushdown + row group skipping).
- Write speed: Moderate (needs to compress).
- File size: Typically 5–10x smaller than CSV.
- Random access: Fast for column reads; slow for row reads (need to decompress all columns).

**When to use:**

- Analytical workloads (most data science).
- Large datasets (> 1 GB).
- Repeated reads (model training, exploration).
- Data sharing between systems (schema is embedded).

**When NOT to use:**

- Small files (< 10 MB) — overhead dominates.
- Streaming / append-only logs (use Avro or JSONL).
- Random row access (use a database).

**Python:**

```python
import pandas as pd
import pyarrow.parquet as pq

# Read — column pruning + filtering
df = pd.read_parquet("data.parquet", columns=["col1", "col2"])  # only read 2 columns
df = pd.read_parquet("data.parquet", filters=[("status", "==", "active")])  # predicate pushdown

# Read partitioned dataset
df = pd.read_parquet("data/partitioned/", filters=[("year", "==", 2024)])

# Write
df.to_parquet("output.parquet", engine="pyarrow",
              compression="zstd", row_group_size=1_000_000)

# Schema preservation
schema = pq.read_schema("data.parquet")
print(schema)  # preserves types, nested types, metadata
```

### 1.4 Avro

**What it is:** Row-based serialization format with schema, designed for streaming and Hadoop.

**Parameters:**

- **Row-based:** Unlike Parquet, Avro stores rows sequentially. Good for writes, bad for columnar reads.
- **Schema:** Separate JSON schema file (or embedded). Schema evolution supported (add/remove fields, type promotion).
- **Compression:** Block-level (Deflate, Snappy).
- **Serialization:** Binary — compact, no redundant field names (schema defines them once).
- **Schema evolution:**
  - **Writer schema:** Schema used when writing.
  - **Reader schema:** Schema used when reading.
  - Reader can handle added/removed fields with defaults.

**Performance:**

- Read speed: Moderate (row-based, but binary is compact).
- Write speed: Fast (binary, no complex encoding).
- File size: Small (binary, no redundant names).

**When to use:**

- Streaming data pipelines (Kafka messages often use Avro with Schema Registry).
- Schema evolution over time (fields added/removed).
- Hadoop ecosystem integration.

**When NOT to use:**

- Analytical queries on subsets of columns (use Parquet).
- Ad-hoc exploration (no built-in tooling like `pq.read_schema()`).

**Python:**

```python
import fastavro

# Write
schema = {
    "name": "User",
    "type": "record",
    "fields": [
        {"name": "id", "type": "int"},
        {"name": "name", "type": "string"},
    ]
}
with open("users.avro", "wb") as f:
    writer = fastavro.writer(f, fastavro.parse_schema(schema),
                              [{"id": 1, "name": "Alice"}])

# Read
with open("users.avro", "rb") as f:
    reader = fastavro.reader(f)
    for record in reader:
        print(record)
```

### 1.5 ORC

**What it is:** Optimized Row Columnar format — Parquet's main competitor in the Hadoop ecosystem.

**Parameters:**

- **Hybrid:** Columnar at block level, row-based within stripes for row access.
- **Compression:** Stripes have separate compression per column.
- **Index:** Per-stripes min/max/sum statistics.
- **Predicate pushdown:** Supported.
- **Hadoop-native:** Deep integration with Hive, Spark, Presto.

**When to use:**

- Hadoop / Hive environments.
- When you need both columnar reads and row access.

**When NOT to use:**

- Non-Hadoop ecosystems (Parquet has wider support).

**Python:**

```python
# Read ORC with pandas (requires pyarrow ORC support)
df = pd.read_orc("data.orc", columns=["col1", "col2"],
                 filters=[("status", "==", "active")])
```

### 1.6 Excel

**What it is:** Microsoft Excel's proprietary format (.xls = legacy BIFF, .xlsx = Office Open XML, a ZIP of XML files).

**Parameters:**

- **Sheets:** Workbooks contain multiple sheets. Which sheet to read must be specified.
- **Formulas:** Cells can contain formulas. Reading Excel with `pandas` evaluates formulas (returns computed values, not formulas).
- **Merged cells:** Common in reports. Breaks `pandas` parsing — merged cells produce NaN in non-top-left cells.
- **Data types:** Excel is typeless — a column of "2024-01-01" may be parsed as a date or string depending on locale and cell formatting.
- **Limits:** .xls: 65,536 rows × 256 columns. .xlsx: 1,048,576 rows × 16,384 columns.
- **Hidden rows / filters:** Data may be filtered or hidden in the UI but present in the file.

**Performance:**

- Read speed: Slow (XML parsing, formula evaluation).
- File size: Moderate (.xlsx is ZIP of XML, so compressed).
- Memory: Loading large Excel files into `pandas` can OOM.

**When to use:**

- Small data exchanges with stakeholders (they expect Excel).
- Reports with formatting / charts (not machine-readable).

**When NOT to use:**

- Large datasets (> 100k rows).
- Automated pipelines (fragile, format-specific).
- Data with formulas (you get the computed value, not the logic).

**Python:**

```python
import pandas as pd

# Read specific sheet, skip merged-cell issues
df = pd.read_excel("report.xlsx", sheet_name="Data",
                   engine="openpyxl", skiprows=3)

# Read all sheets
xl = pd.ExcelFile("report.xlsx", engine="openpyxl")
sheets = {name: xl.parse(name) for name in xl.sheet_names}
```

### 1.7 PDF

**What it is:** Portable Document Format — fixed-layout document format. Not designed for data storage.

**Parameters:**

- **Text-based PDFs:** Searchable text can be extracted.
- **Image-based PDFs:** Scanned documents — require OCR (Tesseract, AWS Textract, Google Vision).
- **Tables in PDFs:** No standard table structure. Tables are positioned text boxes. Extraction is heuristic.
- **Layout:** Columns, footnotes, multi-page tables, merged cells — all break naive extraction.

**Tools:**

| Tool | Best for |
|---|---|
| `pdfplumber` | Tables with clear borders, text extraction |
| `pymupdf` (fitz) | Fast extraction, OCR support |
| `tabula-py` | Simple table extraction (Java-based) |
| `camelot` | Tables in PDFs (grid and stream mode) |
| `PyPDF2` / `pypdf` | Text extraction, page manipulation |
| `tesseract` | OCR for scanned documents |
| `AWS Textract` | Cloud OCR + table extraction |

**Python:**

```python
import pdfplumber

with pdfplumber.open("report.pdf") as pdf:
    for page in pdf.pages:
        tables = page.extract_tables()
        for table in tables:
            df = pd.DataFrame(table[1:], columns=table[0])  # skip header row
```

**Pitfalls:**

- PDFs are not data formats. Every PDF table extraction is a one-off.
- Layout changes break extraction scripts.
- Always store the extracted data separately and version it.

### 1.8 Image and audio files

See [`../data_types/image.md`](../data_types/image.md) and [`../data_types/audio.md`](../data_types/audio.md) for format details.

| Format | Type | Notes |
|---|---|---|
| PNG | Image | Lossless, supports transparency |
| JPEG | Image | Lossy, most common photo format |
| TIFF | Image | Lossless, scientific / medical imaging |
| WebP | Image | Modern, better compression than JPEG |
| RAW (CR2, NEF, ARW) | Image | Camera raw, needs `rawpy` or `dcraw` |
| WAV | Audio | Uncompressed PCM |
| FLAC | Audio | Lossless compression |
| MP3 | Audio | Lossy, most common |
| OGG / OPUS | Audio | Lossy, low latency, web |

### 1.9 Archive formats

| Format | Extension | Compression | Notes |
|---|---|---|---|
| **gzip** | `.gz` | DEFLATE | Single file; `zcat`, `gzip -d` |
| **bzip2** | `.bz2` | BWT | Better ratio than gzip, slower |
| **xz** | `.xz` | LZMA | Best ratio, slowest |
| **zstd** | `.zst` | Zstandard | Facebook; great speed/ratio tradeoff |
| **zip** | `.zip` | Various | Multiple files; no streaming |
| **tar** | `.tar` | None | Archiving (no compression); combine with gzip/bzip2/xz/zstd |
| **tar.gz / .tgz** | `.tar.gz` | DEFLATE | Most common Unix archive |

**Python:**

```python
import gzip
import zstandard as zstd
import tarfile

# gzip
with gzip.open("data.csv.gz", "rt", encoding="utf-8") as f:
    for line in f:
        process(line)

# zstd
cctx = zstd.ZstdCompressor()
dctx = zstd.ZstdDecompressor()
with open("data.csv.zst", "rb") as f:
    data = dctx.decompress(f.read())

# tar
with tarfile.open("archive.tar.gz", "r:gz") as tar:
    names = tar.getnames()
```

## 2. File systems and cloud storage

### 2.1 Local and network file systems

- **Local disk:** Fast I/O, limited capacity, single-machine.
- **NFS / SMB:** Network-mounted; slower than local, shared across machines.
- **HDFS (Hadoop Distributed File System):** Distributed file system for Hadoop ecosystem.

### 2.2 Cloud object storage

| Provider | Service | Protocol | Python SDK |
|---|---|---|---|
| **AWS** | S3 | HTTP/REST | `boto3`, `s3fs` |
| **Google Cloud** | GCS | HTTP/REST | `google-cloud-storage`, `gcsfs` |
| **Azure** | Blob Storage | HTTP/REST | `azure-storage-blob`, `adlfs` |
| **Cloudflare** | R2 | S3-compatible | `boto3` (S3 endpoint override) |

**Characteristics:**

- **Object storage:** Files (objects) with metadata, no directories (flat namespace with "/" in keys for hierarchy).
- **Consistency:** Eventually consistent for overwrites (S3 now offers strong consistency for PUTS).
- **Durability:** 99.999999999% (11 nines) for standard tier.
- **Cost:** Per-GB stored + per-request + egress. Frequent reads/writes get expensive.
- **Latency:** Higher than local disk (network round-trip).

### 2.3 Unified file system access: `fsspec`

`fsspec` provides a unified interface to local, S3, GCS, Azure, HDFS, and more.

```python
import fsspec
import pandas as pd

# Same API for local and cloud paths
fs = fsspec.filesystem("s3")
with fs.open("my-bucket/data/file.parquet", "rb") as f:
    df = pd.read_parquet(f)

# Works with all backends
paths = fsspec.glob_from_path("s3://my-bucket/data/*.parquet")
```

**Common filesystems:**

| Protocol | Backend |
|---|---|
| `file://` | Local |
| `s3://` | AWS S3 (`s3fs`) |
| `gs://` | Google Cloud Storage (`gcsfs`) |
| `az://` | Azure Blob (`adlfs`) |
| `hdfs://` | Hadoop (`pyarrow.HadoopFileSystem`) |
| `http://` / `https://` | HTTP (read-only) |

## 3. Reproducibility and data versioning

### 3.1 File hashing

```python
import hashlib

def file_hash(path, algorithm="sha256"):
    h = hashlib.new(algorithm)
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            h.update(chunk)
    return h.hexdigest()
```

Track file hashes in a manifest to detect changes.

### 3.2 Data versioning tools

| Tool | Approach | Notes |
|---|---|---|
| **DVC (Data Version Control)** | Git-annalogy for data files | Tracks hashes, stores in cloud, integrates with Git |
| **LakeFS** | Git-like branching for data lakes | Branch, merge, commit data in S3/GCS/Azure |
| **Delta Lake** | Transaction log on Parquet | ACID transactions, time travel on data lakes |
| **Apache Iceberg** | Table format with transaction log | Hidden partitioning, schema evolution, time travel |
| **Hudi** | Hoodie updateable dataset | Upserts, time travel, incremental reads |
| **Pachyderm** | Git-like pipeline versioning | Version data + code + pipelines together |

### 3.3 Manifest files

For simple projects, maintain a `manifest.json`:

```json
{
    "files": [
        {"path": "data/raw/customers.csv", "sha256": "abc123...", "added": "2024-01-15"},
        {"path": "data/processed/customers.parquet", "sha256": "def456...", "added": "2024-01-16"}
    ]
}
```

## 4. Common pitfalls for data scientists

### 4.1 Encoding mismatches

CSV with Latin-1 read as UTF-8 → garbled characters. Always detect encoding (chardet) or specify explicitly.

### 4.2 Silent type coercion

`pandas` infers types from a sample of rows. A column of numbers with a single non-numeric value becomes `object` (string). Always specify `dtype`.

### 4.3 Trailing newlines in CSV

Creates an extra row of NaNs. Use `skip_blank_lines=True` (default) and drop rows where all columns are NaN.

### 4.4 Excel merged cells

Merged cells in reports produce NaN in non-top-left cells. Manually forward-fill or use `merge_cells=False` and fill.

### 4.5 Loading everything into memory

`pd.read_csv("huge.csv")` on a 100 GB file will OOM. Use:
- `chunksize` parameter for iterative reading.
- `polars` or `duckdb` for out-of-core processing.
- Parquet with column pruning and filtering.

### 4.6 Stale file caches

Reading a file that was updated on disk but cached by the OS or Python. Use fresh file handles or invalidate caches.

### 4.7 Partition explosion

Writing Parquet files with too many partitions (e.g., one per day per region per SKU) creates thousands of tiny files — slow to read. Aim for 128 MB – 1 GB per file.

## 5. Format comparison summary

| Format | Columnar | Schema | Compression | Nested | Streaming | Best for |
|---|---|---|---|---|---|---|
| **CSV** | No | No | No | No | Yes | Small exchanges |
| **JSON** | No | No | No | Yes | No | Nested data |
| **JSONL** | No | No | No | Yes | Yes | Event streams |
| **Parquet** | Yes | Yes | Yes | Yes | No | Analytical queries |
| **Avro** | No | Yes | Yes | Yes | Yes | Streaming pipelines |
| **ORC** | Yes | Yes | Yes | Yes | No | Hadoop / Hive |
| **Excel** | No | No | Yes | No | No | Stakeholder reports |
| **Parquet + ZSTD** | Yes | Yes | Yes | Yes | No | Default choice |

## 6. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas` | General-purpose tabular I/O |
| `polars` | Fast, lazy, out-of-core tabular I/O |
| `pyarrow` | Parquet, Avro, ORC, IPC |
| `duckdb` | SQL queries on Parquet/CSV/JSON files |
| `ijson` | Streaming JSON parse |
| `orjson` | Fastest JSON serialization |
| `fastavro` | Avro read/write |
| `fsspec` | Unified file system interface |
| `s3fs`, `gcsfs`, `adlfs` | Cloud storage backends |
| `dvc` | Data version control |
| `pdfplumber`, `pymupdf` | PDF table extraction |
| `openpyxl` | Excel .xlsx read/write |
| `chardet` | Encoding detection |
| `tabulate` | Pretty-print tables |

## 7. References

- Apache Parquet Documentation — https://parquet.apache.org/docs/
- Apache Avro Documentation — https://avro.apache.org/docs/
- Google BigQuery Storage Read/Write APIs — https://cloud.google.com/bigquery/docs/storage-api
- Kleppmann, M. *Designing Data-Intensive Applications*. O'Reilly (2017). (Ch. 2: Data Models and Languages, Ch. 3: Encoding and Evolution)
- pandas Documentation — https://pandas.pydata.org/docs/user_guide/io.html
- `fsspec` Documentation — https://filesystem-spec.readthedocs.io/
