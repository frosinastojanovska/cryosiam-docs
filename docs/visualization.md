# Visualization (CryoSiam‑Vis)

The **cryosiam-vis** package provides interactive visualization utilities (via **napari** and **dash**) for inspecting
results produced by CryoSiam:

- Denoised tomograms
- Semantic segmentation results
- Instance segmentation results (including filtered instances)
- Particle/coordinate sets from **.star** or **.csv** files
- Subtomogram embedding spaces (UMAP / clustering)

Most visualization commands open an interactive **napari** viewer, while embedding exploration uses a lightweight
**dash** interface.

---

## Install

Refer to the [CryoSiam‑Vis installation guide](installation_vis.md).

---

## Command overview

All visualizations are accessed through a single CLI with **subcommands**:

```bash
cryosiam_vis --version
cryosiam_vis <subcommand> [arguments]
```

> **Important:**
> All visualization commands reuse the **same configuration files** used during CryoSiam inference.
> No separate visualization configuration is required.

### Available subcommands

- `visualize_denoising` – visualize denoised vs raw tomograms
- `visualize_semantic` – overlay semantic segmentation predictions
- `visualize_instance` – visualize instance segmentation results
- `visualize_filtered_instance` – visualize filtered instance segmentation results
- `visualize_coordinates` – plot particle coordinates from `.star` / `.csv` files
- `visualize_embeddings` – interactive embedding visualization
- `visualize_embeddings_clusters` – embedding visualization with clustering labels

---

## `visualize_denoising`

Visualize the denoised tomogram produced by `cryosiam denoise_predict` and optionally compare it to the raw input.

### Usage

```bash
cryosiam_vis visualize_denoising --config_file configs/denoise.yaml --filename TS_01.mrc
```

### Arguments

| Argument        | Required | Description                                             |
|-----------------|:--------:|---------------------------------------------------------|
| `--config_file` |    ✅     | The **same YAML** used with `cryosiam denoise_predict`. |
| `--filename`    |    ✅     | Tomogram filename (including extension).                |

### What it loads

- Raw tomogram from `data_folder`
- Denoised tomogram from `prediction_folder`
- Opens both as layers in **napari** for side‑by‑side inspection

---

## `visualize_semantic`

Overlay semantic segmentation predictions on the denoised tomogram.

### Usage

```bash
cryosiam_vis visualize_semantic --config_file configs/semantic.yaml --filename TS_01.mrc
```

### Arguments

| Argument        | Required | Description                                              |
|-----------------|:--------:|----------------------------------------------------------|
| `--config_file` |    ✅     | The **same YAML** used with `cryosiam semantic_predict`. |
| `--filename`    |    ✅     | Tomogram filename (including extension).                 |

### What it loads

- Denoised tomogram (`data_folder`)
- Semantic probabilities and/or segmentation masks (`prediction_folder`)
- Lamella mask (if `mask_folder` is defined)

---

## `visualize_instance`

Visualize instance segmentation volumes and labels.

### Usage

```bash
cryosiam_vis visualize_instance --config_file configs/instance.yaml --filename TS_01.mrc
```

### Arguments

| Argument        | Required | Description                                              |
|-----------------|:--------:|----------------------------------------------------------|
| `--config_file` |    ✅     | The **same YAML** used with `cryosiam instance_predict`. |
| `--filename`    |    ✅     | Tomogram filename (including extension).                 |

### What it loads

- Denoised tomogram (`data_folder`)
- Instance segmentation masks from `prediction_folder`

---

## `visualize_filtered_instance`

Visualize **filtered** instance segmentation results produced by `cryosiam instance_filter`.

### Usage

```bash
cryosiam_vis visualize_filtered_instance --config_file configs/instance_filter.yaml --filename TS_01.mrc
```

### Arguments

| Argument        | Required | Description                                    |
|-----------------|:--------:|------------------------------------------------|
| `--config_file` |    ✅     | The YAML used with `cryosiam instance_filter`. |
| `--filename`    |    ✅     | Tomogram filename (including extension).       |

### What it loads

- Denoised tomogram
- Filtered instance masks

---

## `visualize_coordinates`

Plot particle coordinates from a **.star** or **.csv** file onto a tomogram.

### Usage

```bash
cryosiam_vis visualize_coordinates --config_file configs/semantic_particle.yaml --filename TS_01.mrc --point_size 15
```

### Arguments

| Argument        | Required | Default | Description                                           |
|-----------------|:--------:|:-------:|-------------------------------------------------------|
| `--config_file` |    ✅     |    –    | Config file used for CryoSiam particle identification |
| `--filename`    |    ✅     |    –    | Tomogram filename (including extension)               |
| `--point_size`  |    ❌     |  `15`   | Marker size for plotted points                        |

### Notes

- Coordinates are read from the **.star** or **.csv** file specified in the configuration.
- Coordinates must be in **voxel space** and aligned with the tomogram.

---

## `visualize_embeddings`

Open an interactive embedding viewer for exploratory analysis.

### Usage

```bash
cryosiam_vis visualize_embeddings --config_file configs/subtomo_embeddings.yaml
```

### What it does

- Loads embedding projections produced by the subtomogram embeddings module
- Opens an interactive **dash** interface for UMAP/PCA exploration

---

## `visualize_embeddings_clusters`

Visualize embedding projections with cluster assignments.

### Usage

```bash
cryosiam_vis visualize_embeddings_clusters --config_file configs/subtomo_embeddings.yaml --clustering kmeans
```

### Arguments

| Argument        | Required | Description                                                 |
|-----------------|:--------:|-------------------------------------------------------------|
| `--config_file` |    ✅     | Configuration file used for embedding generation/clustering |
| `--clustering`  |    ✅     | Clustering type: `kmeans` or `spectral`                     |

---

## See also

- [CryoSiam usage overview](usage.md)
