# NYU-Greene-HPC-Notes <!-- omit from toc -->
This repository contains comprehensive notes and examples for using Singularity on the NYU Greene High-Performance Computing (HPC) cluster.

# Table of Contents <!-- omit from toc -->
- [Useful Commands](#useful-commands)
- [Request Compute Note](#request-compute-note)
- [List Your Jobs](#list-your-jobs)
- [Cancel Your Jobs](#cancel-your-jobs)
- [Singularity using Overlay](#singularity-using-overlay)
- [Files in a Container](#files-in-a-container)
- [Using GPU in a Container](#using-gpu-in-a-container)

## Useful Commands
Check current usage and quota ⬇️
```bash
myquota
```
```bash
Filesystem   Environment   Backed up?   Allocation       Current Usage
Space        Variable      /Flushed?    Space / Files    Space(%) / Files(%)

/home        $HOME         Yes/No       50.0GB/30.0K       0.19GB(0.39%)/1684(5.61%)
/scratch     $SCRATCH      No/Yes        5.0TB/1.0M       44.54GB(0.87%)/98718(9.87%)
/archive     $ARCHIVE      Yes/No        2.0TB/20.0K       0.00GB(0.00%)/2(0.01%)
/vast        $VAST         NO/YES        2TB/5.0M           0.0TB(0.0%)/2(0%)
```
Show status of the GPU ⬇️
```bash
nvidia-smi
```
```bash
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.154.05             Driver Version: 535.154.05   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA A100-SXM4-80GB          On  | 00000000:E3:00.0 Off |                    0 |
| N/A   37C    P0             214W / 500W |   6568MiB / 81920MiB |     99%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A   3009270      C   python                                     6558MiB |
+---------------------------------------------------------------------------------------+
```
Running program in background ⬇️
```bash
nohup $Command &
```

## Request Compute Note
Programs I want to use all require a GPU. To request a compute node with one GPU
```bash
srun --cpus-per-task=4 --time=4:00:00 --mem=8000 --gres=gpu:1 --pty /bin/bash
```
To request for a specific GPU
```bash
srun --cpus-per-task=4 --time=4:00:00 --mem=8000 --gres=gpu:v100:1 --pty /bin/bash
```
```bash
srun --cpus-per-task=4 --time=4:00:00 --mem=8000 --gres=gpu:rtx8000:1 --pty /bin/bash
```
- `--cpus-per-task=4` or `-c 4` specify 4 CPU cores
- `--times=4:00:00` or `-t 4:00:00` specify 4 hours
- `--mem=8192` or `--mem=8G` specifies 8192MB(8GB) of memory
- `--pty /bin/bash` runs bash
  
More details in this official [documentation](https://sites.google.com/nyu.edu/nyu-hpc/training-support/tutorials/slurm-tutorial?authuser=0).

## List Your Jobs
To list all your jobs since June 10, 2024, you can use
```bash
sacct --starttime=2024-06-10
```
To list your **queued** jobs' time and node, you can use
```bash
squeue -u $NetID
```

## Cancel Your Jobs
After you get your job id by listing all jobs, you can kill a job using
```bash
scancel $JobID
```
If you are in a compute node, you can use `exit` to go back to the login node
```bash
exit
```
More details about SLURM in this official [documentation](https://crc-docs.abudhabi.nyu.edu/hpc/jobs/hpc_slurm.html#)

## Singularity using Overlay
NYU HPC has detailed documentation about this [here](https://sites.google.com/nyu.edu/nyu-hpc/hpc-systems/greene/software/singularity-with-miniconda?authuser=0), but you can still follow steps below.\
Instead of building an image on your own, you should use one provided here
```bash
ls /scratch/work/public/singularity
```
Overlay images are here
```bash
ls /scratch/work/public/overlay-fs-ext3
```
<details>
  <summary>what is overlay?</summary>

  Persistent overlay directories allow you to overlay a writable file system on an immutable read-only container for the illusion of read-write access. You can run a container and make changes, and these changes are kept separately from the base container image. Read more about overlay [here](https://docs.sylabs.io/guides/3.5/user-guide/persistent_overlays.html)
</details>

Choose an overlay image with the desired size. I chose `overlay-15GB-500K.ext3.gz` for my application. It has 15GB free space inside and is able to hold 500K files.\
\
Make a directory in your scratch folder and cd into it
```bash
mkdir /scratch/<NetID>/myproject && cd /scratch/<NetID>/myproject
```
Copy it and unzip
```bash
cp -rp /scratch/work/public/overlay-fs-ext3/overlay-15GB-500K.ext3.gz .
gunzip overlay-15GB-500K.ext3.gz
```
Finally, you can launch the container in read/write mode (with the :rw flag)
```bash
singularity exec --overlay overlay-15GB-500K.ext3:rw /scratch/work/public/singularity/cuda11.6.124-cudnn8.4.0.27-devel-ubuntu20.04.4.sif /bin/bash
```
Now you can do what you want inside. Maybe install conda
```bash
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p /ext3/miniconda3
rm Miniconda3-latest-Linux-x86_64.sh
```
But what if your program generates a file? How should you access it and transfer it outside the container? In this case, we can use `--bind`

## Files in a Container
The container’s filesystem is isolated from the host filesystem. To access files on the physical filesystem from within the container, you need to mount the directory of the host filesystem.\
Simply add `--bind <Host Dir>:<Container Dir>`. Then the command becomes
```bash
singularity exec --bind .:/mnt /scratch/work/public/singularity/cuda11.6.124-cudnn8.4.0.27-devel-ubuntu20.04.4.sif /bin/bash
```
or
```bash
singularity exec --overlay overlay-15GB-500K.ext3:rw --bind .:/mnt /scratch/work/public/singularity/cuda11.6.124-cudnn8.4.0.27-devel-ubuntu20.04.4.sif /bin/bash
```
Here I'm binding my current work directory `.` with `/mnt` in the container. If you go to `/mnt` inside the container, it's just your current work directory.

## Using GPU in a Container
When you need to use GPU in a container, add `--nv` when you launch the image. For example
```bash
singularity exec --nv --overlay overlay-15GB-500K.ext3:rw --bind .:/mnt /scratch/work/public/singularity/cuda11.6.124-cudnn8.4.0.27-devel-ubuntu20.04.4.sif /bin/bash
```
