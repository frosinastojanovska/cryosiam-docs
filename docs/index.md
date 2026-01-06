# CryoSiam Documentation


<p align="center">
  <img src="images/CryoSiam_logo.svg" alt="CryoSiam" width="200">
</p>

CryoSiam is a self-supervised deep learning framework for cryo-electron tomography (cryo-ET) that enables 
denoising, semantic and instance segmentation, and particle identification without requiring ground-truth annotations.

Cryo-ET data are often challenging to analyze due to high noise levels and the lack of labeled training data. 
CryoSiam addresses these challenges by learning directly from simulated tomograms and predicting on real data.

This documentation is intended for both new and experienced users and provides step-by-step guidance on installing 
CryoSiam, running its analysis pipelines, and visualizing results.

---

## Key features

- Provided trained models with self-supervised learning
- Denoising and segmentation of cryo-ET tomograms  
- Support for semantic and instance segmentation workflows  
- Command-line driven and configurable pipelines  
- Integrated visualization of the outputs through cryosiam-vis  

---

## Getting started

If you are new to CryoSiam, we recommend starting here:

- [Installation](installation.md) – set up CryoSiam and its dependencies  
- [Usage](usage.md) – run denoising and segmentation workflows  
- [Tutorial](tutorial.md) – follow end-to-end examples on real data


For methodological details and validation, see the CryoSiam preprint: [CryoSiam: Self-Supervised Learning for Cryo-Electron Tomography](https://www.biorxiv.org/content/10.1101/2025.11.11.687379)

---

*Developed by Frosina Stojanovska in the Zaugg Lab at EMBL, in collaboration with the Mahamid Lab and the Kreshuk Lab.*
