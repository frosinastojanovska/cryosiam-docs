# Semantic Segmentation Training

This page explains how to **train CryoSiam’s semantic segmentation model** using your own annotated (or simulated)
cryo-ET tomograms. Training can start from pretrained weights (or from scratch) and supports large-scale, multi-GPU
training with extensive data augmentation.

Semantic segmentation training in CryoSiam follows a **five-step** CLI workflow:

1. **Ground-truth filtering** (optional)
2. **Normalize / scale input tomograms** (optional, but strongly recommended for tomograms which are not denoised with
   CryoSiam)
3. **Preprocess - generate distance maps**
4. **Patch creation**
5. **Model training**

Each step is executed explicitly to give full control and reproducibility. Example configuration file is
given [below](#example-configuration-semantic_trainyaml).

---

## Pretrained weights

To fine-tune instead of training from scratch, download the pretrained checkpoint:

- [CryoSiam DenseSimSiam pretrained checkpoint (v1.0)](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/dense_simsiam_pretrained.ckpt)

Set it in your config under `pretrained_model`.

---

## Training workflow overview

## Step 1: Filter ground-truth labels (optional)

This step optionally filters and remaps existing semantic label volumes to create a new set of training masks.

It is designed for cases where:

- Only a **subset of classes** should be used for training (e.g. simulated datasets with many labeled structures)
- **Binary masks** are needed instead of full multi-class labels
- Existing labels need to be **cleaned and validated** before training

### What this step does

Given a folder of existing semantic label volumes, CryoSiam can:

- Select **one label value** or **multiple label values**
- Create new masks containing **only the selected class(es)**

If a single label value is selected, the output is a **binary foreground mask**.
If multiple label values are selected, the output contains **only those classes**.

This step **does not modify the original labels**. Filtered masks are written to a new output folder.

### Configuration options

- `labels_folder_for_filtering` – path to the folder containing existing semantic label volumes
- `selected_labels` – a single label value or a list of label values to keep

### Command

```bash
cryosiam semantic_filter_ground_truth --config_file=configs/semantic_training.yaml
```

Produces

- cleaned / filtered semantic label volumes

### Creating label masks when none are provided

If ground-truth semantic labels are **not available**, CryoSiam provides preprocessing utilities to **generate training
masks automatically** from given coordinates and/or RELION type orientation angles.

These processing functions allow you to:

- Create **spherical masks** from particle center coordinates
- Generate masks directly from **density maps or probability maps**

> **Note:**
> Mask generation is handled by dedicated preprocessing tools and is **not part of the semantic training pipeline itself
**.

For details, refer to [Processing functions](processing_functions.md).

--- 

## Step 2: Normalize and scale input tomograms (optional)

This step ensures that intensities of the input tomograms are **scaled and oriented consistently** with the expectations
of the pretrained semantic segmentation model.

The model expects tomograms to be:

- **Scaled to the range [0, 1]**
- **White on black** (high density = high intensity)

Correct intensity scaling is critical for stable training and convergence.

### When is this step required?

- **Required** if your input tomograms were generated externally or come from a different preprocessing pipeline.
- If your data is already white-on-black, you can omit `--invert` and only apply scaling.
- **Can be skipped** if you use CryoSiam denoised tomograms and denoising was run with `scale_prediction: True`.

### What this step does

When required, this step:

- (Optionally) `inverts` intensities to enforce white-on-black contrast
- Rescales tomograms to the range `[0, 1]`
- Clips extreme intensity values using percentile thresholds `[0.1, 99.9]`

### Command

```bash
cryosiam processing_invert_scale \
 --input_path=folder \
 --output_path=out_folder \
 --invert \
 --lower_end_percentage 0.1 \
 --upper_end_percentage 99.9
```

For a detailed explanation of this function and its parameters, refer to
[Processing functions](processing_functions.md).

---

## Step 3: Generate distance maps for training

This step generates **distance maps** from the provided semantic segmentation masks.
These distance maps are used as **auxiliary training targets** by CryoSiam’s semantic segmentation model.

This step does **not** modify, validate, or align the semantic labels. It strictly computes distance-based targets
derived from the existing masks.


> Note: the CLI command name includes “preprocess”, but in this workflow it is used only for distance-map creation.

### Command

```bash
cryosiam semantic_train_preprocess --config_file=configs/semantic_training.yaml
```

Outputs

- distance map volumes corresponding to the semantic label masks

These outputs are consumed by the patch creation step and used during training as auxiliary targets


--- 

## Step 4: Create training patches

This step splits full tomograms and their corresponding training targets into **overlapping 3D patches** suitable for
GPU-based training. Patch creation is required because tomograms are big 3D volumes that can't fit into GPU-memory.

### What this step does

Given:

- Input tomograms (scaled as required in Step 2)
- Semantic segmentation masks
- Distance maps generated in Step 3

CryoSiam:

- Extracts overlapping 3D patches from tomograms
- Extracts the corresponding label patches
- Extracts distance-map patches
- Optionally discards patches containing only background labels
- Stores all patches on disk for efficient training

No data augmentation is applied at this stage.

### Configuration options

- `parameters.data.patch_size` - size of the 3D training patches.
- `parameters.data.patch_overlap` - fractional overlap between adjacent patches.
- `parameters.data.remove_only_background` - if `True`, patches containing only background labels are discarded.

### Command:

```bash
cryosiam semantic_train_create_patches --config_file=configs/semantic_training.yaml
```

**Outputs**

- Tomogram patches
- Semantic label patches
- Distance-map patches

---

## Step 5. Train the semantic segmentation model

This step runs the full semantic segmentation training pipeline using **PyTorch Lightning**.
Training is performed on the patches generated in Step 4 and optionally initialized from pretrained weights.

### What this step does

During training, CryoSiam:

- Loads tomogram, label, and distance-map patches
- Applies data augmentation on-the-fly
- Optimizes the semantic segmentation network
- Optionally predicts auxiliary distance maps (if enabled)
- Periodically evaluates the model on validation data
- Saves checkpoints, logs, and training statistics

Training supports **single-GPU, multi-GPU, and multi-node** execution.

### Command

```bash
cryosiam semantic_train --config_file=configs/semantic_training.yaml
```

### Training on Slurm clusters

When running on a Slurm-managed system, use:

```bash
srun cryosiam semantic_train --config_file=configs/semantic_training.yaml
```

Ensure that Slurm resource requests (GPUs, nodes, CPUs) match the configuration parameters.

### Outputs

Training produces:

- Model checkpoints (`.ckpt`)
- Training and validation logs
- TensorBoard files

### Resuming and fine-tuning

To resume interrupted training, set:

```yaml
continue_training: True
```

To fine-tune from pretrained weights, set:

```yaml
pretrained_model: /path/to/model.ckpt
continue_training: False
```

---

## Example Configuration (semantic_train.yaml)

:octicons-download-16: [Download example config](configs/config_semantic_train.yaml)

```yaml
data_folder: '/scratch/stojanov/dataset1/predictions/denoised'
labels_folder: '/scratch/stojanov/dataset1/semantic_gt_for_training'
noisy_data_folder: null
patches_folder: '/scratch/stojanov/dataset1/models/dense_simsiam_semantic_complexes/patches'
temp_dir: '/scratch/stojanov/dataset1/models/dense_simsiam_semantic_complexes'
log_dir: '/scratch/stojanov/dataset1/models/dense_simsiam_semantic_complexes'
prediction_folder: '/scratch/stojanov/dataset1/models/dense_simsiam_semantic_complexes/predictions'
pretrained_model: '/scratch/stojanov/trained_models/dense_simsiam_pretrained.ckpt'
file_extension: '.mrc'

train_files: [ 'sample_1.mrc', 'sample_2.mrc', 'sample_3.mrc', 'sample_4.mrc', 'sample_5.mrc',
               'sample_6.mrc', 'sample_7.mrc', 'sample_8.mrc', 'sample_9.mrc', 'sample_10.mrc',
               'sample_11.mrc', 'sample_12.mrc', 'sample_13.mrc', 'sample_14.mrc', 'sample_15.mrc',
               'sample_16.mrc', 'sample_17.mrc', 'sample_18.mrc', 'sample_19.mrc', 'sample_20.mrc',
               'sample_21.mrc', 'sample_22.mrc', 'sample_23.mrc', 'sample_24.mrc', 'sample_25.mrc',
               'sample_26.mrc', 'sample_27.mrc', 'sample_28.mrc', 'sample_29.mrc', 'sample_30.mrc' ]

val_files: null
validation_ratio: 0.1

continue_training: False

labels_folder_for_filtering: null
selected_labels: null

parameters:
  nodes: 1
  gpu_devices: 8
  data:
    patch_size: [ 128, 128, 128 ]
    patch_overlap: 0.5
    remove_only_background: False
    min: 0
    max: 1
    mean: 0
    std: 1
  transforms:
    low_pass_sigma_range: [ 0.1, 0.6 ]
    high_pass_sigma_range: [ 0.1, 0.2 ]
    high_pass_sigma2_range: [ 4, 6 ]
    noise_sigma_range: [ 0.01, 0.05 ]
    combine_transforms: False
    use_noisy_input: False
    scale_intensity_factors: null
    elastic: null
    zoom: [ 0.8, 1.2 ]
    rotate: [ 3.14, 3.14, 3.14 ]
    flip: True
  network:
    in_channels: 1
    spatial_dims: 3
    out_channels: 7
    dense_dim: 64
    filters: [ 32, 64 ]
    kernel_size: 3
    padding: 1
    distance_prediction: True
    use_dice_loss: True
    unfreeze_decoder: True
    unfreeze_backbone: True

hyper_parameters:
  cache_rate: 0
  val_interval: 1
  batch_size: 3
  optimizer: 'adamw' # one of 'sgd', 'adam', 'adamw'
  lr: 0.001
  momentum: 0.9
  weight_decay: 0.00001
  max_epochs: 200
```

---

## Config Reference

### Top‑level keys

| Key                           | Type                         | Must change the default value | Description                                                                                                       |
|-------------------------------|------------------------------|------------------------------:|-------------------------------------------------------------------------------------------------------------------|
| `data_folder`                 | `str`                        |                             ✅ | Tomograms or precomputed predictions used as input                                                                |
| `labels_folder`               | `str`                        |                             ✅ | Ground-truth semantic labels                                                                                      |
| `labels_folder_for_filtering` | `str` or `null`              |                             ❌ | Optional: folder of label masks to filter/remap in Step 1                                                         |
| `selected_labels`             | `int` / `list[int]` / `null` |                             ❌ | Optional: label(s) to keep in Step 1 (single value → binary mask; list → keep subset)                             |
| `noisy_data_folder`           | `str` or `null`              |                             ❌ | Optional additional input folder (used only if your transforms/pipeline uses noisy input)                         |
| `patches_folder`              | `str`                        |                             ✅ | Output folder where training patches will be written (Step 4)                                                     |
| `temp_dir`                    | `str`                        |                             ❌ | Temporary folder for intermediate files                                                                           |
| `log_dir`                     | `str`                        |                             ❌ | Training logs, checkpoints, TensorBoard output                                                                    |
| `prediction_folder`           | `str`                        |                             ❌ | Optional folder for prediction sanity checks (if your pipeline writes them)                                       |
| `pretrained_model`            | `str`                        |                             ❌ | Checkpoint to initialize weights (fine-tuning) (`.ckpt`)                                                          |
| `file_extension`              | `str`                        |                             ❌ | Input tomogram extension (`.mrc` or `.rec`)                                                                       |
| `train_files`                 | `list[str]` or `null`        |                           ✅/❌ | Explicit training file list; null uses all files in data_folder (except those assigned to val/test if applicable) |
| `val_files`                   | `list[str]` or `null`        |                             ❌ | Optional explicit validation list                                                                                 |
| `validation_ratio`            | `float`                      |                             ❌ | Used only if `val_files: null` (fraction of training set used for validation)                                     |
| `continue_training`           | `bool`                       |                             ❌ | If true, `resume` training from latest checkpoint under `log_dir`                                                 |

---

### `parameters`

| Key                                  | Type                 | Must change the default value | Description                                                                                           |
|--------------------------------------|----------------------|------------------------------:|-------------------------------------------------------------------------------------------------------|
| `nodes`                              | `int`                |                             ✅ | Number of compute nodes for distributed training                                                      |
| `gpu_devices`                        | `int` or `list[int]` |                             ✅ | Number of GPUs or GPU indices to use                                                                  |
| `data.patch_size`                    | `list[int]`          |                             ✅ | 3D patch size used for patch creation + training                                                      |
| `data.patch_overlap`                 | `float`              |                             ✅ | Overlap fraction between patches (e.g. `0.5`)                                                         |
| `data.remove_only_background`        | `bool`               |                             ✅ | If `true`, discard patches containing only background labels                                          |
| `data.min`                           | `float`              |                             ❌ | Intensity minimum value for data scaling                                                              |
| `data.max`                           | `float`              |                             ❌ | Intensity maximum value for data scaling                                                              |
| `data.mean`                          | `float`              |                             ❌ | Mean used for normalization                                                                           |
| `data.std`                           | `float`              |                             ❌ | Std used for normalization                                                                            |
| `transforms.low_pass_sigma_range`    | `list[float]`        |                             ❌ | Range for Gaussian low-pass filtering                                                                 |
| `transforms.high_pass_sigma_range`   | `list[float]`        |                             ❌ | Range for high-pass filtering (sigma 1)                                                               |
| `transforms.high_pass_sigma2_range`  | `list[float]`        |                             ❌ | Range for high-pass filtering (sigma 2)                                                               |
| `transforms.noise_sigma_range`       | `list[float]`        |                             ❌ | Additive noise range                                                                                  |
| `transforms.combine_transforms`      | `bool`               |                             ❌ | Combine the blur, high pass and noise transforms in one pass                                          |
| `transforms.use_noisy_input`         | `bool`               |                             ❌ | If `true`, uses `noisy_data_folder` as transformation with input noisy                                |
| `transforms.scale_intensity_factors` | `list[float]`        |                             ❌ | Random intensity scaling factors                                                                      |
| `transforms.elastic`                 | `list[float]`        |                             ❌ | Elastic deformation magnitude range                                                                   |
| `transforms.zoom`                    | `list[float]`        |                             ❌ | Zoom range                                                                                            |
| `transforms.rotate`                  | `float`              |                             ❌ | Rotation range (radians)                                                                              |
| `transforms.flip`                    | `bool`               |                             ❌ | Random flips                                                                                          |
| `network.in_channels`                | `int`                |                             ❌ | Number of input channels (usually `1`)                                                                |
| `network.spatial_dims`               | `int`                |                             ❌ | Dimensionality of the model (`3` for tomograms)                                                       |
| `network.out_channels`               | `int`                |                             ❌ | Number of semantic classes                                                                            |
| `network.dense_dim`                  | `int`                |                             ❌ | Dense feature dimension (architecture-specific)                                                       |
| `network.filters`                    | `list[int]`          |                             ❌ | Convolution filter sizes                                                                              |
| `network.kernel_size`                | `int`                |                             ❌ | Convolution kernel size                                                                               |
| `network.padding`                    | `int`                |                             ❌ | Convolution padding                                                                                   |
| `network.distance_prediction`        | `bool`               |                             ❌ | If `false`, the training ignores the distance map loss when training (from distance maps from Step 3) |
| `network.use_dice_loss`              | `bool`               |                             ❌ | Use Dice loss in addition to standard cross-entropy loss                                              |
| `network.unfreeze_decoder`           | `bool`               |                             ✅ | Fine-tuning: unfreeze decoder layers                                                                  |
| `network.unfreeze_backbone`          | `bool`               |                             ✅ | Fine-tuning: unfreeze backbone/encoder layers                                                         |

---

### `hyper_parameters`

| Key            | Type    | Must change the default value | Description                                                      |
|----------------|---------|------------------------------:|------------------------------------------------------------------|
| `cache_rate`   | `float` |                             ❌ | Dataset cache rate (`0` = no caching)                            |
| `val_interval` | `int`   |                             ❌ | Validate every N epochs                                          |
| `batch_size`   | `int`   |                             ✅ | Number of patches per batch (default `10`).                      |
| `optimizer`    | `str`   |                             ❌ | Type of optimizer; `'sgd'`, `'adam'`, or `'adamw'`               |
| `lr`           | `float` |                             ❌ | Learning rate                                                    |
| `momentum`     | `float` |                             ❌ | Optimizer momentum (used for SGD/Adam variants where applicable) |
| `weight_decay` | `float` |                             ❌ | Weight decay                                                     |
| `max_epochs`   | `int`   |                             ❌ | Number of training epochs                                        |

---

## Next Steps

After training, continue with:

- [Semantic segmentation](semantic.md)
- [Particle identification](particle_identification.md)

For an overview of complete pipelines, see the [Usage overview](usage.md).