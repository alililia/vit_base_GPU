# Contents

<!-- TOC -->

- [Contents](#contents)
- [Vit_base Description](#vit-description)
- [Model architecture](#model-architecture)
- [Dataset](#dataset)
- [Characteristic](#characteristic)
    - [Mixed precision](#mixed-precision)
- [Environment requirements](#environmental-requirements)
- [Quick start](#quick-start)
- [Script description](#script-description)
    - [Script and code structure](#script-and-code-structure)
    - [Script parameter](#script-parameters)
    - [Training process](#training-process)
        - [Standalone training](#standalone-training)
        - [Distributed training](#distributed-training)
    - [Evaluation process](#evaluation-process)
        - [Evaluation](#evaluation)
    - [Export process](#export-process)
        - [Export](#export)
    - [Inference process](#inference-process)
        - [Inference](#inference)
- [Model description](#model-description)
    - [Performance](#performance)
        - [Evaluation performance](#evaluation-performance)
            - [Vit_base on CIFAR-10](#vit_base-on-cifar-10)
        - [Inference performance](#inference-performance)
            - [Vit_base_on CIFAR-10](#vit_base-on-cifar-10)
- [ModelZoo homepage](#modelzoo-home-page)

<!-- TOC -->

# ViT description

Transformer architecture has been widely used in the field of natural language processing. The author of this model found that the Vision Transformer (ViT) model is not necessary to use CNN in the field of computer vision. When it has directly applied to the image block sequence for image classification, it can also be comparable to the current convolutional network accuracy.

[Paper](https://arxiv.org/abs/2010.11929): Dosovitskiy, A. , Beyer, L. , Kolesnikov, A. , Weissenborn, D. , & Houlsby, N.. (2020). An image is worth 16x16 words: transformers for image recognition at scale.

# Model architecture

The overall network architecture is as follows: [Link](https://arxiv.org/abs/2010.11929)

# Dataset

Dataset used: [CIFAR-10](<http://www.cs.toronto.edu/~kriz/cifar.html>)

- Dataset size: 175M, a total of 10 categories, 60,000 color images
    - Training: 146M, a total of 50,000 images
    - Test: 29M, a total of 10,000 images
- Data format: Binary file
    - Note: The data will be processed in src/dataset.py.

# Characteristic

## Mixed precision

Adopt [mixed precision](https://www.mindspore.cn/docs/programming_guide/zh-CN/r1.3/enable_mixed_precision.html) is the training method uses single-precision and half-precision data to improve the training speed of deep learning neural networks while maintaining the network accuracy that can be achieved by single-precision training. Mixed-precision training increases computing speed and reduces memory usage while supporting training larger models or achieving larger batches of training on specific hardware.

# Environmental requirements

- Hardware（GPU）
    - Use GPU to build a hardware environment.
- Framework
    - [MindSpore](https://www.mindspore.cn/install/en)
- For details, please refer to the following resources:
    - [MindSpore version](https://www.mindspore.cn/tutorials/zh-CN/r1.3/index.html)
    - [MindSpore Python API](https://www.mindspore.cn/docs/api/zh-CN/r1.3/index.html)

# Quick start

After installing MindSpore through the official website，you can follow the steps below for training and evaluation，in particular，before training, you need to download the official baseImageNet21k pre-trained model [ViT-B_16](https://console.cloud.google.com/storage/vit_models/) , and convert it to the ckpt format model supported by MindSpore，named "cifar10_pre_checkpoint_based_imagenet21k.ckpt"，place the training set and test set data in the same level directory:
Note: you can use .npz file, but only for ViT-base-16. For that, type the path to .npz in `config.py`.

- GPU

  ```python
  # Run standalone training example
  # GPU
  bash ./scripts/run_standalone_train_gpu.sh [DATASET_NAME] [DEVICE_ID] [LR_INIT] [LOGS_CKPT_DIR]

  # Run the distributed training example
  # GPU
  bash ./scripts/run_distribute_train_gpu.sh [DATASET_NAME] [DEVICE_NUM] [LR_INIT] [LOGS_CKPT_DIR]

  # Run evaluation example
  # GPU
  bash ./scripts/run_standalone_eval_gpu.sh [DATASET_NAME] [DEVICE_ID] [CKPT_PATH]
  ```

  For distributed training, you need to create a configuration file in JSON format in advance.

  Please follow the instructions in the link below：

  <https://gitee.com/mindspore/models/tree/master/utils/hccl_tools.>

- Train at ModelArts (If you want to run on modelarts, you can refer to the following documents [modelarts](https://support.huaweicloud.com/modelarts/))

    - Use Doka to train the CIFAR-10 dataset on ModelArts

      ```python
      # (1) Set the AI engine to MindSpore on the web page
      # (2) Set up on the web"ckpt_url=obs://path/pre_ckpt/"（The pre-trained model is named "cifar10_pre_checkpoint_based_imagenet21k.ckpt"）
      #     Set up on the web "modelarts=True"
      #     Set other parameters on the web page
      # (3) Upload your data set to the S3 bucket
      # (4) Set your code path on the web page as "/path/vit_base"
      # (5) Set the startup file on the web page as "train.py"
      # (6) Set the "training dataset on the web page（path/dataset/cifar10/cifar-10-batches-bin/", "Training output file path", "job log path", etc.
      # (7) Create training job
      ```

# Script description

## Script and code structure

```bash
├── vit_base
    ├── README.md                              // vit_base related description (ENG)
    ├── README_CN.md                           // vit_base related description (CN)
    ├── scripts
    │   ├──run_distribute_train_gpu.sh       // GPU distribute train shell script
    │   ├──run_standalone_eval_gpu.sh          // GPU standalone evaluation shell script
    │   └──run_standalone_train_gpu.sh         // GPU standalone training shell script
    ├── src
    │   ├──config.py                           // parameter configuration
    │   ├──dataset.py                          // create dataset
    │   ├──modeling_ms.py                      // vit_base architecture
    │   ├──net_config.py                       // structure model parameter configuration
    │   └──npz_converter.py                    // converting .npz checkpoints into .ckpt
    ├── eval.py                                // evaluation script
    ├── export.py                              // export the checkpoint file to AIR/MINDIR
    └── train.py                               // training script
```

## Script parameters

Training parameters and evaluation parameters can be configured at the same time in `config.py`.

- Configure vit_base and CIFAR-10 datasets.

  ```python
  'name':'cifar10'         # Dataset name
  'pre_trained':True       # Whether to train based on a pre-trained model
  'num_classes':10         # Number of dataset classes
  'lr_init':0.013          # Initial learning rate
  'batch_size':32          # Training batch size
  'epoch_size':60          # Total number of training epochs
  'momentum':0.9           # Momentum
  'weight_decay':1e-4      # Weight decay value
  'image_height':224       # The height of the image input to the model
  'image_width':224        # The width of the image input to the model
  'data_path':'/dataset/cifar10/cifar-10-batches-bin/'     # Absolute full path of the training data set
  'val_data_path':'/dataset/cifar10/cifar-10-verify-bin/'  # Evaluate the absolute full path of the data set
  'device_target':'Ascend' # Operating platform
  'device_id':0            # The device ID used for training or evaluating the data set, which can be ignored during distributed training
  'keep_checkpoint_max':2  # Save up to 2 ckpt model files
  'checkpoint_path':'/dataset/cifar10_pre_checkpoint_based_imagenet21k.ckpt'  # Save the absolute full path of the pre-trained model
  # optimizer and lr related
  'lr_scheduler':'cosine_annealing'
  'T_max':50
  ```

For more configuration details, please refer to the script `config.py`.

## Training process

### Standalone training

- GPU

  ```bash
  # GPU
  bash ./scripts/run_standalone_train_gpu.sh [DATASET_NAME] [DEVICE_ID] [LR_INIT] [LOGS_CKPT_DIR]
  ```

  The above python command will run in the background, you can view the result through the generated train.log file.

- After training, you can get the training results in the default script folder or chosen logs_dir：

  ```bash
  Load pre_trained ckpt: ./cifar10_pre_checkpoint_based_imagenet21k.ckpt
  epoch: 1 step: 1562, loss is 0.12886986
  epoch time: 289458.121 ms, per step time: 185.312 ms
  epoch: 2 step: 1562, loss is 0.15596801
  epoch time: 245404.168 ms, per step time: 157.109 ms
  {'acc': 0.9240785256410257}
  epoch: 3 step: 1562, loss is 0.06133139
  epoch time: 244538.410 ms, per step time: 156.555 ms
  epoch: 4 step: 1562, loss is 0.28615832
  epoch time: 245382.652 ms, per step time: 157.095 ms
  {'acc': 0.9597355769230769}
  ```

### Distributed training

- GPU

  ```bash
  # GPU
  bash ./scripts/run_distribute_train_gpu.sh [DATASET_NAME] [DEVICE_NUM] [LR_INIT] [LOGS_CKPT_DIR]
  ```

  The above shell script will run distributed training in the background.

  After training, you can get the training results:

  ```bash
  Load pre_trained ckpt: ./cifar10_pre_checkpoint_based_imagenet21k.ckpt
  epoch: 1 step: 781, loss is 0.015172593
  epoch time: 195952.289 ms, per step time: 250.899 ms
  epoch: 2 step: 781, loss is 0.06709316
  epoch time: 135894.327 ms, per step time: 174.000 ms
  {'acc': 0.9853766025641025}
  epoch: 3 step: 781, loss is 0.050968178
  epoch time: 135056.020 ms, per step time: 172.927 ms
  epoch: 4 step: 781, loss is 0.01949552
  epoch time: 136084.816 ms, per step time: 174.244 ms
  {'acc': 0.9854767628205128}
  ```

  Note: If you want to validate model during training, set flag `--do_val=True` in `train.py`

## Evaluation process

### Evaluation

- Evaluate the CIFAR-10 dataset while running in the GPU environment

  ```bash
  # GPU
  bash ./scripts/run_standalone_eval_gpu.sh [DATASET_NAME] [DEVICE_ID] [CKPT_PATH]
  ```

# Model description

## Performance

### Evaluation performance

#### Vit_base on CIFAR-10

| Parameter                  |  GPU                                                   |
| -------------------------- | ---------------------------------------------------- |
| Model                      | vit_base   |
| Hardware                   | 1 Nvidia Tesla V100-PCIE, CPU 3.40GHz; 8 Nvidia RTX 3090, CPU 2.90GHz |
| Upload date                |  2021-11-29 |
| MindSpore version          |  1.5.0 |
| Dataset                    |  CIFAR-10            |
| Training parameters        |  epoch=60, batch_size=32, lr_init=0.0065 or 0.052（1 or 8 GPUs）|
| Optimizer                  |  Momentum |
| Loss function              |  Softmax cross entropy |
| Output                     |  Probability |
| Classification accuracy    |  Single card: 98.84%; Eight cards: 98.83% |
| Speed                      | Single card: 467 ms/step; Eight cards: 469 ms/step |
| Total time                 |  Single card: 12.1 hours/60 epochs; Eight cards: 1.52 hours/60 epochs |


# ModelZoo home page

  Please visit the official website [homepage](https://gitee.com/mindspore/models).
