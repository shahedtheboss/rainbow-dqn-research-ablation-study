# rainbow-dqn-research-ablation-study

Controlled ablation of the scripted guide used to train a Rainbow DQN agent for
mapless LiDAR navigation of a TurtleBot3 Burger (ROS 2 Humble, Gazebo).

Status: project scaffold only. The agent, environment, training and evaluation
code is being written from scratch.

## Layout

| Path | Contents |
|---|---|
| `rainbow_ablation/` | ROS 2 Python package; code in `rainbow_ablation/rainbow_ablation/` (`environment.py`, `agent.py`, `train.py`, `evaluate.py`) |
| `turtlebot3_simulations/`, `turtlebot3_msgs/` | Gazebo worlds, launch files and messages (copied from the earlier workspace) |
| `Graphs/` | Output figures (empty) |
| `results.csv` | Experiment results (empty) |
| `requirements.txt` | Python dependency versions |

## Build

The repo root is itself the colcon workspace. Build it in place:

    cd ~/rainbow-dqn-research-ablation-study && source /opt/ros/humble/setup.bash && colcon build --symlink-install

Then source `install/setup.bash` from this repo only. Do not also source another workspace that contains `turtlebot3_*` packages (e.g. `~/ros2_ws`), since the package names are the same.

## License

MIT, see `LICENSE`.
