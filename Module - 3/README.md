# Module 3 – Combinational and Sequential Optimizations

## Overview

This module focuses on **optimization techniques used during digital synthesis** to simplify RTL designs and generate efficient hardware.

The main goal of optimization is to reduce unnecessary logic while improving:

* **Area**
* **Power**
* **Performance**

The module covers both **combinational** and **sequential** optimization using **Verilog RTL and Yosys**.

---

## 1. Introduction to Optimization

Optimization in digital design involves simplifying the logic required to implement a given functionality.

An optimized design can achieve:

* Reduced hardware area
* Lower power consumption
* Better performance
* Fewer gates and transistors
* Reduced unnecessary switching activity

Optimization can be broadly classified into:

1. **Combinational logic optimization**
2. **Sequential logic optimization**

---

# 2. Combinational Logic Optimization

Combinational optimization simplifies logic that depends only on the current inputs.

### Objectives

The main objective is to obtain the same functionality with simpler hardware.

### Benefits

* Reduced area
* Reduced power consumption
* Improved performance
* Fewer gates and transistors

### Common Techniques

* Constant propagation
* Boolean logic optimization
* K-Map simplification
* Quine–McCluskey method

---

## 2.1 Constant Propagation

Constant propagation is an optimization technique in which known constant values such as `0` or `1` are propagated through the logic.

### Example

Consider:

```text
Y = AB + C̅
```

If:

```text
A = 0
```

then:

```text
Y = 0B + C̅
```

Therefore:

```text
Y = C̅
```

The original logic can now be replaced by a simple inverter:

```text
C → NOT → Y
```

This eliminates unnecessary logic and can reduce both **area and power**.

---

## 2.2 Boolean Logic Optimization

Boolean expressions can be simplified using algebraic rules and logic minimization techniques.

Common approaches include:

* Boolean algebra
* K-Map
* Quine–McCluskey method
* Multiplexer-based simplification

For example, a 2:1 multiplexer can be represented as:

```text
Y = S̅I0 + SI1
```

Simplifying the expression can remove redundant logic and produce a smaller implementation.

---

# 3. Sequential Logic Optimization

Sequential optimization focuses on circuits that contain storage elements such as:

* Flip-flops
* Registers
* Counters
* State machines

The objective is to eliminate unnecessary sequential logic while preserving the required behavior.

### Techniques

### Basic Technique

* Sequential constant propagation

### Advanced Techniques

* State optimization
* Retiming
* Sequential logic cloning
* Floorplan-based optimization
* Synthesis-based optimization

---

# 4. Sequential Constant Propagation

Sequential constant propagation identifies flip-flops whose outputs can be proven to always have a constant value.

If a flip-flop output is always `0` or always `1`, the flip-flop and related logic may be simplified or removed.

## Example

Suppose:

```text
Q = 0
```

and the output logic is:

```text
Y = A · Q̅
```

Substituting `Q = 0`:

```text
Y = A · 0̅
```

Therefore:

```text
Y = 1
```

The output becomes a constant and no longer depends on the input logic.

---

## Set Flip-Flop Example

A flip-flop with a **SET** input does not automatically become a sequential constant.

For example:

* `SET = 1` → `Q = 1`
* `SET = 0` → `Q` depends on the data input and clock

Therefore, the important condition is whether the output `Q` can be proven to **always remain at a constant value**.

---

# 5. State Optimization

State optimization reduces unnecessary or unused states in sequential circuits.

It can simplify the state representation and reduce the hardware required to implement the state machine.

### Benefits

* Fewer flip-flops
* Reduced area
* Reduced power
* Simpler state logic
* Improved performance

---

# 6. State Optimization – Multiplexer Examples

The following examples demonstrate how Yosys can simplify multiplexer-based combinational logic.

---

## Example 1 – `opt_check1.v`

### Verilog Code

```verilog
module opt_check1(input a, input b, output y);
    assign y = a ? b : 0;
endmodule
```

The multiplexer has:

* **Select:** `a`
* **Input 0:** `0`
* **Input 1:** `b`

The expression can be written as:

```text
Y = a̅(0) + ab
```

Therefore:

```text
Y = ab
```

### Yosys Commands

```text
yosys
read_liberty
read_verilog opt_check1.v
synth -top opt_check1
opt_clean -purge
abc -liberty
show
```

---

## Example 2 – `opt_check2.v`

### Verilog Code

```verilog
module opt_check2(input a, input b, output y);
    assign y = a ? 0 : b;
endmodule
```

The multiplexer has:

* **Select:** `a`
* **Input 0:** `b`
* **Input 1:** `0`

Therefore:

```text
Y = a̅b + a(0)
```

which simplifies to:

```text
Y = a̅b
```

### Yosys Commands

```text
yosys
read_liberty
read_verilog opt_check2.v
synth -top opt_check2
opt_clean -purge
abc -liberty
show
```

---

## Example 3 – `opt_check3.v`

### Verilog Code

```verilog
module opt_check3(input a, input b, input c, output y);
    assign y = a ? (c ? b : 0) : 0;
endmodule
```

This design contains two multiplexers.

The logic can be expanded and simplified to:

```text
Y = abc
```

Thus, the original multiplexer structure can be reduced to equivalent simpler combinational logic.

### Yosys Commands

```text
yosys
read_verilog opt_check3.v
synth -top opt_check3
opt_clean -purge
abc -liberty
show
```

---

# 7. Sequential Optimization – D Flip-Flop Constant Propagation

D flip-flops can be used to demonstrate sequential constant propagation.

### Typical Files

```text
dff_const1.v
dff_const2.v
dff_const3.v
tb_dff_const1.v
```

The testbench files are used to simulate the corresponding designs and observe their behavior.

### Simulation Commands

```text
iverilog dff_const1.v tb_dff_const1.v
./a.out
gtkwave tb_dff_const1.vcd
```

Similarly:

```text
iverilog dff_const3.v tb_dff_const3.v
./a.out
gtkwave tb_dff_const3.vcd
```

These simulations help verify the sequential behavior before and after optimization.

---

# 8. Editing and Copying Files

The Verilog source files can be edited and duplicated using commands such as:

```text
gvim dff_const1.v
cp dff_const1.v dff_const2.v
gvim dff_const2.v
```

This allows different optimization cases to be created and tested while preserving the original design.

---

# 9. Yosys Flow for Sequential Optimization

A typical sequential optimization flow is:

```text
yosys
read_liberty
read_verilog dff_const2.v
synth -top <top_module>
opt_clean -purge
abc -liberty
show
```

The synthesized result can then be compared with the original circuit to determine whether constant or redundant sequential logic has been removed.

---

# 10. Sequential Optimization – Unused Outputs

Optimization can also remove unnecessary counter bits or sequential logic that does not affect any observable output.

## Example – Counter

```verilog
module counter_opt(input clk, input reset, output q);

    reg [2:0] count;

    assign q = count[0];

    always @(posedge clk, posedge reset) begin
        if (reset)
            count <= 3'b000;
        else
            count <= count + 1;
    end

endmodule
```

The counter contains:

```text
count[2:0]
```

but only:

```text
count[0]
```

is connected to the output.

Depending on the synthesis and optimization context, the upper bits may be identified as unnecessary logic if they do not affect any required functionality.

---

# 11. Counter Optimization

Counter optimization identifies counter bits or sequential elements that are not required for the observable behavior of the design.

For example:

```text
count[2:0] = 3'b100
```

can be analyzed to determine whether particular counter states or bits can be simplified.

### Main Idea

If certain counter bits do not affect any observable output or required functionality, the synthesis tool may identify them as unnecessary and optimize them.

### Benefits

* Reduced number of flip-flops
* Reduced area
* Reduced power consumption
* Reduced switching activity
* Simplified sequential logic

---

# 12. Yosys Flow for Counter Optimization

A typical flow is:

```text
yosys
read_liberty
read_verilog counter_opt2.v
synth -top <top_module>
dfflibmap
abc -liberty
show
```

The synthesized circuit can then be compared with the original RTL to observe the effect of optimization.

---

# 13. Important Optimization Commands

| Command            | Purpose                                         |
| ------------------ | ----------------------------------------------- |
| `read_verilog`     | Reads the Verilog RTL design                    |
| `read_liberty`     | Reads the standard-cell timing library          |
| `synth -top`       | Performs synthesis for the specified top module |
| `opt_clean -purge` | Removes unused and unnecessary logic            |
| `dfflibmap`        | Maps flip-flops to library cells                |
| `abc -liberty`     | Performs technology mapping using ABC           |
| `show`             | Displays the synthesized circuit                |
| `iverilog`         | Compiles Verilog for simulation                 |
| `./a.out`          | Runs the compiled simulation                    |
| `gtkwave`          | Opens the generated waveform                    |
| `gvim`             | Opens and edits Verilog files                   |

---

# 14. Quick Revision

## Combinational Optimization

### Main Techniques

1. Constant propagation
2. Boolean logic optimization
3. K-Map
4. Quine–McCluskey method

### Main Goal

```text
Less Area + Less Power + Better Performance
```

---

## Sequential Optimization

### Basic Technique

* Sequential constant propagation

### Advanced Techniques

* State optimization
* Retiming
* Sequential logic cloning
* Floorplan-based optimization
* Synthesis-based optimization

### Important Concept

If a sequential element can be proven to always produce a constant output, it may be removed or simplified during synthesis.

---

# 15. Overall Optimization Flow

The general optimization flow covered in this module is:

```text
Verilog RTL
    ↓
Read Design
    ↓
Synthesis
    ↓
Constant Propagation
    ↓
Optimization / Cleanup
    ↓
Technology Mapping
    ↓
ABC
    ↓
Show / Analyze Optimized Circuit
```

### Key Commands

```text
read_liberty
read_verilog
synth -top
opt_clean -purge
dfflibmap
abc -liberty
show
```

---
## Conclusion

This module provided an understanding of **combinational and sequential optimization techniques** used in digital VLSI design. Constant propagation, Boolean logic simplification, state optimization, and removal of unused sequential logic help reduce unnecessary hardware while maintaining the required functionality.

Using **Verilog RTL, Icarus Verilog, GTKWave, and Yosys**, the optimization process can be simulated, synthesized, and analyzed. These techniques contribute to achieving **smaller area, lower power consumption, reduced switching activity, and improved circuit performance**.

Overall, optimization plays an important role in transforming RTL designs into **efficient and practical hardware implementations**.
