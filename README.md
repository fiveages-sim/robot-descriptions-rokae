# Rokae Robots

珞石（Rokae）机械臂描述包：单臂 AR5-SRS、双臂 AR5-CCS（含 CCS V2）。末端由 `ar5_ccs_description` 解析，亦可被其它机型（如 `fiveages_w2r`）复用。

| | |
|:---:|:---:|
| <img src=".images/rokae_ar5.png" alt="Rokae AR5 SRS" height="240"> | <img src=".images/ar5_ccs.png" alt="Rokae AR5 CCS" height="240"> |
| **[AR5-SRS](ar5_srs_description/)** | **[AR5-CCS / CCS V2](ar5_ccs_description/)** |

## Packages

| Package | Launch `robot` | 形态 | 说明 |
|---------|----------------|------|------|
| [ar5_srs_description](ar5_srs_description/) | `ar5_srs` | 单臂 / 双臂 | 左/右臂布局（`type:=left` / `right`） |
| [ar5_ccs_description](ar5_ccs_description/) | `ar5_ccs` | 双臂平台 | CCS / CCS V2（`variant:=ar5_ccs` / `ar5_ccs_v2`）；丰富末端 |

W2R 整机用同套短键挂臂：`arms:=ar5_ccs` / `ar5_ccs_v2` / `ar5_srs`，见 [fiveages_w2r README](../../humanoid/FiveAges/Gen2/fiveages_w2r_description/README.md)。

## Common Commands

### Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to <package> --symlink-install
source ~/ros2_ws/install/setup.bash
```

将 `<package>` 替换为 `ar5_srs_description` 或 `ar5_ccs_description`。

### Visualize

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=<robot> collider:=simple
```

常用参数：

- `robot:=ar5_srs` / `ar5_ccs`
- `variant:=ar5_ccs` / `ar5_ccs_v2`：仅 CCS 包选代际，与 `type` 正交
- `type:=<ee_key>`：对称末端（CCS / SRS 双臂）
- `type:=left` / `right`：单臂布局（不要同时当末端键用）
- `left_type:=… right_type:=…`：左右不同末端（**不要**再传 `type:=`）
- `collider:=simple`：规划用简化碰撞体（xacro 默认多为 `convex`）

末端键表与支持矩阵见 [ar5_ccs_description README §2](ar5_ccs_description/README.md#2-末端执行器)。

### OCS2 / Control

```bash
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=<robot>
ros2 launch ocs2_arm_controller demo.launch.py robot:=<robot>
```

默认硬件为 `mock_components`；真机加 `hardware:=real` 或 `real_usb`（细节见各包 README）。真机还需编译 `rokae_ros2_control`。

要让控制器跟踪虚拟 TCP，还需在 `control.patch` 里设置 `ocs2_arm_controller.ros__parameters.left_ee_frame: left_tcp_offset`（及 `right_ee_frame`）；只定义几何不会自动改 tip。

## Layout

```text
Rokae/
├── .images/
├── ar5_srs_description/
└── ar5_ccs_description/
```
