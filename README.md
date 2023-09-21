## Panoptic Lifting for 3D Scene Understanding (CVPR2023 Highlight)

<hr/>

[**arXiv**](https://arxiv.org/abs/2212.09802) | [**Video**](https://youtu.be/QtsiL-6rSuM) | [**Project Page**](https://nihalsid.github.io/panoptic-lifting/) <br/>


This repository contains the implementation for the paper:

[**Panoptic Lifting for 3D Scene Understanding with Neural Fields**](https://arxiv.org/abs/2212.09802) by Yawar Siddiqui, Lorenzo Porzi, Samuel Rota Bulò, Norman Müller, Matthias Nießner, Angela Dai and Peter Kontschieder.

<div>
<div style="text-align: center">
  <img src="https://user-images.githubusercontent.com/932110/213286408-40bf7b98-a21b-4422-a0b5-e02f98beca9a.gif" alt="animated" />
</div>
<div style="margin-top: 5px;">
Given posed RGB images, Panoptic Lifting optimizes a panoptic radiance field which can be queried for color, depth, semantics, and instances for any point in space. Our method lifts noisy and view-inconsistent machine generated 2D segmentation masks into a consistent 3D panoptic radiance field, without requiring further tracking supervision or 3D bounding boxes.
</div>
</div>

## Dependencies

Install requirements from the project root directory:

```bash
pip install -r requirements.txt
```
In case errors show up for missing packages, install them manually.

## Structure

Overall code structure is as follows:

| Folder                 | Description                                                                                  |
|------------------------|----------------------------------------------------------------------------------------------|
| `config/`              | hydra default configs                                                                        |
| `data/`                | processed scenes for different datasets                                                      |
| `dataset/`             | pytorch `Dataset` implementations                                                            |
| `docs/`                | project webpage files                                                                        |
| `inference/`           | rendering and evaluation code for trained models                                             |
| `model/`               | implementations for radiance field representations, their renderers, and losses              |
| `pretrained-examples/` | pretrained models for scenes from scannet, replica, hypersim and self-captured (in-the-wild) |
| `resources/`           | mappings for scannet, 3d front, coco etc. and misc. mesh and blender files                   |
| `runs/`                | model training logs and checkpoints go here in addition to wandb;                            |
| `trainer/`             | pytorch-lightning module for training                                                        | 
| `util/`                | misc utilities for coloring, cameras, metrics, transforms, logging etc.                      |

## Pre-trained Models and Data

Download the pretrained models from [here](https://drive.google.com/file/d/1dvkfZ9beYVsxG_RftZ6aguHP6FLgjAC3/view?usp=sharing) and the corresponding processed scene data from [here](https://drive.google.com/file/d/1I6Y7IqSEmWl_T4CRUj-TmlUgejjexCa7/view?usp=sharing). Extract both zips in the project root directory, such that trained models are in `pretrained-examples/` directory and data is in `data/` directory. More [pretrained models](https://drive.google.com/file/d/1KsIq4MBIDIa08gREoRWePI0qXYJE_4HW/view?usp=sharing) and [data](https://drive.google.com/file/d/1ks0z8bJgaqDlEWFKQq-W6T6BS2UPj57F/view?usp=sharing) from ScanNet dataset are also provided.

### Running inference

To run inference use the following command

```bash
python inference/render_panopli.py <PATH_TO_CHECKPOINT> <IF_TEST_MODE>
```
This will render the semantics, surrogate-ids and visualizations to `runs/<experiment>` folder. When `<IF_TEST_MODE>` is `True`, the test set is rendered (input to the evaluation script later). When `False`, a custom trajectory stored in `data/<dataset_name>/<scene_name>/trajectories/trajectory_blender.pkl` is rendered.

Example:

```bash
python inference/render_panopli.py pretrained-examples/hypersim_ai001008/checkpoints/epoch=30-step=590148.ckpt False
```

### Evaluation

Use the `inference/evaluation.py` script for calculating metrics on the folder generated by the `inference/render_panopli.py` script (make sure you render the test set, since labels are not available for novel trajectories). 

Example:

```bash
python inference/evaluate.py --root_path data/replica/room_0 --exp_path runs/room_0_test_01171740_PanopLi_replicaroom0_easy-longshoreman
```

## Training

For launching training, use the following command from project root

```
python trainer/train_panopli_tensorf.py experiment=<EXPERIMENT_NAME> dataset_root=<PATH_TO_SCENE> wandb_main=True <HYPERPARAMETER_OVERRIDES>
```

Some example trainings:

#### ScanNet
```bash
python trainer/train_panopli_tensorf.py experiment=scannet042302 wandb_main=True batch_size=4096 dataset_root="data/scannet/scene0423_02/"
```
#### Replica
```bash
python trainer/train_panopli_tensorf.py experiment=replicaroom0 wandb_main=True batch_size=4096 dataset_root="data/replica/room_0/" lambda_segment=0.75
```
#### HyperSim
```bash
python trainer/train_panopli_tensorf.py experiment=hypersim001008 wandb_main=True dataset_root="data/hypersim/ai_001_008/" lambda_dist_reg=0 val_check_interval=1 instance_optimization_epoch=4 batch_size=2048 max_epoch=34 late_semantic_optimization=4 segment_optimization_epoch=24 bbox_aabb_reset_epochs=[2,4,8] decay_step=[16,32,48] grid_upscale_epochs=[2,4,8,16,20] lambda_segment=0.5
```
#### Self Captured
```bash
python trainer/train_panopli_tensorf.py experiment=itw_office0213meeting_andram wandb_main=True batch_size=8192
```

## Data Generation

Preprocessing scripts for data generation are provided in `dataset/preprocessing/` for Hypersim, Replica, ScanNet datasets and in-the-wild captures. For generating training labels, use our test-time augmented version of mask2former from [here](https://github.com/nihalsid/mask2former).  

**ScanNet**: For processing ScanNet folders you will need the scene folder containing ``.sens`` and the label zips.

**Replica**: Use the data provided by authors of SemanticNeRF and place it in `data/replica/raw/from_semantic_nerf` directory.

**HyperSim**: These scripts require the scene data in the `raw` folder in the `data/hypersim/` directory. For example, for processing hypersim scene `ai_001_008`, you'd need the raw data in `data/hypersim/raw/ai_001_008` directory. HyperSim raw data for a scene would typically contain the `_detail` and `images` directories.

**Self Captured Data**: The preprocessing scripts expect `data/itw/raw/<scene_name>` to have `color` directory and `transforms.json` file containing pose information (see InstantNGP to see how to generate this).

## License

The majority of _Panoptic Lifting_ is licensed under CC-BY-NC, however portions of the project are available under separate license terms: _TensoRF_ and _spherical_camera_ is licensed under the MIT license, _Panoptic Quality_ is license under Apache license.

## Citation

If you wish to cite us, please use the following BibTeX entry:

```BibTeX
@InProceedings{Siddiqui_2023_CVPR,
    author    = {Siddiqui, Yawar and Porzi, Lorenzo and Bul\`o, Samuel Rota and M\"uller, Norman and Nie{\ss}ner, Matthias and Dai, Angela and Kontschieder, Peter},
    title     = {Panoptic Lifting for 3D Scene Understanding With Neural Fields},
    booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)},
    month     = {June},
    year      = {2023},
    pages     = {9043-9052}
}
```
