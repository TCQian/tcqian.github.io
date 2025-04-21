---
title: "How to setup SC-GS from CVMI-Lab"
date: 2025-04-19
permalink: /posts/phd/computer-vision/sc-gs
tags:
  - 3D Gaussian
  - Computer Vision
---

A step-by-step guide to setting up the SC-GS 3D Gaussian Splatting framework from CVMI-Lab using Conda and PyTorch.

# Forked repository

I have [forked](https://github.com/TCQian/SC-GS) the repository and updated the `requirements.txt` file. The following sections will assume you use my fork instead. If you want to work on the original repository instead, you may check out the diff [here](https://github.com/CVMI-Lab/SC-GS/compare/master...TCQian:SC-GS:master).

# Clone repository, initialise submodule in the repository

```
git clone https://github.com/TCQian/SC-GS.git
cd SC-GS
git submodule init; git submodule update
```

# Use conda to create a closed virtual environment

```
conda create -n scgs python=3.10
conda activate scgs
```

# Install packages (make sure you have a Nvidia GPU and Cuda runtime when running below installations)

There was some changes made to `requirements.txt` file because:

1. `pytorch-ssim` module is removed because it is found to be no longer used in the repository
1. all module versions removed, since we can rely on `pip` to resolve dependencies

Here, we follow https://pytorch.org/ guidelines and always install the latest torch version that the Nvidia GPU can support for optimal performance.

```
pip install torch torchvision torchaudio
pip install -r requirements.txt
```

# Installing submodule toolkit

## diff-gaussian-rasterization

While installing, you might encounter some header files not found issue. It’s likely that because the system packages are not yet installed in your Linux system. If you have `sudo` privileges, you may google the header files and install them directly on your Linux system.

For me, I do not have `sudo` privileges, and I encounter a `glm` header file not found issue. So I used `conda` to install the corresponding system packages. I also need to export some variable such that the installation will not use the default system path.

```
conda install -c conda-forge glm ninja
export CPLUS_INCLUDE_PATH=$CONDA_PREFIX/include
```

Then, I can install `diff-gaussian-rasterization` successfully.

```
pip install -e submodules/diff-gaussian-rasterization
```

## simple-knn

While installing, the error prompt some missing variables:

```
error: identifier "FLT_MAX" is undefined
```

To resolve this, you need to include the missing import statement `#include <cfloat>` at the top of these two files:

```
simple_knn.cu
spatial.cu
```

Afterwards, you should be able to install `simple-knn` successfully.

```
pip install -e submodules/simple-knn
```

# Running

The following is identical to the code run steps provided in the original repo.

Set up `data` folder by downloading the `D-NeRF` dataset from [dropbox](https://www.dropbox.com/scl/fi/cdcmkufncwcikk1dzbgb4/data.zip?rlkey=n5m21i84v2b2xk6h7qgiu8nkg&e=1&dl=0)

Your folder should look like this:

```
├── data
│   ├── jumpingjacks
│   ├── bouncingballs
│   ├── hook
│   ├── ...
```

Run training.

```
CUDA_VISIBLE_DEVICES=0 python train_gui.py --source_path YOUR/PATH/TO/DATASET/jumpingjacks --model_path outputs/jumpingjacks --deform_type node --node_num 512 --hyper_dim 8 --is_blender --eval --gt_alpha_mask_as_scene_mask --local_frame --resolution 2 --W 800 --H 800
```
