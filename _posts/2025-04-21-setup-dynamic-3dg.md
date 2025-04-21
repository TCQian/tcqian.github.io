---
title: "How to setup Dynamic 3D Gaussians from JonathonLuiten"
date: 2025-04-21
permalink: /posts/phd/computer-vision/dynamic-3dg
tags:
  - 3D Gaussian
  - Computer Vision
---

Guide to set up the Dynamic 3D Gaussians from JonathonLuiten using Conda and PyTorch.

# Forked repository

I have [forked](https://github.com/TCQian/Dynamic3DGaussians) the repository and added a `requirements.txt` file. The following sections will assume you use my fork instead. If you want to work on the original repository instead, you may check out the diff [here](https://github.com/JonathonLuiten/Dynamic3DGaussians/compare/main...TCQian:Dynamic3DGaussians:main).

# Clone repository

```
git clone https://github.com/TCQian/Dynamic3DGaussians.git
cd Dynamic3DGaussians
```

# Use conda to create a closed virtual environment

Try to use python 3.9 or above, as it's the latest python version that `PyTorch` supports.

```
conda create -n 3dg python=3.10 -y
conda activate 3dg
```

# Install packages (make sure you have a Nvidia GPU and Cuda runtime when running below installations)

Added `requirements.txt` file to include the necessary `pip` modules. I normally don't use the `conda` environment files as they are often outdated, or the `cuda` runtime used is too old. Most of the time, you should use the most recent `torch` version that your `cuda` runtime can support to take advantage of improved library.

`Pytorch` no longer officially support installation using `conda`, so you should just use `pip` instead.

```
pip install -r requirements.txt
```

# Installing submodule

## diff-gaussian-rasterization-w-depth

The following is identical to the installation steps provided in the original repo.

```
git clone git@github.com:JonathonLuiten/diff-gaussian-rasterization-w-depth.git
cd diff-gaussian-rasterization-w-depth
python setup.py install
pip install .
```

# Running

The following is also identical to the code run steps provided in the original repo.

Set up `data` folder.

```
cd Dynamic3DGaussians
wget https://omnomnom.vision.rwth-aachen.de/data/Dynamic3DGaussians/data.zip  # Download training data
unzip data.zip
```

Run training

```
python train.py
```
