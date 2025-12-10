---
id: 06-unity-simulation
title: "Chapter 6 — Unity for High-Fidelity Humanoid Simulation"
---

## Introduction to Unity in Robotics

While Gazebo is a staple in the ROS community, Unity has emerged as a powerful alternative for creating high-fidelity simulations, especially for complex robots like humanoids and for applications requiring photorealistic visuals and advanced physics. Unity's real-time development platform, originally built for gaming, provides a rich set of tools for robotics simulation.

## Unity's Physics Engine for Robotics

Unity's built-in physics engine (NVIDIA PhysX) is highly capable for robotics simulation. It provides:

- **Rigid Body Dynamics**: Simulates the motion of complex objects under the influence of forces and torques.
- **Joints**: A variety of joint types (Hinge, Slider, Ball-and-Socket, Configurable) allow for the creation of complex kinematic chains, essential for humanoids.
- **Collision Detection**: Advanced and efficient collision detection systems (Discrete, Continuous, and Continuous Speculative).
- **Material Properties**: Define friction, bounciness, and other physical properties for realistic interactions.

For robotics, it's crucial to set the physics solver to run at a high, fixed frequency (e.g., >200 Hz) to ensure stability, especially for dynamically balanced robots.

## Humanoid Animation and Rigging

Unity's "Mecanim" animation system is a key advantage for humanoid robotics.

- **Avatar System**: Unity can create an "Avatar," a generic representation of a humanoid skeleton. This allows animation clips to be retargeted from one humanoid model to another, regardless of their specific proportions.
- **Inverse Kinematics (IK)**: The animation system includes built-in IK solvers, useful for tasks like ensuring a robot's feet stay planted on uneven ground or reaching for an object.
- **Animation Rigging**: This package provides a suite of tools to create procedural, real-time animation effects on top of existing animations. For example, you can create a rig that makes a humanoid's head track a target.

```text
    +-----------------+
    |   Motion Data   |
    | (Mocap, Keyed)  |
    +-----------------+
            |
            v
+-----------------------+      +-------------------+
| Mecanim Retargeting   |      |  Procedural Rig   |
| (Maps to any Avatar)  | ---> | (e.g., Head Look) | ---> Final Pose
+-----------------------+      +-------------------+
```

## Importing Robot Models (URDF/FBX)

Unity can import robot models from standard formats:

- **URDF (Unified Robot Description Format)**: The standard for ROS. The [Unity Robotics Hub](https://github.com/Unity-Technologies/Unity-Robotics-Hub) provides a URDF importer that converts a URDF file into a Unity prefab. It automatically creates the hierarchy of links, configures the joints, and assigns physical materials.
- **FBX (Filmbox)**: A common format from 3D modeling software. FBX files are imported directly by Unity and can contain complex meshes, materials, and animations. They are often used for creating realistic simulation environments.

## Adding Sensors to a Unity Model

While Unity doesn't have a built-in library of robotics sensors like Gazebo, the Unity Robotics Hub provides scripts for common sensors:

- **Camera**: Publishes images, depth maps, and camera info.
- **LiDAR**: Simulates a rotating laser scanner by firing raycasts into the scene and publishing the point cloud.
- **IMU**: Can be approximated by reading the acceleration and angular velocity of the robot's base link.

Custom sensors can be created using C# scripts that perform raycasts, sphere-casts, or read data directly from the physics engine.

## The Unity Robotics Hub

This official Unity package is essential for any robotics project. It includes:
- **ROS 2 Integration**: A robust TCP endpoint for communicating with a ROS 2 network. It can publish and subscribe to any message type.
- **URDF Importer**: As mentioned, this tool is critical for bringing ROS-based robot models into Unity.
- **Tutorials and Example Projects**: Provides a great starting point for common robotics tasks like pick-and-place and navigation.

**Example: ROS Publisher in C#**
```csharp
using UnityEngine;
using Unity.Robotics.ROSTCPConnector;
using RosMessageTypes.Std; // Assumes you have generated messages for std_msgs

public class MyPublisher : MonoBehaviour
{
    ROSConnection ros;
    public string topicName = "my_topic";

    void Start()
    {
        ros = ROSConnection.GetOrCreateInstance();
        ros.RegisterPublisher<StringMsg>(topicName);
    }

    void Update()
    {
        StringMsg msg = new StringMsg("Hello from Unity!");
        ros.Publish(topicName, msg);
    }
}
```

## Comparing Unity and Gazebo

| Feature               | Gazebo                                      | Unity                                         |
| --------------------- | ------------------------------------------- | --------------------------------------------- |
| **Primary Use Case**  | ROS-centric robotics research               | High-fidelity visualization, gaming, VR/AR    |
| **Visuals**           | Functional, but less realistic by default   | Photorealistic rendering capabilities         |
| **Physics**           | Multiple engine options (ODE, Bullet)       | High-performance PhysX, optimized for games   |
| **ROS Integration**   | Native, deep integration (`gazebo_ros_pkgs`)| Via Unity Robotics Hub (TCP-based)            |
| **Community/Assets**  | Strong in the academic/robotics community   | Massive community, huge asset store           |
| **Extensibility**     | C++ plugins                                 | C# scripts, visual scripting                  |
| **Best For...**       | Traditional ROS workflows, rapid prototyping| Humanoid simulation, sim-to-real (vision), VR |

**Diagram: Architectural Difference**

**Gazebo:**
```text
  [ ROS Master ] <--> [ Gazebo Node (with plugins) ] <--> [ Physics/Sensors ]
   (Direct Communication via shared memory where possible)
```

**Unity:**
```text
  [ ROS Master ] <--> [ ROS-TCP-Endpoint Node ] <--> [ Unity (C# Scripts) ] <--> [ Physics/Sensors ]
   (Communication over TCP network socket)
```