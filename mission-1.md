# 🤖 ROS2 Garbage Collection Bot — Learn by Building

> **Flow:** understand → build → run → inspect.  
> Inspection is kept short. The main focus is writing code, running it, and checking only the most useful ROS 2 commands.

**Official references**
- ROS 2 Jazzy installation: https://docs.ros.org/en/jazzy/Installation.html
- ROS 2 Jazzy Ubuntu deb install: https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html
- ROS 2 Jazzy tutorials: https://docs.ros.org/en/jazzy/Tutorials.html

---

## What You Will Build

1. Install ROS 2 Jazzy
2. Create a workspace
3. Create a package
4. Build with colcon
5. Build a node
6. Add topics
7. Add services
8. Add parameters
9. Add actions
10. Add a launch file

---

## Part 0 — Install ROS 2 Jazzy

> These commands are for **Ubuntu 24.04 Noble**.

### 0.1 Install ROS 2 Jazzy

Copy-paste this whole block:

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y software-properties-common curl git python3-pip

sudo add-apt-repository universe -y
sudo apt update

export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')

curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $VERSION_CODENAME)_all.deb"

sudo dpkg -i /tmp/ros2-apt-source.deb

sudo apt update
sudo apt install -y ros-jazzy-desktop ros-dev-tools
```

### 0.2 Source ROS 2

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 0.3 Verify

```bash
printenv ROS_DISTRO
```

Expected output:

```bash
jazzy
```

### 0.4 Install pygame and graph tool

```bash
pip3 install pygame --break-system-packages
sudo apt install -y ros-jazzy-rqt-graph
```

---

## Part 1 — Workspace

A workspace is the folder where your ROS 2 packages live.

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
```

Later, after building, it looks like this:

```text
ros2_ws/
├── src/       # your code
├── build/     # build files
├── install/   # runnable output
└── log/       # logs
```

---

## Part 2 — Creating a Package

Make sure you are in the `src` folder before running package creation commands.

```bash
cd ~/ros2_ws/src
```

### CMake package

Used mostly for C++ packages.

```bash
ros2 pkg create --build-type ament_cmake --license Apache-2.0 <package_name>
```

### Python package

Easier for this session.

```bash
ros2 pkg create --build-type ament_python --license Apache-2.0 <package_name>
```

Now create our package:

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python --license Apache-2.0 ros2_robot_sim
```

Open it:

```bash
cd ~/ros2_ws/src/ros2_robot_sim
codium .
```

---

## Part 3 — Colcon Build

`colcon` is the build tool we use in ROS 2. It takes the code inside `src/` and creates the runnable output inside `install/`.

The basic flow is always this:

```bash
cd ~/ros2_ws
colcon build --packages-select ros2_robot_sim
source install/setup.bash
```

Use this after creating a package, editing `setup.py`, adding a new node, or changing files that ROS 2 needs to discover.

> Always run `colcon build` from the workspace root: `~/ros2_ws`, not from inside `src/`.

---

## Part 4 — Node

### What is a node?

A node is one running program in ROS 2.

For example:

```python
class RobotSimNode(Node):
    def __init__(self):
        super().__init__('robot_sim')
```

Here, `robot_sim` is the node name.

### 4.1 Add dependencies

Replace `package.xml`:

Open the package in Codium:

```bash
cd ~/ros2_ws/src/ros2_robot_sim
codium .
```

Create/replace `package.xml` and paste this code:

```xml
<?xml version="1.0"?>
<package format="3">
  <name>ros2_robot_sim</name>
  <version>0.0.1</version>
  <description>ROS 2 garbage collection bot simulation</description>

  <maintainer email="student@example.com">student</maintainer>
  <license>Apache-2.0</license>
  
  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <depend>rclpy</depend>
  <depend>geometry_msgs</depend>
  <depend>std_srvs</depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

### 4.2 Create a simple node

Open the package in Codium:

```bash
cd ~/ros2_ws/src/ros2_robot_sim
codium .
```

Create/replace `ros2_robot_sim/robot_sim.py` and paste this code:

```python
import rclpy
from rclpy.node import Node


class RobotSimNode(Node):
    def __init__(self):
        super().__init__('robot_sim')
        self.get_logger().info('robot_sim node started')


def main(args=None):
    rclpy.init(args=args)
    node = RobotSimNode()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 4.3 Register the node

Replace `setup.py`:

Open the package in Codium:

```bash
cd ~/ros2_ws/src/ros2_robot_sim
codium .
```

Create/replace `setup.py` and paste this code:

```python
from setuptools import find_packages, setup

package_name = 'ros2_robot_sim'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='ubuntu',
    maintainer_email='1goelkeshav0@gmail.com',
    description='TODO: Package description',
    license='Apache-2.0',
    extras_require={
        'test': [
            'pytest',
        ],
    },
    entry_points={
        'console_scripts': [
            'robot_sim = ros2_robot_sim.robot_sim:main',
        ],
    },
)
```

### 4.4 Build and run

```bash
cd ~/ros2_ws
colcon build --packages-select ros2_robot_sim
source install/setup.bash
ros2 run ros2_robot_sim robot_sim
```
### This wont open up any window for now , it has just connected the ros to our simulation by creating a node 

### 4.5 Inspect only the node

Open another terminal:

```bash
source ~/ros2_ws/install/setup.bash
ros2 node list
ros2 node info /robot_sim
```

That is enough here.  
No topic tools yet because we have not created a topic.

---

## Part 5 — Add pygame movement to the node

Now replace the simple node with a pygame bot.

Open the package in Codium:

```bash
cd ~/ros2_ws/src/ros2_robot_sim
codium .
```

Create/replace `ros2_robot_sim/robot_sim.py` and paste this code:

```python
import math
import threading

import pygame
import rclpy
from rclpy.node import Node


WIDTH, HEIGHT = 800, 600
FPS = 60
BOT_RADIUS = 30

SPEED = 150.0
TURN_SPEED = 2.5

GRID_SIZE = 40
GRID_COLOR = (28, 28, 36)
BG_COLOR = (18, 18, 24)


class RobotSimNode(Node):
    def __init__(self):
        super().__init__('robot_sim')
        self.get_logger().info('robot_sim node started')

        self._lock = threading.Lock()

        self.x = float(WIDTH / 2)
        self.y = float(HEIGHT / 2)
        self.ang = -math.pi / 2

        self.lin_vel = 0.0
        self.ang_vel = 0.0

        self._running = True

        threading.Thread(target=self._pygame_loop, daemon=True).start()

    def _draw_grid(self, screen):
        for x in range(0, WIDTH, GRID_SIZE):
            pygame.draw.line(screen, GRID_COLOR, (x, 0), (x, HEIGHT), 1)
        for y in range(0, HEIGHT, GRID_SIZE):
            pygame.draw.line(screen, GRID_COLOR, (0, y), (WIDTH, y), 1)

    def _draw_hud(self, screen, font, x, y, ang, lin_vel, ang_vel):
        head_deg = math.degrees(ang)
        ang_vel_deg = math.degrees(ang_vel)

        lines = [
            ('ROS2 Robot Sim', (60, 200, 255)),
            (f'node: /robot_sim', (180, 180, 180)),
            (f'pos x: {int(x)}  y: {int(y)}', (220, 220, 220)),
            (f'head {head_deg:.1f} deg', (220, 220, 220)),
            (f'v:{lin_vel:+.0f}  w:{ang_vel_deg:+.1f} deg/s', (220, 220, 220)),
        ]

        panel_x, panel_y = 10, 10
        padding = 6
        line_h = font.get_linesize() + 2
        panel_w = 160
        panel_h = len(lines) * line_h + padding * 2

        surf = pygame.Surface((panel_w, panel_h), pygame.SRCALPHA)
        surf.fill((0, 0, 0, 140))
        screen.blit(surf, (panel_x, panel_y))

        for i, (text, color) in enumerate(lines):
            rendered = font.render(text, True, color)
            screen.blit(rendered, (panel_x + padding, panel_y + padding + i * line_h))

    def _draw_toolbar(self, screen, font):
        toolbar_h = 24
        surf = pygame.Surface((WIDTH, toolbar_h), pygame.SRCALPHA)
        surf.fill((0, 0, 0, 160))
        screen.blit(surf, (0, HEIGHT - toolbar_h))

        items = [
            ('W/S: drive', (200, 200, 200)),
            ('  A/D: turn', (200, 200, 200)),
            ('  ESC: quit', (200, 200, 200)),
        ]
        text = '    W/S: drive      A/D: turn      ESC: quit'
        rendered = font.render(text, True, (180, 180, 180))
        rect = rendered.get_rect(center=(WIDTH // 2, HEIGHT - toolbar_h // 2))
        screen.blit(rendered, rect)

    def _pygame_loop(self):
        pygame.display.init()
        pygame.font.init()

        screen = pygame.display.set_mode((WIDTH, HEIGHT))
        pygame.display.set_caption('ROS2 Robot Sim — /robot_sim node')
        clock = pygame.time.Clock()

        font = pygame.font.SysFont('monospace', 13)

        while self._running:
            dt = clock.tick(FPS) / 1000.0

            lin = 0.0
            ang = 0.0

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self._running = False
                    rclpy.shutdown()
                    return

            keys = pygame.key.get_pressed()

            if keys[pygame.K_w]:
                lin = 1.0
            elif keys[pygame.K_s]:
                lin = -1.0

            if keys[pygame.K_a]:
                ang = -1.0
            elif keys[pygame.K_d]:
                ang = 1.0

            if keys[pygame.K_ESCAPE]:
                self._running = False
                rclpy.shutdown()
                return

            with self._lock:
                self.lin_vel = lin * SPEED
                self.ang_vel = ang * TURN_SPEED

                self.x += self.lin_vel * math.cos(self.ang) * dt
                self.y += self.lin_vel * math.sin(self.ang) * dt
                self.ang += self.ang_vel * dt

                self.x = self.x % WIDTH
                self.y = self.y % HEIGHT

                x = int(self.x)
                y = int(self.y)
                draw_ang = self.ang
                lin_vel = self.lin_vel
                ang_vel = self.ang_vel

            # Draw
            screen.fill(BG_COLOR)
            self._draw_grid(screen)

            # Bot outer ring
            pygame.draw.circle(screen, (20, 60, 100), (x, y), BOT_RADIUS + 4)
            # Bot body
            pygame.draw.circle(screen, (60, 180, 255), (x, y), BOT_RADIUS)
            # Direction line
            tip_x = int(x + math.cos(draw_ang) * BOT_RADIUS)
            tip_y = int(y + math.sin(draw_ang) * BOT_RADIUS)
            pygame.draw.line(screen, (255, 255, 255), (x, y), (tip_x, tip_y), 3)

            self._draw_hud(screen, font, x, y, draw_ang, lin_vel, ang_vel)
            self._draw_toolbar(screen, font)

            pygame.display.flip()

        pygame.quit()

    def destroy_node(self):
        self._running = False
        super().destroy_node()


def main(args=None):
    rclpy.init(args=args)
    node = RobotSimNode()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        if rclpy.ok():
            rclpy.shutdown()


if __name__ == '__main__':
    main()
```

Build and run:

```bash
cd ~/ros2_ws
colcon build --packages-select ros2_robot_sim
source install/setup.bash
ros2 run ros2_robot_sim robot_sim
```

Use the terminal for keys:

```text
W = forward
S = backward
A = left
D = right
Q = quit
```

---
Again in another terminal introspect the node !!


## Part 6 — Topic

### What is a topic?

A topic is a named channel.

```text
publisher node  --->  topic  --->  subscriber node
```

We will create this:

```text
/teleop_node  --->  /cmd_vel  --->  /robot_sim
```

### 6.1 Check the message type

```bash
ros2 interface show geometry_msgs/msg/Twist
```

Important fields:

```text
linear.x   # forward/backward
angular.z  # rotation
```

### 6.2 Add subscriber code to `robot_sim.py` ( REPLACE THE CURRENT CODE WITH THIS )
```bash
import math
import threading

import pygame
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist

WIDTH, HEIGHT = 800, 600
FPS = 60
BOT_RADIUS = 30

SPEED = 150.0
TURN_SPEED = 2.5

GRID_SIZE = 40
GRID_COLOR = (28, 28, 36)
BG_COLOR = (18, 18, 24)


class RobotSimNode(Node):
    def __init__(self):
        super().__init__('robot_sim')
        self.get_logger().info('robot_sim node started')

        self._lock = threading.Lock()

        self.x = float(WIDTH / 2)
        self.y = float(HEIGHT / 2)
        self.ang = -math.pi / 2

        self.lin_vel = 0.0
        self.ang_vel = 0.0

        self._running = True

        self.create_subscription(Twist, '/cmd_vel', self._cmd_vel_cb, 10)
        threading.Thread(target=self._pygame_loop, daemon=True).start()

    def _cmd_vel_cb(self, msg):
        with self._lock:
            self.lin_vel = msg.linear.x * SPEED
            self.ang_vel = msg.angular.z * TURN_SPEED

    def _draw_grid(self, screen):
        for x in range(0, WIDTH, GRID_SIZE):
            pygame.draw.line(screen, GRID_COLOR, (x, 0), (x, HEIGHT), 1)
        for y in range(0, HEIGHT, GRID_SIZE):
            pygame.draw.line(screen, GRID_COLOR, (0, y), (WIDTH, y), 1)

    def _draw_hud(self, screen, font, x, y, ang, lin_vel, ang_vel):
        head_deg = math.degrees(ang)
        ang_vel_deg = math.degrees(ang_vel)

        lines = [
            ('ROS2 Robot Sim', (60, 200, 255)),
            ('node: /robot_sim', (180, 180, 180)),
            (f'pos x: {int(x)}  y: {int(y)}', (220, 220, 220)),
            (f'head {head_deg:.1f} deg', (220, 220, 220)),
            (f'v:{lin_vel:+.0f}  w:{ang_vel_deg:+.1f} deg/s', (220, 220, 220)),
        ]

        panel_x, panel_y = 10, 10
        padding = 6
        line_h = font.get_linesize() + 2
        panel_w = 160
        panel_h = len(lines) * line_h + padding * 2

        surf = pygame.Surface((panel_w, panel_h), pygame.SRCALPHA)
        surf.fill((0, 0, 0, 140))
        screen.blit(surf, (panel_x, panel_y))

        for i, (text, color) in enumerate(lines):
            rendered = font.render(text, True, color)
            screen.blit(rendered, (panel_x + padding, panel_y + padding + i * line_h))

    def _draw_toolbar(self, screen, font):
        toolbar_h = 24
        surf = pygame.Surface((WIDTH, toolbar_h), pygame.SRCALPHA)
        surf.fill((0, 0, 0, 160))
        screen.blit(surf, (0, HEIGHT - toolbar_h))

        text = '    W/S: drive      A/D: turn      ESC: quit'
        rendered = font.render(text, True, (180, 180, 180))
        rect = rendered.get_rect(center=(WIDTH // 2, HEIGHT - toolbar_h // 2))
        screen.blit(rendered, rect)

    def _pygame_loop(self):
        pygame.display.init()
        pygame.font.init()

        screen = pygame.display.set_mode((WIDTH, HEIGHT))
        pygame.display.set_caption('ROS2 Robot Sim — /robot_sim node')
        clock = pygame.time.Clock()

        font = pygame.font.SysFont('monospace', 13)

        while self._running:
            dt = clock.tick(FPS) / 1000.0

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self._running = False
                    rclpy.shutdown()
                    return

            if pygame.key.get_pressed()[pygame.K_ESCAPE]:
                self._running = False
                rclpy.shutdown()
                return

            with self._lock:
                self.x += self.lin_vel * math.cos(self.ang) * dt
                self.y += self.lin_vel * math.sin(self.ang) * dt
                self.ang += self.ang_vel * dt

                self.x = self.x % WIDTH
                self.y = self.y % HEIGHT

                x = int(self.x)
                y = int(self.y)
                draw_ang = self.ang
                lin_vel = self.lin_vel
                ang_vel = self.ang_vel

            screen.fill(BG_COLOR)
            self._draw_grid(screen)

            pygame.draw.circle(screen, (20, 60, 100), (x, y), BOT_RADIUS + 4)
            pygame.draw.circle(screen, (60, 180, 255), (x, y), BOT_RADIUS)

            tip_x = int(x + math.cos(draw_ang) * BOT_RADIUS)
            tip_y = int(y + math.sin(draw_ang) * BOT_RADIUS)
            pygame.draw.line(screen, (255, 255, 255), (x, y), (tip_x, tip_y), 3)

            self._draw_hud(screen, font, x, y, draw_ang, lin_vel, ang_vel)
            self._draw_toolbar(screen, font)

            pygame.display.flip()

        pygame.quit()

    def destroy_node(self):
        self._running = False
        super().destroy_node()


def main(args=None):
    rclpy.init(args=args)
    node = RobotSimNode()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        if rclpy.ok():
            rclpy.shutdown()

if __name__ == '__main__':
    main()
```
### 6.3 Create teleop node

Open the package in Codium:

```bash
cd ~/ros2_ws/src/ros2_robot_sim
codium .
```

Create/replace `ros2_robot_sim/teleop_node.py` and paste this code:

```python
import sys
import termios
import tty

import rclpy
from geometry_msgs.msg import Twist
from rclpy.node import Node


KEY_MAP = {
    'w': (1.0, 0.0),
    's': (-1.0, 0.0),
    'a': (0.0, 1.0),
    'd': (0.0, -1.0),
}


def get_key():
    fd = sys.stdin.fileno()
    old = termios.tcgetattr(fd)
    try:
        tty.setraw(fd)
        return sys.stdin.read(1)
    finally:
        termios.tcsetattr(fd, termios.TCSADRAIN, old)


class TeleopNode(Node):
    def __init__(self):
        super().__init__('teleop_node')
        self.publisher = self.create_publisher(Twist, '/cmd_vel', 10)
        print('\nW/S: drive | A/D: turn | Q: quit\n')

    def run(self):
        while rclpy.ok():
            key = get_key().lower()
            msg = Twist()
            if key == 'q':
                break
            if key in KEY_MAP:
                msg.linear.x, msg.angular.z = KEY_MAP[key]
            self.publisher.publish(msg)


def main(args=None):
    rclpy.init(args=args)
    node = TeleopNode()
    try:
        node.run()
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        if rclpy.ok():
            rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 6.4 Register teleop

Update the `entry_points` part of `setup.py`:

```python
entry_points={
    'console_scripts': [
        'robot_sim = ros2_robot_sim.robot_sim:main',
        'teleop = ros2_robot_sim.teleop_node:main',
    ],
},
```

### 6.5 Build and run

```bash
cd ~/ros2_ws
colcon build --packages-select ros2_robot_sim
source install/setup.bash
```

Terminal 1:

```bash
ros2 run ros2_robot_sim robot_sim
```

Terminal 2:

```bash
source ~/ros2_ws/install/setup.bash
ros2 run ros2_robot_sim teleop
```

### 6.6 Inspect the topic

```bash
source ~/ros2_ws/install/setup.bash

ros2 topic list
ros2 topic info /cmd_vel
ros2 topic echo /cmd_vel
```

Graph:

```bash
ros2 run rqt_graph rqt_graph
```

---

## Part 7 — Service

### What is a service?

A service is one request and one response.

We will add:

```text
/reset
```

### 7.1 Add service server to `robot_sim.py`

Add this import at the top with the others inside robot_sim.py

```python
from std_srvs.srv import Empty
```

Inside `__init__`:
Add this line inside `__init__ `after `self.create_subscription` line:
```python
self.create_service(Empty, '/reset', self._reset_cb)
```

Inside the class 
Add this method inside the class after `_cmd_vel_cb`:
```python
def _reset_cb(self, req, res):
    with self._lock:
        self.x = float(WIDTH / 2)
        self.y = float(HEIGHT / 2)
        self.ang = -math.pi / 2
        self.lin_vel = 0.0
        self.ang_vel = 0.0

    self.get_logger().info('Robot reset')
    return res
```
intendation should be exactly like this 

#### if you are getting intendation error still , directly replace your robot_sim.py with this , it will contain all the changes done aabove 

```bash
import math
import threading

import pygame
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
from std_srvs.srv import Empty

WIDTH, HEIGHT = 800, 600
FPS = 60
BOT_RADIUS = 30

SPEED = 150.0
TURN_SPEED = 2.5

GRID_SIZE = 40
GRID_COLOR = (28, 28, 36)
BG_COLOR = (18, 18, 24)


class RobotSimNode(Node):
    def __init__(self):
        super().__init__('robot_sim')
        self.get_logger().info('robot_sim node started')

        self._lock = threading.Lock()

        self.x = float(WIDTH / 2)
        self.y = float(HEIGHT / 2)
        self.ang = -math.pi / 2

        self.lin_vel = 0.0
        self.ang_vel = 0.0

        self._running = True

        self.create_subscription(Twist, '/cmd_vel', self._cmd_vel_cb, 10)
        self.create_service(Empty, '/reset', self._reset_cb)
        threading.Thread(target=self._pygame_loop, daemon=True).start()

    def _cmd_vel_cb(self, msg):
        with self._lock:
            self.lin_vel = msg.linear.x * SPEED
            self.ang_vel = msg.angular.z * TURN_SPEED

    def _reset_cb(self, req, res):
        with self._lock:
            self.x = float(WIDTH / 2)
            self.y = float(HEIGHT / 2)
            self.ang = -math.pi / 2
            self.lin_vel = 0.0
            self.ang_vel = 0.0
        self.get_logger().info('Robot reset')
        return res

    def _draw_grid(self, screen):
        for x in range(0, WIDTH, GRID_SIZE):
            pygame.draw.line(screen, GRID_COLOR, (x, 0), (x, HEIGHT), 1)
        for y in range(0, HEIGHT, GRID_SIZE):
            pygame.draw.line(screen, GRID_COLOR, (0, y), (WIDTH, y), 1)

    def _draw_hud(self, screen, font, x, y, ang, lin_vel, ang_vel):
        head_deg = math.degrees(ang)
        ang_vel_deg = math.degrees(ang_vel)

        lines = [
            ('ROS2 Robot Sim', (60, 200, 255)),
            ('node: /robot_sim', (180, 180, 180)),
            (f'pos x: {int(x)}  y: {int(y)}', (220, 220, 220)),
            (f'head {head_deg:.1f} deg', (220, 220, 220)),
            (f'v:{lin_vel:+.0f}  w:{ang_vel_deg:+.1f} deg/s', (220, 220, 220)),
        ]

        panel_x, panel_y = 10, 10
        padding = 6
        line_h = font.get_linesize() + 2
        panel_w = 160
        panel_h = len(lines) * line_h + padding * 2

        surf = pygame.Surface((panel_w, panel_h), pygame.SRCALPHA)
        surf.fill((0, 0, 0, 140))
        screen.blit(surf, (panel_x, panel_y))

        for i, (text, color) in enumerate(lines):
            rendered = font.render(text, True, color)
            screen.blit(rendered, (panel_x + padding, panel_y + padding + i * line_h))

    def _draw_toolbar(self, screen, font):
        toolbar_h = 24
        surf = pygame.Surface((WIDTH, toolbar_h), pygame.SRCALPHA)
        surf.fill((0, 0, 0, 160))
        screen.blit(surf, (0, HEIGHT - toolbar_h))

        text = '    W/S: drive      A/D: turn      ESC: quit'
        rendered = font.render(text, True, (180, 180, 180))
        rect = rendered.get_rect(center=(WIDTH // 2, HEIGHT - toolbar_h // 2))
        screen.blit(rendered, rect)

    def _pygame_loop(self):
        pygame.display.init()
        pygame.font.init()

        screen = pygame.display.set_mode((WIDTH, HEIGHT))
        pygame.display.set_caption('ROS2 Robot Sim — /robot_sim node')
        clock = pygame.time.Clock()

        font = pygame.font.SysFont('monospace', 13)

        while self._running:
            dt = clock.tick(FPS) / 1000.0

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self._running = False
                    rclpy.shutdown()
                    return

            if pygame.key.get_pressed()[pygame.K_ESCAPE]:
                self._running = False
                rclpy.shutdown()
                return

            with self._lock:
                self.x += self.lin_vel * math.cos(self.ang) * dt
                self.y += self.lin_vel * math.sin(self.ang) * dt
                self.ang += self.ang_vel * dt

                self.x = self.x % WIDTH
                self.y = self.y % HEIGHT

                x = int(self.x)
                y = int(self.y)
                draw_ang = self.ang
                lin_vel = self.lin_vel
                ang_vel = self.ang_vel

            screen.fill(BG_COLOR)
            self._draw_grid(screen)

            pygame.draw.circle(screen, (20, 60, 100), (x, y), BOT_RADIUS + 4)
            pygame.draw.circle(screen, (60, 180, 255), (x, y), BOT_RADIUS)

            tip_x = int(x + math.cos(draw_ang) * BOT_RADIUS)
            tip_y = int(y + math.sin(draw_ang) * BOT_RADIUS)
            pygame.draw.line(screen, (255, 255, 255), (x, y), (tip_x, tip_y), 3)

            self._draw_hud(screen, font, x, y, draw_ang, lin_vel, ang_vel)
            self._draw_toolbar(screen, font)

            pygame.display.flip()

        pygame.quit()

    def destroy_node(self):
        self._running = False
        super().destroy_node()


def main(args=None):
    rclpy.init(args=args)
    node = RobotSimNode()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        if rclpy.ok():
            rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 7.2 Test from CLI

```bash
cd ~/ros2_ws
colcon build --packages-select ros2_robot_sim
source install/setup.bash
ros2 run ros2_robot_sim robot_sim
```
Also move your bot using teleop key in another terminal 
```bash
source ~/ros2_ws/install/setup.bash
ros2 run ros2_robot_sim teleop
```
Another terminal:

```bash
source ~/ros2_ws/install/setup.bash

ros2 service list
ros2 service type /reset
ros2 service call /reset std_srvs/srv/Empty "{}"
```

### 7.3 Test as a separate node

Open the package in Codium:

```bash
cd ~/ros2_ws/src/ros2_robot_sim
codium .
```

Create/replace `ros2_robot_sim/reset_client.py` and paste this code:

```python
import rclpy
from rclpy.node import Node
from std_srvs.srv import Empty


class ResetClient(Node):
    def __init__(self):
        super().__init__('reset_client')
        self.client = self.create_client(Empty, '/reset')

    def send_request(self):
        while not self.client.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('Waiting for /reset...')

        request = Empty.Request()
        future = self.client.call_async(request)

        rclpy.spin_until_future_complete(self, future)
        self.get_logger().info('Reset complete')


def main(args=None):
    rclpy.init(args=args)

    node = ResetClient()
    node.send_request()
    node.destroy_node()

    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

Add to `setup.py`:

```python
'reset_client = ros2_robot_sim.reset_client:main',
```

Build:

```bash
cd ~/ros2_ws
colcon build --packages-select ros2_robot_sim
source install/setup.bash
ros2 run ros2_robot_sim reset_client
```

---

## Part 8 — Parameters

### What is a parameter?

A parameter is a value inside a node that we can change while the node is running.

Add inside `__init__`:

```python
self.declare_parameter('speed', 150.0)
self.declare_parameter('turn_speed', 2.5)
```

Update `_cmd_vel_cb`:

```python
def _cmd_vel_cb(self, msg):
    with self._lock:
        speed = self.get_parameter('speed').value
        turn_speed = self.get_parameter('turn_speed').value

        self.lin_vel = msg.linear.x * speed
        self.ang_vel = msg.angular.z * turn_speed
```

Build and run:

```bash
cd ~/ros2_ws
colcon build --packages-select ros2_robot_sim
source install/setup.bash
ros2 run ros2_robot_sim robot_sim
```

Inspect/change:

```bash
source ~/ros2_ws/install/setup.bash

ros2 param list /robot_sim
ros2 param get /robot_sim speed
ros2 param set /robot_sim speed 300.0
ros2 param set /robot_sim speed 80.0
ros2 param set /robot_sim speed 150.0
```

---

## Part 9 — Action

### What is an action?

An action is for longer tasks.

```text
Goal      -> start task
Feedback  -> live progress
Result    -> final output
```

We will create:

```text
/collect_garbage
```

### 9.1 Create interface package

Custom actions should go in a small `ament_cmake` interface package.

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake --license Apache-2.0 garbage_interfaces
mkdir -p ~/ros2_ws/src/garbage_interfaces/action
```

Open the package and create the action file:

Open the package in Codium:

```bash
cd ~/ros2_ws/src/garbage_interfaces
codium .
```

Create/replace `action/CollectGarbage.action` and paste this code:

```text
int32 max_piles
---
int32 collected
---
float32 distance_to_target
string status
```

Replace `package.xml`:

Open the package in Codium:

```bash
cd ~/ros2_ws/src/garbage_interfaces
codium .
```

Create/replace `package.xml` and paste this code:

```xml
<?xml version="1.0"?>
<package format="3">
  <name>garbage_interfaces</name>
  <version>0.0.1</version>
  <description>Custom action interfaces</description>

  <maintainer email="student@example.com">student</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <build_depend>rosidl_default_generators</build_depend>
  <exec_depend>rosidl_default_runtime</exec_depend>

  <member_of_group>rosidl_interface_packages</member_of_group>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

Replace `CMakeLists.txt`:

Open the package in Codium:

```bash
cd ~/ros2_ws/src/garbage_interfaces
codium .
```

Create/replace `CMakeLists.txt` and paste this code:

```cmake
cmake_minimum_required(VERSION 3.8)
project(garbage_interfaces)

find_package(ament_cmake REQUIRED)
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "action/CollectGarbage.action"
)

ament_export_dependencies(rosidl_default_runtime)
ament_package()
```

Build interface:

```bash
cd ~/ros2_ws
colcon build --packages-select garbage_interfaces
source install/setup.bash

ros2 interface show garbage_interfaces/action/CollectGarbage
```

### 9.2 Action server snippet

In `robot_sim.py`, imports:

```python
import time
from rclpy.action import ActionServer, GoalResponse, CancelResponse
from garbage_interfaces.action import CollectGarbage
```

Inside `__init__`:

```python
self.action_server = ActionServer(
    self,
    CollectGarbage,
    '/collect_garbage',
    execute_callback=self._collect_execute,
    goal_callback=self._goal_cb,
    cancel_callback=self._cancel_cb,
)
```

Callbacks:

```python
def _goal_cb(self, goal_request):
    self.get_logger().info(f'Received goal: {goal_request.max_piles}')
    return GoalResponse.ACCEPT


def _cancel_cb(self, goal_handle):
    self.get_logger().info('Cancel accepted')
    return CancelResponse.ACCEPT


def _collect_execute(self, goal_handle):
    feedback = CollectGarbage.Feedback()
    result = CollectGarbage.Result()

    collected = 0

    for i in range(goal_handle.request.max_piles):
        if goal_handle.is_cancel_requested:
            goal_handle.canceled()
            result.collected = collected
            return result

        feedback.distance_to_target = float(100 - i * 10)
        feedback.status = f'collecting pile {i + 1}'
        goal_handle.publish_feedback(feedback)

        self.get_logger().info(feedback.status)

        collected += 1
        time.sleep(1.0)

    goal_handle.succeed()
    result.collected = collected
    return result
```

### 9.3 Action client

Open the package in Codium:

```bash
cd ~/ros2_ws/src/ros2_robot_sim
codium .
```

Create/replace `ros2_robot_sim/collect_client.py` and paste this code:

```python
import rclpy
from garbage_interfaces.action import CollectGarbage
from rclpy.action import ActionClient
from rclpy.node import Node


class CollectClient(Node):
    def __init__(self):
        super().__init__('collect_client')
        self.client = ActionClient(self, CollectGarbage, '/collect_garbage')

    def send_goal(self):
        self.client.wait_for_server()

        goal = CollectGarbage.Goal()
        goal.max_piles = 5

        future = self.client.send_goal_async(
            goal,
            feedback_callback=self.feedback_callback,
        )

        future.add_done_callback(self.goal_response_callback)

    def feedback_callback(self, feedback_msg):
        feedback = feedback_msg.feedback
        self.get_logger().info(f'{feedback.status} | distance={feedback.distance_to_target}')

    def goal_response_callback(self, future):
        goal_handle = future.result()

        if not goal_handle.accepted:
            self.get_logger().info('Goal rejected')
            rclpy.shutdown()
            return

        result_future = goal_handle.get_result_async()
        result_future.add_done_callback(self.result_callback)

    def result_callback(self, future):
        result = future.result().result
        self.get_logger().info(f'Collected {result.collected} piles')
        rclpy.shutdown()


def main(args=None):
    rclpy.init(args=args)

    node = CollectClient()
    node.send_goal()

    rclpy.spin(node)


if __name__ == '__main__':
    main()
```

Add dependencies to `ros2_robot_sim/package.xml`:

```xml
<depend>action_msgs</depend>
<depend>garbage_interfaces</depend>
```

Add to `setup.py`:

```python
'collect_client = ros2_robot_sim.collect_client:main',
```

Build everything:

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
```

Run server:

```bash
ros2 run ros2_robot_sim robot_sim
```

Run client:

```bash
ros2 run ros2_robot_sim collect_client
```

Inspect action:

```bash
ros2 action list
ros2 action info /collect_garbage
ros2 action send_goal /collect_garbage garbage_interfaces/action/CollectGarbage "{max_piles: 5}" --feedback
```

---

## Part 10 — Launch File

### What is a launch file?

A launch file starts nodes using one command.

Create a launch package:

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python --license Apache-2.0 robot_bringup
mkdir -p ~/ros2_ws/src/robot_bringup/launch
```

Replace `setup.py`:

Open the package in Codium:

```bash
cd ~/ros2_ws/src/robot_bringup
codium .
```

Create/replace `setup.py` and paste this code:

```python
from glob import glob
from setuptools import find_packages, setup

package_name = 'robot_bringup'

setup(
    name=package_name,
    version='0.0.1',
    packages=find_packages(exclude=['test']),
    data_files=[
        ('share/ament_index/resource_index/packages', ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        ('share/' + package_name + '/launch', glob('launch/*.launch.py')),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='student',
    maintainer_email='student@example.com',
    description='Launch files for garbage bot',
    license='Apache-2.0',
    entry_points={
        'console_scripts': [],
    },
)
```

Create launch file:

Open the package in Codium:

```bash
cd ~/ros2_ws/src/robot_bringup
codium .
```

Create/replace `launch/garbage_bot.launch.py` and paste this code:

```python
from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description():
    return LaunchDescription([
        Node(
            package='ros2_robot_sim',
            executable='robot_sim',
            name='robot_sim',
            output='screen',
        ),
    ])
```

Build and launch:

```bash
cd ~/ros2_ws
colcon build --packages-select robot_bringup
source install/setup.bash
ros2 launch robot_bringup garbage_bot.launch.py
```

Run teleop separately because it needs keyboard input:

```bash
source ~/ros2_ws/install/setup.bash
ros2 run ros2_robot_sim teleop
```

---

## Final Inspection Cheat Sheet

| Concept | Command |
|---|---|
| Nodes | `ros2 node list` |
| Node details | `ros2 node info /robot_sim` |
| Topics | `ros2 topic list` |
| Topic data | `ros2 topic echo /cmd_vel` |
| Services | `ros2 service list` |
| Call reset | `ros2 service call /reset std_srvs/srv/Empty "{}"` |
| Parameters | `ros2 param list /robot_sim` |
| Change speed | `ros2 param set /robot_sim speed 300.0` |
| Actions | `ros2 action list` |
| Send action goal | `ros2 action send_goal /collect_garbage garbage_interfaces/action/CollectGarbage "{max_piles: 5}" --feedback` |
| Graph | `ros2 run rqt_graph rqt_graph` |

---

## Final Demo Flow

Terminal 1:

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch robot_bringup garbage_bot.launch.py
```

Terminal 2:

```bash
source ~/ros2_ws/install/setup.bash
ros2 run ros2_robot_sim teleop
```

Terminal 3:

```bash
source ~/ros2_ws/install/setup.bash
ros2 run ros2_robot_sim collect_client
```

