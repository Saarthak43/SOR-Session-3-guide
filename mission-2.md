# Session 3 — Entering the 3D World with Gazebo

*Simulating a Differential Drive Robot from Scratch*

![ROS 2 Jazzy](https://img.shields.io/badge/ROS%202-Jazzy-blue)
![Gazebo Harmonic](https://img.shields.io/badge/Gazebo-Harmonic-orange)
![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu-24.04-purple)

In this session we stop driving blind and enter a full 3D simulated world. We will build a robot description, visualize it in RViz, spawn it in Gazebo, add plugins, bridge ROS 2 with Gazebo, and finally drive a differential drive robot using keyboard teleop.

<img width="2033" height="1123" alt="gazebo-5" src="https://github.com/user-attachments/assets/13643d24-7a1e-48a6-a8e8-eb535b1352a2" />

## Table of Contents

- [Quick Recap — What We Know So Far](#quick-recap--what-we-know-so-far)
- [Mission of This Session](#mission-of-this-session)
- [Why Gazebo?](#why-gazebo)
- [Installation](#installation)
- [Understanding the Gazebo GUI](#understanding-the-gazebo-gui)
- [Download the Offline Model Library](#download-the-offline-model-library)
- [Cloning This Repo](#cloning-this-repo)
- [Opening the Pre-Built World](#opening-the-pre-built-world)
- [URDF — Universal Robot Description Format](#urdf--universal-robot-description-format)
- [XACRO — Supercharged URDF](#xacro--supercharged-urdf)
- [Creating the URDF File](#creating-the-urdf-file)
- [Visualizing in RViz](#visualizing-in-rviz)
- [Adding Colors](#adding-colors)
- [Spawning the Robot in Gazebo](#spawning-the-robot-in-gazebo)
- [The Two-Point Problem — Caster Wheel](#the-two-point-problem--caster-wheel)
- [Giving the Robot Life — Gazebo Plugins](#giving-the-robot-life--gazebo-plugins)
- [Gazebo Bridge — Making ROS and Gazebo Talk](#gazebo-bridge--making-ros-and-gazebo-talk)
- [Build and Launch Checklist](#build-and-launch-checklist)
- [Assignment](#assignment)
- [What's Next?](#whats-next)

## Quick Recap — What We Know So Far

| Concept | Description |
|---|---|
| Nodes, Topics, Services | Core ROS 2 communication |
| Parameters & Actions | Advanced node configuration and long-running goals |
| Packages & Workspaces | How ROS 2 organizes code |
| Launch Files | Running multiple nodes together |

## Mission of This Session

> **Mission: Build, spawn, and drive a differential drive robot in Gazebo Harmonic — from zero.**

## Why Gazebo?

Gazebo is a 3D robotics simulator. It gives us a virtual world where robots can move, collide, sense, and interact with objects before we test anything on real hardware.

| Feature | Why it matters |
|---|---|
| **3D Physics Engine** | Simulates collisions, gravity, friction, and dynamics |
| **Realistic Sensors** | Lets us simulate cameras, LiDAR, IMUs, and more |
| **Plugins** | Adds functionality like motion, sensors, and state publishing |
| **Worlds & Models** | Gives us environments, objects, and robots to work with |

ROS 2 Jazzy and Gazebo Harmonic work well together because both are long-term support releases. That makes this pair stable for a robotics workshop setup.

## Installation

### 5.1 Install Gazebo Harmonic
Follow the official guide
```bash
https://gazebosim.org/docs/harmonic/install_ubuntu/
```

Verify the installation:

```bash
gz sim shapes.sdf
```

> If Gazebo opens with the shapes world, the installation is working.

<img width="2031" height="1195" alt="gazebo" src="https://github.com/user-attachments/assets/1e788ad3-fee6-4607-b690-bd189be53236" />


### 5.2 Install ROS-Gazebo Tools

```bash
sudo apt update
sudo apt install ros-jazzy-ros-gz
```

Later, for bridging ROS 2 topics with Gazebo topics, install:

```bash
sudo apt install ros-jazzy-ros-gz-bridge
```

For keyboard control:

```bash
sudo apt install ros-jazzy-teleop-twist-keyboard
```

## Understanding the Gazebo GUI

Run the basic example first:

```bash
gz sim shapes.sdf
```

<img width="2031" height="1194" alt="gazebo-1" src="https://github.com/user-attachments/assets/608d5078-3c8b-4789-91de-e7057f6a45c9" />


| No. | GUI Element | What it does |
|---|---|---|
| 1 | **Play/Pause** | Starts or pauses simulation. Use `-r` to auto-start Gazebo. |
| 2 | **Real Time Factor** | Should stay near 100%. Below 60% means the simulation is struggling. |
| 3 | **Shape/Light Tools** | Add and transform basic geometry and lights. |
| 4 | **Model Hierarchy** | Shows all models, links, collisions, and visuals in the world. |
| 5 | **Model Inspector** | Shows detailed information about the selected model. |
| 6 | **Plugin Browser** | Opens tools like Resource Spawner, Lidar Visualizer, and Image Display. |

## Download the Offline Model Library

After downloading and unzipping the model library to home of your container/ubuntu, export it:

```bash
https://drive.google.com/file/d/1tcfoLFReEW1XNHPUAeLpIz2iZXqQBvo_/view
```

```bash
export GZ_SIM_RESOURCE_PATH=~/gazebo_models
```
 
Make it permanent:

```bash
echo 'export GZ_SIM_RESOURCE_PATH=~/gazebo_models' >> ~/.bashrc
source ~/.bashrc
```

## Cloning This Repo

From now on, every session has its own repository. Clone the starter branch inside your ROS 2 workspace:

```bash
mkdir -p ~/sor_ws/src
cd ~/sor_ws/src
git clone -b starter https://github.com/Saarthak43/sor-ros-session1.git
cd ~/sor_ws
colcon build --packages-select erc_sor_ros_session1
source install/setup.bash
```

Expected package structure:

```text
erc_sor_ros_session1/
├── launch/
│   ├── check_urdf.launch.py
│   ├── spawn_robot.launch.py
│   └── world.launch.py
├── rviz/
│   ├── rviz.rviz
│   └── urdf.rviz
├── urdf/
│   ├── my_robot.xacro
│   ├── sor_bot.gazebo
│   └── materials.xacro
├── worlds/
│   └── world.sdf
├── CMakeLists.txt
└── package.xml
```

Make sure these folders are installed by `CMakeLists.txt`:

```cmake
install(
  DIRECTORY launch urdf worlds rviz
  DESTINATION share/${PROJECT_NAME}
)
```

## Opening the Pre-Built World

Launch only the world first:

```bash
ros2 launch erc_sor_ros_session1 world.launch.py
```

<img width="2031" height="1122" alt="gazebo-3" src="https://github.com/user-attachments/assets/d773c951-6e10-4658-aa51-32624c2b829a" />

> Gazebo gives us physics, collisions, and world dynamics. But right now, there is still no robot inside the world.

## URDF — Universal Robot Description Format

### 10.1 What is URDF?

URDF means **Universal Robot Description Format**. It is an XML format used to describe the robot's physical structure.

| URDF Part | Meaning |
|---|---|
| **Links** | Physical components like body, wheels, sensors |
| **Joints** | Connections between links |

## Create a basic design in URDF 

Adding base link , wheels , corresponding joints and inertia to our URDF file , refer to the slides to see exactly how to navigate and update the .xacro folder 
 ( extension should be .xacro ) 
```bash
<?xml version='1.0'?>

<robot name="my_robot" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <!-- STEP 1 - Robot footprint -->
  <link name="base_footprint"></link>

  <!-- Reusable inertia macro for a box -->
  <xacro:macro name="box_inertia" params="m l w h">
    <inertia ixx="${(m*(w*w+h*h))/12}" ixy="0" ixz="0"
             iyy="${(m*(l*l+h*h))/12}" iyz="0"
             izz="${(m*(l*l+w*w))/12}"
    />
  </xacro:macro>

  <!-- Reusable inertia macro for a cylinder (wheel) -->
  <xacro:macro name="cylinder_inertia" params="m r h">
    <inertia ixx="${(m*(3*r*r+h*h))/12}" ixy="0" ixz="0"
             iyy="${(m*(3*r*r+h*h))/12}" iyz="0"
             izz="${(m*r*r)/2}"
    />
  </xacro:macro>

  <!-- STEP 2 - Robot chassis = base_link -->
  <joint name="base_footprint_joint" type="fixed">
    <origin xyz="0 0 0" rpy="0 0 0" />
    <parent link="base_footprint"/>
    <child link="base_link" />
  </joint>

  <link name='base_link'>
    <pose>0 0 0.1 0 0 0</pose>

    <inertial>
      <mass value="15.0"/>
      <origin xyz="0.0 0 0" rpy="0 0 0"/>
      <xacro:box_inertia m="15.0" l="0.4" w="0.2" h="0.1"/>
    </inertial>

    <collision name='collision'>
      <origin xyz="0 0 0" rpy="0 0 0"/> 
      <geometry>
        <box size=".4 .2 .1"/>
      </geometry>
    </collision>

    <visual name='base_link_visual'>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <box size=".4 .2 .1"/>
      </geometry>
    </visual>
  </link>

  <!-- STEP 3 - Wheel macro: reusable wheel definition -->
  <xacro:macro name="wheel" params="prefix y_pos">
    <joint type="continuous" name="${prefix}_wheel_joint">
      <origin xyz="0 ${y_pos} 0" rpy="0 0 0"/>
      <child link="${prefix}_wheel"/>
      <parent link="base_link"/>
      <axis xyz="0 1 0" rpy="0 0 0"/>
      <limit effort="100" velocity="10"/>
      <dynamics damping="1.0" friction="1.0"/>
    </joint>

    <link name='${prefix}_wheel'>
      <inertial>
        <mass value="5.0"/>
        <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
        <xacro:cylinder_inertia m="5.0" r="0.1" h="0.05"/>
      </inertial>

      <collision>
        <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/> 
        <geometry>
          <cylinder radius=".1" length=".05"/>
        </geometry>
      </collision>

      <visual name='${prefix}_wheel_visual'>
        <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
        <geometry>
          <cylinder radius=".1" length=".05"/>
        </geometry>
      </visual>
    </link>
  </xacro:macro>

  <!-- STEP 4 - Use the macro to create both wheels -->
  <xacro:wheel prefix="left" y_pos="0.15"/>
  <xacro:wheel prefix="right" y_pos="-0.15"/>

</robot>
```
## Now to view it , we will first use the Rviz tool 

#### Step 1 — Install required URDF packages
```bash
sudo apt update
sudo apt install ros-jazzy-urdf ros-jazzy-urdf-tutorial ros-jazzy-urdf-launch -y
```
#### Step 2 — Create the launch folder and navigate to it
```bash
mkdir -p ~/sor_ws/src/sor-ros-session1/erc_sor_ros_session1/launch
cd ~/sor_ws/src/sor-ros-session1/erc_sor_ros_session1/launch
```
#### Step 3 — Open Codium in this folder
```bash
codium .
```
#### Step 4 — Create the file inside Codium

In the Codium sidebar: right-click → New File → name it check_urdf.launch.py → it opens blank, ready to type/paste.

#### Step 5 — Paste this content
```bash
pythonimport os
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution
from launch_ros.substitutions import FindPackageShare


def generate_launch_description():
    
    pkg_erc_sor_ros_session1 = FindPackageShare('erc_sor_ros_session1')
    default_rviz_config_path = PathJoinSubstitution([pkg_erc_sor_ros_session1, 'rviz', 'urdf.rviz'])

    # Show joint state publisher GUI for joints
    gui_arg = DeclareLaunchArgument(name='gui', default_value='true', choices=['true', 'false'],
                                    description='Flag to enable joint_state_publisher_gui')
    
    # RViz config file path
    rviz_arg = DeclareLaunchArgument(name='rvizconfig', default_value=default_rviz_config_path,
                                    description='Absolute path to rviz config file')
    
    # URDF/xacro model path within the package
    model_arg = DeclareLaunchArgument(
        'model', default_value='my_robot.xacro',
        description='Name of the URDF/xacro description to load'
    )

    # Use built-in ROS2 URDF launch package with our own arguments
    urdf = IncludeLaunchDescription(
        PathJoinSubstitution([FindPackageShare('urdf_launch'), 'launch', 'display.launch.py']),
        launch_arguments={
            'urdf_package': 'erc_sor_ros_session1',
            'urdf_package_path': PathJoinSubstitution(['urdf', LaunchConfiguration('model')]),
            'rviz_config': LaunchConfiguration('rvizconfig'),
            'jsp_gui': LaunchConfiguration('gui')}.items()
    )

    launchDescriptionObject = LaunchDescription()

    launchDescriptionObject.add_action(gui_arg)
    launchDescriptionObject.add_action(rviz_arg)
    launchDescriptionObject.add_action(model_arg)
    launchDescriptionObject.add_action(urdf)

    return launchDescriptionObject

```
CTRL + S to save 

#### Step 6 — Build and launch
```bash
cd ~/sor_ws && colcon build --packages-select erc_sor_ros_session1 && source install/setup.bash
ros2 launch erc_sor_ros_session1 check_urdf.launch.py
```
RViz opens, your robot (base + two wheels) appears, and a joint_state_publisher GUI lets you slide the wheel joints live.

<img width="2032" height="1127" alt="rviz-2" src="https://github.com/user-attachments/assets/2eeaef4c-bbc3-4c7f-92e8-814f8722e6db" />
