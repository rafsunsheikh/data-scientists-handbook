# Text Data

> **TL;DR** — Text is the most flexible and most cleanup-heavy data type. It can be a single label (short string) or a long document. The same pipeline rarely works for tweets, product reviews, legal contracts, and code. Always characterize *length distribution*, *language*, and *vocabulary* before doing anything else.

## 1. Sub-types

| Sub-type | Length | Example | Typical task |
|---|---|---|---|
| Short label | 1–5 tokens | product names, addresses | normalization, matching |
| Tweet / status | ~10–50 tokens | social posts | sentiment, classification |
| Review / comment | ~50–500 tokens | Yelp, Amazon | sentiment, summarization |
| Document | 500–10000+ tokens | news articles, contracts | classification, NER, summarization |
| Conversation | turn-structured | chat logs, call transcripts | intent, dialogue modeling |
| Code | structured token streams | source files | autocomplete, bug detection |
| Structured text | tagged or templated | XML, HTML, JSON, Markdown | parsing, extraction |

## 2. What to characterize first

Before any modeling or cleaning, measure:

- **Length distribution.** Characters and tokens. Plot the histogram — text length is almost always right-skewed.
- **Language(s).** Use `langdetect` or `fasttext-langid`. Mixed-language data needs separate pipelines.
- **Encoding.** UTF-8 is the default. Older datasets may be Latin-1, Windows-1252, or worse — and look fine until you hit a non-ASCII character.
- **Vocabulary size.** How many unique tokens? Are there rare tokens you should `<UNK>`?
- **Markup / boilerplate.** HTML tags, signatures, quoted replies, ads.
- **Sensitive content.** PII (emails, phones, SSNs), PHI, profanity, hate speech. Decide policy *before* training.

## 3. Tokenization

A "token" is whatever your tokenizer says it is. Different choices give different results:

| Tokenizer | Tokens for "I don't know" | When to use |
|---|---|---|
| Whitespace | `["I", "don't", "know"]` | Quick exploration |
| Word | `["I", "do", "n't", "know"]` | Classical NLP (NLTK, spaCy) |
| BPE / WordPiece | `["I", "don", "'t", "know"]` | LLMs (GPT, BERT) |
| Character | `["I", " ", "d", "o", ...]` | Languages without spaces |

For LLM token counting use `tiktoken` (OpenAI), the model's own tokenizer (Hugging Face), or `anthropic.Anthropic().messages.count_tokens()`.

## 4. Storage and dtype

- pandas: `object` dtype by default; `string` (pyarrow-backed) is faster and properly handles `NA`.
- For columns with thousands of duplicates (e.g., a 5-value enum stored as text), use `category`.
- For very long text, consider storing in a separate file system or column store rather than in-memory.

## 5. Common pitfalls

1. **Bad encoding silently produces mojibake.** `"café"` printed as `"cafÃ©"` means Latin-1 was read as UTF-8 (or vice versa). Fix at the source if possible.
2. **Lowercasing destroys signal in some tasks.** `"US"` vs. `"us"`, `"Apple"` the company vs. `"apple"` the fruit. NER cares; bag-of-words for spam often doesn't.
3. **Stop-word removal destroys signal in others.** "Not good" → "good" reverses sentiment if "not" is stripped.
4. **Stemming over-aggregates.** "universe" and "university" both stem to "univers" with Porter.
5. **Train/test leakage via fitted vectorizers.** Fit `TfidfVectorizer` on train only, transform test. Never fit on the combined data.
6. **Duplicate documents** inflate any metric. Deduplicate (exact + near-dup) before splitting.
7. **PII in training data.** Once a model memorizes a phone number, you cannot make it forget. Redact first.

## 6. Cleaning checklist

See [`data_cleaning/text_cleaning.md`](../data_cleaning/text_cleaning.md) for code. At minimum:

- [ ] Normalize encoding to UTF-8.
- [ ] Apply Unicode normalization (NFC or NFKC).
- [ ] Strip / decode HTML if applicable.
- [ ] Decide on case-folding (depends on task).
- [ ] Decide on punctuation, numbers, emojis (don't blindly strip).
- [ ] Decide on stop-words (don't blindly strip).
- [ ] Detect and isolate non-target languages.
- [ ] Deduplicate (exact + MinHash near-duplicate).
- [ ] Detect and handle PII.

## 7. Representation for downstream tasks

| Representation | Captures | Cost | Use when |
|---|---|---|---|
| Bag of words / TF-IDF | Vocabulary frequencies | Cheap | Spam, topic classification, baselines |
| n-grams | Local order | Higher memory | Authorship, language ID |
| Word embeddings (Word2Vec, GloVe, fastText) | Static semantics | One-time train | Small models, interpretability |
| Contextual embeddings (BERT, RoBERTa, Sentence-BERT) | Context-sensitive semantics | GPU inference | Most modern NLP |
| LLM embeddings (`text-embedding-3`, Voyage, Cohere) | Best-in-class semantics | API calls | RAG, semantic search |
| Token IDs for an LLM | Everything (input to a model) | API or GPU | Generation, completion |

## 8. Visualizations

| Question | Chart |
|---|---|
| Document length distribution | Histogram, ECDF |
| Vocabulary | Word frequency bar; word cloud (with caveats) |
| Topic structure | LDA topic-word table; t-SNE / UMAP of embeddings |
| Semantic similarity | Heatmap of cosine similarity matrix |
| Sentiment over time | Line chart of mean sentiment per period |

See [`data_visualization/text_viz.md`](../data_visualization/text_viz.md).

## 9. References

- Bird, Klein, Loper. *Natural Language Processing with Python* (NLTK book).
- Jurafsky & Martin. *Speech and Language Processing*, 3rd ed. (free draft online).
- Hugging Face *NLP Course* — https://huggingface.co/learn/nlp-course
