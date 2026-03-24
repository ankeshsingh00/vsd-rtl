# SKY130 RTL Design and Synthesis Workshop

## Day 1 - Introduction to Verilog RTL Design and Synthesis

### Theory
- **Simulator**: Tool used for simulating RTL design (iverilog used in this course)
- **Design**: Actual Verilog code with intended functionality to meet required specifications
- **Testbench**: Setup to apply stimulus (test vectors) to the design to check its functionality

### How Simulator Works
- Simulator looks for changes in input signals
- Upon change in input, output is evaluated
- If no change in input, no change in output

### Iverilog Based Simulation Flow
```
Design + Testbench → iverilog → VCD File → GTKWave
```

### Lab - Simulation Flow
```bash
# Step 1: Compile
iverilog good_mux.v tb_good_mux.v

# Step 2: Simulate (generates .vcd file)
./a.out

# Step 3: View waveform
gtkwave tb_good_mux.vcd
```

### Introduction to Yosys and Logic Synthesis
- **Synthesizer**: Tool used for converting RTL to netlist (Yosys used in this course)
- **Netlist**: Gate level representation of the design

### What is .lib?
- Collection of logical modules
- Includes basic logic gates like AND, OR, NOT etc.
- Different flavors of same gate (slow, medium, fast)
- Faster cells → more area and power
- Slower cells → less area and power

### Lab - Synthesis Flow (Yosys)
```bash
# Step 1: Open Yosys
yosys

# Step 2: Read liberty file
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Step 3: Read Verilog design
read_verilog good_mux.v

# Step 4: Synthesize
synth -top good_mux

# Step 5: Technology mapping
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Step 6: View schematic
show

# Step 7: Write netlist
write_verilog -noattr good_mux_netlist.v

# Step 8: View netlist
!gvim good_mux_netlist.v
```

---

## Day 2 - Timing Libs, Hierarchical vs Flat Synthesis and Efficient Flop Coding Styles

### Theory
- **lib file name**: sky130_fd_sc_hd__tt_025C_1v80
  - tt → typical process
  - 025C → temperature
  - 1v80 → voltage
- **PVT (Process Voltage Temperature)**: Variations due to fabrication - we want to design circuits that work in all conditions

### How to Read lib File
```bash
# Open lib file
gvim ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Turn off syntax highlighting inside gvim
:syntax off
```
- **cell** → start of cell definition
- **:se nu** → show line numbers
- **sp** → open verilog model file

### Hierarchical vs Flat Synthesis

#### Hierarchical Synthesis
```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show multiple_module
write_verilog multiple_modules_hier.v
!gvim multiple_modules_hier.v
write_verilog -noattr multiple_modules_hier.v
```

#### Flat Synthesis
```bash
yosys
flatten
show
write_verilog multiple_modules_flat.v
!gvim multiple_modules_flat.v
write_verilog -noattr multiple_modules_flat.v
```

### Why Use Module Level Synthesis?
1. If we need multiple instances of same module
2. Divide and conquer approach

### Various Flop Coding Styles

#### Why Flops?
- To avoid glitches caused by propagation delay in combinational circuits
- Flop output is stable and changes only at clock edge

#### D Flip Flop - Async Reset
```verilog
module dff_asyncres(input clk, input async_reset, output reg q);
always @(posedge clk, posedge async_reset)
begin
  if(async_reset)
    q <= 1'b0;
  else
    q <= d;
end
endmodule
```

#### D Flip Flop - Sync Reset
```verilog
always @(posedge clk)
begin
  if(sync_reset)
    q <= 1'b0;
  else
    q <= d;
end
```

### Lab - Flop Synthesis
```bash
# Simulation
iverilog dff_asyncres.v tb_dff_asyncres.v
./a.out
gtkwave tb_dff_asyncres.vcd

# Synthesis in Yosys
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### Interesting Optimisations
```bash
# mult2 - multiply by 2 (no hardware needed)
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mult2.v
synth -top mult2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr mul2_net.v
!gvim mul2_net.v
```

---

## Day 3 - Combinational and Sequential Optimizations

### Combinational Logic Optimisation
- **Squeezing logic** to get most optimized design (area and power savings)
- **Constant Propagation** - Direct optimisation
- **Boolean Logic Optimisation** - K-Map, Quine McCluskey

#### Example: Constant Propagation
```
Y = ((AB) + C)' → if A=0, Y = C'
```

#### Boolean Logic Optimisation Example
```
assign Y = a?(b?c:(c?a:0)):(!c)
→ Y = a'c' + ac + a'c + abc → Y = a⊕c
```

### Sequential Logic Optimisation
- Sequential Constant Propagation
- State Optimisation
- Retiming
- Sequential Logic Cloning

### Lab - Combinational Logic Optimisations
```bash
# In verilog_files
ls *opt*
ls *opt_check*

# Synthesis steps
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check.v
synth -top opt_check
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
# Repeat for opt_check2, opt_check3
```

### Lab - Sequential Logic Optimisations
```bash
# dff_const1
iverilog dff_const1.v tb_dff_const1.v
./a.out
gtkwave tb_dff_const1.vcd

# Synthesis
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const1.v
synth -top dff_const1
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
# Repeat for dff_const2, dff_const3
```

### Sequential Optimisation for Unused Outputs
```bash
# counter_opt - 3bit up counter
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog counter_opt.v
synth -top counter_opt
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
# 1 flop shown for 3-bit counter (unused outputs optimised away)
```

---

## Day 4 - GLS, Blocking vs Non-Blocking and Synthesis-Simulation Mismatch

### What is GLS (Gate Level Simulation)?
- Running testbench with netlist as design under test
- Netlist is logically same as RTL code
- Same testbench will work

### Why GLS?
- Verify logical correctness of design after synthesis
- Ensure timing of design is met

### GLS Flow using iverilog
```
Design + Gate Level Verilog + Test Bench → iverilog → VCD File → GTKWave
```

### Synthesis-Simulation Mismatch Causes
1. **Missing Sensitivity List**
2. **Blocking vs Non-Blocking Assignments**
3. **Non Standard Verilog Coding**

### Blocking vs Non-Blocking Statements
- **Blocking (=)**: Executes statements in order written
- **Non-Blocking (<=)**: Executes all RHS when always block is entered, then assigns to LHS (parallel evaluation)

### Lab - GLS and Synth Sim Mismatch
```bash
# Simulation
gvim ternary_operator_mux.v
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd

# Synthesis
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr ternary_operator_mux_net.v
show

# GLS
iverilog -lblibrary- ../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
ternary_operator_mux_net.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

### Lab - Synth Sim Mismatch Blocking Statement
```bash
# Synthesis
yosys
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr blocking_caveat_net.v
show

# GLS
iverilog ../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
blocking_caveat_net.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
# Shows synthesis simulation mismatch!
```

---

## Day 5 - Optimization in Synthesis - If Case Constructs

### If Case Constructs
- **if** statement → mainly used to create priority logic
- **Danger with incomplete if** → Inferred Latches (bad coding style)
- Always have an **else** statement to avoid inferred latches

### Case Statement
- Used inside always block
- With default → avoids inferred latches
- **Caveats with case**:
  - Incomplete case → Inferred latches
  - Partial assignments → Latching action

### Lab - Incomplete IF
```bash
ls *incomp*
gvim incomp_if.v
iverilog incomp_if.v tb_incomp_if.v
./a.out
gtkwave tb_incomp_if.vcd

# Synthesis
yosys
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
# Shows inferred latch!
```

### Lab - Incomplete Overlapping Case
```bash
# Synthesis
yosys
read_verilog incomp_case.v
synth -top incomp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### For Loop and For Generate

#### Difference
| For Loop | For Generate |
|----------|-------------|
| Inside always block | Outside always block |
| Multiple value evaluation | Hardware instantiation |

#### For Loop Example - MUX
```verilog
always @(*)
begin
  for(k=0; k<4; k=k+1) begin
    if(k == sel)
      y = i_int[k];
  end
end
```

#### For Generate Example - Ripple Carry Adder
```verilog
genvar i;
generate
  for(i=1; i<8; i=i+1) begin
    fa u_fa(.a(num1[i]), .b(num2[i]),
            .c(int_co[i-1]), .co(int_co[i]),
            .sum(int_sum[i]));
  end
endgenerate
```

### Lab - For and For Generate
```bash
# MUX using for loop
iverilog mux_generate.v tb_mux_generate.v
./a.out
gtkwave tb_mux_generate.vcd

# Synthesis
yosys
read_verilog mux_generate.v
synth -top mux_generate
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
# Does for 256:1!

# RCA using for generate
iverilog rca.v fa.v tb_rca.v
./a.out
gtkwave tb_rca.vcd
```

---

## Key Concepts Summary

| Concept | Description |
|---------|-------------|
| RTL | Register Transfer Level Verilog design |
| Testbench | File to test design functionality |
| .vcd file | Value Change Dump - viewed in GTKWave |
| .lib file | Standard cell library |
| Synthesis | RTL to gate level translation |
| Netlist | Gate level output after synthesis |
| GLS | Gate Level Simulation |
| Inferred Latch | Created due to incomplete if/case |
| Blocking (=) | Sequential execution |
| Non-Blocking (<=) | Parallel execution |
| sky130 | Google/Skywater 130nm technology |

## Tools Used
- **iverilog** - Verilog compiler and simulator
- **GTKWave** - Waveform viewer
- **Yosys** - Open source synthesis tool
- **gvim** - Text editor
- **sky130 PDK** - Standard cell library