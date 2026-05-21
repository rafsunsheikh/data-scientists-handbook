# Files as Data Sources

> **Stub — contributions welcome.**

## What to cover

- CSV / TSV: encodings, delimiters, quoting, header detection.
- JSON / JSONL: streaming parse with `ijson` or `orjson`; nested-to-flat with `pd.json_normalize`.
- Parquet: columnar, predicate pushdown, partition pruning.
- Avro: schema-evolution-friendly streaming format.
- ORC: similar to Parquet in Hadoop ecosystem.
- Excel: `.xls` vs. `.xlsx`, multiple sheets, formulas, merged cells.
- PDF: text vs. tables (`pdfplumber`, `pymupdf`, `tabula-py`, `camelot`); OCR via `tesseract`.
- Archive formats: zip, tar, gzip, bzip2, zstd; streaming reads.
- File systems: local, NFS, S3, GCS, Azure Blob, HDFS — uniform access via `fsspec`.
- Reproducibility: hashing, manifest files, versioned datasets (DVC, LakeFS).
