
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
<img width="1600" height="839" alt="image" src="https://github.com/user-attachments/assets/907701f3-62a6-4aca-bfba-c59be1965f45" />
<img width="1042" height="787" alt="image" src="https://github.com/user-attachments/assets/da23d2c7-956f-4acf-af5b-557a718942b5" />

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
<img width="608" height="551" alt="image" src="https://github.com/user-attachments/assets/948dddc8-869e-437e-8925-c76dbae116b9" />
<img width="1600" height="846" alt="image" src="https://github.com/user-attachments/assets/9b6c62de-d955-4e9e-8dbf-9efd45d8a68c" />

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
<img width="1600" height="847" alt="image" src="https://github.com/user-attachments/assets/7a781eae-a99f-4c94-9b0e-d7f50311c2d8" />

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
<img width="1600" height="849" alt="image" src="https://github.com/user-attachments/assets/0c7a2da9-88fc-46b6-865b-262d85682006" />

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
<img width="607" height="652" alt="image" src="https://github.com/user-attachments/assets/e65da272-f565-4169-be34-749aabbd9a7f" />


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
<img width="1600" height="844" alt="image" src="https://github.com/user-attachments/assets/843b4048-8cd3-4e2d-8c59-7af2275c10f3" />

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
<img width="1600" height="844" alt="image" src="https://github.com/user-attachments/assets/7826b2e2-8853-4985-b098-e7274d95b084" />

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
<img width="1600" height="839" alt="image" src="https://github.com/user-attachments/assets/95bf1f18-3e48-455e-a119-69bf2ccf6369" />
<img width="615" height="654" alt="image" src="https://github.com/user-attachments/assets/89422728-4ba4-4c1c-8e80-c4b3572fb4d2" />

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
<img width="1600" height="848" alt="image" src="https://github.com/user-attachments/assets/b1fd04fe-64ef-49d1-aecb-b38a52465a5f" />

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
<img width="1600" height="850" alt="image" src="https://github.com/user-attachments/assets/4f9a01df-a35d-41b2-9c07-ecb9b1fcccd6" />

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

<img width="601" height="640" alt="image" src="https://github.com/user-attachments/assets/d6ccb53c-d0e8-4756-b45e-75de50b02ec2" />

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

<img width="1600" height="839" alt="image" src="https://github.com/user-attachments/assets/cf50a6a6-ffbe-4dc8-acf2-96c1c8ec7e00" />




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
<img width="1600" height="849" alt="image" src="https://github.com/user-attachments/assets/085cb4e1-a5e9-4cd8-9f4d-27412e1b94cb" />


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
