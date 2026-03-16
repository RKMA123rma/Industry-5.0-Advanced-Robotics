# System Validation and Performance

## Table of Contents

1. [Robot Operation and Task Execution](#1-robot-operation-and-task-execution)
2. [Functional Analysis of Core Modules](#2-functional-analysis-of-core-modules)
3. [FPGA Resource Utilization and Board Selection](#3-fpga-resource-utilization-and-board-selection)
4. [Timing Closure and Performance](#4-timing-closure-and-performance)
5. [Latency Comparison with Conventional Robotic Architectures](#5-latency-comparison-with-conventional-robotic-architectures)
6. [Future Work and Considerations](#6-future-work-and-considerations)

---

# 1. Robot Operation and Task Execution

The FPGA-based **RISC-V robotic platform** was implemented on the **DE0-Nano (Cyclone IV E)** FPGA board and experimentally validated for autonomous operation in an Industry 5.0 environment. The system demonstrated real-time navigation, sensing, and manipulation capabilities through coordinated hardware–software execution.

The robot successfully performed the following tasks:

- **Color-coded scanning**
  - Red → defective component  
  - Blue → replacement required  
  - Green → operational component  
  - Yellow → transport item  

- **Pick-and-place operation** using a **2-DOF actuator arm**

- **Obstacle avoidance** triggered by the IR sensor with **BFS-based path re-routing**

- **Obstacle handling** through actuator-based removal

- **Human–cobot collaboration** via low-latency command communication

These tasks validated the integration of sensing, processing, and actuation modules within the FPGA-based SoC.

![Robot executing navigation and pick-and-place tasks](robot.png)

*Figure: Mobile robot executing navigation and pick-and-place operations.*

---

# 2. Functional Analysis of Core Modules

The performance of the key hardware modules was evaluated during real-time operation.

**UART Transceiver**

Maintained stable bidirectional communication with external systems. The baud rate was derived from the **3.125 MHz system clock**, enabling reliable command and status transmission.

**Color Sensor Controller**

Converted RGB frequency measurements into digital color categories, allowing the CPU to prioritize scanning and object-handling tasks.

**IR Obstacle Detection Module**

Detected obstacles along the robot path and triggered immediate **BFS-based route recomputation**.

**Clock Domain Crossing (CDC)**

Enabled safe synchronization between:

| Domain | Frequency |
|------|------|
| FPGA logic | 3.125 MHz |
| Servo control | 50 Hz |

Handshake-based synchronization ensured **metastability-free operation**.

**Servo Controller**

Generated PWM signals to control the **2-DOF actuator**, enabling precise lifting, transport, and placement of objects.

**Reset Logic and Sampling Circuits**

Ensured reliable system initialization and stable timing behavior across modules.

Overall, the system demonstrated **cohesive integration of sensing, computation, communication, and actuation** for collaborative industrial robotics.

---

# 3. FPGA Resource Utilization and Board Selection

The system was implemented on the **Cyclone IV E FPGA (EP4CE22F17C6)**, selected for its balanced logic resources and I/O availability.

### Resource Utilization

| Parameter | Value |
|------|------|
| FPGA Family | Cyclone IV E |
| Device | EP4CE22F17C6 |
| Timing Models | Final |
| Logic Elements Used | 16,403 / 22,320 |
| Registers | 4,307 |
| I/O Pins Used | 34 / 154 (22%) |

Most **on-chip RAM blocks, multipliers, and PLL resources remained unused**, indicating that the architecture supports future expansion.

System integration was completed using:

- **Block Design File (BDF)**
- **Pin assignments**
- **SOF / JIC configuration file generation**

---

# 4. Timing Closure and Performance

Timing analysis performed using **Quartus Prime 20.1.1** confirmed successful timing closure across the entire system.

Two primary clock domains were validated:

| Clock Domain | Frequency |
|------|------|
| CPU and communication logic | 3.125 MHz |
| Servo actuator control | 50 Hz |

Initial slow-corner timing violations were resolved through **Clock Domain Crossing synchronization techniques**.

Final timing analysis confirmed that all:

- setup time constraints
- hold time constraints
- recovery constraints
- pulse-width requirements

were satisfied.

This enabled stable real-time operation during **navigation, obstacle handling, and pick-and-place tasks**.

---

# 5. Latency Comparison with Conventional Robotic Architectures

Real-time responsiveness is critical in industrial robotic environments, particularly for **human–robot collaboration and dynamic task execution**.

Traditional robotic systems commonly rely on:

- **microcontroller-based control**
- **CPU-based frameworks such as Robot Operating System (ROS)**

Although these approaches provide flexibility, they often introduce **latency and non-deterministic timing** due to sequential execution and operating system scheduling.

Studies analyzing ROS communication pipelines report **millisecond-level delays** caused by middleware processing and OS scheduling overhead.

In contrast, **FPGA-based robotic systems exploit hardware parallelism**, allowing sensing, computation, and actuation modules to operate concurrently with deterministic timing.

The proposed system integrates a **single-cycle RISC-V CPU with dedicated FPGA modules**, enabling rapid decision-making during navigation.

The **BFS-based path planning algorithm** requires only tens of instruction cycles for node exploration and path computation. Combined with controller processing and actuator interfacing, this results in control decision latencies in the **tens of microseconds range**.

### Latency Comparison

| Architecture | Processing Model | Typical Latency | Determinism |
|------|------|------|------|
| Microcontroller Robot | Sequential CPU | 1–10 ms | Medium |
| ROS-based Robot | Middleware + OS scheduling | >1 ms to tens of ms | Low |
| FPGA Accelerated Robot | Hardware parallelism | Few µs | High |
| **Proposed FPGA RISC-V SoC** | Parallel FPGA + CPU | ~10–30 µs | High |

This demonstrates the advantage of FPGA-based robotic architectures for **low-latency industrial control systems**.

---

# 6. Future Work and Considerations

The presented FPGA-based RISC-V robotic platform provides a strong foundation for autonomous industrial robotics, but several enhancements can further extend its capabilities.

Future work may explore **advanced RISC-V microarchitectures**, such as pipelined, superscalar, or vector processors, to support computationally intensive tasks including sensor fusion and dynamic motion optimization.

Expanding communication interfaces such as **SPI, I²C, and CAN** would enable integration of additional sensors and distributed robotic systems. The Clock Domain Crossing framework could also be extended to support asynchronous peripherals such as **LiDAR and vision sensors**, improving system perception capabilities.

Another promising direction is integrating the FPGA SoC with **edge or cloud-based intelligence platforms**, enabling predictive analytics, collaborative decision-making, and adaptive task scheduling.

In addition, incorporating **lightweight neural accelerators** or configurable hardware accelerators could enhance on-device intelligence while preserving deterministic FPGA performance.

Energy-efficient design techniques such as:

- dynamic frequency scaling  
- power gating  
- hardware resource sharing  

could further improve system efficiency for long-duration industrial operation.

Overall, the architecture serves as a flexible experimental platform for future research in **autonomous robotics, Industry 5.0 collaboration, and reconfigurable intelligent systems**.

---
