# NaVILA Installation

Author: Richard Wang richard98hess444@gmail.com

### Required Repositories
* [NaVILA](https://github.com/AnjieCheng/NaVILA)
* [Habitat-Sim](https://github.com/facebookresearch/habitat-sim)
* [Habitat-Lab](https://github.com/facebookresearch/habitat-lab)

### 0. Folder Structure
```
navila_ws
  |__habitat-sim
  |__habitat-lab
  |__NaVILA
    |__evaluation/data
      |__datasets
        |__R2R_VLNCE_v1-3_preprocessed
        |__RxR_VLNCE_v0
      |__scene_datasets
        |__mp3d
```

### 1. Create NaVILA Env
Create workspace
```
mkdir navila_ws
cd navila_ws
```
Create virtual environment
```
conda create -n navila-eval python=3.10
conda activate navila-eval
```

### 2. Build Habitat-Sim from Source 
[Reference](https://github.com/facebookresearch/habitat-sim/blob/v0.1.7/BUILD_FROM_SOURCE.md)
Clone the repository and checkout to `v0.1.7`
```
git clone --branch stable https://github.com/facebookresearch/habitat-sim.git
cd habitat-sim
git checkout v0.1.7
```
Install requirements
```
pip install -r requirements.txt
```
Install dependencies
```
sudo apt install -y --no-install-recommends \
     libjpeg-dev \
     libglm-dev \
     libegl1-mesa-dev \
     mesa-utils \
     xorg-dev \
     freeglut3-dev
 ```
 Check gcc, g++ version
 ```
 gcc --version
 ```
 If not 11.x, switch to it. First, install the 11.x version.
 ```
 sudo apt install gcc-11 g++-11 
 ```
 Add them to the current selections
 ```
 sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 50 
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 40 
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-13 50 
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 40 
 ```
 Select gcc, g++ version
 ```
 sudo update-alternatives --config gcc 
sudo update-alternatives --config g++
```
Check again `gcc --version`, the output should be something similar
```
gcc (Ubuntu 11.5.0-1ubuntu1~24.04) 11.5.0
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE
```
Build Habitat-Sim, very **important** to use `--headless`
```
python setup.py install --headless
```

### 3. Build Habitat-Lab from Source
[Reference](https://github.com/jacobkrantz/VLN-CE?tab=readme-ov-file#setup)
Clone the repository and checkout to `v0.1.7`
```
cd ~/navila_ws
git clone --branch v0.1.7 git@github.com:facebookresearch/habitat-lab.git
cd habitat-lab
```
Remove `tensorflow==1.13.1` from `habitat_baselines/rl/requirements.txt`
Install requirements
```
python -m pip install -r requirements.txt
python -m pip install -r habitat_baselines/rl/requirements.txt
python -m pip install -r habitat_baselines/rl/ddppo/requirements.txt
python setup.py develop --all
```

### 4. Build up NaVILA
[Reference](https://github.com/AnjieCheng/NaVILA?tab=readme-ov-file#-evaluation)
Cloen the repository
```
cd ~/navila_ws
git clone https://github.com/AnjieCheng/NaVILA.git
cd NaVILA
```
Manually replace the `common.py` file in Habitat-Sim since the script from the authors is not working
```
cp evaluation/replace/common.py ../habitat-sim/habitat_sim/utils/common.py
```
Install VLN-CE dependencies
```
pip install -r evaluation/requirements.txt
```
Install VILA dependencies
```
# Install VILA (assum in root dir)
pip install -e .
pip install -e ".[train]"
pip install -e ".[eval]"

# Install HF's Transformers
pip install git+https://github.com/huggingface/transformers@v4.37.2
site_pkg_path=$(python -c 'import site; print(site.getsitepackages()[0])')
cp -rv ./llava/train/transformers_replace/* $site_pkg_path/transformers/
cp -rv ./llava/train/deepspeed_replace/* $site_pkg_path/deepspeed/
```

Fix WebDataset version for VLN-CE compatibility
```
pip install webdataset==0.1.103
```

### 5. Fix Pytorch and Flash Attention Version
#### Pytorch
[Reference](https://pytorch.org/get-started/previous-versions/)
Remove torch libraries
```
pip uninstall torch -y
pip uninstall torchvision -y
pip uninstall torchaudio -y
```
Install Pytorch 2.7 with Blackwell architecture GPU support
```
pip install torch==2.7.0 torchvision==0.22.0 torchaudio==2.7.0 --index-url https://download.pytorch.org/whl/cu128 
```
Check installation
```
import torch
torch.cuda.is_available()
torch.cuda.get_arch_list()
```
Output should include `sm_120`
```
True
['sm_75', 'sm_80', 'sm_86', 'sm_90', 'sm_100', 'sm_120', 'compute_120']
```

#### Flash Attention
Remove flash_attn library
```
pip uninstall flash_attn -y
```
Download the compiled flash attention library from [here](https://github.com/Zarrac/flashattention-blackwell-wheels-whl-ONLY-5090-5080-5070-5060-flash-attention-/releases/tag/FlashAttention). Download the `cp310` version (python 3.10) and install it
```
mv flash_attn-2.7.4.post1-rtx5090-torch2.7.0cu128cxx11abiTRUE-cp310-linux_x86_64.whl flash_attn-2.7.4.post1-0rtx5090torch270cu128cxx11abiTRUE-cp310-cp310-linux_x86_64.whl
pip install flash_attn-2.7.4.post1-0rtx5090torch270cu128cxx11abiTRUE-cp310-cp310-linux_x86_64.whl
```

#### Datasets
The dataset folder structure is shown [here](https://github.com/AnjieCheng/NaVILA?tab=readme-ov-file#data).
Download the `R2R` dataset from [here](https://github.com/jacobkrantz/VLN-CE?tab=readme-ov-file#episodes-room-to-room-r2r) and `RxR` dataset from [here](https://github.com/jacobkrantz/VLN-CE?tab=readme-ov-file#episodes-room-across-room-rxr).
Link the Matterport3D dataset from
```
/path_to_storage/Matterport3D/data/scene_datasets/mp3d/v1/scans
```
The best approach is
```
cd ~/navila_ws/NaVIAL/evaluation/data
mkdir -p scene_datasets/mp3d
ln -s /path_to_storage/Matterport3D/data/scene_datasets/mp3d/v1/scans ~/navila_ws/NaVILA/evaluation/data/scene_datasets/mp3d
```

### 6. Evaluation
Follow the instruction from [here](https://github.com/AnjieCheng/NaVILA/tree/main?tab=readme-ov-file#running-evaluation)

### 7. Others (some other notes)
#### A. Download Matterport3D dataset
Go to `/path_to_storage/Matterport3D`, create a python 2.7 environment and run
```
# requires running with python 2.7
python download_mp.py --task habitat -o data/scene_datasets/mp3d/
```

#### B. Convert `.glb` and `.navmesh` files
Habitat-Sim takes only `.glb` and `.navmesh` files for evaluation, which are not included from the original Matterport3D dataset. 
To get these two files, we have to get the `.glb` file firstly from the `.obj` file inside `matterport_mesh.zip`, then convert the `.glb` file to `.navmesh` file. We wrote two scripts for doing this convertion for 90 scenes (Contact the author for the code).

Copy the files to NaVILA repository
```
cd /path_to_storage/Matterport3D
cp gen_obj.py gen_mesh.py ~/navila_ws/NaVILA/evaluation/
```
**Important**: these scripts must run under `navila-eval` environment, which contains Habitat-Sim version 0.1.7
```
python gen_obj.py
python gen_mesh.py
```