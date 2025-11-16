# CryoSiam Installation Guide

This guide provides instructions for installing CryoSiam using either pip or conda, with full support for GPU-enabled PyTorch.

---

## Requirements

- Python >= 3.8
- Conda (for environment-based setup)
- (Optional) NVIDIA GPU with CUDA 11.8+ drivers for GPU acceleration

---

## Step 1: Create the Environment

```bash
conda env create -f https://github.com/frosinastojanovska/cryosiam/blob/main/environment.yml
conda activate cryosiam
```

## Step 2: Install `cryosiam`

After activating the environment:

```bash
git clone https://github.com/frosinastojanovska/cryosiam.git
cd cryosiam
pip install --no-deps .
```

---

## Verify Installation

To verify the CryoSiam installation, run:

```bash
cryosiam --version
```

To check that PyTorch is installed correctly and can detect your GPU:

```python
import torch
print(torch.cuda.is_available())  # Should return True if GPU is available
```

---

## Troubleshooting

- **CUDA errors**: Make sure your system has the correct NVIDIA drivers and CUDA version.
- **Missing packages**: Double-check that you followed the correct conda install steps.
- **Conflicts**: It's recommended to use a clean conda environment.

---

## Need Help?

If you run into issues, feel free to open an issue on the [GitHub repository](https://github.com/frosinastojanovska/cryosiam/issues).

---


##  Don't Have Conda Installed?

If you don't have `conda` installed yet, we recommend using **Miniforge** (lightweight) or **Anaconda** (full-featured).

### Option 1: Install Miniforge (recommended)

Miniforge is a minimal installer for Conda that supports `conda-forge` by default.

**Linux / macOS:**

```bash
# Download and install Miniforge
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
```

**Windows:**

1. Download the latest installer from:  
   https://github.com/conda-forge/miniforge/releases

2. Run the `.exe` installer and follow the setup.

### Option 2: Install Anaconda (Full Distribution)

Anaconda includes Conda, Python, and hundreds of data science packages.

Download from: https://www.anaconda.com/products/distribution

---

After installing Miniforge or Anaconda, return to the [main Conda installation steps](#step-1-create_the_environment) above.
