---
id: 04-ros2-in-depth
title: Chapter 4 — ROS 2 In-Depth
---

## Core ROS 2 Concepts

ROS 2 is built on a set of core concepts that enable distributed, modular robotics applications. Understanding these concepts is essential for designing robust systems.

### Nodes
A **node** is the smallest unit of computation in ROS 2. Each node should be responsible for a single, well-defined task, such as controlling a motor, reading a sensor, or planning a path. Nodes communicate with each other through the ROS 2 graph using topics, services, actions, and parameters.

### Communication Primitives

**Topics**
- **Purpose**: For continuous, one-to-many data streams. A publisher sends messages to a topic, and any number of subscribers can listen.
- **Analogy**: A public announcement board. Anyone can post a message, and anyone can read it.
- **Example**: A camera node publishing `sensor_msgs/Image` messages to the `/camera/image_raw` topic.

**Services**
- **Purpose**: For synchronous, one-to-one request/reply interactions. A client sends a request, and a server provides a single response.
- **Analogy**: A function call. You ask for something and wait for the result.
- **Example**: A service `/set_robot_speed` where the client requests a new speed and the server confirms it has been set.

**Actions**
- **Purpose**: For long-running, asynchronous tasks that provide continuous feedback and can be preempted.
- **Analogy**: Ordering a complex task with progress updates. You can track the progress and cancel the order if needed.
- **Example**: An action `/navigate_to_pose` where the client sends a goal pose, receives regular updates on the robot's progress, and gets a final result (success or failure).

**ASCII Diagram of Communication:**
```
            [Node A]                       [Node B]
               |                              ^
 (Publisher)   | --<sensor_msgs/Image>---> | (Subscriber)
               +------- (Topic) ----------+

            [Node C]                       [Node D]
               |                              ^
(Service Client) | --<Request>--+             | (Service Server)
               |             |-->[Service]--|
               | <--<Response>-+             |
               +----------------------------+

            [Node E]                       [Node F]
               |                              ^
(Action Client)  | --<Goal>-----+             | (Action Server)
               |             |-->[Action]---|
               | <--<Feedback>--+             |
               | <--<Result>----+             |
               +----------------------------+
```

## Quality of Service (QoS) Profiles

QoS settings give you fine-grained control over the reliability and behavior of communication. This is crucial for handling different types of data, from lossy sensor streams to critical control commands.

Key QoS Policies:
- **History**: Keep all messages or only the last N.
- **Depth**: The size of the queue when `History` is set to keep last.
- **Reliability**: Best-effort (faster, may drop messages) or reliable (guaranteed delivery).
- **Durability**: Volatile (only future subscribers receive messages) or transient local (new subscribers receive the last message).

## Lifecycle Nodes

Lifecycle nodes (or managed nodes) introduce a state machine to control the startup and shutdown sequence of your nodes. This ensures that a system starts, stops, and recovers in a predictable and orderly manner.

The standard states are:
- `Unconfigured`
- `Inactive`
- `Active`
- `Finalized`

Transitions between states (e.g., `on_configure`, `on_activate`) allow you to load configurations, open hardware connections, and start processing at the right time.

## Parameters and Callbacks

Parameters allow you to configure nodes externally without recompiling code. They can be set from launch files, the command line, or other nodes.

You can also set up **parameter callbacks**, which are functions that automatically execute when a parameter's value is changed. This is extremely useful for dynamically reconfiguring a running system.

**Example: Parameter Callback in Python**
```python
import rclpy
from rclpy.node import Node
from rcl_interfaces.msg import ParameterDescriptor

class MyDynamicNode(Node):
    def __init__(self):
        super().__init__('my_dynamic_node')
        # Declare a parameter with a descriptor
        my_param_descriptor = ParameterDescriptor(description='This is my dynamic parameter.')
        self.declare_parameter('my_param', 'default_value', my_param_descriptor)
        
        # Register a callback
        self.add_on_set_parameters_callback(self.parameters_callback)

    def parameters_callback(self, params):
        from rcl_interfaces.msg import SetParametersResult
        for param in params:
            if param.name == 'my_param':
                self.get_logger().info(f'Parameter "my_param" changed to: {param.value}')
        return SetParametersResult(successful=True)

# ... main function ...
```

## Multi-Machine Communication

ROS 2 is designed for distributed systems. Communication across multiple computers works out-of-the-box as long as they are on the same network and the underlying DDS (Data Distribution Service) middleware is correctly configured.

For discovery to work, ensure:
1.  All machines are on the same LAN.
2.  The `ROS_DOMAIN_ID` environment variable is the same on all machines (defaults to `0`).
3.  Firewalls are not blocking DDS discovery traffic (typically uses UDP multicast).

## Best Practices for Structuring Large Projects

1.  **One Node, One Job**: Keep nodes focused on a single responsibility.
2.  **Package by Capability**: Group related nodes and logic into packages (e.g., `my_robot_driver`, `my_robot_navigation`).
3.  **Use Launch Files for Composition**: Define your application by composing nodes in launch files. Avoid hardcoding node relationships in the code.
4.  **Define Clear Interfaces**: Use custom message (`.msg`), service (`.srv`), and action (`.action`) definitions in a dedicated `my_robot_interfaces` package.
5.  **Parameterize Everything**: Expose important constants and settings as ROS 2 parameters.
6.  **Use Lifecycle Nodes**: For any node that manages a resource (like hardware), use lifecycle management to ensure clean startup and shutdown.
```