# Learning to Sense: Efficient Semantic Transmission for Cooperative Object Detection at Intersections

This repository contains the code of **RL-VoIS**, a reinforcement learning (DDPG) framework that decides when a vehicle uploads its sensing data to a roadside unit (RSU), based on the Value of Information (VoI) of camera images, the wireless channel and the energy cost. It is built on top of [CoopDet3D](https://github.com/tum-traffic-dataset/coopdet3d) and evaluated on the TUMTraf-V2X cooperative perception dataset.

## Research information

- **Paper**: Q. Liu, S. Bayhan, Y. Huang and G. Heijenk, "Learning to Sense: Efficient Semantic Transmission for Cooperative Object Detection at Intersections," *2026 IEEE Vehicular Networking Conference (VNC)*, Montreal, QC, Canada, 2026, pp. 1-8.
- **DOI**: [10.1109/VNC69225.2026.11629173](https://doi.org/10.1109/VNC69225.2026.11629173)
- **Author / contact**: Qianqian Liu, University of Twente (qianqian.liu@utwente.nl)

## Repository contents

| Path | Description |
|------|-------------|
| `Test_coop.py` | Evaluates a trained RL-VoIS agent (**use this to reproduce the results**) |
| `Training_coop.py` | Trains the RL-VoIS agent from scratch (optional) |
| `Trained-models/Alpha0` … `Alpha6` | Trained DDPG actor/critic weights, one folder per value of the reward weight α (`Alpha5-ac_th-infras` is an additional variant). All share the same network shape and can be used interchangeably with `Test_coop.py`. |
| `Reward-infras.txt` | Per-frame detection accuracy when only infrastructure data is used (baseline read by `Test_coop.py`, 400 frames) |
| `Reward-full-1.txt` | Per-frame detection accuracy when all vehicle data is uploaded (baseline read by `Test_coop.py`, 400 frames) |
| `mmdet3d/models2/` | Modified copy of CoopDet3D's `mmdet3d/models`, imported by both scripts (`from mmdet3d.models2 import build_coop_model`) |
| `docker/Dockerfile` | Docker environment used for the experiments (CoopDet3D's Dockerfile with small changes) |

## Requirements

- Linux with an **NVIDIA GPU** and CUDA (the code calls `torch.cuda.set_device(0)`; CPU-only is not supported). Experiments were run in the Docker image below.
- Software (installed by the Dockerfile): Python 3.9, PyTorch 1.10.1 + CUDA 11.3, mmcv-full 1.4.0, mmdet 2.20.0, torchpack, torchsparse, numba, pypcd, open3d.
- YOLOv5 from [ultralytics/yolov5](https://github.com/ultralytics/yolov5), commit `d3ea0df` (2022-08-17). Newer versions renamed `scale_coords`, which the scripts import.

## Step-by-step reproduction

### 1. Get CoopDet3D and add the RL-VoIS files

```bash
git clone https://github.com/tum-traffic-dataset/coopdet3d.git
git clone https://github.com/Qqianliu/rl-voi-aware-transmission-and-computing-scheme-for-vehicle-cooperative-sensing-in-a-intersection- rlvois

# copy the RL-VoIS files into the CoopDet3D root
cp -r rlvois/Test_coop.py rlvois/Training_coop.py rlvois/Reward-infras.txt rlvois/Reward-full-1.txt rlvois/Trained-models coopdet3d/
cp -r rlvois/mmdet3d/models2 coopdet3d/mmdet3d/
cp rlvois/docker/Dockerfile coopdet3d/docker/Dockerfile
cd coopdet3d
```

### 2. Get the data, checkpoint and YOLOv5 (reproduction package)

Download the reproduction package from the link below (to be added before the CodeCheck session):

> **Download link:** _will be added here_
 It consists of 7 zip files, `codecheck_data_part1of7.zip` … `codecheck_data_part7of7.zip` (1.36 GB in total). Unzip all of them **in the CoopDet3D root**:

```bash
cd coopdet3d
for f in /path/to/codecheck_data_part*of7.zip; do unzip -o "$f"; done
```

It contains exactly what `Test_coop.py` needs, already in the right place:

| Path | Content |
|------|---------|
| `data/tumtraf_v2x_cooperative_perception_dataset_mini/val/` | Validation split of TUMTraf-V2X-mini (images, labels) |
| `data/tumtraf_v2x_cooperative_perception_dataset_processed/val/` + `*.pkl` | Preprocessed point clouds and info files (output of CoopDet3D's `create_tumtraf_v2x_data.py`) |
| `weights/coopdet3d_vi_cl_pointpillars512_2xtestgrid_yolos_transfer_learning_best.pth` | Pretrained CoopDet3D checkpoint |
| `yolov5/`, `yolov5s.pt` | YOLOv5 code (commit `d3ea0df`) and weights for the VoI calculation |

**Important:** the `.pkl` info files store absolute paths starting with `/home/coopdet3d/`, so the CoopDet3D folder must be mounted at `/home/coopdet3d` (the Docker command below does this). With the package, steps 4 and 5 can be skipped.

### 3. Build and start the Docker environment

```bash
cd docker && docker build . -t coopdet3d && cd ..
nvidia-docker run -it -v `pwd`:/home/coopdet3d --shm-size 16g coopdet3d /bin/bash
```

Inside the container:

```bash
cd /home/coopdet3d
python setup.py develop
pip install -r yolov5/requirements.txt
export PYTHONPATH=/home/coopdet3d/yolov5:$PYTHONPATH   # needed because YOLOv5 uses "from utils..." imports
```

### 4. (Only without the package) Prepare the dataset

Download **TUMTraf-V2X** (or **TUMTraf-V2X-mini**) from the [TUMTraf dataset page](https://innovation-mobility.com/tumtraf-dataset), link it to `data/tumtraf_v2x_cooperative_perception_dataset` (with `train/` and `val/` inside), then run CoopDet3D's preprocessing:

```bash
mkdir -p data
ln -s /home/data/tumtraf_v2x_cooperative_perception_dataset data/tumtraf_v2x_cooperative_perception_dataset
python ./tools/create_tumtraf_v2x_data.py \
  --root-path /home/coopdet3d/data/tumtraf_v2x_cooperative_perception_dataset \
  --out-dir /home/coopdet3d/data/tumtraf_v2x_cooperative_perception_dataset_processed \
  --splits training,validation
```

### 5. (Only without the package) Download the checkpoint and YOLOv5

Download `coopdet3d_vi_cl_pointpillars512_2xtestgrid_yolos_transfer_learning_best.pth` from the [CoopDet3D README](https://github.com/tum-traffic-dataset/coopdet3d) (section "Pretrained Weights") and place it in `weights/`. Then:

```bash
git clone https://github.com/ultralytics/yolov5.git
cd yolov5 && git checkout d3ea0df && cd ..
wget https://github.com/ultralytics/yolov5/releases/download/v6.2/yolov5s.pt   # place in the CoopDet3D root
```

### 6. Run the evaluation

```bash
python Test_coop.py configs/tumtraf_v2x/det/transfusion/secfpn/cooperative/camera+lidar/yolov8/pointpillars.yaml \
  --mode pred \
  --checkpoint weights/coopdet3d_vi_cl_pointpillars512_2xtestgrid_yolos_transfer_learning_best.pth \
  --actor Trained-models/Alpha2/actor_ep5_step300.pth \
  --critic Trained-models/Alpha2/critic_ep5_step300.pth
```

Notes:
- `--mode pred` is required: it loads the CoopDet3D model.
- Any `Trained-models/AlphaX` folder can be passed to `--actor` / `--critic`.
- `--reward-infras` and `--reward-full` default to the two baseline files in the current directory.
- The random seed is fixed to 17 in `Test_coop.py`.

### 7. Outputs

The script writes these files to the working directory (one value per line):

| File | Content |
|------|---------|
| `Accuray-test.txt` | Detection accuracy per frame |
| `Decision-test.txt` | Upload decision per frame (1 = upload, 0 = no upload) |
| `Reward-test.txt`, `EP-Reward-test.txt` | Reward per step / per episode |
| `Energy-test.txt` | End-to-end energy of RL-VoIS |
| `Raw-energy-test.txt` | Energy when raw data is always transmitted |
| `Mini-energy-test.txt` | Minimum-energy reference |

Averaging these files gives the accuracy, upload ratio and energy figures reported in the paper.

## Optional: train from scratch

```bash
python Training_coop.py configs/tumtraf_v2x/det/transfusion/secfpn/cooperative/camera+lidar/yolov8/pointpillars.yaml \
  --mode pred \
  --checkpoint weights/coopdet3d_vi_cl_pointpillars512_2xtestgrid_yolos_transfer_learning_best.pth \
  --save_dir ./RL-model-new/
```

## Citation

```bibtex
@inproceedings{liu2026learning,
  title     = {Learning to Sense: Efficient Semantic Transmission for Cooperative Object Detection at Intersections},
  author    = {Liu, Qianqian and Bayhan, Suzan and Huang, Yanqiu and Heijenk, Geert},
  booktitle = {2026 IEEE Vehicular Networking Conference (VNC)},
  pages     = {1--8},
  year      = {2026},
  doi       = {10.1109/VNC69225.2026.11629173}
}
```

Please also cite CoopDet3D and the TUMTraf-V2X dataset if you use their code, checkpoints or data.

## Acknowledgement

This implementation is based on the [CoopDet3D](https://github.com/tum-traffic-dataset/coopdet3d) framework and the TUMTraf-V2X cooperative perception dataset. This work is part of the FNS 6G Dutch project.
