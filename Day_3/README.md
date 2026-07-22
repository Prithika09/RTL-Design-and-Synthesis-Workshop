
# **Day 3: Combinational and Sequential Optimization**

## **Overview**

Day 3 of the RTL Design and Synthesis Workshop focuses on optimization techniques performed during the logic synthesis phase. Instead of converting Verilog code directly into gates, synthesis tools such as **Yosys**, **Synopsys Design Compiler**, or **Cadence Genus** analyze the logic, remove redundant hardware, simplify Boolean expressions, and optimize the design for:

* Area
* Delay
* Power (ADP)

while maintaining the exact functionality of the design.

During this session, the following tasks were completed:

* Studied Constant Propagation, State Optimization, Cloning, and Retiming.
* Designed, simulated, and synthesized six Verilog modules (Lab 1 to Lab 6).
* Verified designs using **Icarus Verilog (iverilog)** and **GTKWave**.
* Performed synthesis using **Yosys** and the **Sky130 Standard Cell Library**.
* Resolved common synthesis issues such as module mismatches, missing libraries, and file handling errors.

---

## **Objectives**

* Understand combinational logic optimization.
* Learn sequential optimization techniques.
* Observe how constant values simplify logic.
* Understand Yosys optimization commands.
* Perform complete RTL-to-Gate synthesis.

---

# **Theory and Concepts**

## **RTL Optimization Flow**

```
RTL Verilog Code
        │
        ▼
Synthesis Engine (Yosys)
        │
        ▼
Boolean Optimization
(Constant Propagation,
Redundancy Removal)
        │
        ▼
Technology Mapping
(Sky130 Standard Cells)
```

---

# **1. Constant Propagation**

### What is it?

Constant propagation is an optimization where signals permanently connected to **0** or **1** are propagated through Boolean expressions.

### Why is it used?

It removes unnecessary gates, reducing:

* Area
* Delay
* Power

### Boolean Rules

```
A · 0 = 0
A · 1 = A
A + 1 = 1
A + 0 = A
```

### Hardware Example

A 2:1 multiplexer selecting between **B** and **0**

```
Y = A ? B : 0
```

simplifies to

```
Y = A & B
```

The multiplexer disappears.

---

# **2. State Optimization**

### What is it?

State optimization removes:

* redundant states
* unreachable states

and optimizes state encoding.

### Benefits

* fewer flip-flops
* simpler next-state logic
* lower power

### Example

If two FSM states always behave identically,

```
State1
State2
```

they are merged into one state.

---

# **3. Cloning**

### What is it?

A gate driving many loads is duplicated.

Instead of

```
Gate
 │
100 Loads
```

it becomes

```
Gate A → 50 Loads

Gate B → 50 Loads
```

### Benefit

Less capacitive load

↓

Faster circuit.

---

# **4. Retiming**

### What is it?

Registers are moved across combinational logic without changing functionality.

Example

Before

```
Register

900 ps Logic

Register

100 ps Logic

Register
```

After retiming

```
Register

500 ps Logic

Register

500 ps Logic

Register
```

This increases the maximum clock frequency.

---

# **Software Tools**

| Tool           | Purpose                     |
| -------------- | --------------------------- |
| Icarus Verilog | Compile & simulate RTL      |
| VVP            | Execute compiled simulation |
| GTKWave        | View waveforms              |
| Yosys          | Logic synthesis             |
| Sky130 Liberty | ASIC technology library     |

---
<img width="917" height="606" alt="image" src="https://github.com/user-attachments/assets/c41129eb-fbc5-4d64-bd02-97af3252a246" />

# **Lab 1**

## Constant Propagation in MUX Logic

### Verilog

```verilog
module opt_check(
input a,
input b,
output y
);

assign y = a ? b : 1'b0;

endmodule
```

### Logic

```
a=0 → y=0

a=1 → y=b
```

Truth table becomes

```
Y = A & B
```

### Commands

Compile

```bash
iverilog -o opt_check_tb.out opt_check.v opt_check_tb.v
```

Run

```bash
vvp opt_check_tb.out
```

Waveform

```bash
gtkwave opt_check.vcd
```
<img width="1600" height="841" alt="image" src="https://github.com/user-attachments/assets/d2f082fd-823f-4262-8dd6-27e1ea285f30" />

### Yosys

```tcl
read_liberty -lib sky130.lib

read_verilog opt_check.v

synth -top opt_check

opt_clean -purge

abc -liberty sky130.lib

show
```
<img width="1600" height="791" alt="image" src="https://github.com/user-attachments/assets/defd1a34-f116-41f3-b881-94765b5f0eb9" />
<img width="600" height="553" alt="image" src="https://github.com/user-attachments/assets/31f42f19-abcf-4c77-8878-3e09c9c50969" />


### Result

MUX

↓

AND Gate

---

# **Lab 2**

### Verilog

```verilog
assign y = a ? 1'b1 : b;
```

Logic

```
a=1

↓

y=1

a=0

↓

y=b
```

Simplifies to

```
Y = A | B
```
### Commands

Compile

```bash
iverilog -o opt_check2_tb opt_check2.v opt_check2_tb.v
```

Run

```bash
vvp opt_check2_tb
```

Waveform

```bash
gtkwave opt_check2.vcd
```

<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/8d83454b-4b59-4cd0-824f-8dd4a115b20f" />


### Yosys

```tcl
read_liberty -lib /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog opt_check2.v

synth -top opt_check2

opt_clean -purge

abc -liberty /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```
<img width="1600" height="837" alt="image" src="https://github.com/user-attachments/assets/7f24f590-615b-4812-8c8f-c0c253c3550a" />
<img width="602" height="644" alt="image" src="https://github.com/user-attachments/assets/9ea6af58-8c00-40a1-8538-ca14d2223fad" />



Result

MUX

↓

OR Gate

---

# **Lab 3**

### Verilog

```verilog
assign y = a ? (c ? b : 1'b0) : 1'b0;
```

Boolean reduction

```
Y

=

A · (C · B)

=

A · B · C
```
### Commands

Compile

```bash
iverilog -o opt_check3_tb opt_check3.v opt_check3_tb.v
```

Run

```bash
vvp opt_check3_tb
```

Waveform

```bash
gtkwave opt_check3.vcd
```
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/86513c46-916c-4fd6-b51c-b94d2448dbdf" />

### Yosys

```tcl
read_liberty -lib /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog opt_check3.v

synth -top opt_check3

opt_clean -purge

abc -liberty /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```
<img width="1600" height="836" alt="image" src="https://github.com/user-attachments/assets/11070fb7-d210-40b7-9c00-13d7bc4a4d04" />
<img width="606" height="605" alt="image" src="https://github.com/user-attachments/assets/29dc5574-c99c-4f33-adde-d95a04c3e462" />

Result

3-input AND Gate

---

# **Lab 4**

### Verilog

```verilog
assign y = a ? (b ? (a & c) : c) : (!c);
```

Simplification

When

```
a=1
```

output becomes

```
c
```

When

```
a=0
```

output becomes

```
!c
```

Final equation

```
Y = A ? C : !C
```

Equivalent gate

```
XNOR
```

Input **b** is completely removed.

---
### Commands

Compile

```bash
iverilog -o opt_check4_tb opt_check4.v opt_check4_tb.v
```

Run

```bash
vvp opt_check4_tb
```

Waveform

```bash
gtkwave opt_check4.vcd
```
<img width="1600" height="784" alt="image" src="https://github.com/user-attachments/assets/aeccc5b1-0b4c-4c76-9a06-48aa51c9df07" />

### Yosys

```tcl
read_liberty -lib /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog opt_check4.v

synth -top opt_check4

opt_clean -purge

abc -liberty /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```
<img width="1600" height="845" alt="image" src="https://github.com/user-attachments/assets/d6028fe9-1bd0-4403-8e2d-3e2d81d2580b" />
<img width="616" height="635" alt="image" src="https://github.com/user-attachments/assets/ceb3c4aa-bc3a-4b3d-9f42-1c8040fdd0a4" />

# **Lab 5**

## Sequential Optimization

### Verilog

```verilog
always @(posedge clk or posedge reset)

if(reset)

q<=0;

else

q<=1;
```

Behavior

* Reset makes q = 0
* First clock changes q to 1
* q remains 1 afterwards

### Result

Flip-flop **cannot be removed** because it stores state.

---
### Commands

Compile

```bash
iverilog -o opt_check5_tb opt_check5.v opt_check5_tb.v
```

Run

```bash
vvp opt_check5_tb
```

Waveform

```bash
gtkwave opt_check5.vcd
```
<img width="1600" height="849" alt="image" src="https://github.com/user-attachments/assets/13f5c217-3d79-4976-a1a4-acc9ea89125e" />

### Yosys

```tcl
read_liberty -lib /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog opt_check5.v

synth -top opt_check5

opt_clean -purge

abc -liberty /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```
<img width="1600" height="851" alt="image" src="https://github.com/user-attachments/assets/ed5191f3-6215-4e2e-ab82-fe8e92a502bd" />
<img width="607" height="654" alt="image" src="https://github.com/user-attachments/assets/50ada95b-ff50-4bf2-8f97-8b7bbee346a2" />

# **Lab 6**

### Verilog

```verilog
always @(posedge clk or posedge reset)

if(reset)

q<=1;

else

q<=1;
```

Behavior

Regardless of reset or clock,

```
q = 1
```

always.
### Commands

Compile

```bash
iverilog -o opt_check6_tb opt_check6.v opt_check6_tb.v
```

Run

```bash
vvp opt_check6_tb
```

Waveform

```bash
gtkwave opt_check6.vcd
```

### Yosys

```tcl
read_liberty -lib /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog opt_check6.v

synth -top opt_check6

opt_clean -purge

abc -liberty /home/prithi/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```
<img width="1600" height="843" alt="image" src="https://github.com/user-attachments/assets/51ad32a8-672b-49f3-896a-535fc23eb47d" />

<img width="605" height="646" alt="image" src="https://github.com/user-attachments/assets/cbaa5d80-8238-4742-9c01-9de3fd94a52e" />

### Result

Yosys removes

* Flip-flop
* Clock
* Reset

Output is directly connected to

```
Logic High (VDD)
```

---

# **Issues Encountered**

### 1. Module Name Mismatch

Problem

```
module opt_check2
```

inside

```
opt_check3.v
```

Solution

Rename module to

```
module opt_check3
```

---

### 2. VCD Filename Mismatch

Problem

GTKWave couldn't find the waveform.

Solution

Use matching filenames in

```verilog
$dumpfile()
```

---

### 3. Missing Liberty File

Problem

```
sky130.lib not found
```

Solution

Provide the full absolute path.

---

# **Interview Questions**

### Difference between Constant Propagation and Constant Folding

**Constant Propagation**

Known constants replace variables.

Example

```
A=0

Y=A&B

↓

Y=0
```

---

**Constant Folding**

Compile-time arithmetic simplification.

Example

```
2+1

↓

3
```

---

### What does `opt_clean -purge` do?

Removes

* unused wires
* dead logic
* unconnected cells

---

### Why Retiming?

Instead of manually moving registers,

the synthesis tool balances delays automatically.

---

# **Common Beginner Mistakes**

* Thinking synthesized hardware exactly matches RTL.
* Forgetting `opt_clean -purge`.
* Module name mismatch.
* Incorrect top module name in `synth -top`.

---

# **Summary**

* **Combinational optimization** reduces multiplexers and redundant logic into simpler gates like **AND**, **OR**, and **XNOR**.
* **Sequential optimization** removes unnecessary flip-flops and replaces constant outputs with **VDD** or **VSS**.
* Using **Icarus Verilog**, **GTKWave**, **Yosys**, and the **Sky130** library provides a complete RTL-to-Gate-Level synthesis flow.
