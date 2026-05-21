# Image Visualization

> **TL;DR** — In computer-vision projects, "visualization" almost always means showing the *images themselves* alongside the model's output. Grids of samples, confusion matrices with example tiles per cell, and saliency maps are the workhorses.

## Pick a viz

| Goal | Viz |
|---|---|
| Inspect random samples | **Image grid** |
| Inspect class balance | Bar of count per class + sample tile per bar |
| Diagnose model errors | **Confusion matrix** with sample tiles per cell |
| Find near-duplicates | t-SNE / UMAP on embeddings (colored by class) |
| Locate model's attention | **Grad-CAM / SmoothGrad / Integrated Gradients** overlay |
| Diagnose segmentation | Mask overlay + IoU per class |
| Diagnose detection | Bounding boxes overlaid on image |
| Inspect augmentations | Grid of one image with N augmentations |
| Compare model versions | Side-by-side predictions per image |
| Track training | Loss / metric curves + sample predictions per epoch |

## Image grid

```python
import torch, torchvision

grid = torchvision.utils.make_grid(batch[:64], nrow=8, padding=2, normalize=True)
plt.imshow(grid.permute(1, 2, 0))
plt.axis('off')
```

Conventions:

- 8 columns is a comfortable default.
- Border around each tile to separate.
- Caption images with class label and prediction (and ✓ / ✗).
- Sort by confidence or by class to make patterns visible.

## Confusion matrix + sample tiles

A confusion matrix alone shows where the model confuses A for B, but not *which* examples. Tile a small grid of misclassified examples in each off-diagonal cell — patterns are usually obvious (wrong crops, mislabeled, near-classes).

## Grad-CAM and friends

For CNNs and ViTs, generate a heatmap of where the model "looked":

```python
from pytorch_grad_cam import GradCAM
cam = GradCAM(model=model, target_layers=[model.layer4[-1]])
heat = cam(input_tensor=img_tensor, targets=[ClassifierOutputTarget(cls)])[0]

plt.imshow(img)
plt.imshow(heat, cmap='jet', alpha=0.5)
```

Methods:

- **Grad-CAM / Grad-CAM++** — class-discriminative, low resolution.
- **SmoothGrad** — averaged noisy gradients, higher resolution.
- **Integrated Gradients** — axiomatic attribution.
- **Score-CAM** — gradient-free; expensive but reliable.
- **Attention rollout** (for transformers).

Caveat: attribution methods are correlated, not authoritative. Always sanity-check with the "model randomization" test (Adebayo et al., 2018).

## Segmentation overlays

```python
plt.imshow(img)
plt.imshow(mask, alpha=0.5, cmap='tab20')
```

For multi-class masks, ensure the palette is consistent across images.

## Detection overlays

Draw bounding boxes with class + confidence. For dense detection, color boxes by class and limit display to top-K by confidence to avoid clutter.

## Augmentation grids

When debugging an augmentation pipeline, show **one source image with N augmented variants**. If you can't recognize the image as the same object, your augmentation is too aggressive.

## Embedding scatter

```python
from umap import UMAP
xy = UMAP(n_components=2).fit_transform(features)
plt.scatter(xy[:, 0], xy[:, 1], c=labels, s=3, cmap='tab20')
```

For interactivity, plotly + image-on-hover is excellent for spotting label noise: hover over an outlier point, see the actual image.

## Training curves + sample predictions

For each epoch, save a small grid of (image, true label, predicted label). Reviewing this *grid* alongside the loss curve catches qualitative regressions that loss alone misses.

## Pitfalls

1. **Showing only the highest-confidence correct predictions** — you'll miss the failure modes.
2. **Grad-CAM as ground truth** for "what the model uses." It's a clue, not a proof.
3. **Wrong color channel order.** RGB looks correct; BGR is the same with red and blue swapped — easy to mistake.
4. **Forgetting to denormalize** an ImageNet-normalized tensor before display.
5. **Confusion matrix without per-class normalization** when classes are imbalanced.
6. **Reporting metrics without showing a sample of errors.**
