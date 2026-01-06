# Particle Identification (Semantic Candidates → Particle Centers)

The **Particle Identification** module in CryoSiam detects **candidate macromolecular complexes** and converts them into
**particle coordinates** suitable for downstream workflows (e.g. `.csv` and `.star` files).

This task is typically used when the goal is to **localize particles** of given type.

---

## Overview

Particle identification is performed in **two steps**:

1. **Semantic prediction of candidate classes**
   A multi-class semantic segmentation model predicts candidate particle regions.
2. **Conversion to particle centers**
   High-confidence regions are converted into particle coordinates.

**Recommended pipeline:**
`Denoising → Lamella Prediction → Particle identification → Particle centers (.star / .csv)`

> **Important:** Always use a **lamella mask** to restrict predictions to the lamella region.
> Models were trained only on lamella data and may produce false positives outside this region.

------

## Example result

Semantic particle prediction (particle centers):
![Semantic particle prediction](images/semantic/particle_prediction.png){ width="400" }

------

## Trained Models

Pre-trained models are provided for particle identification:

- **Lamella prediction model:**  
  [CryoSiam lamella model (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_lamella.ckpt)

- **Particle candidate semantic model:**  
  [CryoSiam particle identification model (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_semantic_myco_candidates.ckpt)

You can also train your own model and then perform prediction with that model. Review
the [Semantic training](semantic_training.md) for explanation of the model training procedure.

------

## :octicons-command-palette-16: Running particle identification

Particle identification is executed in **two stages**, using the same configuration file.

### Stage 1: Semantic prediction of particle candidates

```bash
cryosiam semantic_predict --config_file=configs/semantic_particle.yaml
```

**What this step does**

- Loads the particle-candidate semantic model
- Performs sliding-window 3D inference
- Produces semantic prediction volumes

### Stage 2: Convert semantic predictions to particle centers

```bash
cryosiam semantic_to_centers --config_file=configs/semantic_particle.yaml
```

**What this step does**

- Identifies connected high-confidence regions
- Extracts particle center coordinates
- Writes particle lists in .csv and .star formats

------

### Visualization

Particle coordinates and semantic predictions can be visualized using CryoSiam-Vis.
See visualization instructions [here](visualization.md#visualize_coordinates).

------

## Example Configuration (`configs/semantic_particle.yaml`)

```yaml
data_folder: '/scratch/stojanov/dataset1/predictions/denoised'
mask_folder: '/scratch/stojanov/dataset1/predictions/lamella'
prediction_folder: '/scratch/stojanov/dataset1/predictions/semantic_particle'

trained_model: '/scratch/stojanov/trained_models/cryosiam_semantic_myco_candidates.ckpt'
file_extension: '.mrc'

test_files: null

parameters:
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
    min_voxels: 10

hyper_parameters:
  batch_size: 2
```

---

## Configuration reference

### Top-level keys

| Key                 | Type                  | Must change | Description                                                    |
|---------------------|-----------------------|------------:|----------------------------------------------------------------|
| `data_folder`       | `str`                 |           ✅ | Directory containing denoised tomograms                        |
| `mask_folder`       | `str`                 |           ✅ | Directory containing lamella masks                             |
| `prediction_folder` | `str`                 |           ✅ | Output directory for semantic predictions and particle centers |
| `trained_model`     | `str`                 |           ✅ | Path to particle-candidate model checkpoint (`.ckpt`)          |
| `file_extension`    | `str`                 |           ❌ | Input file extension (`.mrc` or `.rec`)                        |
| `test_files`        | `list[str]` or `null` |           ❌ | Specific tomograms to process; `null` processes all            |

---

### `parameters`

| Key                    | Type        | Must change | Description                                                       |
|------------------------|-------------|------------:|-------------------------------------------------------------------|
| `data.patch_size`      | `list[int]` |           ❌ | Sliding-window patch size for 3D inference                        |
| `data.min`             | `float`     |           ❌ | Intensity minimum value for data scaling                          |
| `data.max`             | `float`     |           ❌ | Intensity maximum value for data scaling                          |
| `data.mean`            | `float`     |           ❌ | Mean used for normalization                                       |
| `data.std`             | `float`     |           ❌ | Std used for normalization                                        |
| `network.in_channels`  | `int`       |           ❌ | Number of input channels (usually `1`)                            |
| `network.spatial_dims` | `int`       |           ❌ | Dimensionality of the model (`3` for tomograms)                   |
| `network.threshold`    | `float`     |           ❌ | Probability threshold for candidate detection.                    |
| `network.min_voxels`   | `int`       |           ❌ | Minimum connected-component size (voxels) to keep. Default is 100 |

---

### `hyper_parameters`

| Key          | Type  | Must change | Description                                     |
|--------------|-------|------------:|-------------------------------------------------|
| `batch_size` | `int` |           ❌ | Number of 3D patches processed per forward pass |

---

## Outputs

- **Semantic prediction volumes** (intermediate)
- **Particle center files** in `.csv` and `.star` formats

Output filenames follow the input basenames with appropriate suffixes.

---

## Troubleshooting

| Symptom                  | Suggested Fix                                                        |
|--------------------------|----------------------------------------------------------------------|
| Too many false positives | Increase `threshold` or `min_voxels`; ensure lamella mask is correct |
| No particles detected    | Lower `threshold` or verify trained model                            |
| CUDA out of memory       | Reduce `batch_size` or `patch size`                                  |

---

## Next Steps

- [Instance segmentation](instance.md)
- [Semantic segmentation training](semantic_training.md)
- [Usage overview](usage.md)
