# ManiSkill Baselines 使用指南

所有命令在 `~/WorkXCJ/ManiSkill/examples/baselines/` 对应子目录下执行。

---

## 1. RL（不需要 demo 数据，从零训练）

### 1.1 PPO

```bash
cd ppo

# 训练（state-based）
python ppo.py --env_id="PushCube-v1" --num_envs=64 --total_timesteps=1000000 --control-mode="pd_ee_delta_pos"

# 训练（visual-based，用摄像头图像）
python ppo_rgb.py --env_id="PushCube-v1" --num_envs=64 --total_timesteps=1000000 --control-mode="pd_ee_delta_pos"

# 评估并保存视频
python ppo.py --evaluate --env_id="PushCube-v1" --control-mode="pd_ee_delta_pos" \
  --checkpoint="runs/<run_name>/final_ckpt.pt"

# 查看结果
xdg-open runs/<run_name>/test_videos/0.mp4
```

### 1.2 SAC

```bash
cd sac

# 训练
python sac.py --env_id="PushCube-v1" --num_envs=32 --total_timesteps=500000 \
  --utd=0.5 --buffer_size=500000 --control-mode="pd_ee_delta_pos"

# 评估
python sac.py --evaluate --env_id="PushCube-v1" --control-mode="pd_ee_delta_pos" \
  --checkpoint="runs/<run_name>/final_ckpt.pt"
```

### 1.3 TD-MPC2

```bash
cd tdmpc2

# 训练
python train.py --env_id="PushCube-v1" --num_envs=32 --total_timesteps=500000

# 评估
python evaluate.py --checkpoint="runs/<run_name>/checkpoint.pt"
```

### 1.4 Stable Baselines3（最简单但最慢）

```bash
cd stable_baselines3
python example.py
```

---

## 2. 模仿学习（需要先下载 demo 数据）

### 2.1 下载 demo 数据

```bash
# 下载指定任务的 demo（state-based）
python -m mani_skill.utils.download_demo -e "PickCube-v1" -o state

# demo 保存位置
# ~/.maniskill/demos/<env_id>/motionplanning/trajectory.state.<control_mode>.physx_cpu.h5
```

### 2.2 ACT（Action Chunking with Transformers）

```bash
cd act

# 训练（100 条 demo，PickCube）
python train.py --env-id PickCube-v1 \
  --demo-path ~/.maniskill/demos/PickCube-v1/motionplanning/trajectory.state.pd_ee_delta_pos.physx_cpu.h5 \
  --control-mode "pd_ee_delta_pos" --sim-backend "physx_cpu" \
  --num_demos 100 --max_episode_steps 100 --total_iters 30000

# 训练（visual-based，RGB 图像输入）
python train_rgbd.py --env-id PickCube-v1 \
  --demo-path ~/.maniskill/demos/PickCube-v1/motionplanning/trajectory.rgb.pd_ee_delta_pos.physx_cpu.h5 \
  --control-mode "pd_ee_delta_pos" --num_demos 100 --total_iters 30000

# 模型保存在 runs/<run_name>/checkpoints/
```

### 2.3 Behavior Cloning

```bash
cd bc

# 训练（state-based）
python bc.py --env-id PickCube-v1 \
  --demo-path ~/.maniskill/demos/PickCube-v1/motionplanning/trajectory.state.pd_ee_delta_pos.physx_cpu.h5 \
  --control-mode "pd_ee_delta_pos" --num_demos 100
```

### 2.4 Diffusion Policy

```bash
cd diffusion_policy

# 训练
python train.py --env-id PickCube-v1 \
  --demo-path ~/.maniskill/demos/PickCube-v1/motionplanning/trajectory.state.pd_ee_delta_pos.physx_cpu.h5 \
  --control-mode "pd_ee_delta_pos" --num_demos 100
```

---

## 3. 通用说明

### 可用环境

| 任务 | env_id | 说明 |
|------|--------|------|
| 推方块 | PushCube-v1 | 最简单，PPO 1 分钟 |
| 抓方块 | PickCube-v1 | 中等难度 |
| 堆方块 | StackCube-v1 | 较难 |
| 插钉 | PegInsertionSide-v1 | 高难度 |
| 拉/画 | PullCube-v1, PushT-v1, DrawTriangle-v1 | 其他 |

### 控制模式

| 模式 | 说明 |
|------|------|
| `pd_ee_delta_pos` | 末端执行器增量位置（推荐，多数 baseline 用这个） |
| `pd_ee_delta_pose` | 末端执行器增量位姿（含旋转） |
| `pd_joint_delta_pos` | 关节增量位置 |

### 训练产出

- 模型权重：`runs/<run_name>/checkpoints/` 或 `runs/<run_name>/final_ckpt.pt`
- 评估视频：`runs/<run_name>/videos/` 或 `test_videos/`
- TensorBoard 日志：`runs/<run_name>/`

### 依赖安装

各 baseline 可能需要额外依赖：
```bash
pip install tensorboard wandb
# PPO/SAC 已包含在 mani_skill 环境中
# ACT/BC/Diffusion Policy 参考各自目录下的 setup.py
```
