---
title: Building ROS 2 Packages
---

In the previous chapters, we learned about the core communication concepts in ROS 2. But how do we organize, build, and run our code in a scalable way? The answer lies in ROS 2 packages and launch files.

## What is a ROS 2 Package?

A ROS 2 package is a directory containing your source code, build files, and a `package.xml` manifest file. It's the fundamental unit for organizing and sharing ROS 2 code. A well-structured package is self-contained and reusable.

To create a new Python package, you can use the `ros2 pkg create` command:
```bash
ros2 pkg create --build-type ament_python --node-name my_first_node my_package
```
This command creates a new directory `my_package` with the following structure:
```
my_package/
├── my_package/
│   ├── __init__.py
│   └── my_first_node.py
├── resource/
│   └── my_package
├── test/
│   ├── test_copyright.py
│   ├── test_flake8.py
│   └── test_pep257.py
├── package.xml
└── setup.py
```

### Key Files in a Package:

*   **`package.xml`:** This file contains meta-information about the package, such as its name, version, author, and dependencies. The ROS 2 build tools use this file to understand how to build and link your package against other packages.

*   **`setup.py`:** This is the build script for a Python-based ROS 2 package. It tells the build system where to find your executable scripts (your ROS 2 nodes) and how to install them. You must register your nodes in the `entry_points` section:
    ```python
    # setup.py
    # ...
    entry_points={
        'console_scripts': [
            'my_node = my_package.my_first_node:main',
        ],
    },
    ```

## Building and Sourcing

Once you have your package set up, you need to build it using `colcon`, the standard build tool for ROS 2. From the root of your workspace, you run:
```bash
colcon build
```
This command will find all the ROS 2 packages in the current directory, build them, and place the output in the `install/` directory.

After building, you need to "source" the setup files to make your new packages available in your terminal environment:
```bash
# For Windows
call install/setup.bat

# For Linux/macOS
source install/setup.bash
```
Now you can run your node using `ros2 run`:
```bash
ros2 run my_package my_node
```

## Launch Files: Running Multiple Nodes

Running each node in a separate terminal is tedious. ROS 2 launch files allow you to define a multi-node system in a single Python script.

Here is an example `my_launch_file.py`:
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='my_package',
            executable='talker_node',
            name='my_talker'
        ),
        Node(
            package='my_package',
            executable='listener_node',
            name='my_listener'
        )
    ])
```
You can run this launch file using the `ros2 launch` command:
```bash
ros2 launch my_package my_launch_file.py
```
This will start both the `talker_node` and the `listener_node` at the same time. Launch files are incredibly powerful and can be used to manage complex systems with many parameters and configurations.