# System Architecture Overview

This architecture repository folder contains three key architecture illustrations supporting the proposed **FPGA Based Autonomous Cobot for Adaptive Industrial Assistance in Industry 5.0**:

1. **Industry Map Setup** – illustrates the industrial environment and robot navigation workflow.
2. **System Architectural Block Diagram** – shows the conceptual hardware architecture of the proposed system.
3. **Quartus Block Diagram** – presents the FPGA implementation of the system modules inside the hardware design environment.

---

# Industry 5.0 Operational Model and Industry Map Concept

The proposed system is built upon an **Industry 5.0 implementation model** that extends traditional Industry 4.0 automation by introducing a **Human–Cobot Center** responsible for collaborative decision making between humans and autonomous robots.

The **Industry Map Setup** represents a simplified model of an electronics manufacturing floor where the autonomous robot operates. The industrial environment is divided into multiple functional units connected through predefined navigation paths across the factory floor.

The industrial units are categorized into two main groups:

## Inspection Units of Industry

Inspection units represent the primary production and monitoring areas within the factory. In the demonstrated setup, the system includes four inspection units:

- **Prototyping Unit** – responsible for model development and prototype validation.
- **Fabrication Unit** – handles physical hardware manufacturing processes.
- **Design Unit** – performs design verification and digital validation tasks.
- **Waste Unit** – identifies and classifies defective or discarded components.

Each inspection unit contains several **subunits** that represent individual workstations. The robot periodically scans these units to determine their operational condition.

---

## Maintenance Units of Industry

When anomalies are detected within inspection units, corrective actions must be performed. For this purpose, the industrial layout includes dedicated **maintenance units**:

- **Service Unit** – responsible for repair, calibration, and maintenance tasks.
- **Replace Unit** – handles component replacement operations.
- **Warehouse Unit** – stores spare components and equipment required for replacement tasks.

These maintenance units ensure that detected faults are resolved efficiently while maintaining continuous industrial operation.

---

## Human–Cobot Center

The **Human–Cobot Center** acts as the central control and decision hub of the system. It receives operational data transmitted by the robot and allows human operators to analyze industrial conditions.

Based on this analysis, the human operator determines the appropriate corrective action, such as sending the robot to the service unit, replacing components, or delivering equipment from the warehouse.

This architecture establishes a **human-in-the-loop industrial automation model**, combining human intelligence with robotic precision to achieve flexible and adaptive industrial operation.

---

# Working Mechanism of the Autonomous Cobot

The robot operates across the industrial floor through a sequence of coordinated stages.

## 1. Scanning Phase

The robot navigates through predefined industrial paths and sequentially scans each subunit using onboard sensors. A **color-based identification system** is used to represent different operational conditions of industrial units.

Example color codes include:

| Color | Meaning | Robot Action |
|------|--------|-------------|
| Red | Defective component detected | Move item to Service Unit |
| Blue | Replacement required | Retrieve component from Warehouse |
| Green | Unit operating normally | No action required |
| Yellow | New equipment request | Deliver equipment to target unit |

These codes can be modified for different industrial environments, making the framework adaptable to multiple domains.

---

## 2. Data Processing

Sensor data collected during scanning are processed in real time by the robot’s **single-cycle RISC-V processor implemented on FPGA**. The processor converts sensor signals into digital status codes that are transmitted to the Human–Cobot Center.

---

## 3. Human–Cobot Interaction

Operators at the Human–Cobot Center interpret the received information and determine the appropriate industrial response. Instructions are then transmitted back to the robot for execution.

---

## 4. Autonomous Execution

After receiving instructions, the robot computes the **shortest navigation path** to the required industrial subunit using a graph-based navigation algorithm. The robot then travels to the location and performs the assigned operation using its robotic actuator.

---

## 5. Emergency Aiding Phase

If a high-priority emergency request arises during scanning or task execution, the robot immediately prioritizes the emergency task. The current task state and navigation path are temporarily stored in memory. After resolving the emergency, the robot resumes its previous task without disrupting the industrial workflow.

This mechanism ensures **continuous industrial operation and adaptive robotic assistance**.

---

# FPGA-Based System Architecture

The **System Architectural Block Diagram** illustrates the complete FPGA-based robotic architecture implemented as a hardware-oriented System-on-Chip (SoC).

The architecture integrates sensing, computation, communication, and actuation within a unified FPGA hardware platform to achieve deterministic real-time performance.

The system is divided into four primary subsystems.

---

## 1. Input and Preprocessing Unit

This unit captures real-time environmental data from multiple sensors and converts them into digital signals suitable for processing.

Key components include:

- Line-following sensor arrays for navigation
- Infrared sensors for obstacle detection
- Color sensors for anomaly identification
- ADC controller for sensor signal conversion
- UART communication interface

These modules provide environmental information required for navigation and inspection.

---

## 2. Processing Unit

The core processing module is a **single-cycle RISC-V processor implemented on FPGA**.

This unit performs:

- sensor data interpretation
- navigation computation
- industrial status analysis
- control decision generation

A **Breadth-First Search (BFS) based shortest path algorithm** is implemented to determine optimal navigation routes across the industrial map.

---

## 3. Post-Processing and Motion Control Unit

The post-processing subsystem converts computational outputs into physical robot movement.

This module performs:

- direction control (left, right, forward)
- PWM-based motor speed control
- motion coordination and stabilization

It ensures precise navigation along the industrial pathways.

---

## 4. Object Handling and Communication Unit

This subsystem enables interaction with the industrial environment and communication with the Human–Cobot Center.

Key functions include:

- robotic manipulation using a **2-DOF robotic actuator**
- pick-and-place operations
- obstacle removal
- status communication

These capabilities allow the robot to perform maintenance and logistics tasks within the industrial setup.

---

# FPGA Implementation

The **Quartus Block Diagram** shows the internal hardware implementation of the system inside the FPGA design environment.

It illustrates the interconnection of the RISC-V processor, sensor interface modules, navigation logic, motor controllers, and communication blocks. The design emphasizes hardware parallelism and deterministic timing, enabling microsecond-level responsiveness while efficiently utilizing FPGA resources.

---

# Summary

The proposed system demonstrates a scalable **Industry 5.0 human–robot collaborative architecture** where autonomous robots assist industrial operations under human supervision.

By integrating sensing, decision-making, navigation, and actuation within an FPGA-based robotic SoC, the system achieves:

- deterministic real-time control
- autonomous industrial navigation
- human–robot collaborative decision making
- efficient hardware resource utilization

The three provided architecture images collectively illustrate the **industrial environment, conceptual system architecture, and FPGA implementation**, providing a comprehensive understanding of the proposed robotic framework.
