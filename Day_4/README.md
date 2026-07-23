
---

# Day 4: Gate-Level Simulation (GLS) & Synthesis-Simulation Mismatch

## Objective

Learn how **Gate-Level Simulation (GLS)** compares the synthesized hardware netlist with the original RTL simulation to identify synthesis-simulation mismatches.

---

# Table of Contents

1. Lab 1: Ternary Operator Multiplexer (`mux_ternary`)
2. Lab 2: Bad MUX – Sensitivity List Mismatch (`bad_mux`)
3. Lab 3: Blocking Assignment Caveat (`blocking_caveat`)
4. Lab 4: Corrected Blocking Assignment (`blocking_caveat_correct`)
5. Summary of Key Learnings

---
### 1. Introduction to Gate-Level Simulation (GLS)
**Gate-Level Simulation (GLS)** means running a simulation using the post-synthesis gate-level netlist instead of raw Verilog code.

#### Why is GLS Important?
* **Validates Synthesis:** Confirms that Yosys translated the RTL design into correct hardware logic.
* **Catches Mismatches:** Detects coding errors (like incomplete sensitivity lists) that behavioral simulation misses.
* **Timing Checks:** Simulates logic with actual standard cell delays from cell library models.

---

### 2. RTL Design Flow
[ RTL Verilog Code ] ──> [ Functional Simulation (iVerilog) ] ──> [ Synthesis (Yosys) ] ──> [ Gate Netlist ] ──> [ GLS Simulation ]


---

### 3. Synthesis-Simulation Mismatch
A **Synthesis-Simulation Mismatch** happens when pre-synthesis RTL simulation behavior does not match post-synthesis netlist behavior.

#### Common Causes:
1. **Incomplete Sensitivity Lists:** Leaving inputs out of `always @(...)` causes simulation to freeze outputs while physical hardware continues to update.
2. **Incorrect Statement Order:** Reading variables before writing them in blocking statements introduces unwanted delays in RTL simulation.

---

### 4. Blocking (`=`) vs Non-Blocking (`<=`) Assignments

| Feature | Blocking (`=`) | Non-Blocking (`<=`) |
| :--- | :--- | :--- |
| **Execution** | Sequential (executes line-by-line instantly). | Concurrent (scheduled at the end of time step). |
| **Used For** | **Combinational Logic** (`always @(*)`) | **Sequential Logic** (`always @(posedge clk)`) |
| **Inferred Hardware** | Logic gates / temporary variables | Flip-Flops / Registers |

---
# Lab 1: Ternary Operator Multiplexer (`mux_ternary`)

## Objective

Implement a 2:1 multiplexer using the Verilog ternary (`?:`) operator and verify that RTL Simulation and Gate-Level Simulation produce identical results.

### RTL Code

```verilog
module mux_ternary (
    input i0,
    input i1,
    input sel,
    output y
);

assign y = sel ? i1 : i0;

endmodule
```

### Testbench

```verilog
module tb_mux_ternary;

reg i0, i1, sel;
wire y;

mux_ternary uut(
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin
    $dumpfile("tb_mux_ternary.vcd");
    $dumpvars(0, tb_mux_ternary);

    i0=0; i1=0; sel=0; #300;
    i0=1; i1=0; sel=0; #300;
    i0=1; i1=1; sel=1; #300;
    i0=0; i1=1; sel=1; #300;

    $finish;
end

endmodule
```

### RTL Simulation Commands

```bash
iverilog -o sim_mux_ternary.out mux_ternary.v tb_mux_ternary.v

vvp sim_mux_ternary.out

gtkwave tb_mux_ternary.vcd
```
<img width="1600" height="839" alt="image" src="https://github.com/user-attachments/assets/282b5fca-e781-44fe-a5ef-a3918f7d64e0" />

<img width="1042" height="787" alt="image" src="https://github.com/user-attachments/assets/3593765c-af94-4766-824f-7850a2b74ac2" />

### Yosys Synthesis

```bash
yosys

read_verilog mux_ternary.v

synth -top mux_ternary

abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

write_verilog -noattr mux_ternary_netlist.v

show

exit
```
<img width="1600" height="847" alt="image" src="https://github.com/user-attachments/assets/95f9dbff-5522-4c6f-8547-4ae4fc2f52b3" />
<img width="608" height="551" alt="image" src="https://github.com/user-attachments/assets/62d47c94-746c-48f1-9dd4-ede034a480b7" />


### Gate-Level Simulation

```bash
iverilog -o gls_mux_ternary.out \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
mux_ternary_netlist.v \
tb_mux_ternary.v

vvp gls_mux_ternary.out

gtkwave tb_mux_ternary.vcd
```
<img width="1600" height="846" alt="image" src="https://github.com/user-attachments/assets/c2bc4160-37be-4d6b-8bdc-c7ee2b2897f0" />


### Result

* RTL Output = GLS Output
* No mismatch observed.
* Yosys synthesizes a single `sky130_fd_sc_hd__mux2_1` cell.

---

# Lab 2: Bad MUX – Sensitivity List Mismatch (`bad_mux`)

## Objective

Demonstrate how an incomplete sensitivity list causes RTL simulation to differ from synthesized hardware.

### RTL Code

```verilog
module bad_mux(
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(sel)
begin
    if(sel)
        y=i1;
    else
        y=i0;
end

endmodule
```

### Testbench

```verilog
module tb_bad_mux;

reg i0,i1,sel;
wire y;

bad_mux uut(
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin

    $dumpfile("tb_bad_mux.vcd");
    $dumpvars(0,tb_bad_mux);

    i0=0; i1=0; sel=0; #100;
    i0=1; i1=0; sel=0; #100;
    i0=1; i1=1; sel=1; #100;
    i0=0; i1=1; sel=1; #100;

    $finish;

end

endmodule
```

### RTL Simulation

```bash
iverilog -o sim_bad_mux.out bad_mux.v tb_bad_mux.v

vvp sim_bad_mux.out

gtkwave tb_bad_mux.vcd
```
<img width="1600" height="852" alt="image" src="https://github.com/user-attachments/assets/30e9688c-99f6-46c6-afbb-ac26d9938d7a" />

### Yosys Synthesis

```bash
yosys

read_verilog bad_mux.v

synth -top bad_mux

abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

write_verilog -noattr bad_mux_netlist.v

show

exit
```
<img width="1600" height="849" alt="image" src="https://github.com/user-attachments/assets/6ba2203f-ae87-42c9-a245-d5bd6212def4" />
<img width="607" height="652" alt="image" src="https://github.com/user-attachments/assets/553696f1-8cc7-4063-a7f7-bb661fec3ee4" />



### Gate-Level Simulation

```bash
iverilog -o gls_bad_mux.out \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
bad_mux_netlist.v \
tb_bad_mux.v

vvp gls_bad_mux.out

gtkwave tb_bad_mux.vcd
```
<img width="1600" height="844" alt="image" src="https://github.com/user-attachments/assets/283a6a28-cdf7-44d7-83a9-e529d60f174c" />

### Result

RTL:

* `y` changes only when `sel` changes.

GLS:

* `y` responds immediately to `i0` and `i1`.

**Result:** Synthesis-Simulation Mismatch.

---
Lab 3: Good MUX (good_mux)
Objective

Fix the incomplete sensitivity list used in the bad_mux design by replacing it with always @(*). This ensures the RTL simulation behaves exactly like the synthesized hardware and eliminates synthesis-simulation mismatches.
````bash
RTL Code
module good_mux(
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(*)
begin
    if(sel)
        y = i1;
    else
        y = i0;
end

endmodule
````
###Testbench
````bash
module tb_good_mux;

reg i0, i1, sel;
wire y;

good_mux uut(
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin

    $dumpfile("tb_good_mux.vcd");
    $dumpvars(0, tb_good_mux);

    i0 = 0; i1 = 0; sel = 0; #100;
    i0 = 1; i1 = 0; sel = 0; #100;
    i0 = 1; i1 = 1; sel = 1; #100;
    i0 = 0; i1 = 1; sel = 1; #100;

    $finish;

end

endmodule
````
RTL Simulation
````
iverilog -o sim_good_mux.out good_mux.v tb_good_mux.v

vvp sim_good_mux.out

gtkwave tb_good_mux.vcd
````
<img width="1600" height="841" alt="image" src="https://github.com/user-attachments/assets/36fb1e67-26dd-42db-91b2-c87e9eb9432e" />


Yosys
````
yosys

read_verilog good_mux.v

synth -top good_mux

abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

write_verilog -noattr good_mux_netlist.v

show

exit
````
<img width="1600" height="839" alt="image" src="https://github.com/user-attachments/assets/46a83313-c884-427a-bb2d-451b47537e2b" />
<img width="615" height="654" alt="image" src="https://github.com/user-attachments/assets/69c109e4-31dd-4e1d-a5b8-066a2657e4e7" />

GLS
````
iverilog -o gls_good_mux.out \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
good_mux_netlist.v \
tb_good_mux.v

vvp gls_good_mux.out

gtkwave tb_good_mux.vcd
````

<img width="1600" height="844" alt="image" src="https://github.com/user-attachments/assets/d0e1a900-a097-4278-8662-cace6af7b379" />

Result

RTL and GLS produce identical outputs.

No synthesis-simulation mismatch occurs because always @(*) automatically includes all input signals in the sensitivity list, ensuring the RTL simulator accurately models the intended combinational hardware.

# Lab 4: Blocking Assignment Caveat (`blocking_caveat`)

## Objective

Understand how incorrect ordering of blocking assignments creates RTL and GLS mismatches.

### RTL Code

```verilog
module blocking_caveat(
    input a,
    input b,
    input c,
    output reg d
);

reg x;

always @(*)
begin
    d = x & c;
    x = a | b;
end

endmodule
```

### Testbench

```verilog
module tb_blocking_caveat;

reg a,b,c;
wire d;

blocking_caveat uut(
    .a(a),
    .b(b),
    .c(c),
    .d(d)
);

initial begin

    $dumpfile("tb_blocking_caveat.vcd");
    $dumpvars(0,tb_blocking_caveat);

    a=0; b=0; c=0; #100;
    a=1; b=0; c=1; #100;
    a=0; b=1; c=1; #100;

    $finish;

end

endmodule
```

### RTL Simulation

```bash
iverilog -o sim_blocking_caveat.out blocking_caveat.v tb_blocking_caveat.v

vvp sim_blocking_caveat.out

gtkwave tb_blocking_caveat.vcd
```
<img width="1600" height="848" alt="image" src="https://github.com/user-attachments/assets/c89b5d34-b2b3-4173-b2b2-ed42f9678d37" />

### Yosys

```bash
yosys

read_verilog blocking_caveat.v

synth -top blocking_caveat

abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

write_verilog -noattr blocking_caveat_netlist.v

show

exit
```
<img width="1600" height="850" alt="image" src="https://github.com/user-attachments/assets/cc417e60-1cab-4ba6-94fc-84ed2dd782e6" />


### GLS

```bash
iverilog -o gls_blocking_caveat.out \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
blocking_caveat_netlist.v \
tb_blocking_caveat.v

vvp gls_blocking_caveat.out

gtkwave tb_blocking_caveat.vcd
```
<img width="1600" height="851" alt="image" src="https://github.com/user-attachments/assets/e07306a9-a892-4ac1-8efa-8985dfe03304" />


### Result

RTL:

* `d` uses the old value of `x`.

GLS:

* Hardware simplifies logic to:

```text
d = (a | b) & c
```

Mismatch occurs.

---

# Lab 5: Corrected Blocking Assignment (`blocking_caveat_correct`)

## Objective

Fix the statement ordering and verify that RTL and GLS now match.

### RTL Code

```verilog
module blocking_caveat_correct(
    input a,
    input b,
    input c,
    output reg d
);

reg x;

always @(*)
begin
    x = a | b;
    d = x & c;
end

endmodule
```

### Testbench

```verilog
module tb_blocking_caveat_correct;

reg a,b,c;
wire d;

blocking_caveat_correct uut(
    .a(a),
    .b(b),
    .c(c),
    .d(d)
);

initial begin

    $dumpfile("blocking_caveat_correct.vcd");
    $dumpvars(0,tb_blocking_caveat_correct);

    a=0; b=0; c=0; #100;
    a=1; b=0; c=1; #100;
    a=0; b=1; c=1; #100;

    $finish;

end

endmodule
```

### RTL Simulation

```bash
iverilog -o sim_blocking_caveat_correct.out blocking_caveat_correct.v tb_blocking_caveat_correct.v

vvp sim_blocking_caveat_correct.out

gtkwave blocking_caveat_correct.vcd
```
<img width="1600" height="839" alt="image" src="https://github.com/user-attachments/assets/9a8a31ea-4615-4145-b89d-a0ec3df80ed9" />



### Yosys

```bash
yosys

read_verilog blocking_caveat_correct.v

synth -top blocking_caveat_correct

abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

write_verilog -noattr blocking_caveat_correct_netlist.v

show

exit
```
<img width="601" height="640" alt="image" src="https://github.com/user-attachments/assets/58139464-7c92-4dcf-b421-87a5db65a364" />


<img width="1600" height="846" alt="image" src="https://github.com/user-attachments/assets/a4bef5d3-5be2-4458-8a46-3bfcb8108806" />


### GLS

```bash
iverilog -o gls_blocking_caveat_correct.out \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
blocking_caveat_correct_netlist.v \
tb_blocking_caveat_correct.v

vvp gls_blocking_caveat_correct.out

gtkwave blocking_caveat_correct.vcd
```
<img width="1600" height="849" alt="image" src="https://github.com/user-attachments/assets/348f6f06-070d-4040-8247-b7d7ff4a8dfd" />



### Result

RTL and GLS produce identical outputs.

No mismatch occurs.

---

# Summary of Key Learnings

* Gate-Level Simulation (GLS) validates the synthesized hardware against the RTL design.
* Behavioral simulation alone is not enough to guarantee correct hardware behavior.
* Always use `always @(*)` (or `always_comb` in SystemVerilog) for combinational logic to avoid sensitivity list issues.
* Incorrect sensitivity lists can cause synthesis-simulation mismatches.
* The order of blocking (`=`) assignments matters in procedural blocks.
* Compute intermediate variables before using them.
* Comparing RTL and GLS waveforms is essential for detecting hidden design bugs before hardware implementation.
