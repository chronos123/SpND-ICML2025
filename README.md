# SpND: Spherical-Nested Diffusion Model for Panoramic Image Outpainting

This is the official repository of the paper "Spherical-Nested Diffusion Model for Panoramic Image Outpainting" in ICML 2025.

- [Paper](https://openreview.net/forum?id=JVDFkVf4QY)


## Overview

(a) Existing methods learn panoramic content by soft regularisations, in a *macro way*, thus outpainting irregular 3D arrangement and artefacts. (b) The proposed method incorporates the sphere nature within the design, thus in a *micro way*, which is able to outpaint realistic and high-quality  panoramic images. ![alt text](assets/Overview.png)


Network architecture.![alt text](assets/Network.png)


## Results

Quantitative results. ![alt text](assets/Table.jpg)

Qualitative results. ![alt text](assets/Subjective-result.png)

## :wrench: Requirements

```sh
conda env create -f env.yaml

conda activate spnd
```

## Data Prepare

### Matterport3D
- Download Matterport3D scans split (skybox image) : https://niessner.github.io/Matterport/.
- Run preprocessing/stitch_mp3d.py to get the Matterport3D dataset
- use 'find /path_to_Matterport3D/*.png > image_Matterport3D.txt' to get image index


### Structured3D
- Download the Structured3D dataset from "https://structured3d-dataset.org/"
- Use image_Structured3D.txt to get the training and testing images
- use 'find /path_to_Structured3D/*.png > image_Structured3D.txt' to get image index

### Prompts
The prompts for Matterport3D and Structured3D datasets are saved in Matter_prompt.json and Structure_prompts.json, respectively. Note that the prompts are generated by the BLIP-2 model.

## Pretrained Weights
- The weights for our SpND model without prompt is in the SpND folder and the weights for the SPND with prompt is in the SPND\_prompt folder.
- All the weights are given in the [Huggingface](https://huggingface.co/aberts/SpND/tree/main).
- Structure
    
    | Model_name | Description |
    | ---------------  | ------------------------------   |
    | SpND_Matterport3D.ckpt | SpND model trained on Matterport3D |
    | SpND_Structured3D.ckpt | SpND model trained on Structured3D |
    | SpND_prompt_Matterport3D.ckpt | SpND model with propmt as input trained on Matterport3D |
    | SpND_prompt_Structured3D.ckpt | SpND model with propmt as input trained on Structured3D |
    | SpND_prompt_pers_Structured3D.ckpt | SpND model with propmt as input and a perspective mask trained on Structured3D |


## <a name="train"></a>:computer: Train

First, download the [control_sd15_ini.ckpt](https://huggingface.co/aberts/SpND/tree/main) and put it in 'models/control_sd15_ini.ckpt'

### Without Prompts
```sh
CUDA_VISIBLE_DEVICES=0 python train.py --config models/SpND.yaml --data-file image_Matterport3D.txt --gpus 1 --max-epoch 100
```

### With Prompts
```sh
CUDA_VISIBLE_DEVICES=0 python train_prompt.py --config models/SpND.yaml --data-file image_Matterport3D.txt --mask-file masks/center_mask.png --prompt-file Matter_prompt.json --gpus 1 --max-epoch 100
```

### Chaneg the mask
- Modify the down_mask_path config models/SpND.yaml
- Set the --mask-file path for training


## <a name="inference"></a>:zap: Inference

### Without Prompts
The input mask should match the `down_mask_path` config in the model config file.
```sh
CUDA_VISIBLE_DEVICES=0 python inference_black_new__cfg_diff_mask.py --config models/SpND.yaml --ckpt <path_to_checkpoint> --data-path image_Matterport3D.txt --mask-path masks/center_mask.png --down-mask-path masks/8_down_center_mask.png
```

### With Prompts
The input mask should match the `down_mask_path` config in the model config file.
```sh
python -m torch.distributed.launch --nproc_per_node=1 multi_inference_black_new_cfg_prompt.py --config models/SpND.yaml --ckpt <path_to_checkpoint> --data-path image_Matterport3D.txt --prompt-file Matter_prompt.json --mask-file masks/center_mask.png --world-size 1
```

