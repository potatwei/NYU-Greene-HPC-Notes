# NYU-Greene-HPC-Singularity-Notes
This repository contains comprehensive notes, tutorials, and examples for using Singularity on the NYU Greene High-Performance Computing (HPC) cluster.

## Request Compute Note
Programs I want to use all require a GPU. To request a compute node with one GPU
```
srun --cpus-per-task=4 --time=2:00:00 --mem=4000 -gres=gpu:1 --pty /bin/bash
```
To request for a specific GPU
```
srun --cpus-per-task=4 --time=2:00:00 --mem=4000 -gres=gpu:1:v100 --pty /bin/bash
```
```
srun --cpus-per-task=4 --time=2:00:00 --mem=4000 -gres=gpu:1:rtx8000 --pty /bin/bash
```
More details in this official [documentation](https://sites.google.com/nyu.edu/nyu-hpc/training-support/tutorials/slurm-tutorial?authuser=0).
