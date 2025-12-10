--- 
id: 10-bipedal-locomotion
title: Chapter 10 — Bipedal Locomotion, Balance & Manipulation
---

## Introduction to Bipedal Locomotion

Bipedal locomotion, or walking on two legs, is a cornerstone of humanoid robotics. It offers unparalleled mobility in human-centric environments but presents significant challenges in stability and control. Unlike wheeled robots, which are statically stable, a bipedal robot is an inherently unstable system that must constantly adjust its posture to avoid falling. This chapter explores the fundamental principles of achieving stable walking, maintaining balance, and performing manipulation tasks.

## The Pillars of Stability

To understand how a humanoid walks, we must first grasp the core physics concepts that govern its stability.

### Center of Mass (COM)
The Center of Mass is the average location of the mass of the robot. In simple terms, it's the point where the robot would balance perfectly if you could support it from underneath. The projection of the COM onto the ground is a critical variable in balance control.

### Support Polygon & Zero Moment Point (ZMP)
The **Support Polygon** is the convex hull of all contact points between the robot's feet and the ground. For the robot to be stable, the vertical projection of its COM must remain within this area.

The **Zero Moment Point (ZMP)** is the point on the ground where the net moment of the inertial forces and the gravity forces has no horizontal component. For the robot to remain stable without tilting over, the ZMP must stay within the support polygon.

```text
      COM (Center of Mass)
        |
        |
        g (gravity)
        |
        V
  +-----------+
  |  Robot    |
  |   Body    |
  +-----------+
      /   \
     /     \
   Leg1   Leg2
    |       |
  Foot1   Foot2
<---- Support Polygon ---->
    ^
   ZMP (Zero Moment Point)
```
**Stability Margin:** The shortest distance from the ZMP to the edge of the support polygon. A larger margin means greater stability.

## Gait Cycles

A gait is a repeating pattern of leg movements. Different gaits are required for different speeds and terrains.

- **Walking:** A stable gait where at least one foot is always in contact with the ground. It involves a "single support phase" (one foot on the ground) and a "double support phase" (both feet on the ground).
- **Running:** A dynamic gait that includes a "flight phase" where both feet are off the ground. It is faster but less stable than walking.
- **Stair Climbing:** A hybrid gait that requires precise foot placement, vertical lift, and forward propulsion, carefully coordinated to ascend or descend steps.

## Balance Control Strategies

Maintaining balance is an active process. Controllers continuously adjust the robot's joints to keep the ZMP within the support polygon.

- **PID Control:** A simple and effective feedback controller that calculates an "error" value (e.g., the difference between the desired ZMP and the actual ZMP) and applies a correction based on Proportional, Integral, and Derivative terms.
  
  *Pseudo-code for a simple PID balance controller:*
  ```
  // Desired ZMP is the center of the support polygon
  desired_zmp = calculate_support_polygon_center();
  current_zmp = sense_current_zmp();
  
  error = desired_zmp - current_zmp;
  integral_error += error * dt;
  derivative_error = (error - previous_error) / dt;
  
  // Calculate ankle torque adjustment
  torque_adjustment = Kp * error + Ki * integral_error + Kd * derivative_error;
  
  // Apply torque to ankle motors
  robot.ankle.apply_torque(torque_adjustment);
  previous_error = error;
  ```

- **Model Predictive Control (MPC):** An advanced technique where the controller uses a dynamic model of the robot to predict its future state. It optimizes a sequence of control moves (e.g., foot placements, torso angles) over a short time horizon to maintain stability.

- **Whole-Body Control (WBC):** A holistic approach that treats the entire robot as a single optimization problem. It simultaneously solves for all joint torques required to achieve multiple objectives, such as tracking a COM trajectory, maintaining balance, and moving the arms, while respecting constraints like joint limits and friction.

## Footstep Planning

Where should the robot place its next foot? Footstep planners answer this question by generating a sequence of feasible foot poses that lead the robot to a goal location without colliding with obstacles or losing balance. Planners must consider:
- Reachability of the next step.
- Kinematic constraints of the legs.
- Obstacle avoidance.
- Maintaining the stability margin.

## Manipulation and Grasping

A truly useful humanoid must do more than just walk; it must interact with its environment.

### Manipulation While Walking
This is a highly complex task where the robot must simultaneously maintain walking stability while moving its arms to perform a task (e.g., opening a door, carrying an object). The movement of the arms shifts the robot's COM, and the locomotion controller must compensate for this disturbance in real-time.

### Hands & Grasping Fundamentals
The end-effector is the "hand" of the robot. Its design determines what the robot can interact with.

- **End-Effector Types:**
  - **Simple Grippers:** Two-fingered or parallel-jaw grippers.
  - **Multi-fingered Hands:** Complex, human-like hands that offer dexterity for complex grasps.
  - **Specialized Tools:** End-effectors designed for a single purpose, like a drill or a welding torch.

- **Grasp Taxonomies:** Grasps are classified based on the number of contact points and the shape of the object.
  - **Power Grasp:** The object is held securely against the palm, using the entire hand for maximum stability (e.g., holding a hammer).
  - **Precision Grasp:** The object is held between the fingertips, allowing for fine manipulation (e.g., picking up a key).

```text
     Power Grasp                   Precision Grasp
+---------------------+
|   Object firmly     |
|   enclosed by       |
|   fingers and palm. |
|                     |
|   (e.g., Hammer)    |
+---------------------+
        |
        |
        V
  +-----------+
  |  Robot    |
  |   Body    |
  +-----------+
      /   \
     /     \
   Leg1   Leg2
    |       |
  Foot1   Foot2
<---- Support Polygon ---->
    ^
   ZMP (Zero Moment Point)
```

By integrating these locomotion, balance, and manipulation principles, engineers can build humanoid robots capable of navigating complex environments and performing meaningful work.
