# Anycost GAN

### [video](https://youtu.be/_yEziPl9AkM) | [paper](https://arxiv.org/abs/2103.03243) | [website](https://hanlab.mit.edu/projects/anycost-gan/) [![](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mit-han-lab/anycost-gan/blob/master/notebooks/intro_colab.ipynb)

[Anycost GANs for Interactive Image Synthesis and Editing](https://arxiv.org/abs/2103.03243)

[Ji Lin](http://linji.me/), [Richard Zhang](https://richzhang.github.io/), Frieder Ganz, [Song Han](https://songhan.mit.edu/), [Jun-Yan Zhu](https://www.cs.cmu.edu/~junyanz/)

MIT, Adobe Research, CMU

In CVPR 2021

![flexible](https://hanlab.mit.edu/projects/anycost-gan/images/flexible.gif)

Anycost GAN generates consistent outputs under various computational budgets.



<a href="https://youtu.be/_yEziPl9AkM?t=90"><img src='assets/figures/demo.gif' width=600></a>

We can use the anycost generator for **interactive image editing**. A full generator takes **~3s** to render an image, which is too slow for editing. While with anycost generator, we can provide a visually similar preview at **5x faster speed**. After adjustment, we hit the "Finalize" button to give the high-qaulity, edited output. Check [here](https://youtu.be/_yEziPl9AkM?t=90) for the full demo.



## Overview

Anycost generators can be run at *diverse computation costs* by using different *channel* and *resolution* configurations. Sub-generators achieve high output consistency compared to the full generator, providng a fast preview.

![overview](https://hanlab.mit.edu/projects/anycost-gan/images/overall.jpg)



With (1) Sampling-based multi-resolution training; (2) adaptive-channel training; (3) generator-conditioned discriminator, we can achieve high image quality and consistency at different resolutions and channels.

![method](https://hanlab.mit.edu/projects/anycost-gan/images/method_pad.gif)

## Results

Anycost GAN (uniform channel version) supports 4 resolutions and 4 channel ratios, producing visually consistent images at different qualities.

![uniform](https://hanlab.mit.edu/projects/anycost-gan/images/uniform.gif)



The consistency retains during image projection and editing:

![](https://hanlab.mit.edu/projects/anycost-gan/images/teaser.jpg)

![](https://hanlab.mit.edu/projects/anycost-gan/images/editing.jpg)



## Usage

### Getting Started

- Clone this repo:

```bash
git clone https://github.com/mit-han-lab/anycost-gan.git
cd anycost-gan
```

- Install PyTorch 1.7 and other dependancies.

We recommend setting up the environment using anaconda: `conda env create -f environment.yml`



### Introduction Notebook

We provide a jupyter notebook example to show how to use the anycost generator for image synthesis at diverse costs: `notebooks/intro.ipynb`. 

We also provide a colab version of the notebook: [![](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mit-han-lab/anycost-gan/blob/master/notebooks/intro_colab.ipynb). Be sure to select the GPU as the accelerator in runtime options.



### Interactive Demo

We provide an interactive demo showing how we can use anycost GAN to enable interactive image editing. To run the demo:

```bash
python demo.py
```

You can find a video recording of the demo [here](https://youtu.be/_yEziPl9AkM?t=90).



### Using Pre-trained Models

To get the pre-trained generator, encoder, and editing directions, run:

```python
import model

pretrained_type = 'generator'  # choosing from ['generator', 'encoder', 'boundary']
config_name = 'anycost-ffhq-config-f'  # replace the config name for other models
model.get_pretrained(pretrained_type, config=config_name) 
```

We also provide the face attribute classifier (which is general for different generators) for computing the editing directions. You can get it by running:

```python
model.get_pretrained('attribute-predictor')
```

The attribute classifier takes in the face images in FFHQ format.



After loading the anycost generator, we can run it at diverse cost. For example:

```python
from model.dynamic_channel import set_uniform_channel_ratio, reset_generator

g = model.get_pretrained('generator', config='anycost-ffhq-config-f')  # anycost uniform
set_uniform_channel_ratio(g, 0.5)  # set channel
g.target_res = 512  # set resolution
out, _ = g(...)  # generate image
reset_generator(g)  # restore the generator
```

For a detailed usage and *flexible-channel* anycost generator, please refer to `notebooks/intro.ipynb`.



### Model Zoo

Currently, we provide the following pre-trained generators, encoders, and editing directions. We will add more in the future.

For anycost generators, by default, we refer to the uniform setting.

| config name                    | generator          | encoder            | edit direction     |
| ------------------------------ | ------------------ | ------------------ | ------------------ |
| anycost-ffhq-config-f          | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| anycost-ffhq-config-f-flexible | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| anycost-car-config-f           | :heavy_check_mark: |                    |                    |
| stylegan2-ffhq-config-f        | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |

`stylegan2-ffhq-config-f` refers to the official StyleGAN2 generator converted from the [repo](https://github.com/NVlabs/stylegan2).



### Datasets

We prepare the [FFHQ](https://github.com/NVlabs/ffhq-dataset), [CelebA-HQ](https://github.com/switchablenorms/CelebAMask-HQ), and [LSUN Car](https://github.com/fyu/lsun) datasets into a directory of images, so that it can be easily used with `ImageFolder` from `torchvision`. The dataset layout looks like:

```
├── PATH_TO_DATASET
│   ├── images
│   │   ├── 00000.png
│   │   ├── 00001.png
│   │   ├── ...
```

Due to the copyright issue, you need to download the dataset from official site and process them accordingly.



### Evaluation

We provide the code to evaluate some metrics presented in the paper. Some of the code is written with [`horovod`](https://github.com/horovod/horovod) to support distributed evalution and reduce the cost of inter-GPU communication, which greatly improves the speed. Check its website for a proper installation.

#### Fre ́chet Inception Distance (FID)

Before evaluating the FIDs, you need to compute the inception features of the real images using scripts like:

```bash
python tools/calc_inception.py \
    --resolution 1024 --batch_size 64 -j 16 --n_sample 50000 \
    --save_name assets/inceptions/inception_ffhq_res1024_50k.pkl \
    PATH_TO_FFHQ
```

or you can download the pre-computed inceptions from [here](https://www.dropbox.com/sh/bc8a7ewlvcxa2cf/AAD8NFzDWKmBDpbLef-gGhRZa?dl=0) and put it under `assets/inceptions`.

Then, you can evaluate the FIDs by running:

```bash
horovodrun -np N_GPU \
    python metrics/fid.py \
    --config anycost-ffhq-config-f \
    --batch_size 16 --n_sample 50000 \
    --inception assets/inceptions/inception_ffhq_res1024_50k.pkl
    # --channel_ratio 0.5 --target_res 512  # optionally using a smaller resolution/channel
```

#### Perceptual Path Lenght (PPL)

Similary, evaluting the PPL with:

```bash
horovodrun -np N_GPU \
    python metrics/ppl.py \
    --config anycost-ffhq-config-f 
```

#### Attribute Consistency

Evaluating the attribute consistency by running:

```bash
horovodrun -np N_GPU \
    python metrics/attribute_consistency.py \
    --config anycost-ffhq-config-f \
    --channel_ratio 0.5 --target_res 512  # config for the sub-generator; necessary
```

#### Encoder Evaluation

To evaluate the performance of the encoder, run:

```bash
python metrics/eval_encoder.py \
    --config anycost-ffhq-config-f \
    --data_path PATH_TO_CELEBA_HQ
```



### Training

The training code will be updated shortly.



## Citation

If you use this code for your research, please cite our paper.

```
@inproceedings{lin2021anycost,
  author    = {Lin, Ji and Zhang, Richard and Ganz, Frieder and Han, Song and Zhu, Jun-Yan},
  title     = {Anycost GANs for Interactive Image Synthesis and Editing},
  booktitle = {IEEE Conference on Computer Vision and Pattern Recognition (CVPR)},
  year      = {2021},
}
```



## Related Projects

**[GAN Compression](https://github.com/mit-han-lab/gan-compression) | [Once for All](https://github.com/mit-han-lab/once-for-all) | [iGAN](https://github.com/junyanz/iGAN) | [StyleGAN2](https://github.com/NVlabs/stylegan2)**



## Acknowledgement

We thank Taesung Park, Zhixin Shu, Muyang Li, and Han Cai for the helpful discussion. Part of the work is supported by NSF CAREER Award #1943349, Adobe, Naver Corporation, and MIT-IBM Watson AI Lab.

The codebase is build upon a PyTorch implementation of StyleGAN2: [rosinality/stylegan2-pytorch](https://github.com/rosinality/stylegan2-pytorch). For editing direction extraction, we refer to [InterFaceGAN](https://github.com/genforce/interfacegan).