# Image Data

> **TL;DR** — Images are tensors of pixel intensities, but the way a single image is *encoded* (format, color space, bit depth, channel order, metadata) determines what's safe to do with it. A 5-line "load image, train model" pipeline hides a dozen decisions, and most production image bugs trace back to those defaults.

## 1. Sub-types

| Sub-type | Description | Example |
|---|---|---|
| Grayscale | Single channel, intensity only | MRI slices, OCR |
| RGB | 3 channels | photographs |
| RGBA | RGB + alpha (transparency) | UI assets, sprites |
| CMYK | Print color space | print pipelines |
| Multispectral | >3 channels (e.g., visible + NIR + SWIR) | satellite imagery |
| Hyperspectral | Dozens to hundreds of bands | remote sensing, chemistry |
| Depth | Per-pixel distance | Kinect, LiDAR, ToF |
| Volumetric | 3D voxel grid | CT, MRI, microscopy |
| Vector | Mathematical primitives, not pixels | SVG, CAD |
| Raster scientific | Geo-referenced grids | GeoTIFF (also: see [geospatial](geospatial.md)) |
| Medical (DICOM) | Pixel data + extensive metadata | radiology |

## 2. Storage formats

| Format | Lossy? | Bit depth | Notes |
|---|---|---|---|
| JPEG | Yes | 8-bit | Ubiquitous; never re-encode JPEGs repeatedly |
| PNG | No | 8 or 16-bit | Good for screenshots, UI, masks |
| WebP / AVIF / HEIF | Configurable | 8–12-bit | Modern web; better compression |
| TIFF | Optional | 8/16/32-bit | Scientific, multi-page, multi-channel |
| GeoTIFF | Optional | varies | TIFF + geo metadata |
| DICOM | Optional | 12/16-bit | Medical; carries patient metadata (PHI!) |
| RAW (CR2, NEF, ARW) | No | 12–14-bit | Camera sensor data before processing |
| HDF5 / Zarr | No | Any | Large multi-dimensional arrays |
| NPY / NPZ | No | Any | NumPy native |

## 3. What to characterize first

- **Shape and dtype.** Is it `H × W × C` or `C × H × W`? `uint8`, `uint16`, or `float32`? Models expect different things.
- **Channel order.** OpenCV reads as BGR. PIL and most others use RGB. Many production bugs come from this.
- **Value range.** `[0, 255]`, `[0, 1]`, `[-1, 1]`, or arbitrary float (medical)? Normalize consistently.
- **Color space.** sRGB is default for photos. Linear RGB for compositing. Lab/HSV for some classical CV.
- **EXIF orientation.** Phones often store the image rotated and rely on EXIF to display upright. Forget to apply it and your training data is sideways.
- **Aspect ratios and resolutions.** Highly variable in the wild; need a consistent resize strategy.
- **Class balance** (for labeled data) — usually very imbalanced.

## 4. Common pitfalls

1. **Mixing BGR and RGB** silently swaps red and blue channels, hurting accuracy for color-sensitive tasks.
2. **Resizing changes aspect ratio.** Squashing portraits into squares distorts faces. Use letterbox / center-crop instead.
3. **JPEG re-encoding.** Loading a JPEG and saving as JPEG loses information every time. Save as PNG/TIFF intermediates.
4. **Mismatched normalization.** Training with ImageNet mean/std but inference with raw `[0, 255]` → model output is nonsense.
5. **PHI in DICOM headers.** DICOM files carry patient name, ID, birth date by default. Strip before sharing.
6. **EXIF metadata leaks location.** Phone photos contain GPS coordinates.
7. **Class imbalance** that wasn't sampled-aware during training.
8. **Train-test leakage via near-duplicates.** Same scene, different crops, in both sets.
9. **Watermarks, logos, and timestamps** that the model learns to predict instead of the actual class.

## 5. Cleaning checklist

See [`data_cleaning/image_cleaning.md`](../data_cleaning/image_cleaning.md).

- [ ] Verify dimensions and dtypes are uniform (or handle variability explicitly).
- [ ] Normalize color space and channel order.
- [ ] Apply EXIF rotation.
- [ ] Remove corrupted files (try-decode pass).
- [ ] Detect duplicates and near-duplicates (perceptual hash).
- [ ] Strip metadata before publishing.
- [ ] Audit labels — most datasets are surprisingly mislabeled.

## 6. Representations for downstream tasks

| Representation | Use |
|---|---|
| Raw pixels | CNN / ViT input |
| HOG, SIFT, ORB | Classical CV, low-resource matching |
| CNN feature maps | Transfer learning |
| Image embeddings (CLIP, DINO, SAM features) | Search, clustering, few-shot, zero-shot |
| Saliency / attention maps | Explainability |
| Segmentation masks | Per-pixel labels |
| Bounding boxes / keypoints | Detection / pose |

## 7. Visualizations

| Question | Chart |
|---|---|
| What do random samples look like? | Image grid |
| Class distribution | Bar chart |
| Are predictions correct? | Confusion matrix + sample grid per cell |
| Where is the model looking? | Grad-CAM, integrated gradients overlay |
| Are there near-duplicates? | t-SNE / UMAP of embeddings |
| What's the resolution distribution? | Scatter of width vs. height |

See [`data_visualization/image_viz.md`](../data_visualization/image_viz.md).

## 8. References

- Szeliski. *Computer Vision: Algorithms and Applications*, 2nd ed.
- Goodfellow, Bengio, Courville. *Deep Learning*, Chap. 9.
- Hugging Face *Computer Vision Course*.
