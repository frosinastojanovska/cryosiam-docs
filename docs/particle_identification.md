# Particle Identification (Semantic Candidates → Particle Centers)

The **Particle Identification** task in CryoSiam is designed to detect **candidate macromolecular complexes** and
convert them into **particle coordinates** suitable for downstream workflows (csv and star files).

It works in **two steps**:

1. **Semantic prediction of candidate classes** (multi-class semantic model)
2. **Conversion to particle centers** (extracting coordinates from high-confidence regions)

**Workflow:**
> `Denoising → Lamella Prediction → Semantic (particle candidates) → Particle centers (.star / .csv)`
>
> **Recommended:** Always use a **lamella mask** to remove false positives outside the lamella, since models are trained
> only on lamella regions.

------

## Example Results

Semantic particle prediction (centers):  
![Semantic particle prediction](images/semantic/particle_prediction.png){ width="400" }

------

## Trained Models

You can download the trained lamella model from here: [CryoSiam lamella model (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_lamella.ckpt), and the semantic
segmentation model from here: [CryoSiam particle identification model (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_semantic_myco_candidates.ckpt).
You can also train your own model and then perform prediction with that model. Review
the [Semantic training](semantic_training.md) for explanation of the model training procedure.

------

## :octicons-command-palette-16: Commands

### Semantic Prediction for Particle Candidates

```bash
cryosiam semantic_predict --config_file=configs/semantic_particle.yaml
```

### Convert Semantic Predictions to Particle Centers

```bash
cryosiam semantic_to_centers --config_file=configs/semantic_particle.yaml
```

**Visualization**
For visualization of the results, CryoSiam-Vis can be used as described [here](visualization.md#visualize_coordinates).

---

## Example Configuration (`configs/semantic_particle.yaml`)

```yaml
data_folder: '/scratch/stojanov/dataset1/predictions/denoised'
mask_folder: '/scratch/stojanov/dataset1/predictions/lamella'
log_dir: '/scratch/stojanov/dataset1/'
prediction_folder: '/scratch/stojanov/dataset1/predictions/semantic_particle'

trained_model: '/scratch/stojanov/trained_models/cryosiam_semantic_myco_candidates.ckpt'
file_extension: '.mrc'

test_files: null

parameters:
  gpu_devices: 1
  data:
    patch_size: [ 128, 128, 128 ]
    min: 0
    max: 1
    mean: 0
    std: 1
  network:
    in_channels: 1
    spatial_dims: 3
    out_channels: 14
    threshold: 0.7

hyper_parameters:
  batch_size: 2
```

---

## Config Reference

### Top-level keys

| Key                 | Type                  | Must change | Description                                                |
|---------------------|-----------------------|------------:|------------------------------------------------------------|
| `data_folder`       | `str`                 |           ✅ | Folder with denoised tomograms.                            |
| `mask_folder`       | `str`                 |           ✅ | Folder with lamella masks to restrict predictions.         |
| `log_dir`           | `str`                 |           ❌ | Where logs and debug files are written.                    |
| `prediction_folder` | `str`                 |           ✅ | Output folder for semantic predictions + particle centers. |
| `trained_model`     | `str`                 |           ✅ | Path to `.ckpt` particle‑candidate model.                  |
| `file_extension`    | `str`                 |           ❌ | `.mrc` or `.rec`.                                          |
| `test_files`        | `list[str]` or `null` |           ❌ | Which tomograms to run.                                    |

---

### `parameters`

| Key                      | Type            | Must change | Description                          |
|--------------------------|-----------------|------------:|--------------------------------------|
| `gpu_devices`            | `int/list[int]` |           ❌ | GPU to use.                          |
| `data.patch_size`        | `list[int]`     |           ❌ | Patch size for inference.            |
| `data.min` / `data.max`  | `float`         |           ❌ | Intensity clipping.                  |
| `data.mean` / `data.std` | `float`         |           ❌ | Normalization stats.                 |
| `network.in_channels`    | `int`           |           ❌ | Number of input channels (always 1). |
| `network.spatial_dims`   | `int`           |           ❌ | 3 for 3D tomograms.                  |
| `network.out_channels`   | `int`           |           ✅ | Number of semantic classes (14).     |
| `network.threshold`      | `float`         |           ❌ | Probability threshold for detection. |

---

### `hyper_parameters`

| Key          | Type  | Must change | Description                            |
|--------------|-------|------------:|----------------------------------------|
| `batch_size` | `int` |           ❌ | Number of patches processed per batch. |


