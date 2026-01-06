# Semantic Segmentation

The semantic segmentation module classifies each voxel of a tomogram into biologically meaningful categories, such as
membranes, particles, filaments (DNA/RNA), microtubules, or actin.

Semantic segmentation in CryoSiam is typically performed **inside the lamella only**.  
Therefore, the module also includes lamella prediction as a preparatory step, which is **strongly recommended** to
avoid false positives outside the lamella region (models were not trained on the surrounding noisy volume).


---

## Overview

Semantic segmentation in CryoSiam has **two distinct capabilities**:

1. **Lamella Prediction**
    - Identifies lamella (sample) region
    - Produces binary masks and probability maps
    - Used to restrict downstream analysis

2. **Voxel-wise Semantic Segmentation**
    - Classifies voxels *within the lamella* into biological classes.
    - Trained to work with denoised tomograms.

**Recommended pipeline:**  
`Denoising → Lamella Prediction → Semantic Segmentation`

---

## Example Results

- **Lamella mask**: restricts analysis to lamella regions
- **Semantic segmentation**: predicted classes inside lamella

### Input (denoised tomogram)

![Denoised tomogram](images/semantic/denoised_tomogram.png){ width="300" }

### Lamella prediction

![Lamella mask](images/semantic/lamella_mask.png){ width="400" }

### Semantic segmentation

![Semantic segmentation output](images/semantic/segmentation.png){ width="800" }


---

## Trained Models

Pre-trained models are provided for both steps:

- **Lamella prediction model:**  
  [CryoSiam lamella model (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_lamella.ckpt)
- **Semantic segmentation model:**  
  [CryoSiam semantic model (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_semantic_segmentation.ckpt)

You can also train your own semantic segmentation model and use it for prediction.
See [Semantic segmentation  training](semantic_training.md) for details.

___

## :octicons-command-palette-16: Running semantic segmentation

Semantic segmentation is usually run in **two stages**.

---

### Stage 1: Lamella prediction

Run lamella prediction using a YAML configuration file:

```bash
cryosiam semantic_predict --config_file=configs/config_lamella.yaml
```

**What it does**

- Loads the trained lamella model and your denoised tomogram/s
- Performs sliding-window 3D inference
- Writes lamella masks and probability maps to disk

### Stage 2: Semantic segmentation

Run voxel-wise semantic segmentation using the lamella masks from Stage 1:

```bash
cryosiam semantic_predict --config_file=configs/config_semantic.yaml
```

**What this step does**

- Loads the trained semantic segmentation model
- Uses lamella masks to restrict predictions
- Writes semantic segmentation outputs to disk

---

### Visualization

Results can be visualized using CryoSiam-Vis.
See the visualization instructions [here](visualization.md#visualize_semantic).

---

## Example Configurations

### 1. Lamella Prediction (`configs/config_lamella.yaml`)

📥 [Download lamella config](configs/config_lamella.yaml)

```yaml
data_folder: '/scratch/stojanov/dataset1/predictions/denoised'
prediction_folder: '/scratch/stojanov/dataset1/predictions/lamella'

trained_model: '/scratch/stojanov/trained_models/cryosiam_lamella.ckpt'
file_extension: '.mrc'

test_files: null

save_internal_files: False

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
    threshold: 0.9
    postprocessing: True
    3d_postprocessing: False

hyper_parameters:
  batch_size: 2
```

### Configuration reference

#### Top‑level keys

| Key                   | Type                  | Must change the default value | Description                                                         |
|-----------------------|-----------------------|------------------------------:|---------------------------------------------------------------------|
| `data_folder`         | `str`                 |                             ✅ | Directory containing denoised tomograms used for lamella prediction |
| `prediction_folder`   | `str`                 |                             ✅ | Directory where lamella masks and optional intermediates are saved  |
| `trained_model`       | `str`                 |                             ✅ | Path to the lamella prediction model checkpoint (`.ckpt`)           |
| `file_extension`      | `str`                 |                             ❌ | Input file extension (`.mrc` or `.rec`, default: `.mrc`)            |
| `test_files`          | `list[str]` or `null` |                             ❌ | Specific tomograms to process; `null` processes all files           |
| `save_internal_files` | `bool`                |                             ❌ | Save intermediate outputs (probability maps)                        |

---

#### `parameters`

| Key                         | Type        | Must change the default value | Description                                                               |
|-----------------------------|-------------|------------------------------:|---------------------------------------------------------------------------|
| `data.patch_size`           | `list[int]` |                             ❌ | Sliding-window patch size for 3D inference                                |
| `data.min`                  | `float`     |                             ❌ | Intensity minimum value for data scaling                                  |
| `data.max`                  | `float`     |                             ❌ | Intensity maximum value for data scaling                                  |
| `data.mean`                 | `float`     |                             ❌ | Mean used for normalization                                               |
| `data.std`                  | `float`     |                             ❌ | Std used for normalization                                                |
| `network.in_channels`       | `int`       |                             ❌ | Number of input channels (usually `1`)                                    |
| `network.spatial_dims`      | `int`       |                             ❌ | Dimensionality of the model (`3` for tomograms)                           |
| `network.threshold`         | `float`     |                             ✅ | Probability threshold used to binarize the lamella mask. Default to `0.9` |
| `network.postprocessing`    | `bool`      |                             ❌ | Apply morphological postprocessing to clean the mask                      |
| `network.3d_postprocessing` | `bool`      |                             ❌ | Apply postprocessing in full 3D (instead of slice-wise)                   |

> **Tips**  
> Lower `network.threshold` (e.g. 0.7) if lamella masks are too restrictive.  
> Keep `postprocessing: True` for cleaner masks.

---

#### `hyper_parameters`

| Key          | Type  | Must change the default value | Description                                     |
|--------------|-------|------------------------------:|-------------------------------------------------|
| `batch_size` | `int` |                             ❌ | Number of 3D patches processed per forward pass |

---

### 2. Semantic Segmentation (`configs/config_semantic.yaml`)

:octicons-download-16: [Download semantic config](configs/config_semantic.yaml)

```yaml
data_folder: '/scratch/stojanov/dataset1/predictions/denoised'
mask_folder: '/scratch/stojanov/dataset1/predictions/lamella'
prediction_folder: '/scratch/stojanov/dataset1/predictions/semantic'

trained_model: '/scratch/stojanov/trained_models/cryosiam_semantic.ckpt'
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
    threshold: 0.1
    postprocessing_sizes: [ -1, 5000, -1, -1, -1 ]

hyper_parameters:
  batch_size: 2
```

### Configuration reference

#### Top‑level keys

| Key                   | Type                  | Must change the default value | Description                                                         |
|-----------------------|-----------------------|------------------------------:|---------------------------------------------------------------------|
| `data_folder`         | `str`                 |                             ✅ | Directory containing denoised tomograms used for lamella prediction |
| `mask_folder`         | `str`                 |                             ✅ | Directory containing lamella masks from the lamella prediction step |
| `prediction_folder`   | `str`                 |                             ✅ | Directory where semantic segmentation outputs are saved             |
| `trained_model`       | `str`                 |                             ✅ | Path to the lamella prediction model checkpoint (`.ckpt`)           |
| `file_extension`      | `str`                 |                             ❌ | Input file extension (`.mrc` or `.rec`, default: `.mrc`)            |
| `test_files`          | `list[str]` or `null` |                             ❌ | Specific tomograms to process; `null` processes all files           |
| `save_internal_files` | `bool`                |                             ❌ | Save intermediate outputs (probability maps)                        |

---

#### `parameters`

| Key                            | Type        | Must change the default value | Description                                                                                                                                           |
|--------------------------------|-------------|------------------------------:|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| `data.patch_size`              | `list[int]` |                             ❌ | Sliding-window patch size for 3D inference                                                                                                            |
| `data.min`                     | `float`     |                             ❌ | Intensity minimum value for data scaling                                                                                                              |
| `data.max`                     | `float`     |                             ❌ | Intensity maximum value for data scaling                                                                                                              |
| `data.mean`                    | `float`     |                             ❌ | Mean used for normalization                                                                                                                           |
| `data.std`                     | `float`     |                             ❌ | Std used for normalization                                                                                                                            |
| `network.in_channels`          | `int`       |                             ❌ | Number of input channels (usually `1`)                                                                                                                |
| `network.spatial_dims`         | `int`       |                             ❌ | Dimensionality of the model (`3` for tomograms)                                                                                                       |
| `network.threshold`            | `float`     |                             ✅ | Probability cutoff to remove very low-confidence predictions. Default `0.1`.                                                                          |
| `network.postprocessing_sizes` | `list[int]` |                             ✅ | Size thresholds for connected components postprocessing. Example: `[ -1, 5000, -1, -1, -1 ]` keeps only connected components >5000 voxels for label 2 |

> **Tips**  
> • Tune `postprocessing_sizes` depending on dataset type and noise in predictions.

---

#### `hyper_parameters`

| Key          | Type  | Must change the default value | Description                                      |
|--------------|-------|------------------------------:|--------------------------------------------------|
| `batch_size` | `int` |                             ❌ | Number of 3D patches processed per forward pass. |

---

## Outputs

- **Lamella masks** saved to `prediction_folder` (.h5).
- **Segmentation mask** saved to `prediction_folder` (.h5).
- **Optional intermediate files**, depending on configuration.

Output filenames follow the input basenames with appropriate suffixes.

---

## Troubleshooting

| Symptom                         | Suggested Fix                                                        |
|---------------------------------|----------------------------------------------------------------------|
| False positives outside lamella | Ensure lamella prediction was run and `mask_folder` is set correctly |
| CUDA OOM                        | Reduce `batch_size` or `patch size`                                  |
| Blank segmentation              | Verify model path and reduce thresholds                              |

---

## Next Steps

- [Semantic segmentation training](semantic_training.md)
- [Instance segmentation](instance.md)
- [Particle identification](particle_identification.md)
- [Usage overview](usage.md)