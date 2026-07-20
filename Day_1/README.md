# Day 1: Introduction to RTL Design Flow using Verilog, Icarus Verilog, GTKWave, and Yosys

## 🎯 Objective

The goal of Day 1 is to understand the complete front-end RTL design flow, from writing Verilog code to synthesizing it into hardware using open-source EDA tools. By the end of this session, you will know how to simulate a digital design, verify its functionality through waveforms, and synthesize it using the Sky130 standard cell library.

---

# Learning Outcomes

After completing Day 1, you will be able to:

* Understand the RTL design flow.
* Write and modify Verilog RTL code.
* Create and use a testbench for simulation.
* Compile and simulate designs using Icarus Verilog.
* Analyze digital waveforms using GTKWave.
* Understand the purpose of Liberty (`.lib`) files.
* Perform RTL synthesis using Yosys.
* Map RTL logic to Sky130 standard cells.
* Visualize the synthesized circuit.

---

# Prerequisites

Before starting, ensure the following tools are installed:

| Tool                   | Purpose                       |
| ---------------------- | ----------------------------- |
| Ubuntu/Linux           | Development Environment       |
| Git                    | Clone the workshop repository |
| Icarus Verilog         | Verilog Compiler & Simulator  |
| GTKWave                | Waveform Viewer               |
| Yosys                  | RTL Synthesis Tool            |
| Sky130 Liberty Library | Technology Mapping            |

---

# RTL Design Flow

```text
Specification
      │
      ▼
Verilog RTL Design
      │
      ▼
Testbench
      │
      ▼
Compilation (Icarus Verilog)
      │
      ▼
Simulation
      │
      ▼
Waveform (.vcd)
      │
      ▼
GTKWave Verification
      │
      ▼
RTL Synthesis (Yosys)
      │
      ▼
Technology Mapping (Sky130)
      │
      ▼
Gate-Level Netlist
```

---

# Step 1: Clone the Repository

### Command

```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

### Purpose

Downloads the complete workshop repository containing Verilog source files, testbenches, and Sky130 libraries.

---

# Step 2: Navigate to the Verilog Files

### Command

```bash
cd sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

### Purpose

Moves into the directory containing the Verilog design files.

---

# Step 3: Install Icarus Verilog

### Command

```bash
sudo apt install iverilog
```

### Purpose

Installs the Verilog compiler used for simulation.

### Verification

```bash
iverilog -V
```

---

# Step 4: Install GTKWave

### Command

```bash
sudo apt install gtkwave
```
<img width="954" height="989" alt="image" src="https://github.com/user-attachments/assets/2ca4fb3b-5879-49a6-94c2-9ce7359c1e30" />


### Purpose

Installs the waveform viewer used to analyze simulation results.

### Verification

```bash
gtkwave --version
```
<img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/b04a724c-37ca-4183-b64c-80579cb04b7b" />

---

# Step 5: Understand the Design

The design used in Day 1 is a **2:1 Multiplexer**.

### Truth Table

| sel | Output |
| --- | ------ |
| 0   | i0     |
| 1   | i1     |

RTL Code

```verilog
always @(*) begin
    if(sel)
        y = i1;
    else
        y = i0;
end
```
<img width="953" height="1020" alt="image" src="https://github.com/user-attachments/assets/7e259296-efe5-407a-afd8-bb8d79ab0940" />

This is **combinational logic**, meaning the output depends only on the current input values.

---

# Step 6: Compile the Design

### Command

```bash
iverilog good_mux.v tb_good_mux.v
```

### Purpose

Compiles both the design (`good_mux.v`) and the testbench (`tb_good_mux.v`) into an executable simulation file.

### Output

```text
a.out
```

---

# Step 7: Run the Simulation

### Command

```bash
./a.out
```

### Purpose

Executes the compiled simulation.

### Expected Output

```text
VCD info: dumpfile tb_good_mux.vcd opened for output.
```

A **Value Change Dump (VCD)** file is generated to record signal transitions during simulation.

---

# Step 8: View the Waveform

### Command

```bash
gtkwave tb_good_mux.vcd
```

### Purpose

Opens the generated waveform file.

### Observe

* `i0`
* `i1`
* `sel`
* `y`

Verify that:

* When `sel = 0`, `y = i0`
* When `sel = 1`, `y = i1`

This confirms the multiplexer operates correctly.

---

# Step 9: Modify the RTL (Optional)

### Command

```bash
nano good_mux.v
```

### Purpose

Open the RTL file for editing.

Useful shortcuts:

* `Ctrl + O` → Save
* `Ctrl + X` → Exit

---

# Step 10: Launch Yosys

### Command

```bash
yosys
```
<img width="715" height="363" alt="image" src="https://github.com/user-attachments/assets/be88c817-c950-48fb-935b-fd8a930a430e" />

### Purpose

Starts the Yosys synthesis shell.

---

# Step 11: Load the Sky130 Standard Cell Library

### Command

```tcl
read_liberty -lib /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Purpose

Loads the Sky130 standard cell library, which contains timing, area, and functional information for available logic cells.

Expected message:

```text
Imported 418 cell types from liberty file.
```
<img width="912" height="92" alt="image" src="https://github.com/user-attachments/assets/da58d77a-8ec6-45b2-940e-3d385fa574ca" />

---

# Step 12: Read the Verilog Design

### Command

```tcl
read_verilog good_mux.v
```

### Purpose

Parses the Verilog RTL and converts it into Yosys's internal RTL representation.

Expected message:

```text
Successfully finished Verilog frontend.
```

---

# Step 13: Synthesize the RTL

### Command

```tcl
synth -top good_mux
```

### Purpose

Synthesizes the RTL design into generic logic.

During synthesis, Yosys performs:

* Hierarchy analysis
* Process conversion
* Logic optimization
* Latch detection
* Constant propagation
* Technology-independent mapping

Since this is a combinational multiplexer, Yosys reports:

```text
No latch inferred
```

which indicates correct combinational coding.

---

# Step 14: Technology Mapping

### Command

```tcl
abc -liberty /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Purpose

Maps the synthesized logic to actual Sky130 standard cells using the ABC optimization engine.

In your run, ABC reported:

```text
ABC RESULTS:
MUX cells : 1
```

This confirms the RTL was implemented as a single multiplexer cell.

---

# Step 15: Visualize the Synthesized Circuit

### Command

```tcl
show
```
<img width="1600" height="846" alt="image" src="https://github.com/user-attachments/assets/4578ae65-82eb-4ea3-9a9d-9a9283acfed0" />
<img width="898" height="635" alt="image" src="https://github.com/user-attachments/assets/e006b6b5-7932-4be6-bf41-70e572a9a979" />

### Purpose

Generates and displays a graphical representation of the synthesized netlist using Graphviz.

The circuit should show one multiplexer connected to three inputs (`i0`, `i1`, `sel`) and one output (`y`).

---

# Key Concepts Learned

* RTL (Register Transfer Level) Design
* Verilog HDL Basics
* Combinational Logic
* Testbench and Functional Verification
* Compilation and Simulation
* VCD Waveform Generation
* Waveform Analysis using GTKWave
* Standard Cell Libraries (`.lib`)
* RTL Synthesis using Yosys
* Technology Mapping with ABC
* Gate-Level Netlist Visualization

---

# Commands Summary

| Command                             | Description                       |
| ----------------------------------- | --------------------------------- |
| `git clone`                         | Download the workshop repository  |
| `cd`                                | Navigate to the project directory |
| `sudo apt install iverilog`         | Install Verilog compiler          |
| `sudo apt install gtkwave`          | Install waveform viewer           |
| `iverilog good_mux.v tb_good_mux.v` | Compile the design and testbench  |
| `./a.out`                           | Run the simulation                |
| `gtkwave tb_good_mux.vcd`           | View the waveform                 |
| `nano good_mux.v`                   | Edit the Verilog file             |
| `yosys`                             | Launch the synthesis tool         |
| `read_liberty -lib ...`             | Load the Sky130 library           |
| `read_verilog good_mux.v`           | Read the RTL design               |
| `synth -top good_mux`               | Synthesize the design             |
| `abc -liberty ...`                  | Perform technology mapping        |
| `show`                              | Display the synthesized circuit   |

---

# Conclusion

On Day 1, I completed the complete RTL design flow for a 2:1 multiplexer using open-source EDA tools. I compiled and simulated the design with Icarus Verilog, verified the output through GTKWave, synthesized the RTL using Yosys, and successfully mapped the logic to the Sky130 standard cell library. This provided a practical understanding of the fundamental front-end ASIC design workflow and the relationship between RTL code, simulation, synthesis, and hardware implementation.
