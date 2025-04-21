---
title: "How to setup 4DGaussians from hustvl"
date: 2025-04-21
permalink: /posts/phd/computer-vision/4dg
tags:
  - 3D Gaussian
  - Computer Vision
---

Guide to set up the 4DGaussians from hustvl using Conda and PyTorch.

# Forked repository

I have [forked](https://github.com/TCQian/4DGaussians?tab=readme-ov-file) the repository and updated the `requirements.txt` file. The following sections will assume you use my fork instead. If you want to work on the original repository instead, you may check out the diff [here](https://github.com/hustvl/4DGaussians/compare/master...TCQian:4DGaussians:master).

# Clone repository, initialise submodule in the repository

```
git clone https://github.com/TCQian/4DGaussians.git
cd 4DGaussians
git submodule init; git submodule update
```

# Use conda to create a closed virtual environment

```
conda create -n 4dg python=3.10
conda activate 4dg
```

# Install packages (make sure you have a Nvidia GPU and Cuda runtime when running below installations)

There was some changes made to `requirements.txt` file to remove `module` versions. We could rely on `pip` to resolve the dependencies.

Here, we follow https://pytorch.org/ guidelines and always install the latest torch version that the Nvidia GPU can support for optimal performance.

```
pip install torch torchvision torchaudio
pip install -r requirements.txt
```

# Installing submodule toolkit (make sure you have a Nvidia GPU and Cuda runtime when running below installations)

```
pip install -e submodules/depth-diff-gaussian-rasterization
pip install -e submodules/simple-knn
```

# Running

## synthetic scene

Set up `data` folder by downloading the `D-NeRF` dataset from [dropbox](https://www.dropbox.com/scl/fi/cdcmkufncwcikk1dzbgb4/data.zip?rlkey=n5m21i84v2b2xk6h7qgiu8nkg&e=1&dl=0)

Your folder should look like this:

```
├── data
├── ├── dnerf
│       ├── jumpingjacks
│       ├── bouncingballs
│       ├── hook
│       ├── ...
```

Run training, rendering and evaluation on `bouncingballs`.

```
python train.py -s data/dnerf/bouncingballs --port 6017 --expname "dnerf/bouncingballs" --configs arguments/dnerf/bouncingballs.py
python render.py --model_path "output/dnerf/bouncingballs/"  --skip_train --configs arguments/dnerf/bouncingballs.py
python metrics.py --model_path "output/dnerf/bouncingballs/"
```

## multipleview scene

Organise your `data` folder as follow:

```
├── data
|   | multipleview
│     | <your dataset name>
│   	  | cam01
|     		  ├── frame_00001.jpg
│     		  ├── frame_00002.jpg
│     		  ├── ...
│   	  | cam02
│     		  ├── frame_00001.jpg
│     		  ├── frame_00002.jpg
│     		  ├── ...
│   	  | ...
```

Then, generate the necessary files `sparse_`, `points3D_multipleview.ply` and `poses_bounds_multipleview.npy`, using the provided script. You will need `colmap` to be able to run the script. Please check the next section on `colmap` installation guide.

```
bash multipleviewprogress.sh <your dataset name>
```

Run training, rendering and evaluation on `multipleview` data.

```
python train.py -s  data/multipleview/<your dataset name> --port 6017 --expname "multipleview/<your dataset name>" --configs arguments/multipleview/default.py
python render.py --model_path "output/multipleview/<your dataset name>/"  --skip_train --configs arguments/multipleview/default.py
python metrics.py --model_path "output/multipleview/<your dataset name>/"
```

# `colmap` installation (make sure you have a Nvidia GPU and Cuda runtime when running below installations)

As I don't have `sudo` privileage in my server setup, all my installation is made using `conda`.

First install the necessary packages (You may use `apt-get` for this, but you will have to figure out youself)

```
conda install -c conda-forge -c nvidia \
  cmake ninja eigen glew boost suitesparse gflags glog opencv qt \
  libpng libjpeg-turbo flann metis freeimage cgal sqlite zlib lz4 \
  openmp mpfr gmp libglu libgl \
  xorg-libx11 xorg-libxext xorg-libxau xorg-libxdmcp xorg-libxrender libgl-devel \
  mesa-libgl-cos7-x86_64 \
  cuda-toolkit ceres-solver
```

Then, clone the official `colmap` repository and setup

```
git clone https://github.com/colmap/colmap.git
cd colmap
mkdir build && cd build
```

Run the `colmap` installation

```
cmake .. -GNinja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCUDA_ENABLED=ON \
  -DCOLMAP_GUI=OFF \
  -DCMAKE_CUDA_ARCHITECTURES=80
```

Finally, add the `colmap` path

```
export PATH=$PWD/src/colmap/exe:$PATH
```

## `DCMAKE_CUDA_ARCHITECTURES` Explanation

The `-DCMAKE_CUDA_ARCHITECTURES` flag in CMake specifies the **target GPU architecture(s)** that the CUDA code should be compiled for.

Find your GPU's **Compute Capability** on [NVIDIA's official chart](https://developer.nvidia.com/cuda-gpus) and pass it as an integer.

| GPU Model        | Compute Capability | Value to Use |
| ---------------- | ------------------ | ------------ |
| Tesla V100       | 7.0                | `70`         |
| RTX 2080 Ti      | 7.5                | `75`         |
| RTX 3080 / A100  | 8.0                | `80`         |
| RTX 3090 / A6000 | 8.6                | `86`         |
| RTX 40xx / H100  | 8.9                | `89`         |
