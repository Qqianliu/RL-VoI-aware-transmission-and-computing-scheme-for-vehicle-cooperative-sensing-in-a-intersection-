# Learning to Sense: Efficient Semantic Transmission for Cooperative Object Detection at Intersections

This repository provides the implementation of **RL-VoIS**, a reinforcement learning-based value-aware semantic transmission framework for cooperative object detection at intersections.

The implementation is built on top of **CoopDet3D: Deep Multi-Modal Cooperative 3D Object Detection of Traffic Participants Using Onboard and Roadside Sensors**. We use CoopDet3D as the cooperative perception backbone and extend it with Value of Information (VoI)-based transmission control and reinforcement learning-based upload decision-making.

## Code Overview

The main scripts are:

```text
Trained-models       # Pre-trained RL models
Training_coop.py     # Train the RL-VoIS agent
Test_coop.py         # Evaluate a trained RL-VoIS agent
```

RL-VoIS includes:

* image-based VoI calculation using YOLO object detection output;
* channel-aware vehicle-to-RSU transmission simulation;
* end-to-end energy consumption calculation;
* DDPG-based RL agent for adaptive upload decisions;
* cooperative 3D object detection evaluation using the CoopDet3D backbone.

## Step 1: Prepare CoopDet3D

First, clone and install the original CoopDet3D framework following its official instructions.

Our RL-VoIS scripts should be placed inside the CoopDet3D project directory.

A typical project structure is:

```text
coopdet3d/
├── mmdet3d/
├── tools/
├── configs/
├── data/
├── checkpoints/
├── RL-model/
├── Training_coop.py
├── Test_coop.py
└── yolov5s.pt
```

## Step 2: Prepare the Dataset

This work uses the **TUMTraf-V2X Cooperative Perception Dataset** adopted by CoopDet3D.

Please download either:

* `TUMTraf-V2X`
* or `TUMTraf-V2X-mini`

Then place the dataset under the `data/` directory following the CoopDet3D format:

```text
coopdet3d/
├── data/
│   ├── tumtraf_v2x_cooperative_perception_dataset/
│   │   ├── train/
│   │   ├── val/
```

Make sure the dataset path is correctly set in the CoopDet3D config file.

## Step 3: Prepare Models and Required Files

Before running RL-VoIS, please prepare:

1. A pretrained CoopDet3D checkpoint. (i chosed the best one)

```text
 coopdet3d_vi_cl_pointpillars512_2xtestgrid_yolos_transfer_learning_best.pth
```
2. YOLOv5 weights for image-based VoI calculation:

```text
yolov5s.pt
```

3. The provided trained RL-VoIS actor and critic models:

```text
actor_*.pth
critic_*.pth
```

4. Baseline accuracy files used for reward calculation:

```text
Reward-infras.txt
Reward-full-1.txt
```

Please update the file paths in `Test_coop.py` if needed, especially:

```python
actor_path = "path/to/actor.pth"
critic_path = "path/to/critic.pth"
```

## Step 4: Run RL-VoIS

We provide trained RL-VoIS models for reproducing the main results. Therefore, users can directly run the evaluation script without retraining the RL agent.

Run:

```bash
python Test_coop.py <CONFIG_FILE> \
    --mode pred \
    --checkpoint <COOPDET3D_CHECKPOINT> \
    --split val
```

Example:

```bash
python Test_coop.py configs/your_config.py \
    --mode pred \
    --checkpoint checkpoints/your_coopdet3d_checkpoint.pth \
    --split val
```

The script loads the trained RL-VoIS agent and evaluates adaptive upload decisions on the cooperative perception dataset.

## Optional: Train RL-VoIS from Scratch

If you want to retrain the RL-VoIS agent, run:

```bash
python Training_coop.py <CONFIG_FILE> \
    --mode pred \
    --checkpoint <COOPDET3D_CHECKPOINT> \
    --split val \
    --save_dir ./RL-model/
```

Example:

```bash
python Training_coop.py configs/your_config.py \
    --mode pred \
    --checkpoint checkpoints/your_coopdet3d_checkpoint.pth \
    --split val \
    --save_dir ./RL-model/
```

The trained actor and critic models will be saved in:

```text
./RL-model/
```

For reproducing the paper results, we recommend using the provided trained models.

## Output Files

The evaluation script generates `.txt` files for result analysis.

Typical output files include:

```text
Accuray-test.txt
Decision-test.txt
EP-Reward-test.txt
Energy-test.txt
Raw-energy-test.txt
Mini-energy-test.txt
Reward-test.txt
```

These files record:

* detection accuracy;
* upload decisions;
* reward values;
* RL-VoIS energy consumption;
* raw transmission energy consumption;
* minimum-energy reference values.

If training from scratch, additional training logs may be generated, such as:

```text
Accuray-scmch0.txt
Decision-scmch0.txt
EP-Reward-scmch0.txt
Energy-scmch0.txt
Raw-energy-scmch0.txt
Mini-energy-scmch0.txt
Reward-scmch.txt
```

## Reproducing the Main Results

To reproduce the main results:

1. Install CoopDet3D and its dependencies.
2. Download and prepare the TUMTraf-V2X dataset.
3. Download the pretrained CoopDet3D checkpoint.
4. Place `Test_coop.py`, `yolov5s.pt`, the trained RL-VoIS models, and the baseline accuracy files in the CoopDet3D project directory.
5. Update the paths of the actor, critic, dataset, and checkpoint.
6. Run `Test_coop.py`.
7. Use the generated `.txt` files to compute accuracy, data volume, energy consumption, and upload decision statistics.

## Citation

If you use this code, please cite our work:

```bibtex
@inproceedings{liu2026learning,
  title     = {Learning to Sense: Efficient Semantic Transmission for Cooperative Object Detection at Intersections},
  author    = {Liu, Qianqian and Bayhan, Suzan and Huang, Yanqiu and Heijenk, Geert},
  booktitle = {IEEE Vehicular Networking Conference (VNC)},
  year      = {2026}
}
```

Please also cite the original CoopDet3D work if you use the CoopDet3D codebase, pretrained models, or TUMTraf-V2X dataset.

## Acknowledgement

This implementation is based on the CoopDet3D framework and the TUMTraf-V2X cooperative perception dataset.

## Contact

For questions, please contact:

```text
Qianqian Liu
GitHub: https://github.com/Qqianliu
```
