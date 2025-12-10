---
id: 05-digital-twin
title: "Chapter 5 — The Digital Twin: Gazebo Simulation"
---

## What is a Digital Twin?

A digital twin is a virtual model of a physical object, system, or process. In robotics, it's a high-fidelity simulation of a robot and its operational environment. This is more than just a 3D model; it's a dynamic, data-driven representation that mirrors the real-world counterpart's state, behavior, and interactions.

Key characteristics include:
- **Physical Fidelity**: The model accurately represents the robot's geometry, mass, inertia, joints, and materials.
- **Environmental Fidelity**: The simulated world mirrors the real environment, including lighting, textures, and physical obstacles.
- **Sensor Simulation**: Virtual sensors (cameras, LiDAR, IMUs) generate data that closely matches what real sensors would produce.
- **Real-Time Synchronization**: A true digital twin can receive data from the physical robot and update its state, or send commands from the simulation to the physical robot.

```text
  +----------------------+        (Data Sync)         +--------------------+
  |    Physical Robot    | <------------------------> |    Digital Twin    |
  | (Sensors, Actuators) |                            | (Simulated Robot & |
  +----------------------+                            |    Environment)    |
```

## Why Use Digital Twins in Robotics?

Digital twins are crucial for modern robotics development due to several advantages:

- **Safety**: Test dangerous or complex tasks in simulation before deploying them on expensive hardware.
- **Cost-Effectiveness**: Reduce wear and tear on physical robots. Avoid costs associated with hardware damage during early-stage algorithm testing.
- **Speed and Scalability**: Run thousands of tests in parallel, far faster than real-time. Test scenarios that are difficult or time-consuming to set up in the real world.
- **Algorithm Development**: Develop and validate perception, navigation, and manipulation algorithms in a controlled, repeatable environment.
- **Data Generation**: Generate vast amounts of labeled data for training machine learning models (e.g., object detection, segmentation).

## Introduction to Gazebo

Gazebo is a powerful, open-source 3D robotics simulator. It is widely used in the robotics community, especially within the ROS (Robot Operating System) ecosystem.

Core features of Gazebo:
- **Physics Engines**: Supports multiple high-performance physics engines like ODE, Bullet, Simbody, and DART.
- **Sensor Models**: Provides a wide range of realistic sensor models, including cameras, depth sensors, LiDAR, IMUs, and GPS.
- **Robot Models**: Uses the Simulation Description Format (SDF) to define robot models and environments. It can also import URDF (Unified Robot Description Format) models.
- **Plugin Architecture**: Allows users to customize and extend the simulator's functionality with C++ plugins for new sensors, actuators, and world behaviors.
- **ROS Integration**: Offers seamless integration with ROS, allowing you to control and monitor your simulated robot using standard ROS messages, services, and actions.

## Creating a Gazebo World

A Gazebo "world" is an SDF file that defines the environment, including lighting, physics properties, and static or dynamic objects.

A simple world file (`my_world.world`) might look like this:

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <world name="default">
    <!-- A global light source -->
    <include>
      <uri>model://sun</uri>
    </include>

    <!-- A ground plane -->
    <include>
      <uri>model://ground_plane</uri>
    </include>

    <!-- Add a simple box obstacle -->
    <model name="box">
      <pose>2 0 0.5 0 0 0</pose>
      <link name="link">
        <collision name="collision">
          <geometry>
            <box>
              <size>1 1 1</size>
            </box>
          </geometry>
        </collision>
        <visual name="visual">
          <geometry>
            <box>
              <size>1 1 1</size>
            </box>
          </geometry>
        </visual>
      </link>
    </model>
  </world>
</sdf>
```

To run this world, use the command:
```bash
gazebo my_world.world
```

## Spawning a Robot

Robots are "spawned" into the Gazebo world. You can do this from the command line or using a ROS launch file.

Example using a ROS 2 launch file to spawn a TurtleBot3:

```python
# ros2_ws/src/my_robot_spawner/launch/spawn_turtlebot.launch.py

from launch import LaunchDescription
from launch_ros.actions import Node
import os
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():
    # Path to the TurtleBot3 model file
    model_path = os.path.join(get_package_share_directory('turtlebot3_gazebo'),
                              'models', 'turtlebot3_waffle', 'model.sdf')
    
    return LaunchDescription([
        Node(
            package='gazebo_ros',
            executable='spawn_entity.py',
            arguments=['-entity', 'turtlebot3_waffle', '-file', model_path],
            output='screen'
        )
    ])
```

## Linking Gazebo with ROS 2

The `gazebo_ros_pkgs` package provides the necessary bridge between Gazebo and ROS 2. This is typically done through plugins included in your robot's SDF or URDF file.

An essential plugin is the `gazebo_ros_init` which initializes ROS within Gazebo. Another common one is `gazebo_ros_state` which publishes the state of all models.

A differential drive plugin for a wheeled robot would look like this in its URDF/SDF:

```xml
<gazebo>
  <plugin name="differential_drive_controller" filename="libgazebo_ros_diff_drive.so">
    <!-- ROS 2 parameters -->
    <ros>
        <namespace>/my_robot</namespace>
    </ros>

    <!-- wheels -->
    <left_joint>left_wheel_joint</left_joint>
    <right_joint>right_wheel_joint</right_joint>

    <!-- kinematics -->
    <wheel_separation>0.35</wheel_separation>
    <wheel_diameter>0.1</wheel_diameter>

    <!-- output -->
    <publish_odom>true</publish_odom>
    <publish_tf>true</publish_tf>
  </plugin>
</gazebo>
```
This plugin listens to `Twist` messages on the `/my_robot/cmd_vel` topic and publishes odometry on `/my_robot/odom`.

## Simulating Sensors

Gazebo can simulate a wide variety of sensors. Each sensor is added as a plugin.

Example of a camera sensor in a URDF:
```xml
<gazebo reference="camera_link">
  <sensor type="camera" name="camera1">
    <update_rate>30.0</update_rate>
    <camera name="head">
      <horizontal_fov>1.3962634</horizontal_fov>
      <image>
        <width>800</width>
        <height>800</height>
        <format>R8G8B8</format>
      </image>
      <clip>
        <near>0.02</near>
        <far>300</far>
      </clip>
    </camera>
    <plugin name="camera_controller" filename="libgazebo_ros_camera.so">
      <ros>
        <namespace>/my_robot</namespace>
        <image_topic>camera/image_raw</image_topic>
        <camera_info_topic>camera/camera_info</camera_info_topic>
      </ros>
    </plugin>
  </sensor>
</gazebo>
```

This will publish sensor data on the specified ROS 2 topics, which can be consumed by other ROS nodes just like real hardware.

## Testing Navigation with Nav2

Once your robot is spawned with simulated sensors and a drive system, you can launch the ROS 2 Navigation Stack (Nav2) to test autonomous navigation.

The workflow is:
1.  **Launch Gazebo**: Start the simulation with your robot in its environment.
2.  **Launch Nav2**: Run the Nav2 launch file, configured for your robot's topics and parameters.
3.  **Provide a Map**: Use a map created from the simulation environment or a real-world map.
4.  **Set Initial Pose**: In RViz2, tell the robot where it is on the map.
5.  **Send Goal**: Use the RViz2 "Nav2 Goal" tool to command the robot to a destination.

The robot will use its simulated LiDAR and odometry to localize itself and navigate through the world, avoiding obstacles defined in the Gazebo world file.

## The Simulation-to-Reality Workflow

The ultimate goal of a digital twin is to accelerate real-world deployment.

```text
+---------------------+      +----------------+      +------------------+
|      1. Develop     | ---> |   2. Test in   | ---> |  3. Fine-tune on |
| Algorithm (in code) |      |   Simulation   |      |  Physical Robot  |
+---------------------+      +----------------+      +------------------+
        ^     |                    |                         |
        |     +--------------------+-------------------------+
        +------------ (Iterate as needed) -------------------+
```

1.  **Develop**: Write your control, perception, or navigation code against the ROS 2 APIs.
2.  **Sim-Test**: Deploy your code in a ROS 2 workspace that communicates with the Gazebo digital twin. Debug issues and validate performance. This is the fastest iteration loop.
3.  **Real-Test**: Once the algorithm is working reliably in simulation, deploy the *exact same code* to the physical robot.
4.  **Fine-Tune**: The transfer is never perfect. Minor parameter adjustments (e.g., controller gains, sensor noise models) may be needed to account for sim-to-real gaps.
5.  **Iterate**: If significant issues are found on the real robot, go back to the simulation to replicate the failure case, fix the algorithm, and re-verify before deploying again.