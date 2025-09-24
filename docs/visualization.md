# Visualization (CryoSiam‑Vis)

The **cryosiam-vis** package provides interactive visualization utilities (via **napari** and **dash**) for results
produced by CryoSiam:

- Denoised tomograms
- Semantic segmentation results
- Instance segmentation results
- Particle/coordinate sets from **.star** files
- Embedding UMAP spaces

---

## Install

Refer to the [CryoSiam-Vis installation guide](installation_vis.md). 

---

## Command Overview

All visualizations are accessed through a single CLI with **subcommands**:

```bash
cryosiam-vis --version
cryosiam-vis <subcommand> [arguments]
```

Available subcommands:

- `visualize_denoising` — show **denoised** vs **raw** tomogram
- `visualize_semantic` — overlay **semantic** masks on tomogram
- `visualize_instance` — explore **instance** masks
- `visualize_coordinates` — plot points from a **.star** file on a tomogram
- `visualize_embeddings` — open an interactive embedding explorer

---

## `visualize_denoising`

Show the denoised tomogram produced by `cryosiam denoise` (and optionally compare to raw).

### Usage

```bash
cryosiam-vis visualize_denoising --config_file configs/denoise.yaml --filename TS_01.mrc
```

### Arguments

| Arg             | Required | Description                                                                              |
|-----------------|:--------:|------------------------------------------------------------------------------------------|
| `--config_file` |    ✅     | The **same YAML** used with `cryosiam denoise_predict` (paths are read from it).         |
| `--filename`    |    ✅     | Tomogram filename (including extension), must exist under `data_folder` from the config. |

### What it loads

- Input tomogram from `data_folder`
- Denoised output from `prediction_folder`
- Opens layers in **napari** for side‑by‑side inspection

---

## `visualize_semantic`

Overlay semantic predictions over the denoised tomogram.

### Usage

```bash
cryosiam-vis visualize_semantic --config_file configs/semantic.yaml --filename TS_01.mrc
```

### Arguments

| Arg             | Required | Description                                                                                                           |
|-----------------|:--------:|-----------------------------------------------------------------------------------------------------------------------|
| `--config_file` |    ✅     | The **same YAML** used with `cryosiam semantic_predict`. Reads `data_folder`, `mask_folder`, and `prediction_folder`. |
| `--filename`    |    ✅     | Tomogram filename (including extension).                                                                              |

### What it loads

- Denoised tomogram (`data_folder`)
- Semantic **probabilities** and/or **segmentation** (`prediction_folder`)

---

## `visualize_instance`

Visualize instance segmentation volumes and labels.

### Usage

```bash
cryosiam-vis visualize_instance --config_file configs/instance.yaml --filename TS_01.mrc
```

### Arguments

| Arg             | Required | Description                                              |
|-----------------|:--------:|----------------------------------------------------------|
| `--config_file` |    ✅     | The **same YAML** used with `cryosiam instance_predict`. |
| `--filename`    |    ✅     | Tomogram filename (including extension).                 |

### What it loads

- Denoised tomogram (`data_folder`)
- Instance masks/labels from the prediction folder defined in the config

---

## `visualize_coordinates`

Plot particle coordinates from a **.star** file onto a tomogram.

### Usage

```bash
cryosiam-vis visualize_coordinates --tomo_path /path/to/denoised --filename TS_01.mrc --star_file /path/to/coordinates.star --point_size 15
```

### Arguments

| Arg            | Required | Default | Description                                        |
|----------------|:--------:|--------:|----------------------------------------------------|
| `--tomo_path`  |    ✅     |       — | Folder containing tomograms (ideally denoised).    |
| `--filename`   |    ✅     |       — | Tomogram filename (with extension).                |
| `--star_file`  |    ✅     |       — | Path to the **.star** file with point coordinates. |
| `--point_size` |    ❌     |    `15` | Point marker size in napari.                       |

### Notes

- Make sure the voxel size / coordinate system used to generate the **.star** matches the tomogram voxel size.
- Supports common STAR conventions via `starfile`.

---

## `visualize_embeddings`

Open the embedding viewer (for exploratory analysis).

### Usage

```bash
cryosiam-vis visualize_embeddings
```

### What it does

- Loads embedding UMAP (if present in the working directory or a default location used by your inference)
- Opens an interactive explorer (2D UMAP projection, selection, metadata overlays)

---

## See also

- [CryoSiam usage overview](usage.md)
