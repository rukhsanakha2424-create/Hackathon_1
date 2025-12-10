---
title: ROS 2 In-Depth
---

This chapter dives deep into the core communication protocols of ROS 2. We'll explore how to practically implement Nodes, Topics, Services, and Actions using Python, the most common language for high-level robotics programming in ROS 2.

## A Closer Look at Nodes

A ROS 2 system is a graph of nodes. A node is responsible for a single, modular purpose (e.g., controlling a wheel, reading a sensor, planning a path). In Python, a node is a class that inherits from `rclpy.node.Node`.

```python
import rclpy
from rclpy.node import Node

class MyNode(Node):
    def __init__(self):
        super().__init__('my_node_name')
        self.get_logger().info('My ROS 2 node has started!')

def main(args=None):
    rclpy.init(args=args)
    node = MyNode()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

The `rclpy.spin(node)` function enters a loop, keeping the node alive to send and receive messages until the program is shut down.

## Topics: The Data Stream

Topics are for continuous, one-way data flow. Let's create a node that publishes the current time every second and another that subscribes to it.

**Publisher Node (`talker.py`):**
```python
# ... (imports and main function as above)
from std_msgs.msg import String

class Talker(Node):
    def __init__(self):
        super().__init__('talker')
        self.publisher_ = self.create_publisher(String, 'chatter', 10)
        self.timer = self.create_timer(1.0, self.timer_callback)
        self.get_logger().info('Talker node started and publishing.')

    def timer_callback(self):
        msg = String()
        msg.data = f'Hello from ROS 2 at {self.get_clock().now()}'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')
```

**Subscriber Node (`listener.py`):**
```python
# ... (imports and main function as above)
from std_msgs.msg import String

class Listener(Node):
    def __init__(self):
        super().__init__('listener')
        self.subscription = self.create_subscription(
            String,
            'chatter',
            self.listener_callback,
            10)
        self.get_logger().info('Listener node started and subscribing.')

    def listener_callback(self, msg):
        self.get_logger().info(f'I heard: "{msg.data}"')
```
When you run both nodes, the `talker` will publish messages on the `chatter` topic, and the `listener` will print them to the console.

## Services: The Remote Procedure Call

Services are for request/response interactions. Let's create a service that adds two integers.

**Service Server Node (`add_two_ints_server.py`):**
```python
# ... (imports and main function as above)
from example_interfaces.srv import AddTwoInts

class AddTwoIntsServer(Node):
    def __init__(self):
        super().__init__('add_two_ints_server')
        self.srv = self.create_service(AddTwoInts, 'add_two_ints', self.add_two_ints_callback)
        self.get_logger().info('Service server started.')

    def add_two_ints_callback(self, request, response):
        response.sum = request.a + request.b
        self.get_logger().info(f'Incoming request: a={request.a}, b={request.b}. Returning sum={response.sum}')
        return response
```

**Service Client Node (`add_two_ints_client.py`):**
```python
# ... (imports)
from example_interfaces.srv import AddTwoInts
import sys

# ... (client class and main function)
# (For brevity, the full client implementation is omitted, but it involves
# creating a client, waiting for the service, sending a request, and
# spinning until the future (the response) is complete.)
```
The client would send two numbers to the server, and the server would return the sum. This is a synchronous, blocking call.

This chapter has provided a practical, code-first look at the fundamental building blocks of a ROS 2 system. Mastering these concepts is the first major step toward building capable Physical AI.