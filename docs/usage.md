# Workflows

CryoSiam workflows are designed to process cryo-electron tomography (CryoET) data step by step. Each module focuses on a specific task, and you can combine them into analysis pipelines depending on your scientific question.

---

## :octicons-workflow-16: Workflow 1: Denoising → Semantic Segmentation / Particle Identification

1. **Denoising**  
   Clean the raw tomogram to reduce noise while preserving structural details.  
2. **Semantic Segmentation**  
   Classify each voxel into biological classes such as membranes, filaments, or complexes.  
   *Alternative:* **Particle Identification** can be used to locate specific particles of interest directly after denoising.

See details in [Denoising](denoising.md), [Semantic Segmentation](semantic.md), and [Particle Identification](particle_identification.md).

---

## :octicons-workflow-16: Workflow 2: Denoising → Instance Segmentation

1. **Denoising**  
   Prepare a cleaner tomogram for reliable downstream processing.  
2. **Instance Segmentation**  
   Separate individual structures even when they overlap or belong to the same class.

See details in [Denoising](denoising.md) and [Instance Segmentation](instance.md).

---

## :octicons-workflow-16: Workflow 3: Denoising → Instance Segmentation → Subtomogram Embeddings

1. **Denoising**  
   Preprocess tomograms for structural clarity.  
2. **Instance Segmentation**  
   Extract and separate candidate subtomograms.  
3. **Subtomogram Embeddings**  
   Represent subtomograms as feature vectors for clustering, comparison, or downstream analysis.

See details in [Denoising](denoising.md), [Instance Segmentation](instance.md), and [Subtomogram Embeddings](subtomogram_embeddings.md).

---

## Configuration Files

Each module requires a YAML configuration file defining inputs, outputs, and model parameters.  
You can run a module as:

```bash
cryosiam <module> --config_file=configs/<module>.yaml
```

Explanation of the YAML configuration files is given into the specific documentation page. 