# Rokae AR5-SRS Description

珞石 AR5 SRS 机械臂 URDF / ros2_control 描述包。模型在 **`xacro/`**。CCS 代际由 [`ar5_ccs_description`](../ar5_ccs_description/README.md) 的 `arms` 槽选择。

`config/ocs2/single.info` 以 W2R `fixed_base.info` 的单臂参数为基准，仅保留 7 个关节：预测时域 1.0 s，R scaling 为 0.05，对角值为 `[0.5, 0.5, 0.5, 0.5, 0.1, 0.1, 0.1]`，末端过程/终端权重分别为位置/姿态 `100/50`、`200/100`，软限速 ±1.07 rad/s。保留 `base_link` / `link7` 坐标系和原有的自碰撞关闭状态，不引入 CCS 腕部耦合或全身约束。

```bash
cd ~/ros2_ws && colcon build --packages-up-to ar5_srs_description --symlink-install
source install/setup.bash
```

## 1. 可视化与 Launch

通用入口：`robot_common_launch manipulator.launch.py robot:=ar5_srs`  
控制 / 真机：`ocs2_arm_controller demo.launch.py robot:=ar5_srs`  
OCS2 规划：`robot_common_launch manipulator_ocs2.launch.py robot_name:=ar5_srs`

| 场景 | 示例 |
|------|------|
| 双臂（默认） | `… manipulator.launch.py robot:=ar5_srs collider:=simple` |
| 单臂 | 加 `type:=left` 或 `type:=right` |
| 末端（对称） | 加 `type:=<key>`（如 `rg75`），见 [CCS §2](../ar5_ccs_description/README.md#2-末端执行器) |
| 末端（左右不同） | `left_type:=… right_type:=…`，**不要**再传 `type:=` |
| 真机 | 不支持；AR5-SRS 仅用于仿真和 mock |
| Mock 控制 | `demo.launch.py …`（默认 `hardware:=mock_components`） |

![Rokae AR5 SRS](../.images/rokae_ar5.png)

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_srs type:=left collider:=simple
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_srs type:=rg75 collider:=simple
```

W2R 挂 SRS：`arms:=ar5_srs`（见 [fiveages_w2r README](../../../humanoid/FiveAges/Gen2/fiveages_w2r_description/README.md)）。

## 2. 真机部署

```bash
colcon build --packages-up-to rokae_ros2_control --symlink-install
source install/setup.bash

ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_srs hardware:=real
ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_srs hardware:=real type:=rg75
```
