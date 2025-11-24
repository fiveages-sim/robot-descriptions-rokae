# Rokae AR5 Description

This package contains the description files for Rokae AR5 Manipulator.

## 1. Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to rokae_ar5_description --symlink-install
```

## 2. Visualize the robot

### 2.1 SRS Arm without end effector
* Left Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 collider:=simple
    ```
  ![rokae_ar5](../.images/rokae_ar5.png)
* Right Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=right  collider:=simple
    ```
  ![rokae_ar5](../.images/rokae_ar5_right.png)
* Dual Arms
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=dual collider:=simple
    ```
  
### 2.2 SRS Arm with end effector
* Left Arm with Jodell RG75
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=left_rg75
    ```
* Dual Arm with Jodell RG75
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=dual_rg75 collider:=simple
    ```

### 2.3 CCS Arm without end effector
* Left Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=ccs_left collider:=simple
    ```
* Right Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=ccs_right collider:=simple
    ```
* Dual Arms
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=ccs_dual collider:=simple
    ```

### 2.4 CCS Arm with end effector
* Left Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=ccs_left_rg75 collider:=simple
    ```
* Right Arm
    ```bash
    source ~/ros2_ws/install/setup.bash
    ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=ccs_right_rg75 collider:=simple
    ```
* Dual Arm with Jodell RG75
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator.launch.py robot:=rokae_ar5 type:=ccs_dual_rg75 collider:=simple
  ```


## 3. OCS2 Demo

### 3.1 Official OCS2 Mobile Manipulator Demo

#### SRS Type
* Left Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=rokae_ar5
  ```
* Right Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=rokae_ar5 type:=right
  ```

#### CCS Type
* Left Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=rokae_ar5 type:=ccs_left
  ```
* Right Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=rokae_ar5 type:=ccs_right
  ```

### 3.2 OCS2 Arm Controller Demo

#### 3.2.1 Mock Component

* Left Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 
  ```
* Right Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 type:=right
  ```
* Dual Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 type:=dual
  ```

#### 3.2.2 Gazebo Simulation

* Left Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 hardware:=gz
  ```
* Right Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 hardware:=gz type:=right
  ```

### 3.3 Real Robot Deploy
* Build Command
```bash
cd ~/ros2_ws
colcon build --packages-up-to rokae_ros2_control --symlink-install
```

#### 3.3.1 Arm without end effector
* Single Left Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 hardware:=real
  ```
* Single Right Arm
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 hardware:=real type:=right
  ```
* Dual Arms
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 hardware:=real type:=dual
  ```
  
#### 3.3.2 Arm with end effector
* Left Arm with Jodell RG75
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 hardware:=real type:=left_rg75
  ```
* Right Arm with Jodell RG75
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 hardware:=real type:=right_rg75
  ```
* Dual Arms
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller demo.launch.py robot:=rokae_ar5 hardware:=real type:=dual_rg75
  ```