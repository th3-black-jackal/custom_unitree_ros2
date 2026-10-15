# Unitree G1 Walking Simulation — MuJoCo + RL Gym
This guide documents a locally tested way to run a walking Unitree G1 in MuJoCo using the pretrained locomotion policy included with Unitree RL Gym.
The experiment is a standalone Python simulation. It is separate from the unitree_mujoco C++ simulator and does not yet integrate walking control with ROS 2 / SDK2.
https://github.com/unitreerobotics/unitree_rl_gym

## Tested setup
- Windows with WSL 2 and Ubuntu 22.04
- A working graphical display for Linux applications (WSLg)
- Python 3 virtual environment
- Python package mujoco==3.3.6
- PyTorch CPU build
- Pretrained policy: deploy/pre_train/g1/motion.pt

The exact Python, NumPy and PyTorch versions were not recorded during the successful test. These instructions reproduce the commands used, rather than claim a fully pinned or universally tested dependency set.

### 1. Clone the official repository
Run in an Ubuntu terminal:
```bash
cd ~
git clone https://github.com/unitreerobotics/unitree_rl_gym.git
cd ~/unitree_rl_gym
```
If the directory already exists, use it instead of cloning again.

### 2. Create a Python virtual environment
```bash
sudo apt update
sudo apt install -y python3-venv
cd ~/unitree_rl_gym
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Python dependencies
```bash
python -m pip install --upgrade pip
python -m pip install "mujoco==3.3.6" numpy pyyaml
python -m pip install torch --index-url https://download.pytorch.org/whl/cpu
```
Verify imports:
```bash
python -c "import mujoco, torch, numpy, yaml; print('MuJoCo:', mujoco.__version__); print('PyTorch:', torch.__version__); print('Environment OK')"
```

### 4. Check the G1 policy and configuration
```bash
ls ~/unitree_rl_gym/deploy/pre_train/g1/motion.pt
ls ~/unitree_rl_gym/deploy/deploy_mujoco/configs/g1.yaml
```
The included motion.pt is a pretrained walking policy, not a generic controller for arbitrary actions such as running.

### 5. Set Python's import path
From the repository root:
```bash
cd ~/unitree_rl_gym
source .venv/bin/activate
export PYTHONPATH="$PWD:${PYTHONPATH:-}"
Verify that Python finds the legged_gym module:
python -c "from legged_gym import LEGGED_GYM_ROOT_DIR; print(LEGGED_GYM_ROOT_DIR)"
```
This addresses the observed error ModuleNotFoundError: No module named 'legged_gym'.

### 6. Start the walking simulation
```bash
python deploy/deploy_mujoco/deploy_mujoco.py g1.yaml
```
A MuJoCo viewer should open and the G1 should walk using the pretrained policy. The successful local test used this command.

### 7. Extend the run duration (optional)
Edit:
```bash
nano ~/unitree_rl_gym/deploy/deploy_mujoco/configs/g1.yaml
```
Increase simulation_duration if it is present; for example:
```bash
simulation_duration: 3600.0
```
Save with Ctrl+O, Enter, Ctrl+X. Restart the script to apply the change. This sets a simulation-time limit; the corresponding real-world duration can depend on simulation speed.

### 8. Launch again with the shared-project script
From the root of the shared project containing scripts/start_g1.sh:
```bash
bash scripts/start_g1.sh
```
The script assumes that unitree_rl_gym and its .venv have already been prepared under ~/unitree_rl_gym. To use another location, pass it as the first argument:
```bash
bash scripts/start_g1.sh /path/to/unitree_rl_gym
```

Troubleshooting
No module named 'mujoco'	- Activate the correct .venv and install Python MuJoCo there. The C++ installation does not provide the Python package.
No module named 'legged_gym'	- Run from the repository root and export PYTHONPATH="$PWD:${PYTHONPATH:-}".
No graphical window	- Check that your WSL installation can display Linux GUI applications.
Viewer closes after some time -	Check simulation_duration and inspect terminal output for errors.
Missing motion.pt	- Confirm that the official Unitree RL Gym repository was cloned completely.


## Research scope and limitations
Confirmed in the local test: G1 walking in the standalone Python MuJoCo simulation with the pretrained policy.
Separately confirmed in the C++ environment: the SDK2-based MuJoCo simulator publishes telemetry readable from ROS 2 (/lowstate). This is not evidence that the Python walking simulation is publishing its state to ROS 2.
Potential later work includes adding a Python-side ground-truth telemetry exporter, a separate mock monitoring feed, and a controlled integrity experiment comparing actual simulated movement with reported movement. Such a setup would be a conceptual laboratory test, not proof of a vulnerability in the physical G1 or the SDK2 protocol.
References
- Unitree RL Gym https://github.com/unitreerobotics/unitree_rl_gym
- Unitree MuJoCo https://github.com/unitreerobotics/unitree_mujoco
- Unitree ROS 2 https://github.com/unitreerobotics/unitree_ros2
- MuJoCo documentation https://mujoco.readthedocs.io/
