# Rokae Robots

Rokae manipulator robot description packages. Each package contains its own URDF/Xacro, meshes, RViz, and controller configuration for the AR5 arm.

| | |
|:---:|:---:|
| <img src=".images/rokae_ar5.png" alt="Rokae AR5 SRS" height="280"> | <img src=".images/ar5_ccs.png" alt="Rokae AR5 CCS" height="280"> |
| **[AR5 SRS](ar5_srs_description/)** | **[AR5 CCS](ar5_ccs_description/)** |

## Common Commands

### Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to <package> --symlink-install
source ~/ros2_ws/install/setup.bash
```

Replace `<package>` with `ar5_srs_description` or `ar5_ccs_description`.

### Visualize Full Robot

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=<robot> type:=<type> collider:=simple
```

Useful arguments:

- `robot:=<robot>` selects the model: `ar5_srs` or `ar5_ccs`.
- `type:=left`, `type:=right`, or `type:=dual` for single or dual-arm layouts without an end effector.
- `type:=left_rg75`, `type:=right_rg75`, or `type:=dual_rg75` adds a Jodell RG75 gripper on the corresponding arm(s).
- `collider:=simple` uses simple primitive collision geometry. The xacro default is `convex`; use `collider:=convex` to match it.

### Run OCS2 Demos

The default hardware is `mock_components`.

```bash
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=<robot> type:=<type>
ros2 launch ocs2_arm_controller demo.launch.py robot:=<robot> type:=<type>
```

`manipulator_ocs2.launch.py` is the official mobile manipulator demo. `demo.launch.py` is the arm controller demo. Add `hardware:=gz` for Gazebo simulation or `hardware:=real` for real-robot deployment where supported. Real-hardware builds may also require `rokae_ros2_control`. Model-specific launch examples and gripper variants are documented in each package README.

## Packages

```text
Rokae/
├── ar5_srs_description/
└── ar5_ccs_description/
```
