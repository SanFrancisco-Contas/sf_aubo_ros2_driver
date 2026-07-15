# aubo_ros2_driver

Aubo Robotics ROS2 driver

## ROS 2 Jazzy Adaptation Status

This repository has completed an initial adaptation for ROS 2 Jazzy. `aubo_ros2_driver`, `aubo_description`, and `aubo_moveit_config` can perform basic startup, status display, MoveIt planning, and trajectory execution under the Jazzy environment.

Key issues addressed in this adaptation include:

- Adapted the MoveIt OMPL planning configuration to the Jazzy parameter format.
- Completed the MoveIt controller manager parameters to support `joint_trajectory_controller` execution.
- Added acceleration limits to `joint_limits.yaml`, fixing an `AddTimeOptimalParameterization` failure.
- Corrected the `SRDF` virtual joint to `world -> base_link`.
- Fixed the RViz `Planning Scene Topic` to `/monitored_planning_scene`, allowing the MotionPlanning scene's robot state to refresh correctly.

Notes:

- All MoveIt-related launch instructions in this document are based on the Jazzy-adapted configuration.
- For real hardware debugging, it is still recommended to complete URDF calibration first before performing MoveIt validation.

## Viewing the Aubo Robot Model in RViz (using aubo_i5 as an example)

```bash
ros2 launch aubo_description aubo_viewer.launch.py
```

## URDF Calibration Is Recommended Before Driving the Real Robot Arm

It is recommended to first generate a calibrated URDF based on the calibration compensation returned by the robot's current controller, before proceeding with real hardware driving, MoveIt planning, and trajectory validation.

Using the default, uncalibrated URDF directly may carry the following risks:

- The planning model's kinematic parameters may not match the real robot, causing end-effector pose deviations.
- The pose shown in RViz/MoveIt may not fully match the real robot feedback, making troubleshooting harder.
- Functions that depend on model accuracy — such as TCP validation, trajectory reproduction, and offline waypoint comparison — may produce unreliable results.

Recommended method:

```bash
cd <your_ros2_ws>
source /opt/ros/jazzy/setup.bash
python3 src/aubo_description/scripts/calibrate_urdf_dh.py \
  --robot-model aubo_i5 \
  --robot-ip 192.168.127.128
colcon build --packages-select aubo_description
source install/setup.bash
```

Notes:

- The generated result is written by default to `src/aubo_description/urdf/<robot_model>_calibrated.urdf`.
- `--robot-ip` must be provided explicitly.
- Before running, make sure the current Python environment can import `numpy` and `pyaubo_sdk`.
- After generation, the `aubo_description` package must be rebuilt separately.

## Driving the Real aubo_i5 Robot Arm (adjust `robot_ip` and `aubo_type` for your robot)

Terminal 1:

```bash
cd <your_ros2_ws>
source /opt/ros/jazzy/setup.bash
source install/setup.bash
ros2 launch aubo_ros2_driver aubo_control.launch.py aubo_type:=aubo_i5 robot_ip:=192.168.127.128 \
  use_fake_hardware:=false
```

Terminal 2:

```bash
cd <your_ros2_ws>
source /opt/ros/jazzy/setup.bash
source install/setup.bash
ros2 launch aubo_moveit_config aubo_moveit.launch.py aubo_type:=aubo_i5
```

Under ROS 2 Jazzy, `aubo_moveit.launch.py` has already been adapted and can be used directly for RViz + MoveIt integration testing. `launch_rviz` defaults to `true`.

## Real aubo_i5 Single-Point Trajectory Execution Demo (adjust `robot_ip` and `aubo_type` for your robot)

Terminal 1:

```bash
cd <your_ros2_ws>
source /opt/ros/jazzy/setup.bash
source install/setup.bash
ros2 launch aubo_ros2_driver aubo_control.launch.py aubo_type:=aubo_i5 robot_ip:=192.168.127.128 \
  use_fake_hardware:=false
```

Terminal 2:

```bash
cd <your_ros2_ws>
source /opt/ros/jazzy/setup.bash
source install/setup.bash
ros2 launch ros_joints_plan joints_plan.launch.py aubo_type:=aubo_i5
```

Under ROS 2 Jazzy, the launch configuration for `ros_joints_plan` has also been synced with the current MoveIt adaptation.

## Driving the Real Robot Arm via the Service Node (adjust `robot_ip` for your robot)

```bash
cd <your_ros2_ws>
source /opt/ros/jazzy/setup.bash
source install/setup.bash
ros2 launch aubo_ros2_driver aubo_client.launch.py robot_ip:=127.0.0.1 log_level:=info
```

Optional parameters:

- `port`: TCP service port, defaults to `30004`
- `robot`: robot name prefix, defaults to `rob1`
- `log_level`: log level, defaults to `info`

## Call Example

```bash
cd <your_ros2_ws>
source /opt/ros/jazzy/setup.bash
source install/setup.bash
ros2 service call /jsonrpc_service aubo_msgs/srv/JsonRpc \
"{cls: 'RobotState', func: 'getTcpPose', params: '[]'}"
```

## Response Example

```bash
requester: making request: aubo_msgs.srv.JsonRpc_Request(cls='RobotState', func='getTcpPose', params='[]')

response:
aubo_msgs.srv.JsonRpc_Response(result='[0.0, 0.0, 0.0, 0.0, 0.0, 0.0]', error='None')
```

## Error Response Example (using an incorrect class name `RobotStat`)

```bash
requester: making request: aubo_msgs.srv.JsonRpc_Request(cls='RobotStat', func='getTcpPose', params='[]')

response:
aubo_msgs.srv.JsonRpc_Response(result='None', error='{"code": -32601, "message": "method not found: rob1.RobotStat.getTcpPose"}')
```

## aubo_sdk Interface Reference Documentation

[aubo_sdk developer](https://docs.aubo-robotics.cn/arcs_api/index.html)
