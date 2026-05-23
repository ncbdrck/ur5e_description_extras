# ur5e_description_extras

Universal Robots UR5e + Robotiq 2F-85 sim extras for the
`rl_environments` stack. Mirrors `reactorx200_description`'s role for
the RX200:

- Wraps the upstream UR5e URDF (`ur_e_description`) + Robotiq 2F-85
  gripper (`robotiq_description`) into a single xacro that ships the
  Gazebo plugins we need (FT sensor, gripper grasp plugin).
- Mounts the arm on a 4-legged `ur5_base` (~0.59 m tall).
- Spawns a `cafe_table` next to the base (top at z = 0.775).
- Adds a **head-mount Kinect v2** so cube-tracking workflows share the
  same camera topology as the other robots.
- Brings up **ros_control** with plugin-level Gazebo PID gains for the
  6-DOF arm plus a trajectory controller for the Robotiq gripper.
- Ships local copies of `ur5_base`, `cafe_table`, and a red cube under
  `models/` so the scene is self-contained.

## Prerequisites

- ROS Noetic
- `ur_e_description` (ros-industrial — clone `universal_robot.git -b $ROS_DISTRO-devel`)
- `robotiq_description` + `robotiq_gazebo` (Robotiq 2F-85 macros)
- `common_sensors` (Kinect v2 xacro)
- `ur5e_robotiq_85_moveit_config` (MoveIt; optional unless using `start_moveit:=true`)

## Verify the sim works

```bash
roscore                                                       # term 1
roslaunch ur5e_description_extras ur5e_gazebo.launch          # term 2
```

Expected:
- Gazebo opens with the ur5_base at origin, cafe_table at x=0.7, red
  cube on the table, and UR5e + Robotiq 2F-85 mounted on top of the
  base (`base_link` at z = 0.59). The default launch pose is folded
  upright, not all-zeros, so the arm starts above the tabletop.
- `rostopic echo /ur5e/joint_states` publishes 6 arm joints + the
  Robotiq knuckle at ~300 Hz.
- `rostopic list | grep /ur5e/arm_controller` shows `/command` + state
  topics.

The safe startup pose can be overridden from the command line:

```bash
roslaunch ur5e_description_extras ur5e_gazebo.launch \
  shoulder_pan:=0 shoulder_lift:=-1.5707 elbow:=1.5707 \
  wrist_1:=-1.5707 wrist_2:=-1.5707 wrist_3:=0
```

To bring up MoveIt against the same Gazebo controllers:

```bash
roslaunch ur5e_description_extras ur5e_gazebo.launch start_moveit:=true use_moveit_rviz:=true
```

## Layout

```
ur5e_description_extras/
├── package.xml
├── CMakeLists.txt
├── README.md
├── urdf/
│   ├── ur5e_robotiq85.urdf.xacro         # UR5e + Robotiq 2F-85 + coupler + Gazebo plugins
│   └── ur5e_robotiq85_kinect.urdf.xacro  # + head-mount Kinect v2
├── launch/
│   ├── ur5e_gazebo.launch                # world + arm + control + (optional) cube/MoveIt
│   └── ur5e_control.launch               # joint_state + arm + gripper spawn
├── config/
│   └── ur5e_controller.yaml              # PID gains
├── worlds/
│   └── ur5e_scene.world                  # sun + ground + ur5_base + cafe_table baked in
└── models/
    ├── ur5_base/                         # 4-legged 0.59 m platform (from utecrobotics/ur5)
    ├── cafe_table/                       # Gazebo stock cafe_table (top at z=0.775)
    └── block/                            # 4 cm red cube
```

## Why this exists

Separate description package keeps URDF / launch files reusable for
MoveIt / RViz demos independent of the RL stack. Same shape as
`viperx300s_description` (VX300S), `niryo_ned2_description_extras`
(NED2), and `reactorx200_description` (RX200).
