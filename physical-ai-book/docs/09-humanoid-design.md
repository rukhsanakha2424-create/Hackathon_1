---
id: 09-humanoid-design
title: "Chapter 9 — Humanoid Robot Design: Kinematics & Dynamics"
---

## Fundamentals of Humanoid Design

Designing a humanoid robot involves a deep understanding of mechanics, electronics, and control theory. This chapter focuses on the mechanical principles of kinematics and dynamics, which govern the robot's movement and stability.

## Joints and Degrees of Freedom (DOF)

A humanoid robot is a kinematic chain of links (bones) connected by joints. The number and arrangement of these joints determine its mobility.

- **Degree of Freedom (DOF)**: Each independent direction a joint can move. A humanoid robot's total DOF is the sum of the DOF of all its joints.
- **Common Joint Types**:
  - **Revolute Joint (1-DOF)**: A rotating joint, like an elbow or knee.
  - **Prismatic Joint (1-DOF)**: A sliding joint (less common in humanoids).
  - **Universal Joint (2-DOF)**: Two revolute joints combined, like an ankle.
  - **Spherical Joint (3-DOF)**: A ball-and-socket joint, like a hip or shoulder.

A typical humanoid has 20-30 DOF:
- **Legs (6-DOF each)**: 3-DOF hip, 1-DOF knee, 2-DOF ankle.
- **Arms (7-DOF each)**: 3-DOF shoulder, 1-DOF elbow, 3-DOF wrist.
- **Head/Neck (2-3 DOF)**

## Kinematics: The Geometry of Motion

Kinematics describes motion without considering the forces that cause it.

### Forward Kinematics (FK)

Forward Kinematics calculates the position and orientation of the end-effector (e.g., hand or foot) given the angles of all the joints in the kinematic chain. This is relatively straightforward.

**Example**: For a simple 2-link arm in 2D:
- Link 1 has length `L1` and angle `θ1`.
- Link 2 has length `L2` and angle `θ2`.

The position of the end-effector (x, y) is:
```
x = L1 * cos(θ1) + L2 * cos(θ1 + θ2)
y = L1 * sin(θ1) + L2 * sin(θ1 + θ2)
```
This is a simple equation. For a full humanoid, this is usually solved using matrix transformations (Denavit-Hartenberg parameters).

### Inverse Kinematics (IK)

Inverse Kinematics is the reverse problem: given a desired position and orientation for the end-effector, what are the required joint angles? This is much harder because:
- Multiple solutions may exist.
- A solution may not exist (the target is out of reach).
- The calculations are computationally intensive.

IK is crucial for tasks like reaching for an object or placing a foot on a specific spot. It's often solved using numerical optimization methods like the Jacobian inverse method.

```text
       Goal Position          Joint Angles
FK:  [ θ1, θ2, ... ]  --->  [ x, y, z, ... ]  (Easy)

IK:  [ x, y, z, ... ]  --->  [ θ1, θ2, ... ]  (Hard)
```

## Dynamics: The Physics of Motion

Dynamics adds the concept of forces, mass, and inertia. It describes the relationship between the forces acting on the robot and the resulting motion.

- **Forward Dynamics**: Given the joint torques (from motors), calculate the resulting acceleration and motion of the robot. This is used for simulation.
- **Inverse Dynamics**: Given a desired trajectory (position, velocity, acceleration) for the robot's limbs, calculate the joint torques required to achieve that motion. This is essential for control.

**Simple Equation of Motion**:
The general equation for a robot's dynamics is:
`Τ = M(q) * q̈ + C(q, q̇) + G(q)`

Where:
- `Τ` (Tau) is the vector of joint torques.
- `q`, `q̇`, `q̈` are the joint positions, velocities, and accelerations.
- `M(q)` is the mass matrix (inertia of the system).
- `C(q, q̇)` represents Coriolis and centrifugal forces.
- `G(q)` is the gravity vector.

The controller's job is to calculate the necessary torques `Τ` to produce the desired motion `q̈`.

## Stability: Not Falling Over

For a bipedal robot, maintaining balance is the primary challenge. Two key concepts are the Center of Mass (COM) and the Zero Moment Point (ZMP).

- **Center of Mass (COM)**: The single point where the entire mass of the robot can be considered to be concentrated. To be stable, the projection of the COM onto the ground must stay within the support polygon.
- **Support Polygon**: The area on the ground formed by the robot's feet.
- **Zero Moment Point (ZMP)**: The point on the ground where the net moment from inertia and gravity is zero. If the ZMP stays within the support polygon, the robot will not tip over.

**Control Strategy**: The robot's controller continuously adjusts the joints to keep the ZMP inside the support polygon. When walking, the controller moves the COM to shift the ZMP from one foot to the other.

```text
     O (COM)
     |
     |
  +--|--+  (Stable)
  |  v  |
 [ Foot ] (ZMP is inside)


     O (COM)
      \
       \
    +---\\--+ (Unstable)
    |    \ |
   [ Foot ] v (ZMP is outside)
```

## Actuators and Sensors

- **Actuators**: The "muscles" of the robot. In humanoids, these are typically high-torque electric motors with gearboxes (e.g., servo motors, brushless DC motors). Series Elastic Actuators (SEAs) include a spring, which provides compliance and can absorb shocks, making them well-suited for walking.
- **Sensors**:
  - **Proprioceptive Sensors**: Measure the robot's internal state.
    - **Encoders**: Measure joint angles.
    - **Inertial Measurement Unit (IMU)**: Measures orientation and angular velocity (like the inner ear).
    - **Force/Torque Sensors**: Measure forces at the joints or feet.
  - **Exteroceptive Sensors**: Measure the external environment (Cameras, LiDAR, etc.).

## Mechanical Design Considerations

- **Strength-to-Weight Ratio**: The structure must be strong enough to support the robot's weight and dynamic loads, but light enough for the motors to move it efficiently. Carbon fiber and aluminum alloys are common materials.
- **Actuator Placement**: Placing heavy motors closer to the robot's core (hips, torso) reduces the inertia of the limbs, allowing them to move faster.
- **Wiring and Routing**: A humanoid has hundreds of wires. Managing them to prevent pinching, stretching, and electromagnetic interference is a major design challenge.