# Day 2 – RTL Design and Synthesis using Yosys

<p align="center">
  <img src="https://img.shields.io/badge/Workshop-VSD%20RTL%20Design%20%26%20Synthesis-blue" />
  <img src="https://img.shields.io/badge/Tool-Yosys-success" />
  <img src="https://img.shields.io/badge/Language-Verilog-orange" />
  <img src="https://img.shields.io/badge/Library-Sky130-red" />
</p>

---

# Overview

Day 2 of the **RTL Design and Synthesis Workshop** focuses on understanding how Verilog RTL is transformed into a gate-level netlist using the **Yosys Open Synthesis Suite** and the **Sky130 Standard Cell Library**.

Unlike simulation, which verifies the functionality of a design, synthesis converts RTL code into logic gates that can be physically implemented on silicon. During this process, the synthesis tool reads the Verilog description, optimizes the logic, maps it to standard cells available in the technology library, and generates a gate-level netlist.

This session also introduces the concept of **Timing Libraries**, which provide delay, power, and area information required by synthesis tools for technology mapping and optimization.

---

# Objectives

* Understand the purpose of Timing Libraries.
* Learn the RTL-to-Gate-Level synthesis flow.
* Perform synthesis using Yosys.
* Analyze synthesized netlists.
* Compare Hierarchical and Flat synthesis.
* Observe how different coding styles affect synthesized hardware.
* Understand D Flip-Flop inference using various Verilog coding styles.

---

# Tools Used

| Tool                         | Purpose                      |
| ---------------------------- | ---------------------------- |
| Ubuntu Linux                 | Development Environment      |
| Icarus Verilog               | RTL Compilation & Simulation |
| GTKWave                      | Waveform Viewer              |
| Yosys                        | RTL Synthesis                |
| Sky130 Standard Cell Library | Technology Mapping           |

---

# What is RTL?

RTL (Register Transfer Level) is a hardware description that explains how data moves between registers and how logic operations are performed every clock cycle.

RTL is written using Hardware Description Languages such as **Verilog**.

Example:

```verilog
always @(posedge clk)
    q <= d;
```

This code describes a D Flip-Flop that captures the input `d` on every positive edge of the clock.

---

# What is Synthesis?

Synthesis is the process of converting RTL code into a gate-level circuit using logic gates available in a technology library.

### Synthesis Flow

```
RTL Verilog
      │
      ▼
   Yosys Parser
      │
      ▼
Logic Optimization
      │
      ▼
Technology Mapping
      │
      ▼
Gate-Level Netlist
```

---

# Understanding Timing Libraries

A Timing Library ('.lib' file) contains information about every standard cell available in the fabrication technology.

The synthesis tool uses this information to determine:

* Propagation Delay
* Cell Area
* Leakage Power
* Dynamic Power
* Setup Time
* Hold Time
* Input Capacitance
* Output Drive Strength

Without the Timing Library, Yosys cannot determine which physical gates should replace the RTL logic.

For the Sky130 process, the commonly used library is:

```
sky130_fd_sc_hd__tt_025C_1v80.lib
```

Where:

| Parameter | Meaning             |
| --------- | ------------------- |
| tt        | Typical Process     |
| 025C      | 25°C Temperature    |
| 1v80      | 1.8V Supply Voltage |

---
<img width="1600" height="842" alt="image" src="https://github.com/user-attachments/assets/605a0e32-30a0-4e69-92a3-da8d892ec0ac" />


# RTL Simulation

Before synthesis, every RTL design is verified through simulation.

Simulation checks whether the logic behaves as expected.

### Compilation

```bash
iverilog design.v tb_design.v
```

Explanation:

* `iverilog` compiles the RTL and testbench.
* Generates simulation executable.
<img width="934" height="426" alt="image" src="https://github.com/user-attachments/assets/ee9f08e6-72f6-4f66-8815-612fc6308c07" />
<img width="930" height="597" alt="image" src="https://github.com/user-attachments/assets/10e9543c-10ae-4204-93ce-a26049b78615" />
<img width="928" height="912" alt="image" src="https://github.com/user-attachments/assets/29c5d917-df3b-417d-a69d-f691be745f87" />
<img width="927" height="919" alt="image" src="https://github.com/user-attachments/assets/983f3841-def9-4d17-b1f8-d3e5dac22ace" />
<img width="772" height="737" alt="image" src="https://github.com/user-attachments/assets/f8a90638-fb67-4b73-b6b5-f1f33fb863c0" />

---

### Run Simulation

```bash
./a.out
```

or

```bash
vvp a.out
```

Explanation:

Executes the compiled Verilog simulation.

---

### View Waveforms

```bash
gtkwave dump.vcd
```

Explanation:

Opens GTKWave to visualize signal transitions and verify the circuit behavior.
<img width="1600" height="843" alt="image" src="https://github.com/user-attachments/assets/4e674f09-a47c-45f0-9b42-5aca2605359b" />
<img width="1600" height="846" alt="image" src="https://github.com/user-attachments/assets/4ea67ebb-8517-4af1-a212-6ea6145abf58" />
<img width="939" height="481" alt="image" src="https://github.com/user-attachments/assets/0cc41108-b082-4024-9b21-fa827fc3095c" />
<img width="928" height="1006" alt="image" src="https://github.com/user-attachments/assets/4d8b40e7-66c1-4770-b883-10fdf2b0d360" />

---

# Introduction to Yosys

Yosys is an open-source synthesis framework used for digital circuit synthesis.

It converts RTL into optimized gate-level hardware.

---

# Starting Yosys

```bash
yosys
```

Starts the Yosys synthesis shell.

---

# Reading Liberty File

```tcl
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
```

Explanation:

Loads the Sky130 timing library into Yosys.

This library provides:

* Cell Delays
* Area Information
* Power Information
* Available Standard Cells

---

# Reading Verilog

```tcl
read_verilog filename.v
```

Explanation:

Imports the RTL design into Yosys.

---

# Checking Design

```tcl
hierarchy -check -top module_name
```

Explanation:

* Identifies top module.
* Verifies module hierarchy.
* Reports missing modules.

---

# Generic Synthesis

```tcl
synth -top module_name
```

Explanation:

Performs generic synthesis.

Tasks performed:

* Constant propagation
* Boolean optimization
* FSM optimization
* Memory optimization
* Dead code removal

---

# Technology Mapping

```tcl
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

Explanation:

Maps generic logic into Sky130 standard cells.

---

# Writing Netlist

```tcl
write_verilog synthesized_netlist.v
```

Explanation:

Generates synthesized Verilog netlist containing Sky130 standard cells.

---

# Viewing Statistics

```tcl
stat
```

Displays:

* Number of Cells
* Number of Wires
* Number of Inputs
* Number of Outputs
* Area Estimation

---

# Hierarchical Synthesis

Hierarchical synthesis preserves the original module hierarchy.

Command:

```tcl
synth -top module_name
```

Advantages

* Easier debugging
* Better readability
* Individual module optimization

---

# Flat Synthesis

Flattening removes all module boundaries.

Command

```tcl
flatten
```

Then

```tcl
synth -top module_name
```

Advantages

* Better optimization
* Reduced logic
* Improved timing

Disadvantage

* Difficult debugging

---

# D Flip-Flop Inference

Yosys automatically identifies sequential logic.

Example

```verilog
always @(posedge clk)
    q <= d;
```

Inference:

One positive-edge-triggered D Flip-Flop.

Other variations include:

* DFF with Reset
* DFF with Set
* Asynchronous Reset
* Asynchronous Set
* Synchronous Reset


Each coding style results in different standard cells after synthesis.

---

# Complete Yosys Flow

```bash
yosys

read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog design.v

hierarchy -check -top design

proc

opt

fsm

memory

techmap

abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib

clean

stat

show

write_verilog design_netlist.v
```
<img width="606" height="637" alt="image" src="https://github.com/user-attachments/assets/1c86f7bb-a1f3-4c5b-b568-b1cff5cbe081" />
<img width="1600" height="846" alt="image" src="https://github.com/user-attachments/assets/f222f9ee-0df7-4447-b903-96b0d9d2f379" />
<img width="1600" height="842" alt="image" src="https://github.com/user-attachments/assets/436f8032-1a51-4578-81d1-d40e82d3bee6" />

---

# Common Commands Summary

| Command       | Description                  |
| ------------- | ---------------------------- |
| yosys         | Start Yosys                  |
| read_verilog  | Read RTL                     |
| read_liberty  | Load Timing Library          |
| hierarchy     | Check module hierarchy       |
| synth         | Generic synthesis            |
| flatten       | Remove hierarchy             |
| abc           | Technology Mapping           |
| stat          | Display synthesis statistics |
| show          | Display circuit diagram      |
| write_verilog | Generate synthesized netlist |

---

# Screenshots

Include screenshots for:

* RTL Simulation
* GTKWave Output
* Reading Liberty File
* Reading Verilog
* Hierarchy Check
* Synth Command
* ABC Mapping
* Statistics
* Gate-Level Netlist
* Hierarchical Synthesis
* Flat Synthesis


# Key Learning Outcomes

* Learned the difference between RTL simulation and synthesis.
* Understood the role of Timing Libraries in technology mapping.
* Performed synthesis using Yosys.
* Explored hierarchical and flat synthesis techniques.
* Observed how RTL code is converted into gate-level hardware.
* Learned how different Verilog coding styles infer different hardware structures.
* Generated optimized gate-level netlists using the Sky130 standard cell library.

---

# Conclusion

Day 2 provided a strong foundation in RTL synthesis using Yosys and the Sky130 standard cell library. The session demonstrated how Verilog RTL is analyzed, optimized, and transformed into technology-specific logic gates. By comparing hierarchical and flat synthesis, studying timing libraries, and exploring D Flip-Flop inference, this workshop strengthened the understanding of the complete RTL-to-Gate-Level design flow used in modern ASIC design.

This knowledge forms the basis for the next stages of the VLSI design flow, including logic optimization, static timing analysis, placement, routing, and physical design.

---

## Repository Structure

```
Day_2/
│
├── README.md
├── verilog_files/
├── synthesized_netlists/
├── screenshots/
├── timing_library/
└── reports/
```

---

### Author

**Prithika Logaiyan**

Electronics and Communication Engineering (ECE)

RTL Design & Synthesis Workshop Repository
