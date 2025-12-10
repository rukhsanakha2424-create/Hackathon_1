---
title: The Robotics Nervous System (ROS 2 Basics)
---

This chapter covers the basics of the Robot Operating System (ROS 2), the nervous system for many modern robots. If a robot has a physical body, ROS 2 provides the foundational software framework to manage its complex functions, much like a biological nervous system.

## What is ROS 2?

ROS 2 (Robot Operating System 2) is a set of open-source software libraries and tools for building robot applications. Despite its name, it's not a traditional operating system like Windows or Linux. Instead, it's "meta-operating system" that provides services you'd expect from an OS, including:

*   **Hardware Abstraction:** It lets you write code without worrying about the specific hardware you're using.
*   **Low-level Device Control:** It provides drivers and interfaces for common sensors and actuators.
*   **Inter-process Communication:** It allows different parts of your robot's software to communicate with each other.
*   **Package Management:** It provides tools for organizing, sharing, and reusing code.

ROS 2 is the successor to the widely used ROS 1, rebuilt from the ground up to support multi-robot systems, real-time control, and production-grade applications.

## Core Concepts of ROS 2

ROS 2 applications are typically structured as a network of independent programs called **Nodes**. These nodes communicate with each other using a few key mechanisms:

1.  **Topics (Publish/Subscribe):** This is the most common communication method. A node can "publish" messages to a specific topic (e.g., `/camera/image_raw`), and any other node can "subscribe" to that topic to receive those messages. This is a one-way, asynchronous communication model, ideal for continuous data streams like sensor readings.

2.  **Services (Request/Response):** Services are used for two-way, synchronous communication. A node can offer a "service" (e.g., `/calculate_trajectory`), and another node can send a "request" and wait for a "response". This is useful for commands that should be completed before the program continues, like asking for a specific calculation.

3.  **Actions (Long-running Goals):** Actions are for long-running, asynchronous tasks that provide feedback during execution. For example, a robot might have an "action" to navigate to a specific location. A client node can send a goal to the action server, receive continuous feedback on the robot's progress, and be notified when the goal is complete (or has failed).

By combining these simple but powerful concepts, ROS 2 allows developers to build complex, modular, and scalable robotics software.