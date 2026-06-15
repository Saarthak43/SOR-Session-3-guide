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

```bash
# Follow the official install guide:
# https://gazebosim.org/docs/harmonic/install_ubuntu/
```

Verify the installation:

```bash
gz sim shapes.sdf
```

> If Gazebo opens with the shapes world, the installation is working.

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
