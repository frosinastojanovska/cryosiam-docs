# Trained Models

CryoSiam is distributed with a collection of **pretrained models** that enable out-of-the-box denoising, segmentation,
particle identification, and subtomogram embedding generation for cryo-electron tomography (cryo-ET) data.

All pretrained models are hosted on **Hugging Face** and can be downloaded individually as `.ckpt` files.
Using pretrained weights is strongly recommended for inference and for fine-tuning on new datasets, as it significantly
reduces training time and improves convergence.

> **Model repository:**  
> All models listed below are available at  
> https://huggingface.co/frosinastojanovska/cryosiam_v1.0

---

## Overview of available models

| Model file                                                                                                                                              | Task                        | Used in module          | Description                                                                                                             |
|---------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|-------------------------|-------------------------------------------------------------------------------------------------------------------------|
| [cryosiam_denoising.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_denoising.ckpt)                                    | Denoising                   | Denoising               | Self-supervised denoising model trained on simulated WBP tomograms to reduce noise while preserving structural details. |                                                                                                            
| [cryosiam_lamella.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_lamella.ckpt)                                        | Lamella prediction          | Semantic segmentation   | Binary segmentation model for identifying lamella regions and suppressing false positives outside the lamella.          |                                                                                                            
| [cryosiam_semantic_segmentation.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_semantic_segmentation.ckpt )           | Semantic segmentation       | Semantic segmentation   | Multi-class voxel-wise semantic segmentation of biological structures in denoised tomograms.                            |      
| [cryosiam_ribosome_segmentation.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_ribosome_segmentation.ckpt)            | Semantic segmentation       | Semantic segmentation   | Specialized semantic model trained for ribosome segmentation.                                                           |       
| [cryosiam_semantic_myco_candidates.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_semantic_myco_candidates.ckpt)      | Particle identification     | Particle identification | Semantic model trained to predict candidate macromolecular complexes for particle picking.                              |
| [cryosiam_instance.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_instance.ckpt)                                      | Instance segmentation       | Instance segmentation   | Instance segmentation model for separating individual macromolecular complexes.                                         |                    |
| [cryosiam_instance_convex_hull.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_instance_convex_hull.ckpt )             | Instance segmentation       | Instance segmentation   | Variant of the instance model using convex-hull–based instance representations.                                         | 
| [dense_simsiam_pretrained.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/dense_simsiam_pretrained.ckpt)                        | Self-supervised pretraining | Semantic training       | Dense SimSiam backbone pretrained for initializing semantic segmentation training.                                      |             
| [simsiam_embeds_denoised_convex_hull.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/simsiam_embeds_denoised_convex_hull.ckpt ) | Subtomogram embeddings      | Subtomogram embeddings  | Embedding model using convex-hull masking (`masking_type: 1`), recommended default for instance-based embeddings.       | 
| [simsiam_embeds_denoised_no_masking.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/simsiam_embeds_denoised_no_masking.ckpt)    | Subtomogram embeddings      | Subtomogram embeddings  | Embedding model without masking (`masking_type: 0`), **required** for center-based embeddings.                          |   
| [simsiam_embeds_denoised_strict.ckpt](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/simsiam_embeds_denoised_strict.ckpt)            | Subtomogram embeddings      | Subtomogram embeddings  | Embedding model with strict instance masking (`masking_type: 2`) for isolating internal object structure.               |       

---

## Choosing the right model

### Denoising

Use `cryosiam_denoising.ckpt` to preprocess raw WBP tomograms before any downstream task.

### Semantic segmentation

- Use `cryosiam_semantic_segmentation.ckpt` for general multi-class segmentation.
- Use `cryosiam_ribosome_segmentation.ckpt` for ribosome-focused workflows.
- Always combine with `cryosiam_lamella.ckpt` for lamella masking when available.

### Particle identification

Use `cryosiam_semantic_myco_candidates.ckpt` to predict candidate regions and extract particle centers.

### Instance segmentation

Use `cryosiam_instance.ckpt` for general instance segmentation.
The convex hull variant can be used for alternative instance representations.

### Subtomogram embeddings

Select the embedding model **based on the masking strategy**:

- `masking_type: 0` → `simsiam_embeds_denoised_no_masking.ckpt` (**required for center-based embeddings**)
- `masking_type: 1` → `simsiam_embeds_denoised_convex_hull.ckpt` (recommended default)
- `masking_type: 2` → `simsiam_embeds_denoised_strict.ckpt`

> **Important:**  
> When using `simsiam_embeddings_from_centers_predict`, you **must** use the **no-masking** embedding model
> and provide a patch size close to the expected particle size to avoid embedding background signal.

---

## Using pretrained models

After downloading a model checkpoint, reference it in your configuration file:

```yaml
trained_model: /path/to/model.ckpt
```

CryoSiam will automatically load the weights and configure the network accordingly.

---

## Fine-tuning and reproducibility

- All models can be **fine-tuned** on custom datasets using the semantic segmentation training pipeline.
- For reproducibility, record:
    - model filename
    - CryoSiam version
    - configuration file used for inference or training

---

## Citation

If you use CryoSiam pretrained models in your work, please cite:

**Stojanovska et al.**  
*CryoSiam: self-supervised representation learning for automated cryo-ET analysis*  
bioRxiv (2025)

---
## Contributing trained models

If you have trained your own semantic segmentation model using CryoSiam and would like to make it available to the
community, you are very welcome to contribute it.

Please contact Frosina Stojanovska with:
- a short description of the model (task, classes, data type),
- the CryoSiam version used,
- and a link to the trained checkpoint.

After review, the model can be added to the official Hugging Face repository and listed on this page.
