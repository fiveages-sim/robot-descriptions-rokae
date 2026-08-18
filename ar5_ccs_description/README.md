# Rokae AR5-CCS Description

珞石 AR5 CCS 双臂 URDF / ros2_control 描述包（同一包覆盖 CCS 与 CCS V2）。模型在 **`xacro/`**。OCS2 规划 URDF 由 `xacro/robot.xacro` 生成。`xacro/eefs.xacro` 亦被 `fiveages_w2r_description` 复用。

```bash
cd ~/ros2_ws && colcon build --packages-up-to ar5_ccs_description --symlink-install
source install/setup.bash
```

## 1. 可视化与 Launch

通用入口：`robot_common_launch manipulator.launch.py robot:=ar5_ccs`  
控制 / 真机：`ocs2_arm_controller demo.launch.py robot:=ar5_ccs`  
OCS2 规划：`robot_common_launch manipulator_ocs2.launch.py robot_name:=ar5_ccs`

| 场景 | 示例 |
|------|------|
| 双臂 CCS v1（默认） | `… manipulator.launch.py robot:=ar5_ccs collider:=simple` |
| 双臂 CCS V2 | 加 `variant:=ar5_ccs_v2` |
| 单臂 | 加 `type:=left` 或 `type:=right` |
| 末端（对称） | 加 `type:=<key>`（如 `rg75`、`ag2f90`），见 [§2](#2-末端执行器) |
| 末端（左右不同） | `left_type:=… right_type:=…`，**不要**再传 `type:=` |
| 真机 | `demo.launch.py … hardware:=real` 或 `real_usb`；见 [§2.1](#21-真机驱动方式) |
| Mock 控制 | `demo.launch.py …`（默认 `hardware:=mock_components`） |

![Rokae AR5 CCS](../.images/ar5_ccs.png)

**不对称末端示例**

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_ccs \
  variant:=ar5_ccs_v2 left_type:=rg75 right_type:=linkerhand_o7 collider:=simple
```

也可在 `robot_profile` / `robot.local.yaml` 的 `defaults.end_effectors.left` / `.right` 中配置（键名同 §2）。

## 2. 末端执行器

类型映射：`xacro/eef_type_resolver.xacro`；真机接线：`xacro/ros2_control/robot.xacro`。

### 2.1 真机驱动方式

臂体走 [`rokae_ros2_control`](https://github.com/fiveages-sim/rokae-ros2-control)。每侧末端可独立配置 `left_ee_drive_type` / `right_ee_drive_type`（launch 或 `robot.local.yaml`）。

| 驱动键 | 典型 launch | 物理链路 | 说明 |
|--------|-------------|----------|------|
| （空，默认） | `hardware:=real` | 臂体 Rokae 总线 | 夹爪 / 手随 Rokae 插件 |
| `usb` | `hardware:=real_usb`（默认） | 外接 **USB 串口** | [`modbus_ros2_control`](https://github.com/fiveages-sim/modbus-ros2-control)；口默认左 `/dev/ttyUSB0`、右 `/dev/ttyUSB2` |

夹爪**仅** Rokae 总线，没有外接 USB 夹爪路径。

### 2.2 类型与支持矩阵

`type` / `left_type` / `right_type` 与 `xacro/eef_type_resolver.xacro` 一致。列名对应 §2.1（`rokae 485` = 默认空键 / `hardware:=real`，臂体 RS485）。

| Type key | 产品 | `rokae 485` | `usb` |
|----------|------|:-------:|:-----:|
| `rg75` | Jodell RG75 | ✅ | — |
| `rg75_heavy` | Jodell RG75 + heavy pad | ✅ | — |
| `erg32` | Jodell ERG32（旋转 + 夹爪，2-DOF） | ✅ | — |
| `ag2f90`, `ag2f90_d` | ChangingTek AG2F90-D | ✅ | — |
| `ag2f90_c`, `ag2f90_c_soft` | ChangingTek AG2F90-C | ✅ | — |
| `ag2f120s` | ChangingTek AG2F120S | ✅ | — |
| `ag2f120s_d` | ChangingTek AG2F120S-D | ✅ | — |
| `linkerhand_o6` / `l6` / `o7` | LinkerHand | ✅ | ✅ |

要点：

- **夹爪** — 只走臂体 Rokae 插件（`rokae_hardware.xacro` 的 `gripper_type`）。`hardware:=real_usb` 不会给夹爪开 USB 系统。
- **LinkerHand** — 默认随 Rokae 总线；`hardware:=real_usb` 或 `left_ee_drive_type:=usb` 时改走外接 Modbus（`external_ee_systems.xacro`），Rokae 侧 `hand_type` 置空以免双驱动。
- **`type:=left` / `right`** — 只选单臂拓扑，不是末端键。

仿真 / mock 下上表类型均可展开 URDF；真机列只表示 `hardware:=real` / `real_usb` 的接线。

## 3. 平台与代际

### 3.1 `variant:=ar5_ccs` / `ar5_ccs_v2`

同一描述包内切换 CCS 代际，与末端 `type` 正交：

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_ccs \
  variant:=ar5_ccs_v2 type:=rg75 collider:=simple
```

包默认 `variant:=ar5_ccs`（v1），避免 `robot:=ar5_ccs` 无参时静默改代。W2R 整机默认 `arms:=ar5_ccs_v2`，键值相同、参数名不同。

`robot.local.yaml` 片段：

```yaml
platform:
  variant: ar5_ccs_v2
defaults:
  end_effectors:
    type: rg75                 # 或 left/right 分写
  tcp_offset:
    xyz: "0 0 0"               # metres
    rpy: "0 0 ${PI/2}"         # radians；支持 ${PI/2} / pi/2 / radians(90)
hardware:
  left_arm_ip: 192.168.1.160
  right_arm_ip: 192.168.0.160
  usb_left_port: /dev/ttyUSB0
  usb_right_port: /dev/ttyUSB2
```

Launch 可覆盖（CLI > profile）：`tcp_offset_rpy:="0 0 ${PI/2}"`、`hardware_left_arm_ip:=…` 等。

要让控制器跟踪虚拟 TCP，还需 `control.patch` 设置 `left_ee_frame` / `right_ee_frame` 为 `left_tcp_offset` / `right_tcp_offset`。

### 3.2 `skin`（臂外观）

`xacro/arm_ccs_v2.xacro` 中 `AR5_CCS_V2` 的 **visual** 由 `skin` 选择；**碰撞** 始终用 `meshes/ccs_v2/link*.glb`。`skin` 与 `variant` 正交，**仅 CCS V2** 有效（v1 无配色网格）。底座 visual 始终 `ccs_v2/base.glb`。

| `skin` | Visual |
|--------|--------|
| 默认 | `meshes/ccs_v2/link*.glb` |
| `fiveages` | `meshes/ccs_v2/fiveages/` |

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_ccs \
  variant:=ar5_ccs_v2 skin:=fiveages collider:=simple
```

### 3.3 挂到 W2R / WCE3

人形用 `arms:=` 而不是本包的 `variant:=`。W2R / WCE3 默认 CCS V2，并写死 `skin:=fiveages`：

```bash
ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_w2r
ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_wce3
```

W2R 仍可用 `arms:=ar5_ccs` / `ar5_srs` 换套件（无 FiveAges 配色）。详见 [fiveages_w2r README](../../../humanoid/FiveAges/Gen2/fiveages_w2r_description/README.md)、[fiveages_wce3 README](../../../humanoid/FiveAges/Gen3/fiveages_wce3_description/README.md)。

## 4. 真机部署

```bash
colcon build --packages-up-to rokae_ros2_control --symlink-install
source install/setup.bash

ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_ccs hardware:=real
ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_ccs hardware:=real variant:=ar5_ccs_v2 type:=rg75
ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_ccs hardware:=real_usb type:=linkerhand_o7
```

## 5. ros2_control 共享宏

`xacro/ros2_control/robot.xacro` 为薄封装；可复用模块供 `fiveages_w2r_description` 等 include：

| 文件 | 说明 |
|------|------|
| `end_effector_interfaces.xacro` | 末端接口 |
| `rokae_hardware.xacro` | Rokae 插件参数 |
| `external_ee_systems.xacro` | 外接 usb 末端 |

仿真硬件插件见 [`robot_common_launch`](../../../common/robot_common_launch/xacro/ros2_control/simulation_hardware.xacro)。
