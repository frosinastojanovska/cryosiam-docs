# Processing Functions

CryoSiam provides **standalone preprocessing utilities** to prepare tomograms and label masks for training and
downstream workflows. These commands are independent of the main CryoSiam prediction/training modules and are useful
when you need to:

- Enforce CryoSiam’s expected intensity convention (**scaled to [0,1]**, typically **white-on-black**)
- Generate **binary** or **multi-class** training masks from particle coordinates
- Create masks from **STA** average maps + orientations
- Combine per-class binary masks into a single multi-class label volume

---

## Overview

| Command                                          | What it does                                                          | Typical use                                                  |
|--------------------------------------------------|-----------------------------------------------------------------------|--------------------------------------------------------------|
| `processing_invert_scale`                        | Invert and/or scale tomogram intensities (optionally percentile clip) | Prepare external tomograms for training / pretrained weights |
| `processing_create_sphere_mask`                  | Create **binary** sphere masks from particle centers                  | Weak labels / simulated labels                               |
| `processing_create_sphere_mask_multiclass`       | Create **multi-class** sphere masks from class-labeled centers        | Multi-class weak labels                                      |
| `processing_create_binary_map_after_sta`         | Place a thresholded STA map into tomograms using orientations         | Shape-aware labels from STA                                  |
| `processing_create_multiclass_from_binary_masks` | Merge multiple binary masks into one multi-class mask                 | Combine separate labels into one semantic mask               |

---

## `processing_invert_scale`

Invert and/or scale tomogram intensities.

### When to use

- Your tomograms come from a different pipeline and are **not** in the expected **[0,1]** range.
- You need to enforce **white-on-black** contrast (high density → high intensity).
- You want to clip outliers using percentiles before scaling.

### Command

```bash
cryosiam processing_invert_scale --input_path /path/to/tomograms_or_one_tomogram.mrc --output_path /path/to/output_folder_or_output_file.mrc --invert --lower_end_percentage 0.1 --upper_end_percentage 99.9
```

### Parameters

| Argument                 |    Type | Required | Description                                                               |
|--------------------------|--------:|:--------:|---------------------------------------------------------------------------|
| `--input_path`           |   `str` |    ✅     | Path to an input tomogram **file** or a **folder** containing tomograms   |
| `--output_path`          |   `str` |    ✅     | Output file path (if input is file) or output folder (if input is folder) |
| `--invert`               |  `flag` |    ❌     | If set, invert intensities (contrast inversion)                           |
| `--lower_end_percentage` | `float` |    ✅     | Percentile cutoff for lower-end clipping (e.g. `0.1`)                     |
| `--upper_end_percentage` | `float` |    ✅     | Percentile cutoff for upper-end clipping (e.g. `99.9`)                    |

### Notes

- If your data is already white-on-black, omit `--invert` and use only scaling/clipping.
- Percentile clipping is recommended for tomograms.

---

## `processing_create_sphere_mask`

Create a **binary** tomogram mask by placing spheres at particle center coordinates.

### When to use

- You want **weak labels** from coordinate picks (manual / particle identification / external picker).
- You want to generate **synthetic semantic masks** (single class) for training.

### Command (all tomograms referenced by the coordinates file)

```bash
cryosiam processing_create_sphere_mask --coordinates_file /path/to/particles.star --sphere_radius 6 --output_dir /path/to/output_masks --tomogram_path /path/to/tomograms
```

### Command (process only one tomogram)

```bash
cryosiam processing_create_sphere_mask --coordinates_file /path/to/particles.csv --sphere_radius 6 --output_dir /path/to/output_masks --tomogram_path /path/to/tomograms --tomo_name TS_01.mrc
```

### Parameters

| Argument             |  Type | Required | Description                                                                                              |
|----------------------|------:|:--------:|----------------------------------------------------------------------------------------------------------|
| `--coordinates_file` | `str` |    ✅     | STAR or CSV file containing particle centers                                                             |
| `--sphere_radius`    | `int` |    ✅     | Sphere radius in **voxels**                                                                              |
| `--output_dir`       | `str` |    ✅     | Output directory for generated masks                                                                     |
| `--tomogram_path`    | `str` |    ✅     | Path to a tomogram **folder** (preferred) or a single tomogram file used to determine output volume size |
| `--tomo_name`        | `str` |    ❌     | Process only one tomogram (must match `rlnMicrographName` in STAR or `tomo` in CSV)                      |

### Coordinate file formats

**STAR file**

- Must contain headers: `rlnCoordinateX`, `rlnCoordinateY`, `rlnCoordinateZ`
- Tomogram association is typically via `rlnMicrographName`

**CSV file**

- Must contain headers (z, y, x order): `centroid-0`, `centroid-1`, `centroid-2`
- Tomogram association via column `tomo` (recommended)

> Coordinates are expected in **voxel space**.

---

## `processing_create_sphere_mask_multiclass`

Create a **multi-class** mask by placing spheres at centers, using class labels and class-specific radii.

### When to use

- You have multiple particle types and want a **multi-class** semantic training mask.
- You want different radii per class (to reflect expected particle sizes).

### Command

```bash
cryosiam processing_create_sphere_mask_multiclass --coordinates_file /path/to/particles_with_classes.star --sphere_radius 5,7,9 --output_dir /path/to/output_masks --tomogram_path /path/to/tomograms
```

### Parameters

| Argument             |  Type | Required | Description                                                                    |
|----------------------|------:|:--------:|--------------------------------------------------------------------------------|
| `--coordinates_file` | `str` |    ✅     | STAR or CSV file containing centers **and** class labels                       |
| `--sphere_radius`    | `str` |    ✅     | Comma-separated radii (N integers) where N = number of classes (e.g. `5,7,9`)  |
| `--output_dir`       | `str` |    ✅     | Output directory for generated masks                                           |
| `--tomogram_path`    | `str` |    ✅     | Tomogram folder (preferred) or single tomogram to determine output volume size |
| `--tomo_name`        | `str` |    ❌     | Process only one tomogram                                                      |

### Required columns

**STAR file**

- Coordinates: `rlnCoordinateX`, `rlnCoordinateY`, `rlnCoordinateZ`
- Class: `rlnClassNumber`
- Class numbering must start at **1** and be **sequential**.

**CSV file**

- Coordinates: `centroid-0`, `centroid-1`, `centroid-2` (z, y, x)
- Class: `semantic_class`
- Class numbering must start at **1** and be **sequential**.

---

## `processing_create_binary_map_after_sta`

Create a **binary** tomogram mask by placing a thresholded STA average map into a tomogram using particle orientations.

### When to use

- You have a STA pipeline that produced:
    - a STAR file with orientations, and
    - an average map (subvolume) representing the particle
- You want shape-aware labels rather than spheres.

### Command

```bash
cryosiam processing_create_binary_map_after_sta --star_file /path/to/sta_orientations.star --map_file /path/to/average_map.mrc --map_threshold 0.15 --output_dir /path/to/output_masks --example_tomogram /path/to/example_tomogram.mrc
```

### Parameters

| Argument             |    Type | Required | Description                                                           |
|----------------------|--------:|:--------:|-----------------------------------------------------------------------|
| `--star_file`        |   `str` |    ✅     | STAR file with orientations after STA                                 |
| `--map_file`         |   `str` |    ✅     | Map inserted into the tomogram (expected to be a **cubic subvolume**) |
| `--output_dir`       |   `str` |    ✅     | Output directory for generated masks                                  |
| `--map_threshold`    | `float` |    ✅     | Threshold used to binarize the inserted map                           |
| `--example_tomogram` |   `str` |    ✅     | A reference tomogram used to determine output 3D size                 |
| `--tomo_name`        |   `str` |    ❌     | Process only one tomogram (name must match `rlnMicrographName`)       |

---

## `processing_create_multiclass_from_binary_masks`

Combine multiple binary masks (one per class) into a single **multi-class** semantic mask.

### Expected folder structure

`root_binary_masks_folder` must contain one subfolder per class label, each holding binary masks for that label:

```
root_binary_masks_folder/
├── class_1/
│ ├── TS_01.mrc
│ └── TS_02.mrc
├── class_2/
│ ├── TS_01.mrc
│ └── TS_02.mrc
└── class_3/
  ├── TS_01.mrc
  └── TS_02.mrc
```

### Command

```bash
cryosiam processing_create_multiclass_from_binary_masks --root_binary_masks_folder /path/to/root_binary_masks_folder --output_dir /path/to/output_multiclass_masks
```

### Parameters

| Argument                     |  Type | Required | Description                                              |
|------------------------------|------:|:--------:|----------------------------------------------------------|
| `--root_binary_masks_folder` | `str` |    ✅     | Folder containing per-class subfolders with binary masks |
| `--output_dir`               | `str` |    ✅     | Output directory for multi-class masks                   |
| `--tomo_name`                | `str` |    ❌     | Process only one tomogram (include file extension)       |

---

## Notes and best practices

- All coordinates are expected in **voxel space**.
- Always visually inspect a few outputs before training.
- These tools are commonly used in:
    - [Semantic segmentation training](semantic_training.md) (Steps 1–3)
    - [Particle identification](particle_identification.md)
    - [Subtomogram embeddings](subtomogram_embeddings.md) (center-based inputs)

---

## Next steps

- [Semantic segmentation training](semantic_training.md)
- [Usage overview](usage.md)
