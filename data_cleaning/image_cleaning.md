# Image Cleaning

> **TL;DR** — Image "cleaning" is mostly making your image dataset *consistent*: same orientation, same color space, same value range, no corrupt files, no duplicates, no metadata leakage. Augmentation is a separate concern that happens *after* cleaning.

## 1. Verify and reject corrupt files

```python
from PIL import Image
def is_openable(path):
    try:
        with Image.open(path) as im:
            im.verify()
        return True
    except Exception:
        return False
```

Truncated JPEGs and corrupt PNGs cause silent training failures (random tensors fed to the model). Reject up front.

## 2. Apply EXIF orientation

Phones often store images rotated; they rely on the EXIF `Orientation` tag for upright display. Forget to apply it and a quarter of your training data is sideways.

```python
from PIL import Image, ImageOps
im = ImageOps.exif_transpose(Image.open(path))
```

## 3. Standardize color space and channels

```python
im = im.convert('RGB')        # drop alpha, force 3 channels
```

If your pipeline uses OpenCV (BGR), be deliberate about when you swap.

## 4. Resize policy

| Strategy | When |
|---|---|
| Resize to fixed `(H, W)` | Quick; distorts aspect ratio |
| Resize shorter side + center crop | Standard for classification |
| Resize + letterbox (pad to square) | Detection — preserves aspect for boxes |
| Multi-scale | Detection at multiple sizes |
| Variable size with bucketing | Memory-efficient training |

```python
from torchvision import transforms
t = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
])
```

## 5. Normalization

Decide **once** and apply consistently in train and inference. Most ImageNet-pretrained models expect:

```
mean = [0.485, 0.456, 0.406]
std  = [0.229, 0.224, 0.225]
```

For medical / scientific images, compute mean/std from your own training set (not ImageNet).

## 6. Deduplicate

Exact: hash file bytes (MD5/SHA1).
Near: perceptual hash.

```python
import imagehash
phash = imagehash.phash(Image.open(path))   # 64-bit hash
# Group by phash; Hamming distance < 5 ≈ near-duplicate
```

Near-duplicates between train and test are an instant accuracy inflation.

## 7. Strip sensitive metadata

EXIF can contain GPS coordinates and device IDs. Before publishing or even just sharing internally:

```python
from PIL import Image
im = Image.open(path)
data = list(im.getdata())
clean = Image.new(im.mode, im.size)
clean.putdata(data)
clean.save(out_path)
```

For DICOM (medical), `pydicom`'s deidentification routines remove PHI fields — but verify, as datasets often contain burned-in pixel text (patient name overlaid on the image itself), which requires OCR-based redaction.

## 8. Label sanity

Most datasets have label noise (Northcutt et al. (2021) found errors in 3.4% of ImageNet, plus famous misses in MNIST, CIFAR). For small datasets, inspect a stratified random sample. For larger ones, use confident learning (`cleanlab`).

## 9. Augmentation (for completeness — happens later)

Augmentation is *training-time* perturbation, not cleaning. Common: random crop, flip, color jitter, blur, Cutout, Mixup, RandAugment.

Library: `albumentations` (faster, more options), `torchvision.transforms.v2`, `timm`.

## 10. Checklist

- [ ] Filtered out corrupt / unopenable files.
- [ ] Applied EXIF orientation.
- [ ] Converted to a single color space (usually RGB).
- [ ] Standardized resize / crop / normalize pipeline.
- [ ] Deduplicated within and between train/val/test.
- [ ] Stripped metadata before publishing.
- [ ] Audited a sample of labels.
