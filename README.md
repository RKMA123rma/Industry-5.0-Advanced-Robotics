# Industry-5.0-Advanced-Robotics
Supporting materials for the research work titled "FPGA Based Autonomous Cobot for Adaptive Industrial Assistance in Industry 5.0." This repository includes system architecture diagrams and supplementary materials related to the robotic framework submitted to Advanced Robotics.
# FPGA Based Autonomous Cobot for Adaptive Industrial Assistance in Industry 5.0



# Overview

This repository contains supplementary materials supporting the research work titled:

**“FPGA Based Autonomous Cobot for Adaptive Industrial Assistance in Industry 5.0.”**

The project proposes an **FPGA-based robotic System-on-Chip (SoC)** designed for adaptive industrial assistance within an **Industry 5.0 human–robot collaborative environment**.

The system integrates sensing, computation, communication, and actuation within a deterministic FPGA hardware architecture to enable real-time industrial robotic operations.

Unlike conventional robotic systems relying on microcontrollers or software middleware, the proposed architecture executes critical robotic operations directly on FPGA hardware to achieve **deterministic timing, reduced latency, and improved reliability**.

This repository provides supporting artifacts including system architecture diagrams, hardware design components, proof-of-concept implementation materials, and supplementary documentation related to the proposed robotic platform.

---

# Industry 5.0 System Concept

The proposed system follows a **Human–Cobot collaborative framework** designed for Industry 5.0 environments.

In this framework:

* The **autonomous robot performs inspection and monitoring tasks** across industrial units.
* Sensor data collected by the robot is processed by the onboard FPGA-based processing system.
* Information is transmitted to a **Human Cobot Center**, where operators analyze the system status.
* Based on the analysis, the human operator can issue commands to the robot for corrective or support actions.

This approach combines **human decision-making with robotic autonomy**, enabling flexible and adaptive industrial automation.

---

# System Architecture

The robotic platform is implemented as a **hardware-oriented FPGA SoC architecture** consisting of four major subsystems.

---

## 1. Input and Preprocessing Unit

Responsible for capturing and conditioning real-time sensor signals.

Components include:

* Line following sensor array
* Infrared sensors for obstacle detection
* Color sensors for anomaly classification
* UART communication interface
* ADC controller for sensor signal conversion

---

## 2. Processing Unit

The core processing unit is a **single-cycle RISC-V processor implemented on FPGA**.

Key functions:

* Sensor data processing
* Navigation computation
* Decision making
* Execution of control algorithms

A **Breadth-First Search (BFS) algorithm** is implemented for optimal navigation between industrial nodes.

---

## 3. Post-Processing Unit

This unit translates computational decisions into robot movement.

Responsibilities include:

* Navigation control
* Direction generation (left, right, straight)
* PWM-based motor speed control
* Motion coordination

---

## 4. Object Handling and Communication Unit

This module handles robotic manipulation and communication with the human control center.

Functions include:

* Pick-and-place operations
* Obstacle removal
* Task communication
* Status reporting

A **2-DOF robotic actuator** performs industrial manipulation tasks such as object transport or obstruction removal.

---

# Robot Operation Workflow

The robot operates through the following stages:

### 1. Industrial Unit Scanning
The robot scans industrial units using onboard sensors and identifies operational states through color-based anomaly detection.

### 2. Data Processing
Sensor data is processed by the FPGA-based RISC-V processor to determine system status.

### 3. Human–Robot Interaction
Detected anomalies are transmitted to the Human Cobot Center for analysis and decision making.

### 4. Autonomous Navigation
The robot computes the shortest path to the target location using graph-based navigation algorithms.

### 5. Task Execution
The robot performs the assigned operation such as inspection assistance, object movement, or obstacle removal.

### 6. Emergency Handling
High-priority tasks can interrupt current operations, allowing immediate response before resuming the previous task.

---

# Prototype and Experimental Validation

A working prototype of the proposed system was implemented on an FPGA-based robotic platform.

Experimental demonstrations validated the following capabilities:

* autonomous industrial navigation
* real-time anomaly detection
* human–robot collaborative task execution
* pick-and-place operations
* obstacle detection and removal

These experiments demonstrate the feasibility of **hardware-accelerated robotic architectures for Industry 5.0 environments**.

---

# Repository Structure

```
architecture/
    architecture.md
    Industry 5.0 framework model image(industryplan.png) - Contains the Industry 5.0  Plan Model for electronics manufacturing Industey Setup
    Architectural Block Diagram image(BlockDiagram.png) - Contains the technical SoC architecure required for the Industry 5.0 Setup
    Implemented Block Diagram File(coborarchbdf.jpg) Image - Contains the entire block diagram image implemented in Intel Quartus Prime

hardware/
    Hardware design components
    FPGA module descriptions

proof_of_concept/
    Robot prototype images
    Experimental setup
    Demonstration materials

results/
    Supplementary experimental materials
```

---

# Key Contributions

The major contributions of this work include:

* FPGA-based robotic **System-on-Chip architecture**
* Integration of a **custom RISC-V processing core**
* Graph-based **autonomous navigation algorithm**
* Human–robot collaborative **Industry 5.0 framework**
* Deterministic real-time robotic control using FPGA hardware
* Modular architecture adaptable to multiple industrial applications

---


# Authors

Harish V Mekali  
Girish K  
Rohith Kashyap MA  
Rohit G Gadag  
Selva Kumar M  

Department of Electronics and Communication Engineering  
BMS College of Engineering, Bengaluru, India  

---

# License

This repository is provided for **academic and research purposes only**.  
All materials remain the intellectual property of the authors unless otherwise stated.

---

# Contact

For questions or collaboration inquiries:

Girish K  
Email: gk.mca@bmsce.ac.in  

Rohith Kashyap MA  
Email: rohithma.ec22@bmsce.ac.in
