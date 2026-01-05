# La Z Bot - Autonomous Off-Road Navigation

All files necessary for autonomous off-road navigation of the La Z Bot!

[![Demo Video](https://img.youtube.com/vi/h0uvkaR6fvo/0.jpg)](https://youtu.be/h0uvkaR6fvo)

## Overview

La Z Bot is a ROS-based autonomous navigation system designed for off-road environments. The platform uses sensor fusion from multiple sources including stereo vision, IMU, GPS, and lidar to navigate outdoor terrain autonomously.

## Hardware Components

- **Robot Base**: AgileX Bunker mobile platform
- **Camera**: Stereolabs ZED 2i stereo camera
- **IMU**: Microstrain inertial measurement unit
- **GPS**: Dual-antenna GPS system for heading estimation
- **Lidar**: URG laser scanner (likely Hokuyo)

## Software Requirements

### ROS Version
This project uses **ROS Noetic** (or ROS Melodic). Ensure you have a compatible Ubuntu installation:
- ROS Noetic: Ubuntu 20.04
- ROS Melodic: Ubuntu 18.04

### System Dependencies

Install ROS and required packages:

```bash
# Install ROS (Noetic example)
sudo apt update
sudo apt install ros-noetic-desktop-full

# Install required ROS packages
sudo apt install \
  ros-noetic-robot-localization \
  ros-noetic-navigation \
  ros-noetic-move-base \
  ros-noetic-map-server \
  ros-noetic-robot-state-publisher \
  ros-noetic-joint-state-publisher-gui \
  ros-noetic-xacro \
  ros-noetic-pcl-ros \
  ros-noetic-rviz

# Install ZED SDK (required for ZED camera)
# Download from: https://www.stereolabs.com/developers/release/
# Follow Stereolabs installation instructions for your platform

# Install Microstrain driver dependencies
sudo apt install ros-noetic-microstrain-inertial-driver
```

### Build Dependencies

```bash
sudo apt install \
  build-essential \
  cmake \
  git \
  python3-catkin-tools \
  python3-rosdep
```

## Installation

### 1. Clone the Repository

```bash
# Create catkin workspace
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src

# Clone the repository
git clone --recursive https://github.com/thisisformuchfun/La_Z_Bot.git

# If you forgot --recursive, initialize submodules:
cd La_Z_Bot
git submodule update --init --recursive
```

### 2. Install Dependencies

```bash
cd ~/catkin_ws

# Initialize rosdep (first time only)
sudo rosdep init
rosdep update

# Install all dependencies
rosdep install --from-paths src --ignore-src -r -y
```

### 3. Build the Workspace

```bash
cd ~/catkin_ws

# Build using catkin_make
catkin_make

# Or use catkin build (recommended)
catkin build

# Source the workspace
source devel/setup.bash

# Add to bashrc for convenience
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
```

## Hardware Setup

### CAN Interface (Bunker Base)

The Bunker robot communicates via CAN bus. Configure the CAN interface:

```bash
# Bring up CAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify CAN interface
ip link show can0
```

To make this persistent, add to `/etc/network/interfaces`:

```
auto can0
iface can0 inet manual
    pre-up /sbin/ip link set can0 type can bitrate 500000
    up /sbin/ip link set can0 up
    down /sbin/ip link set can0 down
```

### USB Permissions

Grant permissions for USB devices (ZED camera, IMU, lidar):

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# May need to log out and back in for changes to take effect
```

## Running the System

### Launch the Complete System (Hardware)

To start all hardware components and navigation:

```bash
# Source workspace
source ~/catkin_ws/devel/setup.bash

# Launch hardware (sensors + localization)
roslaunch la_z_bot_bringup hardware.launch

# In a new terminal, launch navigation
roslaunch la_z_bot_navigation move_base.launch
```

### Individual Component Launch

You can launch components separately for testing:

```bash
# Bunker base only
roslaunch la_z_bot_bringup bunker.launch

# ZED camera
roslaunch la_z_bot_bringup zed.launch

# Microstrain IMU
roslaunch la_z_bot_bringup microstrain.launch

# Localization (robot_localization EKF)
roslaunch la_z_bot_bringup localization.launch

# URG lidar
roslaunch la_z_bot_bringup urg.launch

# Point cloud processing
roslaunch la_z_bot_bringup pcl.launch
```

### Visualization

View the robot in RViz:

```bash
# View robot model only
roslaunch la_z_bot_viz view_model.launch

# View robot with sensors (requires hardware running)
roslaunch la_z_bot_viz view_robot.launch
```

## Package Descriptions

### `la_z_bot_bringup`
Hardware drivers and sensor launch files. Contains configuration for all sensors and localization.

**Key launch files:**
- `hardware.launch` - Launches all hardware components
- `bunker.launch` - Bunker mobile base driver (CAN interface)
- `zed.launch` - ZED 2i stereo camera
- `microstrain.launch` - IMU driver
- `localization.launch` - Robot localization (EKF sensor fusion)
- `urg.launch` - URG laser scanner
- `pcl.launch` - Point cloud processing

**Scripts:**
- `dual_antenna_to_odom.py` - Converts dual-antenna GPS to odometry
- `utm_path_saver.py` - Saves GPS waypoints
- `utm_path_player.py` - Replays saved GPS paths

### `la_z_bot_description`
URDF robot description files. Defines the robot's physical structure, sensors, and transformations.

### `la_z_bot_navigation`
Navigation stack configuration using ROS move_base. Includes costmap parameters, local and global planners.

**Key files:**
- `move_base.launch` - Main navigation launch file
- `params/` - Navigation parameter configurations

### `la_z_bot_viz`
RViz configurations for visualization and debugging.

### Submodules

- **`bunker_ros`** - Bunker robot ROS driver
- **`bunker_extras`** - Additional Bunker utilities
- **`ugv_sdk`** - AgileX UGV SDK
- **`zed-ros-wrapper`** - ZED camera ROS wrapper

## Navigation Usage

### Setting Navigation Goals

Once the system is running, you can send navigation goals:

```bash
# Using RViz: Click "2D Nav Goal" and click on the map

# Or programmatically via command line:
rostopic pub /move_base_simple/goal geometry_msgs/PoseStamped '{
  header: {frame_id: "odom"},
  pose: {
    position: {x: 10.0, y: 0.0, z: 0.0},
    orientation: {w: 1.0}
  }
}'
```

### Recording and Replaying Paths

```bash
# Save a GPS path (drive the robot along desired path)
rosrun la_z_bot_bringup utm_path_saver.py

# Replay saved path
rosrun la_z_bot_bringup utm_path_player.py
```

## Troubleshooting

### CAN Interface Issues
```bash
# Check if CAN interface exists
ip link show can0

# Check CAN traffic
candump can0

# Restart CAN interface
sudo ip link set can0 down
sudo ip link set can0 type can bitrate 500000
sudo ip link set can0 up
```

### ZED Camera Not Detected
```bash
# Check ZED SDK installation
/usr/local/zed/tools/ZED_Diagnostic

# Verify USB connection
lsusb | grep -i stereo
```

### No Odometry Data
```bash
# Check topics
rostopic list | grep odom

# Monitor odometry
rostopic echo /odometry/filtered
```

### Transform Errors (TF)
```bash
# View TF tree
rosrun tf view_frames
evince frames.pdf

# Monitor transforms
rosrun tf tf_echo odom base_link
```

## Configuration Files

Configuration files are located in `la_z_bot_bringup/config/`:
- `zed2i.yaml` - ZED camera settings
- `microstrain_params.yaml` - IMU configuration
- `robot_localization.yaml` - EKF sensor fusion parameters
- `gps_localization.yaml` - GPS localization settings

Adjust these files to tune sensor performance for your specific hardware setup.

## Topics Reference

Key ROS topics:
- `/bunker_odom` - Wheel odometry from Bunker base
- `/odometry/filtered` - Fused odometry from EKF
- `/zed2i/zed_node/odom` - Visual odometry from ZED
- `/imu/data` - IMU data
- `/gps/fix` - GPS position
- `/scan` - 2D laser scan
- `/cmd_vel` - Velocity commands to robot

## License

<a rel="license" href="http://creativecommons.org/licenses/by/3.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/3.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a>.

## Maintainer

Dave Niewinski (davesarmoury@gmail.com)

## Contributing

Contributions are welcome! Please submit pull requests or open issues for bugs and feature requests.
