# doom-neurobot
RL for classic Doom deathmatch

## Setup instructions

1) Install miniconda for python 3.7 on Ubuntu 18.04: https://docs.conda.io/en/latest/miniconda.html

2) Create conda env: `conda create -n doom-rl python=3.7`

3) Activate conda env: `conda activate doom-rl`

3.5) Install OpenCV from Conda: `conda install -c conda-forge opencv=4.1.0`

4) Install VizDoom system dependencies (from here https://github.com/mwydmuch/ViZDoom/blob/master/doc/Building.md#linux_deps):

```
# ZDoom dependencies
sudo apt install build-essential zlib1g-dev libsdl2-dev libjpeg-dev \
nasm tar libbz2-dev libgtk2.0-dev cmake git libfluidsynth-dev libgme-dev \
libopenal-dev timidity libwildmidi-dev unzip

# Boost libraries
sudo apt install libboost-all-dev

# Python 3 dependencies
sudo apt install python3-dev python3-pip
```

This list might not be entirely comprehensive, maybe on a clean system you need to install 1-2 more packages if you have compilation errors when installing VizDoom.

5) Clone the repo: `git clone https://github.com/alex-petrenko/doom-neurobot.git`

6) Install python dependencies: `pip install -r requirements.txt`

7) Currently a custom version of Ray is used that is installed via "dev-setup" procedure (from here: https://ray.readthedocs.io/en/latest/rllib-dev.html#development-install)

We first install "base" ray package and then replace the Python code of RLLIB inside our conda env.

```
# install Ray dev package for binary compatibility
pip install ./setup/ray-0.8.0.dev2-cp37-cp37m-manylinux1_x86_64.whl

# clone forked custom Ray repo
cd ~/
git clone https://github.com/alex-petrenko/ray.git
cd ray

# switch to dev branch with latest patches
git checkout doom_bot_project

# replace python files in the installed package with files from this folder (actually creats symlinks):
cd python/ray
./setup-dev.py

# say Y to everything!
```

## Testing the installation

From the doom-neurobot root, run the train script:

```
python -m train_rllib -f=experiments/doom-appo.yaml --experiment-name=test-appo
```

The model is checkpointed every 20 iterations, so wait at least a few minutes before stopping. Then you can see the agent's performance with:

```
python -m enjoy_rllib ~/ray_results/test-appo/CUSTOM_APPO_doom_battle_hybrid_0_2019-07-25_17-33-223ci_8euu/checkpoint_20/checkpoint-20 --run APPO
```

## Working with PyTorch implementation

```
python -m train_pytorch --algo=PPO --env=doom_basic --experiment=doom-test-ppo --recurrence=16
python -m enjoy_pytorch --algo=PPO --env=doom_basic --experiment=doom-test-ppo --fps=100
```


## Working with external memory

Start an experiment with Minigrid environment that requires memory:

```
python -m train_pytorch --algo=PPO --train_for_env_steps=300000000 --env MiniGrid-MemoryS7-v0 --use_rnn False --mem_size 4 --experiment test_memory_minigrid
python -m enjoy_pytorch --algo=PPO --env MiniGrid-MemoryS7-v0 --experiment test_memory_minigrid --fps=3
```

Start an experiment with VizDoom environment that requires memory:

```
python -m train_pytorch --algo=PPO --train_for_env_steps=1000000000 --prior_loss_coeff=0.005 --reward_scale=0.5 --env doom_two_colors_easy --use_rnn True --mem_size 4 --experiment test_memory_doom
python -m enjoy_pytorch --algo=PPO --env doom_two_colors_easy --experiment test_memory_doom --fps=35

```

Run a gridsearch:

```
python -m runner.run --run=memory_minigrid --runner=processes
```

Multuple pre-maid gridsearch configurations can be found in the repo (memory_minigrid.py, memory_doom.py)
Modify these files to change the algorithm parameters, as well as runner parameters (e.g. number of GPUs)
