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

This repo has the same package names as `~/ros2_ws/src/turtlebot3_*`, so build it in its own workspace, not next to them:

    mkdir -p ~/rainbow_ws/src && ln -s ~/rainbow-dqn-research-ablation-study ~/rainbow_ws/src/rainbow
    cd ~/rainbow_ws && source /opt/ros/humble/setup.bash && colcon build --symlink-install

## License

MIT, see `LICENSE`.
