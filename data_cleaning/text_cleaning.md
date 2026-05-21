# Text Cleaning

> **TL;DR** — Modern NLP models (BERT, LLMs) are surprisingly robust to "raw" text, so the aggressive cleaning pipelines that were standard for bag-of-words models are often counterproductive today. Clean for **encoding** and **deduplication** aggressively; clean for **lowercasing, punctuation, stopwords, stemming** only when the downstream model demands it.

## 1. Always do these

### 1.1 Fix encoding
```python
text = raw_bytes.decode('utf-8', errors='replace')
```
If you see characters like `Ã©`, `â€™`, `Ã`, that's UTF-8 being read as Latin-1 (or vice versa). Tools: `ftfy` (literally named "fixes text for you").

```python
import ftfy
clean = ftfy.fix_text(broken)
```

### 1.2 Unicode normalization
NFC for storage, NFKC if you want to collapse compatibility forms (e.g., ligatures `ﬁ` → `fi`).

```python
import unicodedata
text = unicodedata.normalize('NFC', text)
```

### 1.3 Strip control characters
```python
import re
text = re.sub(r'[\x00-\x08\x0b\x0c\x0e-\x1f\x7f]', '', text)
```

### 1.4 Deduplicate
Exact and near-duplicate (see [`duplicates.md`](duplicates.md)). For corpora, MinHash + LSH (`datasketch`).

## 2. Do these when appropriate

### 2.1 HTML / markup removal
For scraped data:

```python
from bs4 import BeautifulSoup
text = BeautifulSoup(html, 'lxml').get_text(separator=' ', strip=True)
```

### 2.2 URL / email / mention normalization
Replace with placeholders (`<URL>`, `<EMAIL>`, `<USER>`) when they carry no signal for your task:

```python
text = re.sub(r'https?://\S+', '<URL>', text)
text = re.sub(r'\S+@\S+', '<EMAIL>', text)
text = re.sub(r'@\w+', '<USER>', text)
```

### 2.3 PII redaction
Use a NER model (spaCy, Presidio) for names, addresses, phone numbers. Regex catches the easy cases; ML catches the rest.

### 2.4 Whitespace
```python
text = re.sub(r'\s+', ' ', text).strip()
```

### 2.5 Case folding
Only if you're using a case-insensitive model (older bag-of-words). Modern transformers usually handle case.

## 3. Do these only for classical NLP

### 3.1 Punctuation stripping
Destroys signal in some tasks (sentiment, syntax). Skip for transformer models.

### 3.2 Stop-word removal
Helpful for topic modeling and keyword extraction; harmful for sentiment (`"not"`) and most modern models.

### 3.3 Stemming / lemmatization
Stemming is aggressive and lossy. Lemmatization is gentler. Both have largely been replaced by subword tokenization.

## 4. Language-specific concerns

- **Chinese / Japanese / Thai** — no whitespace word boundaries. Use jieba (zh), MeCab/SudachiPy (ja), pythainlp (th).
- **Arabic / Hebrew** — RTL, diacritics, multiple forms of the same letter. Normalize with `pyarabic`.
- **Code-mixed text** (e.g., Hinglish) — most language-ID and tokenizers underperform; consider task-specific models.
- **Emoji** — meaningful in social text. Either keep, convert to descriptions (`":smile:"` → `"smiling face"`), or strip — never silently.

## 5. Tokenization-safe cleaning

If you'll feed the text to a transformer:

- Keep the original casing if the tokenizer was trained cased.
- Keep punctuation — the tokenizer uses it.
- Keep emoji and special characters — modern tokenizers handle them.
- Truncate very long documents at token boundaries, not character boundaries (use the model's tokenizer).

## 6. Quality filters (for LLM training data)

When building a training corpus, additionally filter:

- Documents below a length threshold (often <50 words).
- Documents with low unique-word ratio (templated junk).
- Documents with very high punctuation / digit ratio (logs, tables-as-text).
- Documents flagged by perplexity from a small reference model.

See Penedo et al. (2023), *The RefinedWeb Dataset for Falcon LLM* for one well-documented pipeline.

## 7. Common pitfalls

1. **Cleaning before deduplication.** You can lose the exact-match signal.
2. **Lowercasing for NER.** Destroys case-based proper-noun cues.
3. **Stripping URLs from a phishing classifier.** The URL was the feature.
4. **Removing emoji from a sentiment classifier on tweets.** Emoji often *is* the sentiment.
5. **Applying a cleaning pipeline written for English to other languages.**
6. **Forgetting the same pipeline at inference time.**

## 8. Companion notebook

[`../notebooks/cleaning/04_text_cleaning.ipynb`](../notebooks/cleaning/04_text_cleaning.ipynb)
