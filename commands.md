# Commands

Every command assumes the repo is at `~/rainbow-dqn-research-ablation-study`. Run each block in a fresh terminal unless it says otherwise.

## 0. One-time setup (only once per machine)

The repo root is the colcon workspace. It holds rainbow_ablation, turtlebot3_gazebo, turtlebot3_msgs and the rest. Build in place; `build/`, `install/` and `log/` are git-ignored. Rebuild only when you change a package.xml, setup.py, a launch file or a world. Python edits take effect without a rebuild, thanks to `--symlink-install`.
```bash
cd ~/rainbow-dqn-research-ablation-study && source /opt/ros/humble/setup.bash && colcon build --symlink-install
```
Never source `~/ros2_ws/install/setup.bash` in the same terminal. It has packages with the same names, and whichever is sourced last wins.
Python packages, installed into the existing PyTorch venv:
```bash
source ~/pytorch-env/bin/activate && pip install -r ~/rainbow-dqn-research-ablation-study/requirements.txt
```

## 1. Terminal 1: start Gazebo

Keep this terminal open for the whole run.
```bash
cd ~/rainbow-dqn-research-ablation-study && source /opt/ros/humble/setup.bash && source install/setup.bash && export TURTLEBOT3_MODEL=burger && ros2 launch turtlebot3_gazebo turtlebot3_drl_stage4.launch.py
```

## 2. Terminal 2: environment for training and evaluation

Run this once in every new terminal before any `python3 -m rainbow_ablation...` command.
```bash
cd ~/rainbow-dqn-research-ablation-study && source /opt/ros/humble/setup.bash && source install/setup.bash && source ~/pytorch-env/bin/activate
```

## 3. Checks before the study

Check that the robot spawns at (0, 0, 0). The pose should be close to 0, 0, 0:
```bash
ros2 topic echo /odom --once --field pose.pose
```
Check the Gazebo reset service exists:
```bash
ros2 service list | grep -E "reset_simulation|reset_world|set_entity_state"
```
Smoke test, one per arm. These write to `results_smoke/` and never mix with real runs. Delete the folder afterwards.
```bash
python3 -m rainbow_ablation.train --arm N --episodes 5 --eval-every 5 --run-name smoke_N --results-dir results_smoke
```
```bash
python3 -m rainbow_ablation.train --arm G --episodes 5 --eval-every 5 --run-name smoke_G --results-dir results_smoke
```
What to check in each smoke folder:
- In `*_episodes.csv`, `guided_steps` is all 0 for N and mostly non-zero for G.
- `*_evals.csv` has 1 row.
- `*_training.png` opens.
- `*_run_meta.json` says `"status": "completed"`.

## 4. Before the first real run: commit and tag

The runs record the git commit. Commit everything first, so `git_dirty` is `false`.
```bash
git add -A && git commit -m "Pre-registration: final code and plan before study runs"
```
```bash
git tag prereg-v1 && git push && git push --tags
```

## 5. Training runs (Stage 1: 5 seeds x 2 arms)

Each run takes about 1.2 to 1.5 h and writes to `results/<arm>_s<seed>/`. Alternate the arms (N1, G1, N2, G2, ...) so that any drift over the days, such as heat or updates, hits both arms equally.
```bash
python3 -m rainbow_ablation.train --arm N --seed 1
```
```bash
python3 -m rainbow_ablation.train --arm G --seed 1
```
```bash
python3 -m rainbow_ablation.train --arm N --seed 2
```
```bash
python3 -m rainbow_ablation.train --arm G --seed 2
```
```bash
python3 -m rainbow_ablation.train --arm N --seed 3
```
```bash
python3 -m rainbow_ablation.train --arm G --seed 3
```
```bash
python3 -m rainbow_ablation.train --arm N --seed 4
```
```bash
python3 -m rainbow_ablation.train --arm G --seed 4
```
```bash
python3 -m rainbow_ablation.train --arm N --seed 5
```
```bash
python3 -m rainbow_ablation.train --arm G --seed 5
```
If a run crashes, keep the folder: it's data, so note it in your log. Re-run into a new name:
```bash
python3 -m rainbow_ablation.train --arm N --seed 1 --run-name N_s1_rerun
```

## 6. Final evaluation (after training)

Greedy test of one finished run. Change `N_s1` to the run you want:
```bash
R=N_s1 && python3 -m rainbow_ablation.evaluate --model results/$R/${R}_model_final.pth --episodes 50 --label $R --out-csv results/$R/${R}_final_eval.csv --plot results/$R/${R}_final_eval.png
```
Evaluate all 10 runs in one go:
```bash
for R in N_s1 G_s1 N_s2 G_s2 N_s3 G_s3 N_s4 G_s4 N_s5 G_s5; do python3 -m rainbow_ablation.evaluate --model results/$R/${R}_model_final.pth --episodes 50 --label $R --out-csv results/$R/${R}_final_eval.csv --plot results/$R/${R}_final_eval.png; done
```
Heuristic baselines (H and H_eta):
```bash
mkdir -p results/H && python3 -m rainbow_ablation.evaluate --policy heuristic --episodes 50 --label H --out-csv results/H/H_final_eval.csv --plot results/H/H_final_eval.png
```
```bash
mkdir -p results/H_eta && python3 -m rainbow_ablation.evaluate --policy heuristic --guide-noise 0.15 --seed 1 --episodes 50 --label H_eta --out-csv results/H_eta/H_eta_final_eval.csv --plot results/H_eta/H_eta_final_eval.png
```

## 7. Monitoring and troubleshooting

Follow a running run's progress: it shows the last 5 episodes.
```bash
tail -n 5 ~/rainbow-dqn-research-ablation-study/results/N_s1/N_s1_episodes.csv
```
Check Gazebo's real-time factor. It should stay near 1.0:
```bash
source /opt/ros/humble/setup.bash && gz stats
```
GPU or CPU load:
```bash
nvidia-smi
```
```bash
htop
```
Gazebo frozen or leftover processes from an old session. Stop them before relaunching:
```bash
pkill -f gzserver; pkill -f gzclient; pkill -f "rainbow_ablation.train"
```

## 8. Backups and git

Back up the results after each training day. The `.pth` files are git-ignored, so this is their only copy.
```bash
rsync -a ~/rainbow-dqn-research-ablation-study/results/ ~/rainbow_backups/results_$(date +%F)/
```
Commit the CSVs, figures and metadata after each day. The models stay local.
```bash
git add results && git commit -m "Add study runs $(date +%F)" && git push
```
Check what has changed:
```bash
git status && git log --oneline -5
```
