## Table of contents
   1. [Environment setup](#environment-setup)
   2. [Dataset setup](#dataset-setup)
   3. [Demo setup](#demo-setup)
   4. [DETIC setup](#install-detic)
   5. [Run!](#run)

## Environment Setup

```

git clone git@github.com:cpaxton/home-robot-dev.git
cd home-robot-dev
git checkout lang-rearrange-baseline

conda create -n home-robot python=3.10 cmake pytorch pytorch-cuda=11.7 -c pytorch -c nvidia -y
conda activate home-robot

git clone https://github.com/3dlg-hcvc/habitat-sim --branch floorplanner
cd habitat-sim
pip install -r requirements.txt
python setup.py install --headless
# (if the above commands runs out of memory) 
# python setup.py build_ext --parallel 8 install --headless

cd ..
git clone --branch v0.2.2 https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab 
pip install -r requirements.txt
python setup.py develop --all
pip install natsort scikit-image scikit-fmm pandas

cd ..
```

**[IMPORTANT]: Add habitat-lab path to PYTHONPATH**:

```
export PYTHONPATH=$PYTHONPATH:/path/to/home-robot-dev/habitat-lab/
```

[TEMPORARY]: Until we port to habitat v0.2.3.

> Comment out L36 in habitat-lab/habitat/tasks/rearrange/rearrange_sim.py

## Dataset Setup

### Scene dataset setup (v0.2.0)

```
wget --no-check-certificate https://aspis.cmpt.sfu.ca/projects/scenebuilder/fphab/v0.2.0/fphab-v0.2.0.zip -O src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/scene_datasets/fphab-v0.2.0.zip
unzip src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/scene_datasets/fphab-v0.2.0.zip -d src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/scene_datasets/
mkdir -p src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/scene_datasets/floorplanner
mv src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/scene_datasets/fphab-v0.2.0 src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/scene_datasets/floorplanner/v0.2.0
```


### Episode dataset setup

```
mkdir -p src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/episode_datasets/floorplanner/indoor_only/
wget https://www.dropbox.com/s/n1g1s6uvowo4tbm/v0.2.0_receptacle_cat_indoor_only_val.zip -O src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/episode_datasets/floorplanner/indoor_only/v0.2.0_receptacle_cat_indoor_only_val.zip
unzip src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/episode_datasets/floorplanner/indoor_only/v0.2.0_receptacle_cat_indoor_only_val.zip -d src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/episode_datasets/floorplanner/indoor_only/
```

[TEMPORARY] Floorplanner dataset episodes need to point to the right scene dataset config for scenes to load correctly:

> Add the below line after L93 of `habitat-lab/habitat/core/env.py`

```
self.current_episode.scene_dataset_config = "/path/to/home-robot-dev/src/home_robot/home_robot/experimental/theo/habitat_projects/datasets/scene_datasets/floorplanner/v0.2.0/hab-fp.scene_dataset_config.json"
```


## Demo setup

Update `GROUND_TRUTH_SEMANTICS:1` in `src/home_robot/home_robot/experimental/theo/habitat_projects/tasks/object_navigation/configs/agent/floorplanner_eval.yaml` and run the following:

```
cd src
python home_robot/home_robot/experimental/theo/habitat_projects/tasks/object_navigation/eval_scripts/eval_specific_episode.py
```

Results are saved to `src/home_robot/home_robot/experimental/theo/habitat_projects/tasks/object_navigation/datadump/images/debug`.

## Install Detic
```
cd /path/to/home-robot-dev/src/home_robot/home_robot/agent/perception/detection/detic
python -m pip install 'git+https://github.com/facebookresearch/detectron2.git'
pip install -r requirements.txt
mkdir models
wget https://dl.fbaipublicfiles.com/detic/Detic_LCOCOI21k_CLIP_SwinB_896b32_4x_ft4x_max-size.pth -O models/Detic_LCOCOI21k_CLIP_SwinB_896b32_4x_ft4x_max-size.pth

# Test it with
wget https://web.eecs.umich.edu/~fouhey/fun/desk/desk.jpg
python demo.py --config-file configs/Detic_LCOCOI21k_CLIP_SwinB_896b32_4x_ft4x_max-size.yaml --input desk.jpg --output out.jpg --vocabulary lvis --opts MODEL.WEIGHTS models/Detic_LCOCOI21k_CLIP_SwinB_896b32_4x_ft4x_max-size.pth
```

## Run

> Note: Ensure `GROUND_TRUTH_SEMANTICS:0` in `src/home_robot/home_robot/experimental/theo/habitat_projects/tasks/object_navigation/configs/agent/floorplanner_eval.yaml` to test DETIC perception.

```
cd /path/to/home-robot-dev/src

# Single episode to debug (ensuring )
python home_robot/home_robot/experimental/theo/habitat_projects/tasks/object_navigation/eval_scripts/eval_specific_episode.py

# Vectorized evaluation
sbatch home_robot/home_robot/experimental/theo/habitat_projects/tasks/object_navigation/eval_scripts/eval_vectorized.sh --config_path home_robot/home_robot/experimental/theo/habitat_projects/tasks/object_navigation/configs/agent/floorplanner_eval.yaml
```