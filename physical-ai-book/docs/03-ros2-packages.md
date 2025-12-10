---
id: 03-ros2-packages
title: Chapter 3 — Building ROS 2 Packages
---

## What is a ROS 2 Package?

A ROS 2 package is the fundamental unit of organization for ROS 2 code. It contains everything needed for a piece of software to run, including nodes, libraries, configuration files, and a manifest file (`package.xml`) that describes the package and its dependencies.

Packages are designed to be modular and reusable. A typical ROS 2 project is composed of multiple packages, each responsible for a specific capability, such as sensor processing, navigation, or manipulation.

## Python Package Structure

A standard Python package in ROS 2 follows a specific layout. This structure ensures that ROS 2 build tools like `colcon` can correctly identify, build, and install your code.

Here is a typical directory structure for a Python package named `my_robot_controller`:

```
my_robot_controller/
├── package.xml
├── setup.py
├── setup.cfg
├── resource/
│   └── my_robot_controller
├── my_robot_controller/
│   ├── __init__.py
│   ├── node_one.py
│   └── node_two.py
└── launch/
    └── example.launch.py
```

- **`package.xml`**: The package manifest. It contains metadata like the package name, version, author, and dependencies.
- **`setup.py`**: The standard Python setup script. It defines how to install the package and where to place its files.
- **`resource/`**: A folder containing a marker file with the same name as the package. This helps ROS 2 identify the package's resources.
- **`my_robot_controller/`**: The main Python module containing your node scripts.
- **`launch/`**: A directory for launch files, which are used to start and configure multiple nodes at once.

## Creating a Package with `ros2 pkg create`

ROS 2 provides a convenient command-line tool to quickly scaffold a new package.

To create a new Python package, use the following command:

```bash
ros2 pkg create --build-type ament_python my_python_pkg --dependencies rclpy
```

- `--build-type ament_python`: Specifies that this is a Python package using the `ament` build system.
- `my_python_pkg`: The name of your new package.
- `--dependencies rclpy`: A list of other ROS 2 packages that this package depends on. `rclpy` is the ROS 2 client library for Python.

## Writing Nodes in Python

A node is an executable that performs a specific task. In Python, a node is typically a script that uses the `rclpy` library to communicate with the ROS 2 graph.

Here is a simple "Hello World" publisher node (`my_python_pkg/my_python_pkg/publisher.py`):

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class HelloWorldPublisher(Node):
    def __init__(self):
        super().__init__('hello_world_publisher')
        self.publisher_ = self.create_publisher(String, 'hello_world', 10)
        self.timer = self.create_timer(0.5, self.timer_callback)
        self.get_logger().info('Publisher node started')

    def timer_callback(self):
        msg = String()
        msg.data = f"Hello, World! {self.get_clock().now()}"
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')

def main(args=None):
    rclpy.init(args=args)
    publisher = HelloWorldPublisher()
    rclpy.spin(publisher)
    publisher.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

To make this node executable, you must add an entry point in `setup.py`:

```python
entry_points={
    'console_scripts': [
        'hello_publisher = my_python_pkg.publisher:main',
    ],
},
```

## Adding Dependencies

Dependencies are declared in two places:

1.  **`package.xml`**: For ROS 2 to understand the package relationships. These are used by tools like `rosdep` to install system dependencies.
    ```xml
    <depend>rclpy</depend>
    <depend>std_msgs</depend>
    ```
2.  **`setup.py`**: For Python's `setuptools` to handle package installation.
    ```python
    install_requires=['setuptools'],
    ```

## Workspaces, `ament`, and `colcon`

- **Workspace**: A directory containing one or more ROS 2 packages. A typical workspace has `src`, `build`, `install`, and `log` subdirectories.
- **`ament`**: The underlying build system that `colcon` uses. It provides the logic for building and linking packages.
- **`colcon`**: The primary command-line tool for building and installing ROS 2 packages. It automates the process of building an entire workspace.

To build your workspace, navigate to the root and run:

```bash
colcon build
```

After a successful build, you must source the workspace's setup file to make the new packages available in your environment:

```bash
# For Windows
call install/setup.bat

# For Linux/macOS
source install/setup.bash
```

## Creating Launch Files

Launch files automate the process of running multiple nodes with specific configurations. In ROS 2, Python-based launch files are standard.

Here is an example (`my_python_pkg/launch/hello.launch.py`) that starts our publisher node:

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='my_python_pkg',
            executable='hello_publisher',
            name='my_hello_publisher',
            output='screen',
            emulate_tty=True
        )
    ])
```

To run this launch file:

```bash
ros2 launch my_python_pkg hello.launch.py
```
