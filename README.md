# Legged Loco
This repo is used to train low-level locomotion policy of Unitree Go2 and H1 in Isaac Lab.

<p align="center">
<img src="./src/go2_teaser.gif" alt="First Demo" width="45%">
&emsp;
<img src="./src/h1_teaser.gif" alt="Second Demo" width="45%">
</p>


## Installation
This branch is ported to **Isaac Sim 5.1 + Isaac Lab 2.3.2** (Python 3.11, PyTorch 2.7 / CUDA 12.8), which supports Ubuntu 24.04 and RTX 50-series (Blackwell) GPUs.

1. Create a Python 3.11 virtual environment.
    ```shell
    uv venv --python 3.11 .venv
    source .venv/bin/activate
    ```

2. Install PyTorch and Isaac Sim.
    ```shell
    uv pip install -U torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
    uv pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com --index-strategy unsafe-best-match
    ```

3. Clone and install the upstream Isaac Lab v2.3.2. The fixes from the previous Isaac Lab 1.1 fork are already included upstream.
    ```shell
    git clone -b v2.3.2 https://github.com/isaac-sim/IsaacLab.git
    cd IsaacLab
    ./isaaclab.sh -i none
    # flatdict needs pkg_resources at build time, which newer setuptools no longer ships
    echo "setuptools<70" > /tmp/build-constraints.txt
    uv pip install -e source/isaaclab --build-constraints /tmp/build-constraints.txt
    cd ..
    ```

4. Install the extension and the bundled rsl_rl of this repo.
    ```shell
    uv pip install -e {THIS_REPO_DIR}/isaaclab_exts/omni.isaac.leggedloco -e {THIS_REPO_DIR}/rsl_rl
    ```

5. Accept the Omniverse EULA once per shell (or answer the prompt on first launch).
    ```shell
    export OMNI_KIT_ACCEPT_EULA=YES
    ```

**Porting notes**
- `omni.isaac.lab*` imports were renamed to `isaaclab`, `isaaclab_tasks`, `isaaclab_assets`.
- The bundled rsl_rl (2.0.2) is not compatible with the TensorDict-based wrapper in `isaaclab_rl`, so the Isaac Lab 1.1 RSL-RL wrapper and configs are vendored in `omni.isaac.leggedloco.utils.rsl_rl_wrapper`.
- `g1_base` disables the `base_com` event that Isaac Lab 2.x added to the default `EventCfg`, to keep the original behavior.


## Usage
* train

    ```shell
    python scripts/train.py --task=go2_base --history_len=9 --run_name=XXX --max_iterations=2000 --save_interval=200 --headless

    python scripts/train.py --task=h1_base --run_name=XXX --max_iterations=2000 --save_interval=200 --headless
    ```

* test

    ```shell
    python scripts/play.py --task=go2_base_play --history_len=9 --load_run=RUN_NAME --num_envs=10
    python scripts/play.py --task=h1_base_play --load_run=RUN_NAME --num_envs=10
    ```

    Use `--headless` to enable headless mode. Add `--enable_cameras --video` for headless rendering and video saving.

## Add New Environments
You can add additional environments by placing them under `isaaclab_exts/omni.isaac.leggedloco/omni/isaac/leggedloco/config`.