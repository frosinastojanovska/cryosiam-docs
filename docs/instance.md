# Instance Segmentation

The **instance segmentation module** in CryoSiam identifies **individual macromolecular complexes** as distinct,
volumetric objects within a tomogram.

Unlike semantic segmentation, which labels voxels by class, instance segmentation separates **individual particles**,
even when they belong to the same class or touch each other.

Instance segmentation is typically run **after lamella prediction** to restrict analysis to the sample region and avoid
false positives outside the lamella.

> **Recommended pipeline:**  
> `Denoising → Lamella Prediction → Instance Segmentation`

---

## Overview

Instance segmentation produces:

- **Instance masks** – each detected object is labeled with a unique ID
- **Probability maps** – voxel-wise confidence of object presence and boundaries
- **Optional lamella masking** – restricts predictions to valid sample regions

Lamella masking is strongly recommended, as the instance model was trained only on lamella regions.


---

## Example results

### Input (denoised tomogram)

![Denoised tomogram](images/instance/denoised_tomogram.png){ width="300" }

### Lamella mask (recommended)

![Lamella mask](images/instance/lamella_mask.png){ width="400" }

### Instance segmentation output

![Instance segmentation output](images/instance/instance_results.png)

---

## Trained Model

A pre-trained instance segmentation model is available:
[CryoSiam instance segmentation model (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_instance.ckpt)

---

## :octicons-command-palette-16: Running instance segmentation

Run instance segmentation using a YAML configuration file:

```bash
cryosiam instance_predict --config_file=configs/config_instance.yaml
```

To process a **single tomogram only**, use:

```bash
cryosiam instance_predict --config_file=configs/config_instance.yaml --filename TS_01.mrc
```

### Visualization

Instance masks and probability maps can be visualized using CryoSiam-Vis.
See visualization instructions [here](visualization.md#visualize_instance).

---

## Example Configuration (`configs/config_instance.yaml`)

:octicons-download-16: [Download instance config](configs/config_instance.yaml)

```yaml
data_folder: '/scratch/stojanov/dataset1/predictions/denoised'
mask_folder: '/scratch/stojanov/dataset1/predictions/lamella'
prediction_folder: '/scratch/stojanov/dataset1/predictions/instance'

trained_model: '/scratch/stojanov/trained_models/cryosiam_instance.ckpt'
file_extension: '.mrc'

test_files: null

save_raw_predictions: False

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
    threshold_foreground: 0.65
    boundary_bias: 0.8
    min_center_distance: 1
    max_center_distance: 5
    distance_type: 1   # 0 for using only predicted distances, 1 for using predicted distances + cdist from foreground, 2 for using only cdist
    postprocessing: False

hyper_parameters:
  batch_size: 10
```

---

## Config Reference

### Top‑level keys

| Key                   | Type                  | Must change the default value | Description                                                  |
|-----------------------|-----------------------|------------------------------:|--------------------------------------------------------------|
| `data_folder`         | `str`                 |                             ✅ | Directory containing denoised tomograms                      |
| `mask_folder`         | `str`                 |                             ✅ | Directory containing lamella masks (recommended              |
| `prediction_folder`   | `str`                 |                             ✅ | Output directory for instance segmentation results           |
| `trained_model`       | `str`                 |                             ✅ | Path to the instance segmentation model checkpoint (`.ckpt`) |
| `file_extension`      | `str`                 |                             ❌ | Input file extension (`.mrc` or `.rec`, default: `.mrc`)     |
| `test_files`          | `list[str]` or `null` |                             ❌ | Specific tomograms to process; `null` processes all files    |
| `save_internal_files` | `bool`                |                             ❌ | Save intermediate outputs (probability maps)                 |

---

### `parameters`

| Key                            | Type        | Must change the default value | Description                                                                               |
|--------------------------------|-------------|------------------------------:|-------------------------------------------------------------------------------------------|
| `data.patch_size`              | `list[int]` |                             ❌ | Sliding-window patch size for 3D inference                                                |
| `data.min`                     | `float`     |                             ❌ | Intensity minimum value for data scaling                                                  |
| `data.max`                     | `float`     |                             ❌ | Intensity maximum value for data scaling                                                  |
| `data.mean`                    | `float`     |                             ❌ | Mean used for normalization                                                               |
| `data.std`                     | `float`     |                             ❌ | Std used for normalization                                                                |
| `network.in_channels`          | `int`       |                             ❌ | Number of input channels (usually `1`)                                                    |
| `network.spatial_dims`         | `int`       |                             ❌ | Dimensionality of the model (`3` for tomograms)                                           |
| `network.threshold_foreground` | `float`     |                             ✅ | Foreground probability cutoff for seed generation (default `0.4`). Lower = more sensitive |
| `network.boundary_bias`        | `float`     |                             ❌ | Bias controlling separation of touching objects; higher values enforce stronger borders   |
| `network.min_center_distance`  | `int`       |                             ❌ | Minimum allowed distance (voxels) between centers (controls splitting)                    |
| `network.max_center_distance`  | `int`       |                             ❌ | Maximum allowed distance (voxels) for merging/association                                 |
| `network.distance_type`        | `int`       |                             ❌ | Distance strategy: `0` predicted, `1` predicted+cdist (default), `2` cdist only           |
| `network.postprocessing`       | `bool`      |                             ❌ | Apply additional connected-component cleanup                                              |

---

#### `hyper_parameters`

| Key          | Type  | Must change the default value | Description                                      |
|--------------|-------|------------------------------:|--------------------------------------------------|
| `batch_size` | `int` |                             ❌ | Number of 3D patches processed per forward pass. |

> **Tips**  
> Lower `threshold_foreground` to detect weaker objects; raise it to reduce false positives.  
> Tune `boundary_bias` and center-distance parameters to better separate touching particles.  
> Balance `batch_size` and `patch_size` to fit GPU memory.

---

## Troubleshooting

| Symptom                 | Suggested Fix                                                        |
|-------------------------|----------------------------------------------------------------------|
| CUDA out of memory      | Reduce `batch_size` or `patch_size`                                  |
| Objects outside lamella | Ensure lamella prediction was run and `mask_folder` is set correctly |
| Over-segmentation       | Increase `threshold_foreground` or adjust center-distance parameters |

---

## Next Steps

- [Subtomogram embeddings](subtomogram_embeddings.md)
- [Particle identification](particle_identification.md)
- [Visualization](visualization.md)
- [Usage overview](usage.md)
