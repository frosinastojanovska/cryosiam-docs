# CryoSiam-Vis Installation Guide

This guide provides instructions for installing CryoSiam-Vis for easy visualization of the predictions from CryoSiam.

---

## Requirements

- Python >= 3.8
- Conda (optional, for environment-based setup)

---

## Option 1: Install via Conda Environment

You can also use Conda to create an isolated environment with all dependencies.

### Step 1: Create the Environment

```bash
conda env create -f https://github.com/frosinastojanovska/cryosiam_vis/blob/main/environment.yml
conda activate cryosiam_vis
```

### Step 2: Install `cryosiam_vis`

After activating the environment:

```bash
git clone https://github.com/frosinastojanovska/cryosiam_vis.git
cd cryosiam_vis
pip install --no-deps .
```

---

## Option 2: Install via pip

Clone the repository and install:

```bash
git clone https://github.com/frosinastojanovska/cryosiam_vis.git
cd cryosiam_vis

# Recommended: use a virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install the package and dependencies
pip install -r requirements.txt
pip install --no-deps .
```

---


## Verify Installation

To verify the CryoSiam-Vis installation, run:

```bash
cryosiam_vis --version
```

## Need Help?

If you run into issues, feel free to open an issue on the [GitHub repository](https://github.com/frosinastojanovska/cryosiam_vis/issues).

---


##  Don't Have Conda Installed?

If you don't have `conda` installed yet, check the instructions [here](installation.md/#dont-have-conda-installed)
