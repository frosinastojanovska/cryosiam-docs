# Parallel Inference on Slurm Clusters

CryoSiam inference commands (e.g. denoising, semantic segmentation, instance segmentation, embeddings)
are executed on **a single GPU per command-line call**.

When processing **many tomograms**, the recommended way to scale inference on HPC systems
is to **parallelize over tomograms**, running *one tomogram per Slurm job*.

This page explains how to do this safely and efficiently using **Slurm job arrays or batched submissions**.

---

## Key idea

- Each CryoSiam inference command uses **1 GPU**
- Parallelism is achieved by:
  - Splitting tomograms by filename
  - Submitting one Slurm job per tomogram
- The `--filename` argument is used to restrict each job to **a single tomogram**

This approach avoids GPU contention and scales linearly with the number of available GPUs.

---

## Typical use cases

- Denoising a large dataset
- Semantic segmentation over many tomograms
- Instance segmentation at scale
- Subtomogram embeddings generation

---

### Step 1: Prepare a submission script

Below is an example Slurm submission script that runs **instance segmentation**
on a **single tomogram** passed as a command-line argument.

Save this as `run_instance_predict.sbatch`:

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks=16
#SBATCH --mem=50G
#SBATCH --time=1-00:00:00
#SBATCH --gres=gpu:1
#SBATCH --job-name="cryosiam_instance_predict"
#SBATCH -o /scratch/dataset/slurm.%N.%j.out
#SBATCH -e /scratch/dataset/slurm.%N.%j.err

echo "Starting CryoSiam instance prediction"
echo "Tomogram: $1"

cd /scratch/dataset/scripts
conda activate cryosiam

cryosiam instance_predict \
  --config_file=config_dense_simsiam_instance.yaml \
  --filename=$1

echo "Done."
```

---

### Step 2: Submit jobs for all tomograms in a folder

Assuming all tomograms are located in:

```text
/scratch/dataset/data/
```

and have extension `.mrc`, you can submit one job per tomogram using:

```bash
for f in /scratch/dataset/data/*.mrc; do
    fname=$(basename "$f")
    sbatch run_instance_predict.sbatch "$fname"
done
```

---

## Job arrays (recommended for large datasets)

If you have **hundreds to thousands** of tomograms, a Slurm **job array** is often cleaner than submitting many
individual jobs.

The idea is:

1. Create a text file containing one filename per line (e.g. `tomograms.txt`)
2. Submit an array job where each task processes one line from that file
3. Use `--array=0-(N-1)` to match the number of tomograms

---

### Step 1: Create a file list (`tomograms.txt`)

From a folder containing `.mrc` tomograms:

```bash
ls -1 /scratch/dataset/data/*.mrc | xargs -n 1 basename > tomograms.txt
```

This produces a plain text file like:

```text
TS_01.mrc
TS_02.mrc
TS_03.mrc
...
```

---

### Step 2: Array sbatch script

Save this as `run_instance_predict_array.sbatch`:

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks=16
#SBATCH --mem=50G
#SBATCH --time=1-00:00:00
#SBATCH --gres=gpu:1
#SBATCH --job-name="cryosiam_instance_predict"
#SBATCH -o /scratch/dataset/slurm.%A_%a.out
#SBATCH -e /scratch/dataset/slurm.%A_%a.err
#SBATCH --array=0-99

set -euo pipefail

LIST_FILE=/scratch/dataset/scripts/tomograms.txt
TOMO=$(sed -n "$((SLURM_ARRAY_TASK_ID+1))p" "$LIST_FILE")

echo "Array task: $SLURM_ARRAY_TASK_ID"
echo "Tomogram: $TOMO"

cd /scratch/dataset/scripts
conda activate cryosiam

cryosiam instance_predict \
  --config_file=config_dense_simsiam_instance.yaml \
  --filename="$TOMO"

echo "Done."
```

**Important:** replace `--array=0-99` with the correct range for your dataset.
For example, if `tomograms.txt` has 237 lines, use `--array=0-236`.

---

### Step 3: Submit the array job

```bash
# Count tomograms
N=$(wc -l < tomograms.txt)

# Submit array using 0..N-1
sbatch --array=0-$((N-1)) run_instance_predict_array.sbatch
```

---

### (Optional) Limit the number of concurrent GPUs

Many clusters allow a concurrency cap:

```bash
sbatch --array=0-$((N-1))%20 run_instance_predict_array.sbatch
```

This example runs **at most 20 jobs at once**, preventing overwhelming the scheduler or filesystem.

---

### Adapting job arrays to other modules

Replace the command in the script with the module you need, for example:

- Denoising:
  ```bash
  cryosiam denoise_predict --config_file=config_denoising.yaml --filename="$TOMO"
  ```
- Semantic segmentation:
  ```bash
  cryosiam semantic_predict --config_file=config_semantic.yaml --filename="$TOMO"
  ```
- Embeddings (per tomogram):
  ```bash
  cryosiam simsiam_embeddings_predict --config_file=config_subtomo_embeddings.yaml --filename="$TOMO"
  ```

---

## Best practices

- Use **one GPU per job**
- Avoid running multiple CryoSiam jobs on the same GPU
- Monitor jobs with `squeue -u $USER`
- Adjust memory and time requests depending on tomogram size

---

## See also

- [Usage overview](usage.md)
