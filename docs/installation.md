# CryoSiam Installation Guide


This guide walks you through installing **CryoSiam** in a clean and reproducible way using Conda.  
The recommended setup supports full support of **GPU** execution via PyTorch.

If you are new to Conda or Python environments, don't worry, each step is explained.

---

## Requirements

- Python 3.8 or newer
- Conda (Miniforge or Anaconda)
- An NVIDIA GPU with CUDA 11.8+ for GPU acceleration

---

## Step 1: Create the environment

CryoSiam uses a predefined Conda environment to ensure that all dependencies (including PyTorch) are installed with compatible versions. 
If you don't have Conda installed, please follow the guide from [here](#dont-have-conda-installed).

Run the following command to create the environment:

```bash
conda env create -f https://github.com/frosinastojanovska/cryosiam/blob/main/environment.yml
```

Once the environment is created, activate it:
```bash
conda activate cryosiam
```

## Step 2: Install `cryosiam`

With the Conda environment activated, clone the CryoSiam repository:

```bash
git clone https://github.com/frosinastojanovska/cryosiam.git
cd cryosiam
```

Install CryoSiam into the active environment:
```bash
pip install --no-deps .
```

> The `--no-deps` flag ensures that dependencies provided by Conda are used instead of reinstalling them with pip.

---

## Step 3: Verify the installation

To confirm that CryoSiam was installed correctly, run:

```bash
cryosiam --version
```
If the command runs without errors, the installation was successful.


To check that PyTorch is installed correctly and can detect your GPU:

```python
import torch
print(torch.cuda.is_available())  # Should return True if GPU is available
```

This should print `True` if a compatible GPU is available and the Pytorch for GPU was installed.

---

## Troubleshooting

- **CUDA errors**: Make sure your system has the correct NVIDIA drivers and CUDA version.
- **Missing packages**: Make sure the Conda environment is activated before installing or running CryoSiam.
- **Dependency conflicts**: Always install CryoSiam in a fresh Conda environment.

---

##  Don't Have Conda Installed?

If you do not have Conda yet, we recommend one of the following options.

### Option 1: Install Miniforge (recommended)

Miniforge is a minimal installer for Conda that uses `conda-forge` by default.

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

### Option 2: Install Anaconda (full distribution)

Anaconda includes Conda, Python, and hundreds of data scientific packages.

Download from: https://www.anaconda.com/products/distribution

---

After installing Miniforge or Anaconda, return to the [main Conda installation steps](#step-1-create_the_environment) above.

---

## Need Help?

If you run into issues, feel free to open an issue on the [CryoSiam GitHub repository](https://github.com/frosinastojanovska/cryosiam/issues).
