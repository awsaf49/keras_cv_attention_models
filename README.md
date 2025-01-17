# Keras_cv_attention_models
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Keras_cv_attention_models](#kerascvattentionmodels)
- [Usage](#usage)
  - [Basic Usage](#basic-usage)
  - [Layers](#layers)
  - [Model surgery](#model-surgery)
  - [AotNet](#aotnet)
  - [ResNetD](#resnetd)
  - [ResNeXt](#resnext)
  - [ResNetQ](#resnetq)
  - [BotNet](#botnet)
  - [VOLO](#volo)
  - [ResNeSt](#resnest)
  - [HaloNet](#halonet)
  - [CoTNet](#cotnet)
  - [CoAtNet](#coatnet)
  - [CMT](#cmt)
  - [CoaT](#coat)
  - [MLP mixer](#mlp-mixer)
  - [ResMLP](#resmlp)
  - [GMLP](#gmlp)
  - [LeViT](#levit)
  - [NFNets](#nfnets)
- [Other implemented keras models](#other-implemented-keras-models)

<!-- /TOC -->
***

# Usage
## Basic Usage
  - **Current under works**: CMT, CoAtNet training.
  - Install as pip package:
    ```sh
    pip install -U keras-cv-attention-models
    # Or
    pip install -U git+https://github.com/leondgarse/keras_cv_attention_models
    ```
    Refer to each sub directory for detail usage.
  - **Basic model prediction**
    ```py
    from keras_cv_attention_models import volo
    mm = volo.VOLO_d1(pretrained="imagenet")

    """ Run predict """
    import tensorflow as tf
    from tensorflow import keras
    from skimage.data import chelsea
    img = chelsea() # Chelsea the cat
    imm = keras.applications.imagenet_utils.preprocess_input(img, mode='torch')
    pred = mm(tf.expand_dims(tf.image.resize(imm, mm.input_shape[1:3]), 0)).numpy()
    pred = tf.nn.softmax(pred).numpy()  # If classifier activation is not softmax
    print(keras.applications.imagenet_utils.decode_predictions(pred)[0])
    # [('n02124075', 'Egyptian_cat', 0.9692954),
    #  ('n02123045', 'tabby', 0.020203391),
    #  ('n02123159', 'tiger_cat', 0.006867502),
    #  ('n02127052', 'lynx', 0.00017674894),
    #  ('n02123597', 'Siamese_cat', 4.9493494e-05)]
    ```
  - **Exclude model top layers by set `num_classes=0`**
    ```py
    from keras_cv_attention_models import resnest
    mm = resnest.ResNest50(num_classes=0)
    print(mm.output_shape)
    # (None, 7, 7, 2048)
    ```
## Layers
  - [attention_layers](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/attention_layers) is `__init__.py` only, which imports core layers defined in model architectures. Like `RelativePositionalEmbedding` from `botnet`, `outlook_attention` from `volo`.
  ```py
  from keras_cv_attention_models import attention_layers
  aa = attention_layers.RelativePositionalEmbedding()
  print(f"{aa(tf.ones([1, 4, 14, 16, 256])).shape = }")
  # aa(tf.ones([1, 4, 14, 16, 256])).shape = TensorShape([1, 4, 14, 16, 14, 16])
  ```
## Model surgery
  - [model_surgery](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/model_surgery) including functions used to change model parameters after built.
  ```py
  from keras_cv_attention_models import model_surgery
  # Replace all ReLU with PReLU
  mm = model_surgery.replace_ReLU(keras.applications.ResNet50(), target_activation='PReLU')
  ```
## AotNet
  - [Keras AotNet](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/aotnet) is just a `ResNet` / `ResNetV2` like framework, that set parameters like `attn_types` and `se_ratio` and others, which is used to apply different types attention layer.
    ```py
    # Mixing se and outlook and halo and mhsa and cot_attention, 21M parameters
    # 50 is just a picked number that larger than the relative `num_block`
    from keras_cv_attention_models import aotnet
    attn_types = [None, "outlook", ["mhsa", "halo"] * 50, "cot"]
    se_ratio = [0.25, 0, 0, 0]
    mm = aotnet.AotNet50V2(attn_types=attn_types, se_ratio=se_ratio, deep_stem=True, strides=1)
    ```
## ResNetD
  - [Keras ResNetD](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/resnet_family#resnetd) includes implementation of [PDF 1812.01187 Bag of Tricks for Image Classification with Convolutional Neural Networks](https://arxiv.org/pdf/1812.01187.pdf)

  | Model      | Params | Image  resolution | Top1 Acc | Download |
  | ---------- | ------ | ----------------- | -------- | -------- |
  | ResNet50D  | 25.58M | 224               | 80.530   | [resnet50d.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnet50d_imagenet.h5) |
  | ResNet101D | 44.57M | 224               | 83.022   | [resnet101d.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnet101d_imagenet.h5) |
  | ResNet152D | 60.21M | 224               | 83.680   | [resnet152d.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnet152d_imagenet.h5) |
  | ResNet200D | 64.69  | 224               | 83.962   | [resnet200d.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnet200d_imagenet.h5) |
## ResNeXt
  - [Keras ResNeXt](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/resnet_family#resnext) includes implementation of [PDF 1611.05431 Aggregated Residual Transformations for Deep Neural Networks](https://arxiv.org/pdf/1611.05431.pdf)
  - `SWSL` means `Semi-Weakly Supervised ResNe*t` from [Github facebookresearch/semi-supervised-ImageNet1K-models](https://github.com/facebookresearch/semi-supervised-ImageNet1K-models). **Please note the CC-BY-NC 4.0 license on theses weights, non-commercial use only**.

  | Model                     | Params | Image  resolution | Top1 Acc | Download            |
  | ------------------------- | ------ | ----------------- | -------- | ------------------- |
  | ResNeXt50 (32x4d)         | 25M    | 224               | 79.768   | [resnext50_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnext50_imagenet.h5)  |
  | - SWSL                    | 25M    | 224               | 82.182   | [resnext50_swsl.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnext50_swsl.h5)  |
  | ResNeXt50D (32x4d + deep) | 25M    | 224               | 79.676   | [resnext50d_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnext50d_imagenet.h5)  |
  | ResNeXt101 (32x4d)        | 42M    | 224               | 80.334   | [resnext101_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnext101_imagenet.h5)  |
  | - SWSL                    | 42M    | 224               | 83.230   | [resnext101_swsl.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnext101_swsl.h5)  |
  | ResNeXt101W (32x8d)       | 89M    | 224               | 79.308   | [resnext101_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnext101_imagenet.h5)  |
  | - SWSL                    | 89M    | 224               | 84.284   | [resnext101w_swsl.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnext101w_swsl.h5)  |
## ResNetQ
  - [Keras ResNetQ](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/resnet_family#resnetq) includes implementation of [Github timm/models/resnet.py](https://github.com/rwightman/pytorch-image-models/blob/master/timm/models/resnet.py)

  | Model     | Params | Image  resolution | Top1 Acc | Download |
  | --------- | ------ | ----------------- | -------- | -------- |
  | ResNet51Q | 35.7M  | 224               | 82.36    | [resnet51q.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnet_family/resnet51q_imagenet.h5) |
## BotNet
  - [Keras BotNet](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/botnet) is for [PDF 2101.11605 Bottleneck Transformers for Visual Recognition](https://arxiv.org/pdf/2101.11605.pdf).

  | Model        | Params | Image resolution | Top1 Acc | Download            |
  | ------------ | ------ | ---------------- | -------- | ------------------- |
  | botnet50     | 21M    | 224              | 77.604   | [botnet50_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/botnet/botnet50_imagenet.h5)  |
  | botnet101    | 41M    | 224              |          |  |
  | botnet152    | 56M    | 224              |          |  |
## VOLO
  - [Keras VOLO](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/volo) is for [PDF 2106.13112 VOLO: Vision Outlooker for Visual Recognition](https://arxiv.org/pdf/2106.13112.pdf).

  | Model        | Params | Image resolution | Top1 Acc | Download            |
  | ------------ | ------ | ---------------- | -------- | ------------------- |
  | volo_d1      | 27M    | 224              | 84.2     | [volo_d1_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d1_224.h5)  |
  | volo_d1 ↑384 | 27M    | 384              | 85.2     | [volo_d1_384.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d1_384.h5)  |
  | volo_d2      | 59M    | 224              | 85.2     | [volo_d2_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d2_224.h5)  |
  | volo_d2 ↑384 | 59M    | 384              | 86.0     | [volo_d2_384.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d2_384.h5)  |
  | volo_d3      | 86M    | 224              | 85.4     | [volo_d3_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d3_224.h5)  |
  | volo_d3 ↑448 | 86M    | 448              | 86.3     | [volo_d3_448.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d3_448.h5)  |
  | volo_d4      | 193M   | 224              | 85.7     | [volo_d4_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d4_224.h5)  |
  | volo_d4 ↑448 | 193M   | 448              | 86.8     | [volo_d4_448.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d4_448.h5) |
  | volo_d5      | 296M   | 224              | 86.1     | [volo_d5_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d5_224.h5) |
  | volo_d5 ↑448 | 296M   | 448              | 87.0     | [volo_d5_448.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d5_448.h5) |
  | volo_d5 ↑512 | 296M   | 512              | 87.1     | [volo_d5_512.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/volo/volo_d5_512.h5) |
## ResNeSt
  - [Keras ResNeSt](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/resnest) is for [PDF 2004.08955 ResNeSt: Split-Attention Networks](https://arxiv.org/pdf/2004.08955.pdf).

  | Model          | Params | Image resolution | Top1 Acc | Download            |
  | -------------- | ------ | ----------------- | -------- | ------------------- |
  | resnest50      | 28M    | 224               | 81.03    | [resnest50.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnest/resnest50_imagenet.h5)  |
  | resnest101     | 49M    | 256               | 82.83    | [resnest101.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnest/resnest101_imagenet.h5)  |
  | resnest200     | 71M    | 320               | 83.84    | [resnest200.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnest/resnest200_imagenet.h5)  |
  | resnest269     | 111M   | 416               | 84.54    | [resnest269.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/resnest/resnest269_imagenet.h5)  |
## HaloNet
  - [Keras HaloNet](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/halonet) is for [PDF 2103.12731 Scaling Local Self-Attention for Parameter Efficient Visual Backbones](https://arxiv.org/pdf/2103.12731.pdf).

  | Model          | Params | Image resolution | Top1 Acc | Download |
  | -------------- | ------ | ---------------- | -------- | -------- |
  | HaloNetH0      | 5.5M   | 256              | 77.9     |          |
  | HaloNetH1      | 8.1M   | 256              | 79.9     |          |
  | HaloNetH2      | 9.4M   | 256              | 80.4     |          |
  | HaloNetH3      | 11.8M  | 320              | 81.9     |          |
  | HaloNetH4      | 19.1M  | 384              | 83.3     |          |
  | - 21k          | 19.1M  | 384              | 85.5     |          |
  | HaloNetH5      | 30.7M  | 448              | 84.0     |          |
  | HaloNetH6      | 43.4M  | 512              | 84.4     |          |
  | HaloNetH7      | 67.4M  | 600              | 84.9     |          |
  | HaloNet26T     | 11.6M  | 256              | 77.6 ?   | [halonet26t_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/halonet/halonet26t_imagenet.h5) |
  | HaloNetSE33T   | 13.7M  | 256              | 79.8 ?   | [halonet_se33t_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/halonet/halonet_se33t_imagenet.h5) |
  | HaloNextECA26T | 10.7M  | 256              | 77.8 ?   | [halonext_eca26t_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/halonet/halonext_eca26t_imagenet.h5) |
## CoTNet
  - [Keras CoTNet](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/cotnet) is for [PDF 2107.12292 Contextual Transformer Networks for Visual Recognition](https://arxiv.org/pdf/2107.12292.pdf).

  | Model          | Params | Image resolution | FLOPs | Top1 Acc | Download            |
  | -------------- |:------:| ---------------- | ----- |:--------:| ------------------- |
  | CoTNet-50      | 22.2M  | 224              | 3.3   |   81.3   | [cotnet50_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/cotnet/cotnet50_224.h5) |
  | CoTNeXt-50     | 30.1M  | 224              | 4.3   |   82.1   |  |
  | SE-CoTNetD-50  | 23.1M  | 224              | 4.1   |   81.6   | [se_cotnetd50_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/cotnet/se_cotnetd50_224.h5) |
  | CoTNet-101     | 38.3M  | 224              | 6.1   |   82.8   | [cotnet101_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/cotnet/cotnet101_224.h5) |
  | CoTNeXt-101    | 53.4M  | 224              | 8.2   |   83.2   |  |
  | SE-CoTNetD-101 | 40.9M  | 224              | 8.5   |   83.2   | [se_cotnetd101_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/cotnet/se_cotnetd101_224.h5) |
  | SE-CoTNetD-152 | 55.8M  | 224              | 17.0  |   84.0   | [se_cotnetd152_224.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/cotnet/se_cotnetd152_224.h5) |
  | SE-CoTNetD-152 | 55.8M  | 320              | 26.5  |   84.6   | [se_cotnetd152_320.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/cotnet/se_cotnetd152_320.h5) |
## CoAtNet
  - [Keras CoAtNet](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/coatnet) is for [PDF 2106.04803 CoAtNet: Marrying Convolution and Attention for All Data Sizes](https://arxiv.org/pdf/2106.04803.pdf).

  | Model                                | Params | Image resolution | Top1 Acc |
  | ------------------------------------ | ------ | ---------------- | -------- |
  | CoAtNet-0                            | 25M    | 224              | 81.6     |
  | CoAtNet-1                            | 42M    | 224              | 83.3     |
  | CoAtNet-2                            | 75M    | 224              | 84.1     |
  | CoAtNet-2, ImageNet-21k pretrain     | 75M    | 224              | 87.1     |
  | CoAtNet-3                            | 168M   | 224              | 84.5     |
  | CoAtNet-3, ImageNet-21k pretrain     | 168M   | 224              | 87.6     |
  | CoAtNet-3, ImageNet-21k pretrain     | 168M   | 512              | 87.9     |
  | CoAtNet-4, ImageNet-21k pretrain     | 275M   | 512              | 88.1     |
  | CoAtNet-4, ImageNet-21K + PT-RA-E150 | 275M   | 512              | 88.56    |
## CMT
  - [Keras CMT](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/cmt) is for [PDF 2107.06263 CMT: Convolutional Neural Networks Meet Vision Transformers](https://arxiv.org/pdf/2107.06263.pdf).

  | Model    | Params | Image resolution | Top1 Acc |
  | -------- | ------ | ---------------- | -------- |
  | CMTTiny  | 9.5M   | 160              | 79.2     |
  | CMTXS    | 15.2M  | 192              | 81.8     |
  | CMTSmall | 25.1M  | 224              | 83.5     |
  | CMTBig   | 45.7M  | 256              | 84.5     |
## CoaT
  - [Keras CoaT](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/coat) is for [PDF 2104.06399 CoaT: Co-Scale Conv-Attentional Image Transformers](http://arxiv.org/abs/2104.06399).

  | Model         | Params | Image resolution | Top1 Acc | Download |
  | ------------- | ------ | ---------------- | -------- | -------- |
  | CoaTLiteTiny  | 5.7M   | 224              | 77.5     | [coat_lite_tiny_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/coat/coat_lite_tiny_imagenet.h5) |
  | CoaTLiteMini  | 11M    | 224              | 79.1     | [coat_lite_mini_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/coat/coat_lite_mini_imagenet.h5) |
  | CoaTLiteSmall | 20M    | 224              | 81.9     | [coat_lite_small_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/coat/coat_lite_small_imagenet.h5) |
  | CoaTTiny      | 5.5M   | 224              | 78.3     | [coat_tiny_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/coat/coat_tiny_imagenet.h5) |
  | CoaTMini      | 10M    | 224              | 81.0     | [coat_mini_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/coat/coat_mini_imagenet.h5) |
## MLP mixer
  - [Keras MLP mixer](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/mlp_family#mlp-mixer) includes implementation of [PDF 2105.01601 MLP-Mixer: An all-MLP Architecture for Vision](https://arxiv.org/pdf/2105.01601.pdf).
  - **Models** `Top1 Acc` is `Pre-trained on JFT-300M` model accuray on `ImageNet 1K` from paper.

  | Model       | Params | Top1 Acc | ImageNet | Imagenet21k | ImageNet SAM |
  | ----------- | ------ | -------- | --------------- | ------------------ | ------------------- |
  | MLPMixerS32 | 19.1M  | 68.70    |                 |                    |                     |
  | MLPMixerS16 | 18.5M  | 73.83    |                 |                    |                     |
  | MLPMixerB32 | 60.3M  | 75.53    |                 |                    | [b32_imagenet_sam.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/mlp_mixer_b32_imagenet_sam.h5) |
  | MLPMixerB16 | 59.9M  | 80.00    | [b16_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/mlp_mixer_b16_imagenet.h5) | [b16_imagenet21k.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/mlp_mixer_b16_imagenet21k.h5) | [b16_imagenet_sam.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/mlp_mixer_b16_imagenet_sam.h5) |
  | MLPMixerL32 | 206.9M | 80.67    |  |  |                     |
  | MLPMixerL16 | 208.2M | 84.82    | [l16_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/mlp_mixer_l16_imagenet.h5) | [l16_imagenet21k.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/mlp_mixer_l16_imagenet21k.h5) |                     |
  | - input 448 | 208.2M | 86.78    |                 |                    |                     |
  | MLPMixerH14 | 432.3M | 86.32    |                 |                    |                     |
  | - input 448 | 432.3M | 87.94    |                 |                    |                     |
## ResMLP
  - [Keras ResMLP](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/mlp_family#resmlp) includes implementation of [PDF 2105.03404 ResMLP: Feedforward networks for image classification with data-efficient training](https://arxiv.org/pdf/2105.03404.pdf)

  | Model      | Params | Image resolution | Top1 Acc | ImageNet |
  | ---------- | ------ | ---------------- | -------- | -------- |
  | ResMLP12   | 15M    | 224              | 77.8     | [resmlp12_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/resmlp12_imagenet.h5) |
  | ResMLP24   | 30M    | 224              | 80.8     | [resmlp24_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/resmlp24_imagenet.h5) |
  | ResMLP36   | 116M   | 224              | 81.1     | [resmlp36_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/resmlp36_imagenet.h5) |
  | ResMLP_B24 | 129M   | 224              | 83.6     | [resmlp_b24_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/resmlp_b24_imagenet.h5) |
  | - imagenet22k | 129M   | 224              | 84.4     | [resmlp_b24_imagenet22k.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/resmlp_b24_imagenet22k.h5) |
## GMLP
  - [Keras GMLP](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/mlp_family#gmlp) includes implementation of [PDF 2105.08050 Pay Attention to MLPs](https://arxiv.org/pdf/2105.08050.pdf).

  | Model      | Params | Image resolution | Top1 Acc | ImageNet |
  | ---------- | ------ | ---------------- | -------- | -------- |
  | GMLPTiny16 | 6M     | 224              | 72.3     |          |
  | GMLPS16    | 20M    | 224              | 79.6     | [gmlp_s16_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/mlp_family/gmlp_s16_imagenet.h5) |
  | GMLPB16    | 73M    | 224              | 81.6     |          |
## LeViT
  - [Keras LeViT](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/levit) is for [PDF 2104.01136 LeViT: a Vision Transformer in ConvNet’s Clothing for Faster Inference](https://arxiv.org/pdf/2104.01136.pdf).

  | Model     | Params | Image resolution | Top1 Acc | ImageNet |
  | --------- | ------ | ---------------- | -------- | -------- |
  | LeViT128S | 7.8M   | 224              | 76.6     | [levit128s_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/levit/levit128s_imagenet.h5) |
  | LeViT128  | 9.2M   | 224              | 78.6     | [levit128_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/levit/levit128_imagenet.h5) |
  | LeViT192  | 11M    | 224              | 80.0     | [levit192_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/levit/levit192_imagenet.h5) |
  | LeViT256  | 19M    | 224              | 81.6     | [levit256_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/levit/levit256_imagenet.h5) |
  | LeViT384  | 39M    | 224              | 82.6     | [levit384_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/levit/levit384_imagenet.h5) |
## NFNets
  - [Keras NFNets](https://github.com/leondgarse/keras_cv_attention_models/tree/main/keras_cv_attention_models/nfnets) is for [PDF 2102.06171 High-Performance Large-Scale Image Recognition Without Normalization](https://arxiv.org/pdf/2102.06171.pdf).

  | Model       | Params | Image  resolution | Top1 Acc | Download |
  | ----------- | ------ | ----------------- | -------- | -------- |
  | NFNetL0     | 35.07M | 288               | 82.75    | [nfnetl0_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/nfnetl0_imagenet.h5) |
  | NFNetF0     | 71.5M  | 256               | 83.6     | [nfnetf0_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/nfnetf0_imagenet.h5) |
  | NFNetF1     | 132.6M | 320               | 84.7     | [nfnetf1_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/nfnetf1_imagenet.h5) |
  | NFNetF2     | 193.8M | 352               | 85.1     | [nfnetf2_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/nfnetf2_imagenet.h5) |
  | NFNetF3     | 254.9M | 416               | 85.7     | [nfnetf3_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/nfnetf3_imagenet.h5) |
  | NFNetF4     | 316.1M | 512               | 85.9     | [nfnetf4_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/nfnetf4_imagenet.h5) |
  | NFNetF5     | 377.2M | 544               | 86.0     | [nfnetf5_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/nfnetf5_imagenet.h5) |
  | NFNetF6 SAM | 438.4M | 576               | 86.5     | [nfnetf6_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/nfnetf6_imagenet.h5) |
  | NFNetF7     | 499.5M | 608               |          |          |
  | ECA_NFNetL0 | 24.14M | 288               | 82.58    | [eca_nfnetl0_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/eca_nfnetl0_imagenet.h5) |
  | ECA_NFNetL1 | 41.41M | 320               | 84.01    | [eca_nfnetl1_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/eca_nfnetl1_imagenet.h5) |
  | ECA_NFNetL2 | 56.72M | 384               | 84.70    | [eca_nfnetl2_imagenet.h5](https://github.com/leondgarse/keras_cv_attention_models/releases/download/nfnets/eca_nfnetl2_imagenet.h5) |
  | ECA_NFNetL3 | 72.04M | 448               |          |          |
***

# Other implemented tensorflow or keras models
  - [Github faustomorales/vit-keras](https://github.com/faustomorales/vit-keras)
  - [Github rishigami/Swin-Transformer-TF](https://github.com/rishigami/Swin-Transformer-TF)
  - [Github tensorflow/resnet_rs](https://github.com/tensorflow/tpu/tree/master/models/official/resnet/resnet_rs)
  - [Github google-research/big_transfer](https://github.com/google-research/big_transfer)
  - [perceiver_image_classification](https://keras.io/examples/vision/perceiver_image_classification/)
***
