# Session 3 — Entering the 3D World with Gazebo

*Simulating a Differential Drive Robot from Scratch*

![ROS 2 Jazzy](https://img.shields.io/badge/ROS%202-Jazzy-blue)
![Gazebo Harmonic](https://img.shields.io/badge/Gazebo-Harmonic-orange)
![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu-24.04-purple)

In this session we stop driving blind and enter a full 3D simulated world. We will build a robot description, visualize it in RViz, spawn it in Gazebo, add plugins, bridge ROS 2 with Gazebo, and finally drive a differential drive robot using keyboard teleop.

<img width="2033" height="1123" alt="gazebo-5" src="https://github.com/user-attachments/assets/13643d24-7a1e-48a6-a8e8-eb535b1352a2" />


Conversation with Gemini
listen to me now , ill provide you with a mogi repo and you have to replicate it with the flow i created in a slide



provide commands just like mogi did but flow will be mine https://github.com/MOGI-ROS/Week-3-4-Gazebo-basics



dont mention mogi no where - just see what i have done in my slides

You stopped this response
listen to me now , ill provide you with a mogi repo and you have to replicate it with the flow i created in a slide



provide commands just like mogi did but flow will be mine https://github.com/MOGI-ROS/Week-3-4-Gazebo-basics



dont mention mogi no where - just see what i have done in my slides



🎯 One-Shot README Generation Prompt

Paste this entire prompt into Claude (with your repo link). That's it.

THE PROMPT

You are a technical documentation writer. Your job is to write a high-quality, production-grade GitHub README for a ROS 2 robotics session repository.

Reference this GitHub repo for technical accuracy on code snippets, file structures, plugin syntax, and launch file patterns:https://github.com/MOGI-ROS/Week-3-4-Gazebo-basics

Fetch the repo contents to understand the code structure, but DO NOT copy their narrative, their flow, or their section order. The README you write must follow my session flow exactly as described below. Do not mention or reference that repo anywhere in the output.

MY SESSION FLOW (follow this order exactly)

The README must follow this exact pedagogical flow — the way I taught it in my session:



1. Header & Badge Section

Repo title: Session 3 — Entering the 3D World with Gazebo

Subtitle: Simulating a Differential Drive Robot from Scratch

Badges: ROS 2 Jazzy | Gazebo Harmonic | Ubuntu 24.04

Short 1-paragraph hook: "In this session we stop driving blind and enter a full 3D simulated world..."

2. Quick Recap — What We Know So Far

A clean table or bullet list of what was covered in previous sessions:

ConceptDescriptionNodes, Topics, ServicesCore ROS 2 communicationParameters & ActionsAdvanced node configurationPackages & WorkspacesHow ROS 2 organizes codeLaunch FilesRunning multiple nodes at once3. Mission of This Session

One bold line:



Mission: Build, spawn, and drive a differential drive robot in Gazebo Harmonic — from zero.

4. Why Gazebo?

Explain Gazebo as a 3D physics simulation environment. Cover these 4 advantages:



3D Physics Engine — collisions, dynamics

Realistic Sensors — cameras, LiDAR, IMUs

Plugins — extend robot functionality with code

Worlds & Models — rich virtual environments

End with a short paragraph on ROS 2 Jazzy + Gazebo Harmonic being the recommended combo for LTS stability.

5. Installation

5.1 Install Gazebo Harmonic

# Follow the official install guide:

# https://gazebosim.org/docs/harmonic/install_ubuntu/

After installation, verify with:



gz sim shapes.sdf

5.2 Install ROS-Gazebo Bridge

sudo apt install ros-jazzy-ros-gz

5.3 Common WSL2 / VM Fixes

Qt Wayland crash fix: export QT_QPA_PLATFORM=xcb

OGRE2 rendering fallback: gz sim shapes.sdf --render-engine ogre

6. Understanding the Gazebo GUI

Include a labeled breakdown of the GUI with these 6 elements (use a numbered list or a table):



Play/Pause — start or pause simulation; use -r flag to auto-start

Real Time Factor — should stay near 100%; drop below 60% = problem

Shape/Light Tools — add and transform basic geometry

Model Hierarchy — tree of all simulation objects and their links

Model Inspector — detailed info on the selected model

Plugin Browser — open tools like Resource Spawner, Lidar Visualizer

7. Download the Offline Model Library

# After downloading and unzipping to ~/gazebo_models:

export GZ_SIM_RESOURCE_PATH=~/gazebo_models

Add this line to ~/.bashrc to make it permanent.

8. Cloning This Repo

Explain that from now on, every session has its own GitHub repo. Students clone the starter branch to begin:



cd ~/sor_ws/src

git clone -b starter https://github.com/Saarthak43/sor-ros-session1.git

cd ~/sor_ws

colcon build --packages-select erc_sor_ros_session1

source install/setup.bash

Show the package tree structure (infer a reasonable one from the context):



erc_sor_ros_session1/

├── urdf/

│ ├── my_robot.xacro

│ ├── sor_bot.gazebo

│ └── materials.xacro

├── launch/

│ ├── check_urdf.launch.py

│ ├── spawn_robot.launch.py

│ └── world.launch.py

├── worlds/

│ └── world.sdf

├── CMakeLists.txt

└── package.xml

9. Opening the Pre-Built World

ros2 launch erc_sor_ros_session1 world.launch.py

Add a note: Gazebo provides physics and world dynamics — but it still has no robot yet.

10. URDF — Universal Robot Description Format

10.1 What is URDF?

Explain: URDF is an XML format that describes a robot's physical structure.



Links = physical components (body, wheels)

Joints = connections between links

10.2 Making the Base Link

<link name="base_footprint"></link>

Note: This defines one solid body — but without visuals or physics yet.



10.3 Adding Visuals

<visual name='base_link_visual'>

<origin xyz="0 0 0" rpy="0 0 0"/>

<geometry>

<box size=".4 .2 .1"/>

</geometry>

</visual>

10.4 Adding Inertia

<inertia ixx="${(m*(w*w+h*h))/12}" ixy="0" ixz="0"

iyy="${(m*(l*l+h*h))/12}" iyz="0"

izz="${(m*(l*l+w*w))/12}"/>

Tip: Use an online inertia matrix calculator for realistic values.



10.5 Adding Wheels

<link name="left_wheel"/>

<link name="right_wheel"/>

10.6 Defining Joints

<joint name="left_wheel_joint" type="continuous">

<!-- parent, child, origin, axis -->

</joint>

Joint types table:

TypeBehaviorcontinuousInfinite rotation (wheels)fixedNo movementrevoluteRotation with limits11. XACRO — Supercharged URDF

Explain: XACRO = XML Macros. Lets you reuse code, create variables, write functions.



<xacro:property name="wheel_radius" value="0.1"/>

Why it matters: Instead of hardcoding coordinates for every wheel, write one macro and call it twice.

12. Creating the URDF File

mkdir -p ~/sor_ws/src/sor-ros-session1/erc_sor_ros_session1/urdf

cd ~/sor_ws/src/sor-ros-session1/erc_sor_ros_session1/urdf

codium .

Create my_robot.xacro and paste the code from this README (provide the full xacro code fetched from the reference repo, adapted to use erc_sor_ros_session1 package name).

Build:



cd ~/sor_ws

colcon build --packages-select erc_sor_ros_session1

source install/setup.bash

13. Visualizing in RViz

13.1 Install URDF Tools

sudo apt install ros-jazzy-urdf

sudo apt install ros-jazzy-urdf-tutorial

sudo apt install ros-jazzy-urdf-launch

13.2 Create the URDF Check Launch File

mkdir -p ~/sor_ws/src/sor-ros-session1/erc_sor_ros_session1/launch

cd ~/sor_ws/src/sor-ros-session1/erc_sor_ros_session1/launch

codium .

Create check_urdf.launch.py (include full launch file code from reference repo, adapted to erc_sor_ros_session1).



13.3 Build and Launch RViz

cd ~/sor_ws

colcon build --packages-select erc_sor_ros_session1

source install/setup.bash

ros2 launch erc_sor_ros_session1 check_urdf.launch.py

In RViz:



Add → RobotModel

Set Fixed Frame to base_footprint

Browse to your URDF file

13.4 Visualise the TF Tree

After launching RViz:



ros2 run rqt_tf_tree rqt_tf_tree

14. Adding Colors

<material name="orange"/>

<material name="green"/>

Include this import inside the <robot> tag:



<xacro:include filename="$(find erc_sor_ros_session1)/urdf/materials.xacro"/>

15. Spawning the Robot in Gazebo

RViz is only a visualizer — it has no physics. We need Gazebo.



cd ~/sor_ws/src/sor-ros-session1/erc_sor_ros_session1/launch

codium .

Create spawn_robot.launch.py and world.launch.py (include full code from reference repo, adapted).

Build and launch:



cd ~/sor_ws

colcon build --packages-select erc_sor_ros_session1

source install/setup.bash

ros2 launch erc_sor_ros_session1 spawn_robot.launch.py

16. The Two-Point Problem — Caster Wheel

When a differential drive robot has only 2 wheels, it tips. Solution: add a caster wheel.



This time the caster is NOT a separate link and joint — it is added directly within base_link.

Hint: Add a sphere collision geometry at the front-bottom of the base_link with zero friction.

17. Giving the Robot Life — Gazebo Plugins

A robot without plugins is just geometry. Plugins add:

PluginPurposeDifferential Drive PluginMakes wheels move based on velocity commandsJoint State Publisher PluginPublishes joint states to ROSSensor PluginsSimulates cameras, LiDAR etc.17.1 Differential Drive — How It Works

Vl = Vr → straight line

Vl = -Vr → rotate in place

Vl = 0 → pivot turn

17.2 Creating the Plugin File

cd ~/sor_ws/src/sor-ros-session1/erc_sor_ros_session1/urdf

codium .

Create sor_bot.gazebo and paste the plugin code (include full .gazebo XML from reference repo).

Add this include inside <robot> in your xacro:



<xacro:include filename="$(find erc_sor_ros_session1)/urdf/sor_bot.gazebo"/>

Build and launch:



cd ~/sor_ws

colcon build --packages-select erc_sor_ros_session1

source install/setup.bash

ros2 launch erc_sor_ros_session1 spawn_robot.launch.py

18. Gazebo Bridge — Making ROS and Gazebo Talk

ROS 2 and Gazebo are two separate universes with different message formats. The ros_gz_bridge translates between them.



18.1 Install

sudo apt install ros-jazzy-ros-gz-bridge

18.2 Update the Launch File

Remove the standalone joint_state_publisher node from spawn_robot.launch.py

Add the gz_bridge_node to LaunchDescription

(Include the updated full spawn_robot.launch.py code with bridge node.)



18.3 Build, Launch, and Drive!

cd ~/sor_ws

colcon build --packages-select erc_sor_ros_session1

source install/setup.bash

ros2 launch erc_sor_ros_session1 spawn_robot.launch.py

In a new terminal:



ros2 run teleop_twist_keyboard teleop_twist_keyboard

19. Assignment

🎯 Record a short video of your Gazebo robot driving around using teleop_twist_keyboard and submit the recording.

20. What's Next?

SessionTopicSession 4Sensors & PerceptionSession 5MappingSession 6NavigationFORMATTING RULES (follow strictly)

Use GitHub-flavored Markdown only

Every section must have a clear ## or ### heading

All terminal commands must be in fenced ```bash ``` blocks

All XML/XACRO/Python code must be in fenced code blocks with the correct language tag

Use tables wherever there are comparisons or lists with 2+ attributes

Use > blockquotes for tips, warnings, and hints

Do NOT use HTML tags

Keep a consistent tone: educational but punchy — like a workshop handout, not a textbook

The README should read like a linear tutorial someone can follow top-to-bottom

Include a Table of Contents at the top with anchor links to every ## section

All code fetched from the reference repo must be adapted to use erc_sor_ros_session1 as the package name, not the original package name

Do NOT mention or credit the reference repo anywhere

Output: A single, complete, copy-paste-ready README.md file. Nothing else.

Session 3 — Entering the 3D World with Gazebo
Simulating a Differential Drive Robot from Scratch
In this session, we stop driving blind and enter a full 3D simulated world. We will design, construct, and actuate a complete physical representation of a differential drive robot using URDF and XACRO, handle physical constraints like weight distribution, and configure full bi-directional communication between ROS 2 and Gazebo Harmonic to bring our virtual hardware to life.

Table of Contents
Quick Recap — What We Know So Far

Mission of This Session

Why Gazebo?

Installation

Understanding the Gazebo GUI

Download the Offline Model Library

Cloning This Repo

Opening the Pre-Built World

URDF — Universal Robot Description Format

XACRO — Supercharged URDF

Creating the URDF File

Visualizing in RViz

Adding Colors

Spawning the Robot in Gazebo

The Two-Point Problem — Caster Wheel

Giving the Robot Life — Gazebo Plugins

Gazebo Bridge — Making ROS and Gazebo Talk

Assignment

What's Next?

Quick Recap — What We Know So Far
Before jumping into the 3D domain, let's verify our baseline understanding of core ROS 2 concepts covered previously:

Concept	Description
Nodes, Topics, Services	Core ROS 2 communication patterns for messaging and synchronous states.
Parameters & Actions	Advanced node configurations and long-running feedback-oriented tasks.
Packages & Workspaces	Logical execution and distribution layout for building and organizing code.
Launch Files	Automating configuration patterns to run multiple nodes and processes concurrently.
