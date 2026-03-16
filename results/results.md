# Results

## Table of Contents

1. [FPGA Resource Utilization and Board Selection](#1-fpga-resource-utilization-and-board-selection)
2. [Timing Closure and Performance](#2-timing-closure-and-performance)
3. [Latency Comparison with Conventional Robotic Architectures](#3-latency-comparison-with-conventional-robotic-architectures)
4. [Future Work and Considerations](#4-future-work-and-considerations)

---

# 1. FPGA Resource Utilization and Board Selection

The proposed FPGA-based robotic SoC was implemented on the **Cyclone IV E FPGA (EP4CE22F17C6)** available on the **DE0-Nano development board**. This device was selected due to its balanced combination of logic elements, I/O resources, and on-chip memory suitable for embedded robotic control systems.

### FPGA Resource Summary

| Parameter | Value |
|------|------|
| FPGA Family | Cyclone IV E |
| Device | EP4CE22F17C6 |
| Timing Models | Final |
| Total Logic Elements | 16,403 / 22,320 |
| Total Registers | 4,307 |
| Total I/O Pins Used | 34 / 154 (22%) |

Approximately **73% of the logic elements** were utilized for implementing the RISC-V CPU, sensor controllers, actuator controllers, communication modules, and navigation logic.

Most **on-chip RAM blocks, multipliers, and PLL resources remained unused**, leaving significant headroom for system scalability and future module integration.

System integration was performed using:

- **Block Design File (BDF)**
- **Pin assignment configuration**
- **SOF / JIC configuration file generation**

---

# 2. Timing Closure and Performance

Timing verification was performed using **Quartus Prime 20.1.1 timing analysis tools**.

The system operates across two primary clock domains:

| Clock Domain | Frequency |
|------|------|
| CPU and logic modules | 3.125 MHz |
| Servo actuator control | 50 Hz |

Initial timing analysis identified minor violations under slow-corner conditions. These were resolved by implementing **Clock Domain Crossing (CDC) synchronization mechanisms** between the logic and actuator domains.

After optimization, all timing constraints were successfully satisfied, including:

- Setup time constraints  
- Hold time constraints  
- Recovery and removal timing  
- Pulse width constraints  

This confirms stable operation of the system during **real-time navigation, sensing, and actuator control**.

---

# 3. Latency Comparison with Conventional Robotic Architectures

Industrial robotic systems require **fast and deterministic control decisions**, particularly in environments involving human–robot collaboration.

Traditional robotic platforms typically rely on:

- **Microcontroller-based control systems**
- **CPU-based middleware frameworks such as Robot Operating System (ROS)**

While these platforms provide software flexibility, they often introduce **latency and nondeterministic execution** due to sequential processing, operating system scheduling, and communication overhead.

In contrast, FPGA-based robotic systems leverage **hardware-level parallelism**, allowing sensing, computation, and actuation modules to operate concurrently without OS intervention.

The proposed system integrates a **single-cycle RISC-V processor with dedicated FPGA hardware modules**, enabling rapid decision making.

The **BFS-based navigation algorithm** requires only tens of instruction cycles for node expansion and path selection, resulting in control decision latencies in the **microsecond range**.

### Latency Comparison

| Architecture | Processing Model | Typical Latency | Determinism |
|------|------|------|------|
| Microcontroller-based Robot | Sequential CPU execution | 1–10 ms | Medium |
| ROS-based Robot | Middleware + OS scheduling | >1 ms to tens of ms | Low |
| FPGA Accelerated Robotics | Hardware parallelism | Few µs | High |
| **Proposed FPGA RISC-V SoC** | Parallel FPGA + CPU | ~10–30 µs | High |

This demonstrates that the proposed FPGA-based architecture provides **significantly lower latency and more deterministic performance** compared to conventional robotic control systems.

---

# 4. Future Work and Considerations

The presented FPGA-based robotic SoC provides a flexible foundation for further research and system enhancements.

Future work may include:

- Integration of **advanced RISC-V microarchitectures** such as pipelined or vector processors to support higher computational workloads.
- Expansion of communication interfaces including **SPI, I²C, and CAN** to support additional sensors and distributed robotic systems.
- Integration of **vision sensors or LiDAR modules** using extended Clock Domain Crossing frameworks.
- Incorporation of **lightweight neural accelerators** for on-device perception and intelligent decision making.
- Exploration of **energy-efficient techniques** such as dynamic frequency scaling, power gating, and resource sharing to support long-duration industrial operation.

These improvements would enhance the system’s capability for **advanced autonomous robotics and Industry 5.0 human–robot collaboration**.
