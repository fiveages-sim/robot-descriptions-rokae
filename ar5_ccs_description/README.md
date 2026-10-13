# Rokae AR5-CCS Description

珞石 AR5 CCS 双臂 URDF / ros2_control 描述包（同一包覆盖 CCS 与 CCS V2）。模型在 **`xacro/`**。OCS2 规划 URDF 由 `xacro/robot.xacro` 生成。`xacro/eefs.xacro` 亦被 `fiveages_w2r_description` 复用。

AR5 CCS 分体 OCS2 配置集中在本包 `config/ocs2`，W2R 与 WCE3 共用 `task.info`（EEF 精定位）和 `task_tcp.info`（TCP 跟踪）。参数分别取自 W2R / WCE3 的 `fixed_base.info` 和 `fixed_base_tcp.info`，状态与输入仅包含 `[左臂7 | 右臂7]` 共 14 个关节。保留输入代价、双臂末端跟踪、关节位置/速度限位及自碰撞项，不引入全身模式约束。

| 参数 | `task.info` | `task_tcp.info` |
|------|-------------|-----------------|
| 预测时域 | 1.0 s | 0.4 s |
| R scaling | 0.05 | 0.001 |
| 每臂 R 对角值（J1–J7） | 0.5, 0.5, 0.5, 0.5, 0.1, 0.1, 0.1 | 同左 |
| 末端过程权重（位置/姿态） | 100 / 50 | 10 / 5 |
| 末端终端权重（位置/姿态） | 200 / 100 | 20 / 10 |
| 关节软限速 | ±1.07 rad/s | J1–J4：±2.07；J5–J7：±2.57 rad/s |

WCE3 在构建时把这两份任务配置共享到其安装目录，控制器的 `robot_name` 保持 `fiveages_wce3`；规划 URDF 仍由 `fiveages_wce3_description/xacro/robot.xacro` 的 `topology:=dual` 生成，保留 ±0.068 m 的安装位置和 WCE3 躯干碰撞几何。整机包源码仅维护全身任务配置，无需修改控制器或 launch。

`single.info` 提取 W2R `fixed_base.info` 的单臂 7 维参数，采用上表精定位档的权重、时域和 ±1.07 rad/s 软限速，保留 `base_link` / `link7` 坐标系。自碰撞保持原配置的关闭状态，不引入双臂耦合或全身约束。

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
| 双臂 CCS V2 | 加 `arms:=ar5_ccs_v2` |
| 单臂 | 加 `type:=left` 或 `type:=right` |
| 末端（对称） | 加 `type:=<key>`（如 `rg75`、`ag2f90`），见 [§2](#2-末端执行器) |
| 末端（左右不同） | `left_type:=… right_type:=…`，**不要**再传 `type:=` |
| 真机 | `demo.launch.py … hardware:=real arm_hardware_driver:=rokae|inex`；见 [§2.1](#21-真机驱动方式) |
| Mock 控制 | `demo.launch.py …`（默认 `hardware:=mock_components`） |

![Rokae AR5 CCS](../.images/ar5_ccs.png)

**不对称末端示例**

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_ccs \
  arms:=ar5_ccs_v2 left_type:=rg75 right_type:=linkerhand_o7 collider:=simple
```

也可在 `robot_profile` / `robot.local.yaml` 的 `defaults.end_effectors.left` / `.right` 中配置（键名同 §2）。

## 2. 末端执行器

类型映射：`xacro/eef_type_resolver.xacro`；真机接线：`xacro/ros2_control/robot.xacro`。

### 2.1 真机驱动方式

臂体通过 `arm_hardware_driver:=rokae|inex` 选择驱动。每侧末端可独立配置 `left_ee_drive_type` / `right_ee_drive_type`。

| 驱动键 | 典型 launch | 物理链路 | 说明 |
|--------|-------------|----------|------|
| `rokae_rs485` | `hardware:=real arm_hardware_driver:=rokae` | 机械臂末端 RS-485 | 夹爪 / 手与机械臂共用 Rokae hardware system |
| `inex_can` | `hardware:=real arm_hardware_driver:=inex` | iNexus CAN | 当前支持 LinkerHand O6/L6/O7 |
| `inex_rs485` | `hardware:=real arm_hardware_driver:=inex` | iNexus RS-485 | 预留，尚未实现 |
| `usb_rs485` | `hardware:=real` | 外接 USB-RS485 | 使用 `modbus_ros2_control` |
| `usb_can` | `hardware:=real` | 外接 USB-CAN | 使用 `can_ros2_control` |
| `ethernet` | `left_ee_drive_type:=ethernet` 等 | 外接以太网（整机扫描，不写 IP） | `wuji_ros2_control`；仅 `wuji_hand2` |

夹爪**仅** Rokae 总线，没有外接 USB 夹爪路径。

### 2.2 类型与支持矩阵

`type` / `left_type` / `right_type` 与 `xacro/eef_type_resolver.xacro` 一致。列名对应 §2.1（`rokae 485` = 默认空键 / `hardware:=real`，臂体 RS485）。

| Type key | 产品 | `rokae 485` | `usb` | `usb_can` | `ethernet` |
|----------|------|:-------:|:-----:|:---------:|:---------:|
| `rg75` | Jodell RG75 | ✅ | — | — | — |
| `rg75_heavy` | Jodell RG75 + heavy pad | ✅ | — | — | — |
| `erg32` | Jodell ERG32（旋转 + 夹爪，2-DOF） | ✅ | — | — | — |
| `ag2f90`, `ag2f90_d` | ChangingTek AG2F90-D（`pad:=default`，与 m6_ccs 一致） | ✅ | — | — | — |
| `ag2f90_c`, `ag2f90_c_soft` | ChangingTek AG2F90-C | ✅ | — | — | — |
| `ag2f120s` | ChangingTek AG2F120S | ✅ | — | — | — |
| `ag2f120s_d` | ChangingTek AG2F120S-D | ✅ | — | — | — |
| `linkerhand_o6` / `l6` / `o7` | LinkerHand | ✅ | ✅ | ✅ | — |
| `xhand1` | XHand1 | — | ✅ | — | — |
| `wuji_hand2` | Wuji Hand2（舞肌） | — | — | — | ✅ |

要点：

- **夹爪** — 当前主要使用 `rokae_rs485`；其他链路保留统一分发接口并逐步适配。
- **LinkerHand** — 可选 `rokae_rs485`、`inex_can`、`usb_rs485` 或 `usb_can`。外接驱动启用时 Rokae 侧 `hand_type` 置空，避免双驱动。
- **XHand1** — 与 LinkerHand L6 共用 ChangingTek AG2F90-C 法兰；仿真可展开，真机走 `usb` / `usb_rs485`。
- **`wuji_hand2`** — 官方 mount 直接固定在 TCP 上（`R_x(π)`，手指 +Z 与 TCP +Z 重合）。仅 **`ethernet`**：手包 `side_systems` 按 `hand_side` 扫描，不要在 `robot.local.yaml` 写地址。臂仍 `hardware:=real`，手不走 RS485/USB/CAN。`left_ee_rw_rate` / `right_ee_rw_rate` 可覆盖手侧 `rw_rate`（未写默认 **200**）。仿真直接 `type:=wuji_hand2`。
- **`type:=left` / `right`** — 只选单臂拓扑，不是末端键。

仿真 / mock 下上表类型均可展开 URDF；真机统一使用 `hardware:=real`。

## 3. 平台与代际

### 3.1 `arms:=ar5_ccs` / `ar5_ccs_v2`

同一描述包内切换 CCS 代际，与末端 `type` 正交：

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_ccs \
  arms:=ar5_ccs_v2 type:=rg75 collider:=simple
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_ccs \
  arms:=ar5_ccs_v2 type:=wuji_hand2 collider:=simple
```

包默认 `arms:=ar5_ccs`（v1），避免 `robot:=ar5_ccs` 无参时静默改代。W2R 整机默认 `arms:=ar5_ccs_v2`。

`robot.local.yaml` 片段：

```yaml
platform:
  arms: ar5_ccs_v2
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

`xacro/arm_ccs_v2.xacro` 中 `AR5_CCS_V2` 的 **visual** 由 `skin` 选择；**碰撞** 始终用 `meshes/ccs_v2/link*.glb`。`skin` 与 `arms` 正交，**仅 CCS V2** 有效（v1 无配色网格）。底座 visual 始终 `ccs_v2/base.glb`。

| `skin` | Visual |
|--------|--------|
| 默认 | `meshes/ccs_v2/link*.glb` |
| `fiveages` | `meshes/ccs_v2/fiveages/` |

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=ar5_ccs \
  arms:=ar5_ccs_v2 skin:=fiveages collider:=simple
```

### 3.3 挂到 W2R / WCE3

独立双臂与人形统一用 `arms:=` 选择机械臂套件。W2R / WCE3 默认 CCS V2，并写死 `skin:=fiveages`：

```bash
ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_w2r
ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_wce3
```

W2R 仍可用 `arms:=ar5_ccs` / `ar5_srs` 换套件（无 FiveAges 配色）。详见 [fiveages_w2r README](../../../humanoid/FiveAges/Gen2/fiveages_w2r_description/README.md)、[fiveages_wce3 README](../../../humanoid/FiveAges/Gen3/fiveages_wce3_description/README.md)。

## 4. 真机部署

```bash
colcon build --packages-up-to rokae_ros2_control --symlink-install
source install/setup.bash

ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_ccs hardware:=real arm_hardware_driver:=inex
ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_ccs hardware:=real arm_hardware_driver:=rokae arms:=ar5_ccs_v2 type:=rg75 left_ee_drive_type:=rokae_rs485 right_ee_drive_type:=rokae_rs485
ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_ccs hardware:=real arm_hardware_driver:=inex type:=linkerhand_o7 left_ee_drive_type:=inex_can right_ee_drive_type:=inex_can
ros2 launch ocs2_arm_controller demo.launch.py robot:=ar5_ccs hardware:=real arms:=ar5_ccs_v2 type:=wuji_hand2 left_ee_drive_type:=ethernet right_ee_drive_type:=ethernet
```

## 5. ros2_control 共享宏

`xacro/ros2_control/robot.xacro` 为薄封装；可复用模块供 `fiveages_w2r_description` 等 include：

| 文件 | 说明 |
|------|------|
| `end_effector_interfaces.xacro` | 末端接口 |
| `rokae_hardware.xacro` | Rokae 插件参数 |
| `external_ee_systems.xacro` | 外接 usb 末端 |

仿真硬件插件见 [`robot_common_launch`](../../../common/robot_common_launch/xacro/ros2_control/simulation_hardware.xacro)。
