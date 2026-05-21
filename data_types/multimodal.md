# Multimodal Data

> **TL;DR** — Multimodal data combines two or more of the previous types — image+text, audio+text, video (image+audio+text), sensor+text+time-series. The hard part isn't any single modality, it's *alignment*: making sure the right caption goes with the right image, the right transcript with the right audio segment, the right sensor reading with the right event.

## 1. Common combinations

| Combination | Example | Typical task |
|---|---|---|
| Image + text | Product page (photo + description) | retrieval, classification |
| Image + audio + text | Video | action recognition, captioning |
| Audio + text | Podcast + transcript | ASR, diarization |
| Time series + categorical + text | Customer events with comments | next-action prediction, churn |
| Tabular + image | Real estate listing + photos | price prediction |
| Sensor + GPS + time | Connected vehicle | anomaly detection |
| Text + graph | Knowledge graph with descriptions | QA, reasoning |

## 2. Alignment

The dominant question is: *for a given record, which pieces from each modality belong together?*

- **Strict 1:1** — e.g., image-caption pairs. Either dedup or keep all valid pairs.
- **1:N** — one image, many tags. Decide aggregation.
- **N:N** — many images, many captions for one product. Model carefully.
- **Time-aligned** — video frames, audio samples, subtitles all on the same clock. Tiny clock drift accumulates over long recordings.
- **Event-aligned** — sensor reading nearest to a user-clicked timestamp. Use as-of joins.

## 3. Representations

### 3.1 Independent encoders (late fusion)
Encode each modality separately, concatenate, then learn a joint head.

### 3.2 Joint embedding space (CLIP-style)
Train so that paired image and text land near each other in a shared vector space.
- Enables zero-shot classification, cross-modal search.
- Modern variants: SigLIP, EVA-CLIP, ALIGN.

### 3.3 Fully fused model
Single transformer processes all modalities tokenized into a common stream.
- Examples: Flamingo, GPT-4 Vision, Claude 3+, Gemini.

### 3.4 Token-interleaved
Treat everything as tokens (text tokens, image patch tokens, audio frames). Increasingly standard in modern LMMs.

## 4. Common pitfalls

1. **Alignment bugs are silent.** A pipeline mistake that pairs the wrong caption with each image makes the model unable to learn — and might be discovered only at evaluation.
2. **Modality dropout in production.** Train assumes both modalities are present; production has missing thumbnails. Train with random modality masking.
3. **Modality leakage.** A text label inadvertently contains the image filename → trivial accuracy that vanishes in deployment.
4. **Scale mismatch.** Image features and text features on wildly different scales without normalization → one dominates.
5. **Computational asymmetry.** Image and audio encoders are expensive; text is cheap. Batch shapes can OOM unexpectedly.
6. **License heterogeneity.** Different modalities often have different sources with different terms.

## 5. Cleaning checklist

- [ ] Verify every record has the required modalities (or explicitly handle the missing case).
- [ ] Validate cross-modal alignment with a sample inspection.
- [ ] Normalize each modality per its own pipeline (see respective chapters).
- [ ] Detect and remove cross-modal duplicates (same image, different caption).
- [ ] Audit for label / metadata leakage across modalities.

## 6. Evaluation

Multimodal evaluation is harder than single-modality:

- For retrieval: Recall@k, MRR, NDCG.
- For generation (captioning, VQA): BLEU, METEOR, ROUGE, CIDEr, **plus** human eval — automatic metrics correlate weakly with quality.
- For grounding: IoU between predicted and gold regions.
- Always include modality-ablation: how does the model do with only one modality? If text-only matches multimodal, your image encoder is contributing nothing.

## 7. Visualizations

| Question | Chart |
|---|---|
| Joint embedding structure | t-SNE / UMAP of paired image+text embeddings |
| Retrieval quality | Side-by-side grids (query + top-k results) |
| Attention across modalities | Cross-attention heatmaps |
| Modality contribution | Ablation bar chart |

## 8. References

- Radford et al. (2021). *Learning Transferable Visual Models From Natural Language Supervision* (CLIP).
- Baltrušaitis, Ahuja, Morency (2018). *Multimodal Machine Learning: A Survey and Taxonomy.*
- Hugging Face Multimodal documentation.
