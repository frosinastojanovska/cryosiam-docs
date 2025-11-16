# Pretrained CryoSiam Models

CryoSiam provides several pretrained models hosted on Hugging Face.  
Each model corresponds to one stage of the CryoSiam processing pipeline and can be used directly in the configuration
files.

Below is a list of all available pretrained models and what tasks they perform.

---

## Available Pretrained Models

| Model Name                                                                                                            | Task                                                 | Description                                                                                                                                       |
|-----------------------------------------------------------------------------------------------------------------------|------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| [**`cryosiam-denoising`**](https://huggingface.co/frosinastojanovska/cryosiam_v1.0/blob/main/cryosiam_denoising.ckpt) | **Denoising**                                        | Trained to remove noise on simulated data and can be directly applied to real cryo-ET data.                                                       |
| **`cryosiam-lamella`**                                                                                                | **Lamella Mask Prediction**                          | Predicts lamella vs. non-lamella regions. Highly recommended before semantic/instance segmentation to remove false positives outside the lamella. |
| **`cryosiam-semantic`**                                                                                               | **Semantic Segmentation (General Classes)**          | Predicts voxel-level classes: **membrane, particles, microtubules, actin, filaments (DNA/RNA)**.                                                  |
| **`cryosiam-semantic-targeted`**                                                                                      | **Semantic Segmentation (Target Complex)**           | Specialized model trained to detect a *specific macromolecular complex* described in the paper.                                                   |
| **`cryosiam-instance`**                                                                                               | **Instance Segmentation**                            | Predicts individual object instances using center-distance and boundary maps. Produces separated masks for each particle.                         |
| **`cryosiam-simsiam-embeddings`**                                                                                     | **Subtomogram Embeddings / Representation Learning** | Generates 1024-dim embeddings for each instance using a self-supervised SimSiam model. Enables clustering and structural discovery.               |
