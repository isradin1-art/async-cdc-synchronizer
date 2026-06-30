# Asynchronous Clock Domain Crossing (CDC) Synchronizer Pipeline

A robust, hardware-validated Verilog implementation of a multi-bit Clock Domain Crossing (CDC) pipeline. This project addresses the challenge of transferring data safely between two completely asynchronous clock domains (Fast Transmitter at 250 MHz to Slow Receiver at 100 MHz) without triggering metastability or data corruption.

## Key Features
* **Multi-Bit Synchronization:** Safely crosses a 4-bit data bus using Gray Code encoding to guarantee single-bit transitions.
* **2-Stage Flip-Flop Synchronizer:** Employs a robust structural 2-stage FF pipeline synchronized to the destination clock domain to settle potential metastable events.
* **Timing Constraints Integration (XDC):** Includes professional Xilinx Design Constraints (XDC) using virtual clocks and asynchronous clock groups to achieve proper timing closure in AMD Xilinx Vivado.
* **Hardware-Centric Testbench:** Features a multi-clock testbench simulating realistic clock phase shifts and edge alignments to stress-test the CDC boundary.

## System Architecture

The architecture consists of three main hardware modules pipelined together:
1. **`bin2gray`:** Converts the binary input vector from the source domain into Gray code, ensuring that only one bit changes per clock cycle.
2. **`cdc_synchronizer`:** A 2-Stage Flip-Flop synchronizer that captures the Gray code bus within the destination clock domain and allows metastable states to resolve.
3. **`gray2bin`:** Converts the synchronized Gray code back into standard binary format for the destination domain's logic.

<img width="1452" height="527" alt="image" src="https://github.com/user-attachments/assets/7e1d8e40-2dd1-4d33-aaae-e48fe8b12c08" />

## Simulation & Analysis

The behavioral simulation under asynchronous conditions demonstrates the **Undersampling effect**. Since the destination clock ($100\text{ MHz}$) is slower than the source clock ($250\text{ MHz}$), it skips some intermediate counter values. 

However, because the data is encoded in Gray Code at the boundary, **only one bit changes at any specific sampling edge**. This eliminates the risk of sampling corrupted or mixed values (e.g., catching a transition mid-way), providing a perfectly stable and clean output bus.

<img width="1250" height="683" alt="image" src="https://github.com/user-attachments/assets/740423de-2019-42ba-bb0f-1e40d8bc33da" />



## Design Constraints (XDC)

To achieve strict timing closure, a `timing_constraints.xdc` file is included. It explicitly informs Vivado's Static Timing Analysis (STA) engine that the source and destination domains are asynchronous, avoiding false setup/hold violations while ensuring the synthesizer optimizes the physical placement of the synchronizing flip-flops.

```tcl
# Primary Destination Clock (100 MHz)
create_clock -period 10.000 -name clk_dest -waveform {0.000 5.000} [get_ports clk]

# Virtual Source Clock (250 MHz)
create_clock -period 4.000 -name clk_src

# Asynchronous CDC Boundary Definition
set_clock_groups -asynchronous -group [get_clocks clk_src] -group [get_clocks clk_dest]
