# Workflows

CryoSiam workflows are modular pipelines designed to process cryo-electron tomography (cryo-ET) data step by step.  
Each workflow consists of one or more modules, each addressing a specific task such as denoising or segmentation.

You can use individual modules independently or combine them into complete analysis pipelines depending on your
scientific question.

For methodological details, please refer to
the [CryoSiam preprint](https://www.biorxiv.org/content/10.1101/2025.11.11.687379).

---

## How to choose a workflow

If you are new to CryoSiam, consider the following guidelines:

- Use [Workflow 1](#workflow-1) for general denoising and semantic segmentation (or particle identification) of
  tomograms
- Use [Workflow 1b](#workflow-1b) to train a new semantic segmentation model
- Use [Workflow 2](#workflow-2) when individual instances of structures need to be separated
- Use [Workflow 3](#workflow-3) for downstream analysis of subtomograms such as clustering or similarity comparisons

---

## :octicons-workflow-16: Workflow 1

### Denoising → Semantic Segmentation / Particle Identification

This workflow is suitable for interpreting cellular context or detecting specific structures in cryo-ET data.

1. **Denoising**  
   Reduce noise in the raw tomogram while preserving structural features.

2. **Semantic Segmentation**  
   Assign each voxel to a biological class (e.g., membranes, filaments, complexes).

   **Alternative:**  
   **Particle Identification** can be used directly after denoising to locate specific particles of interest.

See detailed documentation:

- [Denoising](denoising.md)
- [Semantic Segmentation](semantic.md)
- [Particle Identification](particle_identification.md)

---

## :octicons-workflow-16: Workflow 1b

### Denoising → Semantic Segmentation (Training)

This workflow is used to train a semantic segmentation model on cryo-ET data using CryoSiam’s self-supervised pretrained
weights and user-provided ground truth data.

It is intended for users who want to adapt CryoSiam to new datasets or biological contexts.

1. **Denoising**  
   Preprocess raw tomograms to improve signal quality.

2. **Semantic Segmentation (Training)**  
   Train (fine-tune) a semantic segmentation model using the self-supervised trained weights as starting point.

After training, the resulting model can be used for inference as described in **Workflow 1**.

See detailed documentation:

- [Denoising](denoising.md)
- [Semantic Segmentation – Training](semantic_training.md)

---

## :octicons-workflow-16: Workflow 2

### Denoising → Instance Segmentation

This workflow focuses on separating individual particles.

1. **Denoising**  
   Improve signal quality for reliable downstream processing.

2. **Instance Segmentation**  
   Identify and separate individual instances of particles.

See detailed documentation:

- [Denoising](denoising.md)
- [Instance Segmentation](instance.md)

---

## :octicons-workflow-16: Workflow 3

### Denoising → (Optional) Instance Segmentation → Subtomogram Embeddings

This workflow enables quantitative downstream analysis of extracted subtomograms.

1. **Denoising**  
   Preprocess tomograms for improved structural information.

2. **Instance Segmentation (optional)**  
   Detect and separate candidate particles.

   **Alternative:**  
   Instance segmentation can be skipped if particle centers are provided;
   see  [Subtomogram Embeddings](subtomogram_embeddings.md) for details.

3. **Subtomogram Embeddings**  
   Represent particle subtomograms as feature vectors for clustering, comparison, or further analysis.

See detailed documentation:

- [Denoising](denoising.md)
- [Instance Segmentation](instance.md)
- [Subtomogram Embeddings](subtomogram_embeddings.md)

---

## Trained Models

Pre-trained models available for CryoSiam workflows are described here:

- [Trained Models](trained_models.md)

---

## Running a workflow

Each CryoSiam module is configured using a YAML configuration file that defines inputs, outputs, and model parameters.

To run a module, use:

```bash
cryosiam <module name> --config_file configs/<module>.yaml
```

Detailed explanations of configuration options are provided on the documentation page of each module.

---

## Running CryoSiam at Scale (Slurm / HPC)

CryoSiam prediction commands run on **one GPU per process**.
To efficiently process large datasets on HPC systems, predictions should be
**parallelized across tomograms using Slurm job submission**.

Instead of processing all tomograms in a single command, CryoSiam allows
processing **one tomogram per job** via the `--filename` argument.

This enables:

- Full utilization of multi-GPU clusters
- Independent job retries
- Clean scaling to hundreds of tomograms

For more details see: [Scaling to large datasets on HPC](slurm_parallel.md)