# Instance Segmentation

The instance segmentation task of CryoSiam identifies individual macromolecular complexes as distinct volumetric
objects within a tomogram.  
It is typically used **after lamella prediction** to restrict analysis to relevant sample regions and avoid false
positives outside the lamella.

> **Recommended:** Run [lamella prediction](semantic.md) first and set the `mask_folder` in the configuration to exclude
> background noise.  
> **Workflow:** `Denoising → Lamella Prediction → Instance Segmentation`

---


## Example Results

- **Instance masks:** Each detected object labeled uniquely.
- **Probability map:** Voxel-wise confidence of object presence.
- **Lamella mask (optional):** Restricts predictions to valid sample area.

### Input (Raw Tomogram)

![Denoised tomogram](images/instance/denoised_tomogram.png){ width="300" }

### Lamella Mask (Recommended)

![Lamella mask](images/instance/lamella_mask.png){ width="400" }

### Instance Segmentation Result

![Instance segmentation output](images/instance/instance_results.png)

---

## Trained Model

You can download the trained model from
here: [CryoSiam instance segmentation model (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_instance.ckpt)

---

## :octicons-command-palette-16: Command

Run instance segmentation with:

```bash
cryosiam instance_predict --config_file=configs/config_instance.yaml
```

Optionally, process a specific tomogram file only:

```bash
cryosiam instance_predict --config_file=configs/config_instance.yaml --filename TS_01.mrc
```

---

## Example Configuration (`configs/config_instance.yaml`)

:octicons-download-16: [Download instance config](configs/config_instance.yaml)

```yaml
data_folder: '/scratch/stojanov/datatset1/predictions/denoised'
mask_folder: '/scratch/stojanov/datatset1/predictions/lamella'
log_dir: '/scratch/stojanov/datatset1/'
prediction_folder: '/scratch/stojanov/datatset1/predictions/instance'

trained_model: '/scratch/stojanov/trained_models/cryosiam_instance.ckpt'
file_extension: '.mrc'

test_files: null

save_raw_predictions: False

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

| Key | Type | Must change the default value | Description |
|---|---|---:|---|
| `data_folder` | `str` | ✅ | Directory containing denoised tomograms for instance segmentation. |
| `mask_folder` | `str` | ✅ | Directory with lamella masks (recommended to limit segmentation to lamella regions). |
| `log_dir` | `str` | ❌ | Folder where logs and runtime information are stored. |
| `prediction_folder` | `str` | ✅ | Output directory for instance predictions. |
| `trained_model` | `str` | ✅ | Path to the trained CryoSiam instance model checkpoint (`.ckpt`). |
| `file_extension` | `str` | ❌ | File extension of input tomograms (`.mrc`, `.rec`, etc.). |
| `test_files` | `list[str]` or `null` | ❌ | List of specific files to process. `null` = all in `data_folder`. |
| `save_raw_predictions` | `bool` | ❌ | If `true`, also save raw network outputs before any cleanup. |

---

### `parameters`

| Key | Type | Must change the default value | Description |
|---|---|---:|---|
| `gpu_devices` | `int` or `list[int]` | ❌ | GPU(s) to use for inference. |
| `data.patch_size` | `list[int]` | ❌ | 3D patch size for sliding‑window inference (here `[64,64,64]`). Reduce if OOM. |
| `data.min` | `float` | ❌ | Intensity minimum for normalization. |
| `data.max` | `float` | ❌ | Intensity maximum for normalization. |
| `data.mean` | `float` | ❌ | Normalization mean. |
| `data.std` | `float` | ❌ | Normalization standard deviation. |
| `network.in_channels` | `int` | ❌ | Number of input channels (usually `1`). |
| `network.spatial_dims` | `int` | ❌ | Dimensionality of the model (always `3`). |
| `network.threshold_foreground` | `float` | ✅ | Foreground probability cutoff for seed/region proposals (default `0.4`). Lower = more sensitive. |
| `network.boundary_bias` | `float` | ❌ | Bias toward separating touching objects; higher values enforce stronger borders. |
| `network.min_center_distance` | `int` | ❌ | Minimum allowed distance (voxels) between centers (controls splitting). |
| `network.max_center_distance` | `int` | ❌ | Maximum allowed distance (voxels) for merging/association. |
| `network.distance_type` | `int` | ❌ | Distance strategy: `0`=predicted, `1`=predicted+cdist (default), `2`=cdist only. |
| `network.postprocessing` | `bool` | ❌ | Apply connected‑components cleanup and size filtering if implemented. |

> **Tips**  
> • Lower `threshold_foreground` to recover faint objects; raise it to reduce false positives.  
> • Tune `boundary_bias` and center‑distance settings to improve separation of touching particles.  
> • Keep `patch_size` and `batch_size` balanced to fit GPU memory.

---

### `hyper_parameters`

| Key | Type | Must change the default value | Description                                                                             |
|---|---|---:|-----------------------------------------------------------------------------------------|
| `batch_size` | `int` | ❌ | Number of patches per inference batch (here `10`). Reduce if you hit GPU memory limits. |

---

## Troubleshooting

| Symptom                  | Suggested Fix                                                   |
|--------------------------|-----------------------------------------------------------------|
| GPU memory error         | Reduce `batch_size` or `data.patch_size`.                       |
| Objects outside lamella  | Use lamella mask and set `mask_folder` properly.                |

---

## Next Steps

- [Semantic segmentation](semantic.md) → upstream voxel classification
- [Particle identification](particle_identification.md) → downstream localization
- [Visualization](visualization.md) → explore instance masks in napari
- [Usage overview](usage.md)
