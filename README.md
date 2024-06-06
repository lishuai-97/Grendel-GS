<div align="center">

Grendel-GS
===========================
<h3>Gaussian Splatting At Scale by Distributed Training</h3>

<div align="left">

## Latest News - June 2024

Explore our latest advancements in our new paper, access pre-trained models, and download evaluation images.

### [On Scaling Up 3D Gaussian Splatting Training](https://github.com/microsoft/DeepSpeed/tree/master/blogs/deepspeed-fp6/03-05-2024)
_Team: **Hexu Zhao**, **Haoyang Weng\***, **Daohan Lu\***, **Ang Li**, **Jinyang Li**, **Aurojit Panda**, **Saining Xie**_  (\* *Indicates equal contribution*)
- **[Arxiv Paper](https://github.com/microsoft/DeepSpeed/tree/master/blogs/deepspeed-fp6/03-05-2024/README.md)**
- **[Pre-trained Models (14 GB)](https://github.com/microsoft/DeepSpeed/tree/master/blogs/deepspeed-fp6/03-05-2024/README.md)**
- **[Evaluation Images (7 GB)](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/evaluation/images.zip)**


<div align="center">
    <img src="assets/teaser.png" width="900">
</div>


---

# Grendel-GS Overview
Grendel-GS serves as a ressearch-oriented framework for large scale gaussian splatting training. Its core idea is to leverage more GPU by distributed computation during training and to increase the batch size to utilize these GPU better. Therefore, we could accommodate much more gaussians primitive in large-scale and high resolution scenes, and speed up at the same time. This codebase is developed from original gaussian splatting implementation and serves as the official implementation associated with our paper "On Scaling Up 3D Gaussian Splatting Training". It contains a distributed PyTorch-based optimizer to produce a 3D Gaussian model from SfM inputs. The optimizer uses PyTorch and CUDA extensions in a Python environment to produce trained models. 

<section class="section" id="BibTeX">
  <div class="container is-max-desktop content">
    <h2 class="title">BibTeX</h2>
    <pre><code>@Article{kerbl3Dgaussians,
      author       = {Kerbl, Bernhard and Kopanas, Georgios and Leimk{\"u}hler, Thomas and Drettakis, George},
      title        = {3D Gaussian Splatting for Real-Time Radiance Field Rendering},
      journal      = {ACM Transactions on Graphics},
      number       = {4},
      volume       = {42},
      month        = {July},
      year         = {2023},
      url          = {https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/}
}</code></pre>
  </div>
(TODO:change it to our bib)
</section> 


## Cloning the Repository

The repository contains submodules, thus please check it out with 
```shell
git clone -b release4neurips git@github.com:TarzanZhao/gaussian-splatting.git --recursive
```

## Setup

To use our repository, you should have GPU with compatible driver and cuda environment and simply do:
```
conda env create --file environment.yml
conda activate gaussian_splatting
```

It worth mentioning that this process will compile and install two dependent cuda repo `diff-gaussian-rasterization` and `simple-knn` containing our customized cuda kernels for rendering and etc.

### Training

For single-GPU non-distributed training with batch size of 1:
```shell
python train.py -s <path to COLMAP dataset>
```

For 4 GPU distributed training and batch size of 4:
```shell
torchrun --standalone --nnodes=1 --nproc-per-node=4 train.py --bsz 4 -s <path to COLMAP dataset>
```

<details>
<summary><span style="font-weight: bold;">Command Line Arguments for train.py</span></summary>

  #### --source_path / -s
  Path to the source directory containing a COLMAP data set.
  #### --model_path / -m 
  Path where the trained model and loggings should be stored (```/tmp/gaussian_splatting``` by default).
  #### --eval
  Add this flag to use a MipNeRF360-style training/test split for evaluation.
  #### --bsz
  The batch size(the number of camera views) in single step training. ```1``` by default.
  #### --lr_scale_mode
  The mode of scaling learning rate given larger batch size. ```sqrt``` by default.
  #### --preload_dataset_to_gpu
  Save all groundtruth images from the dataset in GPU, rather than load each image on-the-fly at each training step. 
  If dataset is large, preload_dataset_to_gpu will lead to OOM; when the dataset is small, preload_dataset_to_gpu could 
  speed up the training a little bit by avoiding some cpu-gpu communication. 
  #### --iterations
  Number of total iterations to train for, ```30_000``` by default.
  #### --test_iterations
  Space-separated iterations at which the training script computes L1 and PSNR over test set, ```7000 30000``` by default.
  #### --save_iterations
  Space-separated iterations at which the training script saves the Gaussian model, ```7000 30000 <iterations>``` by default.
  #### --checkpoint_iterations
  Space-separated iterations at which to store a checkpoint for continuing later, saved in the model directory.
  #### --start_checkpoint
  Path to a saved checkpoint to continue training from.
  #### --white_background / -w
  Add this flag to use white background instead of black (default), e.g., for evaluation of NeRF Synthetic dataset.
  #### --sh_degree
  Order of spherical harmonics to be used (no larger than 3). ```3``` by default.
  #### --feature_lr
  Spherical harmonics features learning rate, ```0.0025``` by default.
  #### --opacity_lr
  Opacity learning rate, ```0.05``` by default.
  #### --scaling_lr
  Scaling learning rate, ```0.005``` by default.
  #### --rotation_lr
  Rotation learning rate, ```0.001``` by default.
  #### --position_lr_max_steps
  Number of steps (from 0) where position learning rate goes from ```initial``` to ```final```. ```30_000``` by default.
  #### --position_lr_init
  Initial 3D position learning rate, ```0.00016``` by default.
  #### --position_lr_final
  Final 3D position learning rate, ```0.0000016``` by default.
  #### --position_lr_delay_mult
  Position learning rate multiplier (cf. Plenoxels), ```0.01``` by default. 
  #### --densify_from_iter
  Iteration where densification starts, ```500``` by default. 
  #### --densify_until_iter
  Iteration where densification stops, ```15_000``` by default.
  #### --densify_grad_threshold
  Limit that decides if points should be densified based on 2D position gradient, ```0.0002``` by default.
  #### --densification_interval
  How frequently to densify, ```100``` (every 100 iterations) by default.
  #### --opacity_reset_interval
  How frequently to reset opacity, ```3_000``` by default. 
  #### --lambda_dssim
  Influence of SSIM on total loss from 0 to 1, ```0.2``` by default. 
  #### --percent_dense
  Percentage of scene extent (0--1) a point must exceed to be forcibly densified, ```0.01``` by default.



</details>
<br>

### Render Trained Model

```shell
python render.py -s <path to COLMAP dataset> --model_path <path to trained model directory> 
```

<details>
<summary><span style="font-weight: bold;">Command Line Arguments for render.py</span></summary>

  #### --model_path / -m 
  Path to the trained model directory you want to create renderings for.
  #### --skip_train
  Flag to skip rendering the training set.
  #### --skip_test
  Flag to skip rendering the test set.
  #### --distributed_load
  If point cloud models are saved distributedly during training, we should set this flag to load all of them.
  #### --quiet 
  Flag to omit any text written to standard out pipe. 

  **The below parameters will be read automatically from the model path, based on what was used for training. However, you may override them by providing them explicitly on the command line.** 

  #### --source_path / -s
  Path to the source directory containing a COLMAP or Synthetic NeRF data set.
  #### --images / -i
  Alternative subdirectory for COLMAP images (```images``` by default).
  #### --eval
  Add this flag to use a MipNeRF360-style training/test split for evaluation.
  #### --llffhold
  The training/test split ratio in the whole dataset for evaluation. llffhold=8 means 1/8 is used as test set and others are used as train set.
  #### --white_background / -w
  Add this flag to use white background instead of black (default), e.g., for evaluation of NeRF Synthetic dataset.

</details>

### Evaluating metrics

```shell
python metrics.py --model_path <path to folder of saving model> 
```

<details>
<summary><span style="font-weight: bold;">Command Line Arguments for metrics.py</span></summary>

  #### --model_paths / -m 
  Space-separated list of model paths for which metrics should be computed.
</details>
<br>

---

# Results

TODO: put the table of PSNR and throughput of mip360 dataset here.

---

# New features [Please check regularly!]

- We will release our optimized cuda kernels within gaussian splatting soon for further speed up. 

