# start_g1.sh — quick launcher
This is a readable Markdown companion to [`../scripts/start_g1.sh`](../scripts/start_g1.sh). Use the .sh file to execute the script; Markdown cannot run as a shell script.
What it does
1. Uses ~/unitree_rl_gym by default, or accepts a custom path.
2. Checks that the deployment script, Python virtual environment and G1 policy are present.
3. Activates the virtual environment.
4. Exposes the repository root on PYTHONPATH so legged_gym can be imported.
5. Starts the MuJoCo walking simulation with g1.yaml.
   
##Usage
From the root of the shared project:
```bash
bash scripts/start_g1.sh
```

Or with a custom Unitree RL Gym path:
```bash
bash scripts/start_g1.sh ~/another_folder/unitree_rl_gym
```

Script source
```bash
#!/usr/bin/env bash
# Start the standalone Unitree G1 walking policy in Python MuJoCo.
set -euo pipefail

RL_GYM_DIR="${1:-$HOME/unitree_rl_gym}"
if [[ ! -f "$RL_GYM_DIR/deploy/deploy_mujoco/deploy_mujoco.py" ]]; then
  echo "Error: Unitree RL Gym not found at: $RL_GYM_DIR" >&2
  echo "Usage: bash scripts/start_g1.sh [path/to/unitree_rl_gym]" >&2
  exit 1
fi
if [[ ! -f "$RL_GYM_DIR/.venv/bin/activate" ]]; then
  echo "Error: Python virtual environment not found at: $RL_GYM_DIR/.venv" >&2
  echo "Follow docs/G1_WALKING_SIMULATION.md first." >&2
  exit 1
fi
if [[ ! -f "$RL_GYM_DIR/deploy/pre_train/g1/motion.pt" ]]; then
  echo "Error: Pretrained G1 policy motion.pt is missing." >&2
  exit 1
fi

cd "$RL_GYM_DIR"
# shellcheck disable=SC1091
source .venv/bin/activate
export PYTHONPATH="$PWD:${PYTHONPATH:-}"
exec python deploy/deploy_mujoco/deploy_mujoco.py g1.yaml
```


For installation steps and limitations, see [`G1_WALKING_SIMULATION.md`](G1_WALKING_SIMULATION.md).
