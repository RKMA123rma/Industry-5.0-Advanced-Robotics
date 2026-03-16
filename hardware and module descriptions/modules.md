# FPGA Module Reference — Industrial Cobot Navigation System

> **Platform:** RV32I Single-Cycle RISC-V · **Clock:** 3.125 MHz · **HDL:** Verilog  
> This document describes every hardware module in the system, grouped by functional unit.

---

## Table of Contents

1. [Input Preprocessing Unit](#1-input-preprocessing-unit)
   - [1.1 UART Receiver](#11-uart-receiver)
   - [1.2 Message Decoder & Task Identifier](#12-message-decoder--task-identifier)
   - [1.3 Aiding Tasks & Color-Based Variants](#13-aiding-tasks--color-based-variants)
   - [1.4 Emergency Task Handler](#14-emergency-task-handler)
   - [1.5 CPU Input Arbitrator](#15-cpu-input-arbitrator)
   - [1.6 Synchronization & Reset Controller](#16-synchronization--reset-controller)
   -  [Color Sensor Controller](#17-color-sensor-controller)
   -  [ADC Controller](#18-adc-controller)
2. [Processing Unit](#2-processing-unit)
   - [4.1 RISC-V CPU Core (RV32I)](#41-risc-v-cpu-core-rv32i)
   - [4.2 Shortest Path Loader (BFS)](#42-shortest-path-loader-bfs)
   - [4.3 Reset Module](#43-reset-module)
3. [Post-Processing Unit](#5-post-processing-unit)
   - [5.1 CPU-Path & Turns Arbitrator](#51-cpu-path--turns-arbitrator)
   - [5.2 Turn & Node Controller](#52-turn--node-controller)
   - [5.3 Industry Turns Graph](#53-industry-turns-graph)
   - [5.4 LFA Controller](#54-lfa-controller)
   - [5.5 PWM Generators & Motor Drivers](#55-pwm-generators--motor-drivers)
4. [Object Handling & Communication Unit](#6-object-handling--communication-unit)
   - [6.1 Object Handling](#61-object-handling)
   - [6.2 Obstacle Handling](#62-obstacle-handling)
   - [6.3 Communication Handling & UART Transmission](#63-communication-handling--uart-transmission)
   - [6.4 TX Controller for Message Sequencing](#64-tx-controller-for-message-sequencing)
   - [6.5 Clock Domain Crossing (CDC) Handler](#65-clock-domain-crossing-cdc-handler)

---

## 1. Input Preprocessing Unit

The Input Preprocessing Unit receives serial commands from the Human–Cobot Center, decodes the predefined communication syntax, identifies the task type, and writes the extracted parameters to the CPU's data memory via the Input Arbitrator.

---

### 1.1 UART Receiver

**Module ID:** `uart_rx`

Deserializes incoming serial data at **11,520 bps**. Implements a 4-state FSM — **Idle → Start → Data → Stop** — with an oversampling counter for noise-robust bit detection.

**Operation:**  
When the line goes low (`rx == 0`), the FSM exits Idle and enters the Start state. An internal counter runs to sample each bit at its midpoint. Once all 8 data bits are received, an optional parity check is performed and the complete byte is output with an `rx_done` flag. An internal reset clears all counters after every complete transaction.

**FSM States:**

| State | Condition to Enter | Action |
|---|---|---|
| Idle | Default | Wait for `rx == 0` |
| Start | `rx == 0` | Start oversampling counter |
| Data | `counter > 0` | Shift in bits |
| Stop / Parity | `counter == 0` | Calculate parity, assert `rx_done` |

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `rx` | Input | Serial data line |
| `clk` | Input | System clock (3.125 MHz) |
| `rst` | Input | Synchronous reset |
| `data_out[7:0]` | Output | Received 8-bit byte |
| `rx_done` | Output | Pulses high when byte is complete |

---

### 1.2 Message Decoder & Task Identifier

**Module ID:** `msg_decoder`

After `rx_done` is asserted, the 8-bit message is passed to this module, which interprets the predefined communication syntax — extracting **task type**, **target unit**, **subunit number**, and **navigation parameters**.

**Command Syntax:**

```
SCT-PU1-UID01-SN03-EN07#   → Start Scanning, Prototyping Unit 1, UID 01, Node 3 to 7
EMG-PU2-UID15-SN04-EN06#   → Emergency at Prototyping Unit 2, UID 15, Node 4 to 6
```

**Supported Task Codes:**

| Code | Task |
|---|---|
| `SCT` | Scanning Task |
| `FU` | Fabrication Unit Task |
| `WU` | Waste Unit Task |
| `SU` | Storage Unit Task |
| `EMG` | Emergency Task (highest priority) |

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `data_byte[7:0]` | Input | Byte from UART Receiver |
| `rx_done` | Input | Strobe from UART Receiver |
| `clk` | Input | System clock |
| `task_type[2:0]` | Output | Decoded task identifier |
| `uid[7:0]` | Output | Unit ID |
| `sn[4:0]` | Output | Start node |
| `en[4:0]` | Output | End node |
| `valid_flag` | Output | Asserted when a full command is decoded |

---

### 1.3 Aiding Tasks & Color-Based Variants

**Module ID:** `aiding_task_handler`

When a color is detected by the Color Sensor Controller, the Human–Cobot Center sends a corresponding **aiding message** defining the robot's corrective action for that color condition. This module receives and routes those aiding commands.

**Operation:**  
The module listens for incoming UART packets that carry color-triggered aiding instructions. On receipt of a valid aiding command, the task type and target parameters are forwarded to the CPU Input Arbitrator. The aiding task is treated as a mid-priority request — below Emergency but above standard Scanning tasks.

---

### 1.4 Emergency Task Handler

**Module ID:** `emg_handler`

Detects `EMG`-prefixed commands and immediately elevates them to **highest priority**, preempting any ongoing navigation task. Activates the **red LED** signaling array to distinguish emergencies from normal operations.

**Operation:**  
On detection of the `EMG` header from the Message Decoder, this module asserts an interrupt signal to the CPU Input Arbitrator. The CPU suspends its current task, and the emergency endpoint pair (SN/EN) is forwarded for immediate BFS path computation.

**Emergency Syntax:**
```
EMG-<UnitID>-<SN>-<EN>#
Example: EMG-PU2-UID15-SN04-EN06#
```

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `task_type[2:0]` | Input | From Message Decoder |
| `valid_flag` | Input | Command valid strobe |
| `emg_irq` | Output | Interrupt to CPU Arbitrator |
| `led_red` | Output | Red LED array activation |

---

### 1.5 CPU Input Arbitrator

**Module ID:** `cpu_input_arb`

A **hierarchical FSM** that validates and arbitrates between Scanning, Aiding, and Emergency task requests. The extracted parameters (UID, SN, EN) are decoded and the corresponding start and end points of the route are written to the CPU's data memory externally.

**Priority Scheme:**

```
Emergency  >  Aiding  >  Scanning
```

**Operation:**  
Validates incoming decoded parameters against the 3-tier priority hierarchy. The winning task's (UID, SN, EN) triplet is written to fixed, memory-mapped addresses in the CPU's data memory to ensure deterministic BFS access. Ongoing lower-priority tasks are suspended when a higher-priority request arrives.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `uid[7:0]`, `sn[4:0]`, `en[4:0]` | Input | Decoded task parameters |
| `emg_irq` | Input | Emergency interrupt |
| `task_type[2:0]` | Input | Task priority class |
| `clk`, `rst` | Input | Clock and reset |
| `mem_wr_en` | Output | Data memory write enable |
| `mem_addr[5:0]` | Output | Target memory address |
| `mem_data[31:0]` | Output | Data to write (SN/EN values) |

---

### 1.6 Synchronization & Reset Controller

**Module ID:** `sync_rst_ctrl`

Clears all transaction flags and counters after every complete message cycle. Prevents false triggers and maintains reliable message handling in **continuous multi-task environments**.

**Operation:**  
After both `rx_done` and processing completion are asserted, this module generates a synchronous internal reset pulse. The pulse clears UART FSM states, decoder buffers, and arbitrator flags — providing a clean slate before the next incoming command.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `rx_done` | Input | UART receive complete |
| `proc_done` | Input | Arbitrator write complete |
| `clk` | Input | System clock |
| `global_rst_pulse` | Output | Reset strobe to all submodules |

---

## 1.7 Color Sensor Controller

**Module ID:** `color_sensor_ctrl`

Identifies color-coded object states within the industrial workspace. Each detected color corresponds to a specific robot action. The **TCS3200** color sensor is used due to its direct frequency-based output, which simplifies FPGA interfacing.

**Operation:**  
The controller cycles through three FSM states — **F_RED → F_GREEN → F_BLUE** — each maintaining a counter (C1, C2, C3) that measures the respective color output frequency. A color is classified as dominant if its frequency count is the highest among all three. The dominant color generates a **2-bit color code** and asserts a `color_valid` start signal, which is transmitted to the Human–Cobot coordination unit to trigger the appropriate aiding or scanning action.

**FSM States:**

| State | Counter | Measures |
|---|---|---|
| `F_RED` | C1 | Red frequency output |
| `F_GREEN` | C2 | Green frequency output |
| `F_BLUE` | C3 | Blue frequency output |
| `COLOR_DECISION` | — | Compare C1/C2/C3, output dominant |
| `CLEAR (RESET)` | — | Reset all counters |

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `sensor_freq` | Input | Frequency output from TCS3200 |
| `clk` | Input | System clock |
| `color_code[1:0]` | Output | 2-bit dominant color identifier |
| `color_valid` | Output | Asserted when color decision is ready |

---

## 1.8 ADC Controller

**Module ID:** `adc_ctrl`

Converts analog signals from the **Line Following Array (LFA)** sensors — Left, Center, and Right — into digital values for navigation processing.

**Operation:**  
The controller sequentially samples each LFA channel by issuing multiplexer select and conversion start signals to the ADC. On each End-of-Conversion (`adc_eoc`) pulse, the digitized value is latched and mapped to its corresponding LFA direction output (`lfa_l`, `lfa_c`, `lfa_r`). These digital values are consumed by the LFA Controller to maintain accurate robot trajectory and obstacle detection.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `adc_data[7:0]` | Input | Converted digital value from ADC |
| `adc_eoc` | Input | End-of-conversion strobe |
| `clk` | Input | System clock |
| `mux_sel[1:0]` | Output | Channel select for ADC multiplexer |
| `conv_start` | Output | Conversion trigger |
| `lfa_l` | Output | Digitized left sensor value |
| `lfa_c` | Output | Digitized center sensor value |
| `lfa_r` | Output | Digitized right sensor value |

---

## 2. Processing Unit

The Processing Unit is built around a **single-cycle RV32I RISC-V CPU** running at 3.125 MHz. It receives navigation inputs — start node, end node, and path type — from the CPU Input Arbitrator, executes the BFS algorithm, and stores the computed path in data memory for the Post-Processing Unit.

---

### 2.1 RISC-V CPU Core (RV32I)

**Module ID:** `riscv_cpu`

A **single-cycle RV32I** processor running at **3.125 MHz**, maintaining uniform timing with all FPGA subsystems.

**Architecture:**

| Submodule | Description |
|---|---|
| **Instruction Memory** | 32-bit wide, 512-word addressable memory storing the compiled BFS navigation program |
| **Data Memory** | 64-word × 32-bit memory for variables, graph structures, and BFS results. Fixed memory-mapped entries for start node, end node, visited array, queue, and parent array |
| **Register File** | Low-latency temporary storage for intermediate values, supporting efficient ALU operation |
| **ALU** | Performs arithmetic, logical, and comparison operations for machine instruction execution |
| **Controller & Main Decoder** | Interprets instructions and generates control signals to coordinate data flow across ALU, register file, and memory |
| **Immediate Extension Unit** | Expands immediate fields to 32-bit values for ALU-based computation |

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `clk` | Input | 3.125 MHz system clock |
| `rst` | Input | Synchronous reset |
| `ext_mem_wr_en` | Input | External write enable (from CPU Arbitrator) |
| `ext_mem_addr[5:0]` | Input | External memory address |
| `ext_mem_data[31:0]` | Input | External data (SN/EN written by Arbitrator) |

---

### 2.2 Shortest Path Loader (BFS)

**Module ID:** `bfs_path_loader`  
*(Compiled to RV32I machine code and stored in instruction memory)*

Implements the **Breadth-First Search (BFS)** algorithm to determine the most efficient traversal path between industrial units. Each node represents a distinct part of the workspace; black edges denote valid navigation paths.

**Algorithm Workflow:**

1. Initialize start node and enqueue all directly connected nodes.
2. Maintain a `parent[]` array tracking each node's predecessor.
3. Iteratively explore nodes from the queue, marking each as visited to prevent reprocessing.
4. When the end node is found, reconstruct the path from `parent[]`.
5. Store the reconstructed path sequentially in data memory for post-processing.

**Memory-Mapped Variables (Data Memory):**

| Variable | Description |
|---|---|
| `start_node` | BFS source node (written by Input Arbitrator) |
| `end_node` | BFS destination node (written by Input Arbitrator) |
| `visited[]` | Boolean array — tracks explored nodes |
| `queue[]` | BFS queue buffer |
| `parent[]` | Predecessor map for path reconstruction |
| `path[]` | Final reconstructed path output |

---

### 2.3 Reset Module

**Module ID:** `cpu_reset_mod`

After the shortest path has been extracted and written to data memory, this module clears all temporary registers, counters, and memory-mapped variables used by the BFS algorithm. Prepares the system for subsequent navigation instructions and prevents residual data from affecting future computations.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `path_done` | Input | Asserted when BFS path is written to memory |
| `clk` | Input | System clock |
| `bfs_mem_clear` | Output | Clears queue, visited, and parent arrays |
| `cpu_soft_rst` | Output | Soft reset to BFS program counter context |

---

## 3. Post-Processing Unit

After the CPU computes and stores the optimal path in data memory, the Post-Processing Unit interprets the path values and generates corresponding control signals for robotic actuation. It acts as the hardware interface between the CPU's path-planning logic and the motion-control subsystem.

---

### 3.1 CPU-Path & Turns Arbitrator

**Module ID:** `cpu_path_arb`

Reads the BFS-computed path from CPU data memory into an internal buffer. Tracks node progression with an internal counter and asserts a **completion flag** when all path nodes are loaded. This flag notifies the Turn Controller to begin navigation.

**Operation:**  
Interfaces with CPU data memory to sequentially latch path nodes into a local register array. Manages end-of-path conditions, obstacle events, and unit-specific triggers to preserve path integrity. Acknowledgment signaling supports conditional runtime path reconfiguration.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `cpu_mem_data[31:0]` | Input | Path data from CPU data memory |
| `clk`, `rst` | Input | Clock and reset |
| `path_buf[]` | Output | Internal path register array |
| `node_idx` | Output | Current node index counter |
| `path_done_flag` | Output | Asserted when path buffer is fully loaded |

---

### 3.2 Turn & Node Controller

**Module ID:** `turn_node_ctrl`

Determines robot maneuvers by processing **triplets of nodes** (previous, current, next) and performing a lookup in the Industry Turns Graph to produce deterministic turn commands.

**Operation:**  
Synchronizes with the CPU-Path Arbitrator via a handshake mechanism that samples path data into internal registers to avoid timing uncertainty. Outputs a 3-bit direction code for each node transition. Acknowledgment signaling supports conditional runtime path reconfiguration.

**Direction Codes:**

| Code | Direction |
|---|---|
| `000` | Straight |
| `001` | Left |
| `010` | Right |
| `011` | U-turn |
| `100` | Reverse |

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `path_buf[]` | Input | Path buffer from Path Arbitrator |
| `ack_in` | Input | Handshake acknowledge |
| `clk` | Input | System clock |
| `dir_cmd[2:0]` | Output | Turn direction command |
| `ack_out` | Output | Handshake response |

---

### 3.3 Industry Turns Graph

**Module ID:** `turns_graph_lut`  
*(Combinational lookup table)*

A predefined combinational lookup table that maps all `(previous, current, next)` node triplets to the required robot maneuver direction. Enables deterministic, zero-latency directional decision-making **without real-time computational overhead**.

**Operation:**  
Encodes the complete industrial floor intersection topology. For each unique node triplet, the table outputs the appropriate direction code. Supports exceptional maneuvers for loops and obstructions, integrating seamlessly with the motion control subsystem.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `prev_node[4:0]` | Input | Previous path node |
| `curr_node[4:0]` | Input | Current path node |
| `next_node[4:0]` | Input | Next path node |
| `dir_cmd[2:0]` | Output | Direction (combinational, zero latency) |

---

### 3.4 LFA Controller

**Module ID:** `lfa_ctrl`

Sustains track alignment by digitizing L–C–R sensor values through the ADC Controller and applying **proportional PWM corrections** for straight motion, deviations, sharp turns, and node events.

**Operation:**  
Maps each of the 8 possible LCR sensor patterns to a motion action using a predefined lookup. Governs initial alignment, overshoot recovery, and deterministic node-stop behavior. Does not use PID — applies direct proportional PWM differential. Node detection (all sensors = `1-1-1`) signals the Path Arbitrator.

**LFA Sensor Pattern Table:**

| L | C | R | Action |
|---|---|---|---|
| 0 | 1 | 0 | Straight |
| 1 | 0 | 0 | Left correction |
| 0 | 0 | 1 | Right correction |
| 1 | 0 | 1 | Default straight (rare case) |
| 1 | 1 | 0 | Left merge / alignment correction |
| 0 | 1 | 1 | Right merge / alignment correction |
| 0 | 0 | 0 | Straight / error correction |
| 1 | 1 | 1 | **Stop / U-turn — Node detected** |

> A **node** represents a point in the industrial layout where the robot may stop or perform a directional change — corresponding to units, junctions, or any marked intersection.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `lfa_l`, `lfa_c`, `lfa_r` | Input | Digitized sensor values from ADC Controller |
| `clk` | Input | System clock |
| `pwm_left_duty[7:0]` | Output | Left motor PWM duty cycle |
| `pwm_right_duty[7:0]` | Output | Right motor PWM duty cycle |
| `node_detect` | Output | Asserted when LCR = 1-1-1 |

---

### 3.5 PWM Generators & Motor Drivers

**Module ID:** `pwm_motor_ctrl`

Two independent PWM generators regulate the **left and right motor pairs**, adjusting duty cycles to define both linear speed and turning curvature. Interfaces with the **L298N** motor driver for bidirectional actuation.

**Operation:**  
Each generator runs a free-running counter against a programmable compare register. Asymmetric left/right duty cycles produce differential wheel speeds for smooth directional transitions through coordinated inner–outer wheel speed control. Direction bits are forwarded to L298N IN1–IN4 for forward/reverse control.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `pwm_left_duty[7:0]` | Input | Left motor duty cycle |
| `pwm_right_duty[7:0]` | Input | Right motor duty cycle |
| `dir_l`, `dir_r` | Input | Motor direction bits |
| `clk` | Input | System clock |
| `PWM_L`, `PWM_R` | Output | PWM signals to L298N |
| `IN1`, `IN2`, `IN3`, `IN4` | Output | Direction control to L298N |

---

## 4. Object Handling & Communication Unit

Manages detection and manipulation of objects and obstacles along the industrial path, and handles all UART-based communication back to the Human–Cobot Center.

---

### 4.1 Object Handling

**Module ID:** `obj_handler`

Manages detection and manipulation of objects at nodes. Using proximity information from the **IR Sensor Controller**, it identifies objects when `LFA = 1-1-1` and triggers a synchronized **pick-and-place routine** using a 2-DOF actuator.

**Actuator Roles:**

| Servo | Function |
|---|---|
| Servo 1 (S1) | Gripping and releasing |
| Servo 2 (S2) | Vertical lift and placement |

**Pick-and-Place Sequence:**

```
IR detects object + LFA = 1-1-1
  → S1: grip angle
  → S2: lift angle
  → Navigate to target
  → S2: lower angle
  → S1: release angle
  → Assert obj_done_flag → CPU resumes navigation
```

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `ir_detect` | Input | IR sensor object detection |
| `node_detect` | Input | LFA node signal (LCR = 1-1-1) |
| `clk` | Input | System clock |
| `s1_pwm_duty[7:0]` | Output | Servo 1 PWM duty |
| `s2_pwm_duty[7:0]` | Output | Servo 2 PWM duty |
| `obj_done_flag` | Output | Completion feedback to CPU |

---

### 4.2 Obstacle Handling

**Module ID:** `obs_handler`

Detects and clears obstacles encountered **outside node conditions** (any LFA pattern other than `1-1-1`). Executes a **5-state clearance FSM** to remove the obstruction and return the robot to its planned route.

**FSM States:**

| State | Action |
|---|---|
| `PICK` | S1 grips the obstacle |
| `LIFT` | S2 lifts the obstacle |
| `MOVE` | Robot rotates toward open space |
| `PLACE` | S2 lowers, S1 releases obstacle |
| `RETURN` | Robot returns to planned route |

**Operation:**  
The `ob_flag` is asserted throughout the clearance sequence to suppress normal LFA navigation. The IR Sensor Controller coordinates `pick_signal`, `drop_signal`, and `ob_flag`. Once clearance is complete, normal navigation resumes.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `ir_detect` | Input | IR sensor signal |
| `lfa_pattern[2:0]` | Input | Current LCR sensor reading |
| `clk` | Input | System clock |
| `s1_pwm`, `s2_pwm` | Output | Servo PWM outputs |
| `ob_flag` | Output | Suppresses LFA navigation during clearance |
| `clearance_done` | Output | Asserted when route is restored |

---

### 4.3 Communication Handling & UART Transmission

**Module ID:** `uart_tx`

Serializes and transmits encoded completion messages to the Human–Cobot Center via UART at **11,520 bps**. Implements a **4-state TX FSM**: Idle → Start → Data → Stop.

**Operation:**  
After each pick-place completion by the 2-DOF actuator, a formatted status message is transmitted (e.g., `PSU-P1-Complete-Red#`). The transmitter inserts a start bit, clocks out 8 data bits LSB-first, and appends a stop bit. A **green LED** asserts on successful transmission.

**TX FSM States:**

| State | Action |
|---|---|
| `IDLE` | Wait for `tx_start` |
| `START` | Drive line low (start bit) |
| `DATA` | Shift out 8 bits LSB-first |
| `STOP` | Drive line high (stop bit), assert `tx_done` |

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `tx_byte[7:0]` | Input | Byte to transmit |
| `tx_start` | Input | Transmission trigger |
| `clk` | Input | System clock (3.125 MHz) |
| `tx` | Output | Serial data line out |
| `tx_done` | Output | Byte transmission complete |
| `led_green` | Output | Successful transmission indicator |

---

### 4.4 TX Controller for Message Sequencing

**Module ID:** `tx_ctrl`

Manages ordered multi-character message transmission. Tracks **subunit done flags**, issues `tx_start` pulses, and serializes characters in sync with `tx_done` from the UART Transmitter.

**Operation:**  
Holds the full outgoing message string in a local register array. Steps through each character on every `tx_done` acknowledgment. A `#` terminator finalizes each message. Internal counters and flags reset during global initialization to preserve transmission integrity.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `subunit_done_flags` | Input | Completion signals from Object Handler |
| `tx_done` | Input | Byte-sent acknowledgment from UART TX |
| `clk`, `rst` | Input | Clock and reset |
| `tx_byte[7:0]` | Output | Current character to UART TX |
| `tx_start` | Output | Trigger to UART TX |

---

### 4.5 Clock Domain Crossing (CDC) Handler

**Module ID:** `cdc_handler`

Safely transfers control signals between the **3.125 MHz** computation/communication clock domain and the **50 Hz** servo actuation clock domain. Prevents metastability and maintains timing correctness across both domains.

**Operation:**  
Implements a two-stage flip-flop synchronizer combined with a request/acknowledge handshake. The handshake ensures servo command data is fully stable and correctly latched before actuation begins. This arises because the 2-DOF servo actuators operate at 50 Hz while all computation and communication blocks run at 3.125 MHz.

**Ports:**

| Port | Direction | Description |
|---|---|---|
| `clk_fast` | Input | 3.125 MHz computation clock |
| `clk_slow` | Input | 50 Hz servo actuation clock |
| `data_in[7:0]` | Input | Servo command from fast domain |
| `req` | Input | Transfer request |
| `data_out_sync[7:0]` | Output | Synchronized servo command |
| `ack` | Output | Transfer acknowledged |

---

## System Summary

| Unit | Modules | Key Technology |
|---|---|---|
| Input Preprocessing | 6 | UART Rx, FSM, Priority Arbitration |
| Color Sensor | 1 | TCS3200, Frequency Counting FSM |
| ADC Controller | 1 | Sequential Sampling, Channel Mapping |
| Processing | 3 | RV32I Single-Cycle CPU, BFS Algorithm |
| Post-Processing | 5 | LUT, PWM, Line Following, Turn Graph |
| Object Handling & Comm | 5 | 2-DOF Servo, UART Tx, CDC Synchronizer |
| **Total** | **21** | |

---

**All the modules can be updated based on the industrial scenario and can be scaled based on the requirements**
