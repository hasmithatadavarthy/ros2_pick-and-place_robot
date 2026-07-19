# ROS 2 Franka Emika Panda Color Sorting System

A ROS 2-based autonomous color sorting pipeline using the Franka Emika Panda robotic manipulator. This project integrates OpenCV-based computer vision, MoveIt 2 motion planning, and Gazebo Sim (Ignition) simulation to automate pick-and-place sorting operations.

## Project Architecture

The system is organized into modular ROS 2 packages:
- **`panda_bringup`**: Contains launch scripts to spawn the robot, configure parameters, and load necessary nodes.
- **`panda_controller`**: Defines configuration files and controllers for the manipulator arm and gripper.
- **`panda_description`**: Houses the robot descriptions (URDF/Xacro), link geometry models, and simulation worlds.
- **`panda_moveit`**: Houses parameters and configuration setups for the MoveIt 2 motion planning assistant.
- **`panda_vision`**: Houses the OpenCV color detection node, which estimates the 3D coordinates of objects on the sorting table.
- **`pymoveit2`**: A Python API wrapper used to construct pick-and-place sorting logic via standard trajectories.

## Prerequisites

- **OS**: Ubuntu 22.04 LTS (Jammy Jellyfish)
- **ROS Distro**: ROS 2 Humble Hawksbill
- **Libraries & Tools**:
  - MoveIt 2
  - Gazebo Sim (Ignition Fortress)
  - OpenCV (`python3-opencv`)
  - `ros2_control` and `ros2_controllers`

## Installation

1. Create a ROS 2 workspace (if you don't have one):
   ```bash
   mkdir -p ~/ros2_ws/src
   cd ~/ros2_ws/src
   ```
2. Clone or copy this repository into `src`:
   ```bash
   git clone <your-repository-url> ros2_panda_color_sorter
   ```
3. Build the workspace:
   ```bash
   cd ~/ros2_ws
   colcon build
   source install/setup.bash
   ```

## Usage

### 1. Launch the Simulation
To spawn the robot in the Gazebo environment alongside MoveIt 2 and RViz, run:
```bash
ros2 launch panda_bringup pick_and_place.launch.py
```
Wait until the Gazebo GUI, RViz, and controller managers have fully loaded and transitioned to ready.

### 2. Run the Sorting Controller
Execute the python sorting node by specifying the target color (`R` for Red, `G` for Green, `B` for Blue) you want to sort:
```bash
ros2 run pymoveit2 pick_and_place.py --ros-args -p target_color:=R
```

## Physics & Control Optimizations

This repository includes several improvements to ensure smooth and stable execution:
- **Analytical Box Collisions**: Replaced unstable table mesh collision models (`.DAE` files) with analytical box primitives. This resolves the micro-jittering and bounciness of dynamic objects on contact surfaces.
- **Model Inertia Tensors**: Configured correct `<inertial>` parameters and mass values for the dynamic colored box models, ensuring realistic physics solver behavior.
- **Start State Real-Time Tracking**: Patched planning requests to use empty starting states. This forces MoveIt 2 to query its internal real-time planning scene state monitor, eliminating aborted trajectory errors caused by stale joint caches.
- **OpenCV Red Hue Wrapping**: Enhanced OpenCV color detection masks to merge both lower and upper hue segments, ensuring robust detection under varying Gazebo lighting conditions.