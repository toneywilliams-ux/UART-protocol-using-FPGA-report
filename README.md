# Hardware Implementation of a UART Communication Interface

**Author:** Toney Williams Kakara

This project documents the design, simulation, and physical hardware implementation of a complete Universal Asynchronous Receiver-Transmitter (UART) from scratch on an Artix-7 FPGA. The system establishes a reliable, asynchronous serial loopback communication channel between the physical FPGA board and a host PC at a 9600 baud rate. This project demonstrates full RTL-to-bitstream proficiency, bridging fundamental digital logic with physical hardware interfaces.

## Tools and Environment

| Tool/Platform | Version/Spec | Purpose |
| :--- | :--- | :--- |
| **Vivado** | 2025.2 | RTL Synthesis, Simulation, and Implementation |
| **Verilog HDL** | IEEE 1364-2001 | Hardware Description Language |
| **Tera Term** | Latest | Serial Terminal for Hardware Verification |
| **Digilent Basys 3** | Artix-7 35T | Physical Target Hardware Platform |

## Architectural Breakdown

The design is partitioned into three distinct, parameterizable hardware modules governed by a top-level wrapper.

* **Baud Rate Generator (BRG):** A precision clock divider scaling the 100 MHz system clock to generate a 9600 Hz transmission tick and a 153.6 kHz (16x oversampling) receiver tick to ensure stable data sampling.
* **Transmitter FSM (Tx):** A 4-state Finite State Machine (IDLE, START, DATA, STOP) that serializes 8-bit parallel data, injecting start and stop framing bits for physical transmission over the wire.
* **Receiver FSM (Rx):** A robust edge-detecting state machine utilizing 16x oversampling to identify the center of incoming data bits, mitigating noise and metastable states before shifting the serial bitstream back into parallel data.
* **Top-Level Architecture:** The top-level module instantiates the BRG, Tx, and Rx components, explicitly wiring them together for a continuous loopback verification test.

## RTL Verification Strategy

Before progressing to physical synthesis, the RTL logic was rigorously verified using a self-checking testbench in Vivado.

* Simulated a complete loopback environment by driving the receiver line with an artificial serial data stream, precisely factoring in the 104.167 µs bit-period timing.
* Verified the receiver FSM correctly asserted the `rx_done` flag, seamlessly triggering the transmitter FSM to echo the identical data packet without data loss or framing errors over a 3 ms simulation window.

## Hardware Implementation & Physical Verification

Following successful behavioral simulation, the design was synthesized and physically implemented on the Xilinx Basys 3 development board.

* An XDC constraints file was authored to explicitly map the RTL I/O ports to the physical hardware interfaces on the Basys 3 board, specifically routing the serial data through the onboard USB-UART bridge.
* The Basys 3 board was connected to the host PC via USB, recognized natively on a dedicated COM port.
* A serial connection was established using the Tera Term terminal emulator, configured strictly to a 9600 Baud Rate, 8 data bits, no parity, and 1 stop bit.
* Keyboard inputs originating from the host PC were successfully transmitted over USB, decoded by the custom Verilog Receiver FSM on bare silicon, passed to the Transmitter FSM, and instantaneously echoed back to the Tera Term console screen. 
* This confirmed 100% operational integrity of the physical hardware logic.

## Synthesis Analytics (Timing & Power)

Post-implementation analytics were extracted from Vivado to quantify the efficiency of the physical design on the Artix-7 fabric.

| Metric | Value |
| :--- | :--- |
| **System Clock** | 100 MHz (10.000 ns period) |
| **Worst Negative Slack (WNS)** | +5.408 ns |
| **Total On-Chip Power** | 0.074 W (74 mW) |
| **Dynamic Power** | 0.002 W |
| **Static/Leakage (Device) Power** | 0.072 W |

### Key Takeaways
* The design comfortably meets all setup and hold timing constraints with zero violations. 
* The positive slack margin dictates that the longest critical combinational logic path in the design resolves in under 4.6 ns, proving the FSMs are highly stable.
* The dynamic power consumption of 0.002 W demonstrates highly efficient logic switching during active data transmission. 
* The overall thermal footprint is dominated almost entirely by the baseline static leakage (0.072 W) of the 28nm Artix-7 device.
