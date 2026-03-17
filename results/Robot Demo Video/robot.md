
## Demonstration Video

A short demonstration video of the robot operation is provided below.This is just a prototype video that shows the robot currently performing the following tasks:

- Following a **black guiding line**
- **Picking and placing objects**
- Detecting and avoiding **obstacles**
- Automatically calculating the **shortest navigation path**
- Executing navigation decisions using a **RISC-V CPU**
- The performance of the robot can be improved based on the requirement and scalability 
<video src="robot.mp4" controls width="600"></video>

If the video does not load in the preview, you can download it directly from the repository:

[Download Video](robot.mp4)

---

## System Functionality

The robot demonstrates several key capabilities:

### 1. Line Following Navigation
The robot follows a predefined **black line track**, which represents an industrial path used for navigation inside factories or warehouses.

### 2. Object Pick and Place
The robot is capable of **picking and placing objects** at designated points along the path.

### 3. Obstacle Detection
Obstacles placed on the path are detected by sensors, and the robot adapts its route accordingly or remove the obstacle based on the requirement 

### 4. Shortest Path Computation
The system computes the **shortest path between nodes** using a graph-based shortest path algorithm executed on a **RISC-V CPU**.

### 5. Autonomous Decision Making
Based on sensor inputs and algorithmic output, the robot autonomously determines its next movement.

---

## Prototype Nature of the System

The current demonstration represents a **small-scale working prototype** intended to validate the architecture and operational concept.

The system can be further **improved and scaled** depending on application requirements, including:

- Larger industrial navigation environments
- Multi-robot coordination
- Integration with industrial IoT systems
- Advanced vision-based navigation
- More complex path planning algorithms

This prototype demonstrates the **feasibility of integrating RISC-V based processing with autonomous robotic navigation systems** for future industrial applications.

---

## Future Improvements

Possible extensions include:

- Integration with **vision-based object detection**
- Implementation of **AI-based navigation algorithms**
- Real-time **dynamic obstacle avoidance**
- Scalable **multi-node industrial logistics systems**

---

## License

This repository is provided for **research and academic demonstration purposes**.
