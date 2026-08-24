# Module 4 — GLS, Blocking vs Non-Blocking Assignments & Synthesis-Simulation Mismatch

## 📌 Overview

This module focuses on **Gate-Level Simulation (GLS)**, the correct use of **blocking (`=`)** and **non-blocking (`<=`) assignments**, and the causes of **synthesis-simulation mismatch** in Verilog designs.

The practical work covers RTL simulation, synthesis using **Yosys**, generation of gate-level netlists, GLS using **Icarus Verilog**, and waveform analysis using **GTKWave**. It also demonstrates how incomplete sensitivity lists and improper assignment styles can cause simulation behavior to differ from the synthesized hardware.

---

## 1. Gate-Level Simulation (GLS)

**Gate-Level Simulation (GLS)** verifies the synthesized gate-level netlist using a testbench instead of simulating the original RTL directly.

### Basic Flow

```text
RTL Design + Testbench
          ↓
    RTL Simulation
          ↓
       Synthesis
          ↓
   Gate-Level Netlist
          ↓
 Gate-Level Simulation
```

The same testbench can be used to compare the behavior of the RTL design with the synthesized implementation.

### Why GLS?

GLS helps to:

* Verify the functionality of the synthesized design.
* Confirm that synthesis preserves the intended behavior.
* Identify synthesis-simulation mismatches.
* Verify gate-level implementation.
* Perform timing-related verification when delay information is available.

> **Note:** Timing-aware GLS requires delay-annotated gate-level models.

---

## 2. RTL vs Gate-Level Netlist

### RTL Description

RTL describes the intended hardware behavior using constructs such as:

* `always`
* `assign`
* `if-else`
* `case`
* Logical operators
* Arithmetic operators

Example:

```verilog
assign y = (a & b) | c;
```

### Gate-Level Netlist

After synthesis, the RTL is converted into gates or technology-specific standard cells.

```text
a ───┐
     AND ───┐
b ───┘      │
            OR ─── y
c ──────────┘
```

The resulting netlist represents the hardware at the gate or cell level rather than using high-level RTL constructs.

---

## 3. GLS Using Icarus Verilog

A basic GLS flow using **Icarus Verilog** is:

```text
              RTL Design
                  ↓
               Yosys
                  ↓
         Gate-Level Netlist
                  ↓
             Icarus Verilog
                  ↑
              Testbench
                  ↓
                 VCD
                  ↓
              GTKWave
```

### Simulation Flow

```text
Gate-Level Netlist
        +
    Testbench
        ↓
     IVerilog
        ↓
       VCD
        ↓
    GTKWave
```

The generated waveform can then be compared with the RTL simulation waveform.

---

## 4. Example: 2:1 MUX

A simple combinational MUX can be described as:

```verilog
assign y = (a & b) | c;
```

The corresponding gate-level representation is:

```text
a ───┐
     AND ───┐
b ───┘      │
            OR ─── y
c ──────────┘
```

The same verification methodology can be applied to larger combinational and sequential circuits.

---

## 5. Synthesis-Simulation Mismatch

A **synthesis-simulation mismatch** occurs when the behavior observed during RTL simulation differs from the behavior of the synthesized hardware.

```text
RTL Simulation
      ≠
Gate-Level Simulation
```

Common causes include:

* Incomplete sensitivity lists.
* Incorrect use of blocking assignments.
* Incorrect use of non-blocking assignments.
* Improper coding of sequential logic.
* RTL constructs that do not accurately represent the intended hardware.

---

## 6. Incomplete Sensitivity Lists

Consider the following MUX:

```verilog
module mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(sel)
begin
    if (sel)
        y = i1;
    else
        y = i0;
end

endmodule
```

The sensitivity list contains only:

```verilog
@(sel)
```

However, the output depends on:

```text
sel
i0
i1
```

Therefore, changes in `i0` or `i1` may not trigger the `always` block.

### Example

Suppose:

```text
sel = 0
i0  = 0
i1  = 1
```

Then:

```text
y = i0 = 0
```

If `i0` changes from `0` to `1` while `sel` remains `0`, the block may not execute, causing `y` to remain incorrectly at `0` in simulation.

### Correct Approach

Use:

```verilog
always @(*)
begin
    if (sel)
        y = i1;
    else
        y = i0;
end
```

`@(*)` automatically includes the signals required by the combinational block.

---

## 7. `always @(sel)` vs `always @(*)`

| Feature                     | `always @(sel)` | `always @(*)` |
| --------------------------- | --------------- | ------------- |
| Executes when `sel` changes | Yes             | Yes           |
| Executes when `i0` changes  | No              | Yes           |
| Executes when `i1` changes  | No              | Yes           |
| Suitable for this MUX       | No              | Yes           |
| Simulation mismatch risk    | Higher          | Lower         |

### Rule

For combinational procedural logic:

```verilog
always @(*)
```

is preferred over an incomplete sensitivity list.

---

## 8. Blocking Assignments

The blocking assignment operator is:

```verilog
=
```

Example:

```verilog
q0 = d;
q  = q0;
```

Blocking assignments execute immediately and sequentially.

Conceptually:

```text
Statement 1
    ↓
Statement 2
    ↓
Statement 3
```

The next statement sees the value assigned by the previous statement.

### Typical Use

Blocking assignments are normally used for:

* Combinational logic.
* Intermediate calculations inside combinational blocks.
* Procedural combinational descriptions.

---

## 9. Non-Blocking Assignments

The non-blocking assignment operator is:

```verilog
<=
```

Example:

```verilog
q0 <= d;
q  <= q0;
```

With non-blocking assignments:

1. Right-hand-side values are evaluated.
2. Left-hand-side updates are scheduled.
3. Updates occur after the current procedural evaluation.
4. Multiple registers can model parallel clocked behavior.

Conceptually:

```text
Evaluate RHS values
        ↓
Schedule updates
        ↓
Update registers
```

This behavior makes non-blocking assignments suitable for sequential logic.

---

## 10. Blocking vs Non-Blocking

| Blocking `=`                            | Non-Blocking `<=`                                                |
| --------------------------------------- | ---------------------------------------------------------------- |
| Executes immediately                    | Update is scheduled                                              |
| Procedural order matters                | Models parallel register updates                                 |
| Later statements can see updated values | Later statements see previous values during the same clock event |
| Commonly used for combinational logic   | Commonly used for sequential logic                               |
| Can cause problems in clocked logic     | Preferred for flip-flop/register modeling                        |

### General Rule

```text
Combinational Logic → Blocking `=`

Sequential Logic → Non-Blocking `<=`
```

---

## 11. Blocking Assignment in Sequential Logic

Consider two flip-flops connected in series:

```text
d ───► FF1 ───► FF2 ───► q
       q0
```

The intended behavior is:

```text
q0 ← d
q  ← old q0
```

A problematic implementation is:

```verilog
always @(posedge clk)
begin
    q0 = d;
    q  = q0;
end
```

Because blocking assignments execute immediately:

```text
q0 = d
   ↓
q = q0
```

The second statement sees the newly updated value of `q0`.

This can make the simulation appear as though the data passes through both registers during the same clock event.

---

## 12. Correct Sequential Coding

The preferred implementation is:

```verilog
always @(posedge clk)
begin
    q0 <= d;
    q  <= q0;
end
```

Now both RHS values are evaluated before the register updates.

```text
Clock 1:
q0 gets d
q gets old q0

Clock 2:
q0 gets new d
q gets previous q0
```

This correctly represents two cascaded flip-flops.

### Recommended Style

```text
Flip-Flops
Registers
Counters
State Machines
Clocked Logic
        ↓
Use <=
```

---

## 13. Why Blocking Assignments Can Cause Mismatch

Consider:

```verilog
always @(posedge clk)
begin
    q0 = d;
    q  = q0;
end
```

### RTL Simulation

The simulator follows procedural ordering:

```text
q0 = d
 ↓
q = q0
```

Therefore, `q` can receive the newly assigned value of `q0`.

### Synthesized Hardware

Synthesis interprets the clocked assignments as sequential hardware and may infer:

```text
d → FF(q0) → FF(q)
```

The two flip-flops naturally operate on successive clock events.

Thus, the simulated behavior can differ from the intended hardware behavior.

> This is why non-blocking assignments are the standard coding style for sequential logic.

---

## 14. Order Dependence of Blocking Assignments

Consider:

```verilog
always @(posedge clk)
begin
    q0 = a | b;
    y  = q0 & c;
end
```

The simulator executes:

```text
1. q0 = a | b
2. y  = q0 & c
```

So `y` uses the new value of `q0`.

If the statements are reversed:

```verilog
always @(posedge clk)
begin
    y  = q0 & c;
    q0 = a | b;
end
```

`y` uses the previous value of `q0`.

Therefore:

```text
Statement Order
      ↓
Different Simulation Behavior
```

This order dependence is undesirable when modeling sequential hardware.

---

## 15. Blocking Assignments for Combinational Logic

Blocking assignments are appropriate for combinational procedural logic.

Example:

```verilog
always @(*)
begin
    if (sel)
        y = i1;
    else
        y = i0;
end
```

The behavior is:

```text
Input changes
      ↓
Combinational block executes
      ↓
Output is calculated
```

Therefore:

```text
Combinational → always @(*) + =
Sequential    → always @(posedge clk) + <=
```

---

## 16. Ternary Operator MUX

A 2:1 MUX can also be written using the ternary operator:

```verilog
assign y = sel ? i1 : i0;
```

Meaning:

```text
sel = 0 → y = i0
sel = 1 → y = i1
```

Equivalent procedural description:

```verilog
always @(*)
begin
    if (sel)
        y = i1;
    else
        y = i0;
end
```

Both describe the same 2:1 MUX.

### MUX Representation

```text
        ┌─────┐
i0 ────►│     │
        │ MUX │────► y
i1 ────►│     │
        └──┬──┘
           │
          sel
```

---

## 17. Ternary MUX Implementation

Example RTL:

```verilog
module ternary_operator_mux (
    input  i0,
    input  i1,
    input  sel,
    output y
);

assign y = sel ? i1 : i0;

endmodule
```

### Expected Behavior

```text
sel = 0 → y = i0
sel = 1 → y = i1
```

A testbench can apply different combinations of `i0`, `i1`, and `sel` and verify the output.

---

## 18. RTL Simulation Using Icarus Verilog

The basic simulation flow is:

```text
Verilog Design
      +
Testbench
      ↓
Icarus Verilog
      ↓
Simulation / VCD
      ↓
GTKWave
```

### Compile

```bash
iverilog -o a.out ternary_operator_mux.v tb_ternary_operator_mux.v
```

### Run

```bash
./a.out
```

### View Waveform

```bash
gtkwave tb_ternary_operator_mux.vcd
```

The waveform can be used to verify that the MUX output follows the selected input.

---

## 19. Yosys Synthesis Flow

**Yosys** can synthesize the RTL and generate a gate-level netlist.

Typical commands are:

```text
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty <library>.lib
write_verilog -noattr ternary_operator_mux.net.v
show
```

### Synthesis Flow

```text
Read RTL
   ↓
Synthesis
   ↓
Technology Mapping
   ↓
Gate-Level Netlist
   ↓
View Synthesized Circuit
```

---

## 20. Gate-Level Netlist and GLS

After synthesis, a gate-level netlist such as:

```text
ternary_operator_mux.net.v
```

can be simulated using the same testbench.

### GLS Flow

```text
RTL
 ↓
Yosys
 ↓
Gate-Level Netlist
 ↓
Icarus Verilog
 ↓
VCD
 ↓
GTKWave
```

Using the same testbench makes it easier to compare RTL and synthesized behavior.

---

## 21. GLS with Gate-Level Libraries

When the synthesized netlist contains standard-cell library instances, the corresponding Verilog library models must also be available during simulation.

A typical setup may contain:

```text
my_lib/
└── verilog_model/
    ├── primitives.v
    └── sky130_fd_sc_hd.v

gate-level-netlist.v
testbench.v
```

Representative compilation:

```bash
iverilog \
  my_lib/verilog_model/primitives.v \
  my_lib/verilog_model/sky130_fd_sc_hd.v \
  gate-level-netlist.v \
  testbench.v
```

Run the simulation:

```bash
./a.out
```

View the waveform:

```bash
gtkwave testbench.vcd
```

---

## 22. Blocking-Statement GLS Experiment

This experiment demonstrates how improper blocking assignments in sequential logic can contribute to a synthesis-simulation mismatch.

### RTL Simulation

```bash
iverilog blocking_caveat.v tb_blocking_caveat.v
```

Run:

```bash
./a.out
```

View:

```bash
gtkwave tb_blocking_caveat.vcd
```

### Synthesis

Using Yosys:

```text
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty <library>.lib
write_verilog -noattr blocking_caveat.net.v
show
```

### Gate-Level Simulation

```bash
iverilog \
  my_lib/verilog_model/primitives.v \
  my_lib/verilog_model/sky130_fd_sc_hd.v \
  blocking_caveat.net.v \
  tb_blocking_caveat.v
```

Run:

```bash
./a.out
```

View:

```bash
gtkwave tb_blocking_caveat.vcd
```

The RTL and GLS waveforms can then be compared to observe the difference in behavior.

---

## 23. RTL Simulation vs Gate-Level Simulation

| RTL Simulation                               | Gate-Level Simulation                               |
| -------------------------------------------- | --------------------------------------------------- |
| Uses RTL code                                | Uses synthesized gate-level netlist                 |
| High-level description                       | Gate/cell-level description                         |
| Generally faster                             | Generally slower                                    |
| Verifies intended RTL functionality          | Verifies synthesized implementation                 |
| Normally does not model physical gate delays | Can model delays when delay information is provided |
| Performed before synthesis                   | Performed after synthesis                           |

### Simple Comparison

```text
RTL Simulation
      ↓
Verify RTL Functionality

Gate-Level Simulation
      ↓
Verify Synthesized Implementation
```

---

## 24. Functional GLS vs Timing GLS

### Functional GLS

Functional GLS verifies the logical behavior of the synthesized netlist.

```text
Gate-Level Netlist
       +
Testbench
       ↓
Functional GLS
```

### Timing GLS

Timing GLS uses delay information along with the gate-level design.

```text
Gate-Level Netlist
       +
Delay Information
       +
Testbench
       ↓
Timing GLS
```

Timing GLS is useful for checking behavior when propagation delays are taken into account.

---

## 25. Important Commands

### Open Verilog Files

```bash
gvim <design>.v
```

### Compile with Icarus Verilog

```bash
iverilog -o a.out <design>.v <testbench>.v
```

### Run Simulation

```bash
./a.out
```

### View Waveform

```bash
gtkwave <waveform>.vcd
```

### Yosys Synthesis

```text
read_verilog <design>.v
synth -top <top_module>
abc -liberty <library>.lib
write_verilog -noattr <netlist>.v
show
```

---

## 26. Complete Practical Flow

The complete workflow covered in this module is:

```text
                    RTL Design
                        │
                        ↓
                 RTL Simulation
                        │
                        ↓
                    Synthesis
                        │
                        ↓
              Gate-Level Netlist
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
       Functional GLS        Timing GLS
              │                   │
              ↓                   ↓
             VCD                 VCD
              │                   │
              └─────────┬─────────┘
                        ↓
                    GTKWave
```

### Step-by-Step

```text
1. Write RTL
       ↓
2. Write Testbench
       ↓
3. Run RTL Simulation
       ↓
4. Verify Waveform
       ↓
5. Synthesize Using Yosys
       ↓
6. Generate Gate-Level Netlist
       ↓
7. Include Required Gate/Cell Libraries
       ↓
8. Run GLS Using the Same Testbench
       ↓
9. Generate VCD
       ↓
10. View Waveform Using GTKWave
       ↓
11. Compare RTL and GLS Results
```

---

## 27. Key Takeaways

* **GLS** verifies the behavior of a synthesized gate-level netlist.
* The **same testbench** can be used for RTL simulation and GLS.
* **Yosys** is used to synthesize RTL and generate the gate-level netlist.
* **Icarus Verilog** can compile and simulate both RTL and gate-level Verilog.
* **GTKWave** is used to inspect simulation waveforms.
* Incomplete sensitivity lists can produce incorrect RTL simulation behavior.
* Use **blocking (`=`)** for combinational procedural logic.
* Use **non-blocking (`<=`)** for sequential and clocked logic.
* Timing GLS requires **delay-annotated gate-level models**.
* Comparing RTL and GLS helps identify synthesis-simulation mismatches.

---

## 🎯 Final Rule to Remember

```text
Combinational Logic
        ↓
always @(*)
        ↓
Blocking Assignment (=)

Sequential Logic
        ↓
always @(posedge clk)
        ↓
Non-Blocking Assignment (<=)

Gate-Level Verification
        ↓
Synthesized Netlist
        +
Same Testbench
        ↓
GLS

Timing Verification
        ↓
Gate-Level Netlist
        +
Delay Information
        ↓
Timing GLS
```

> **Write RTL that clearly represents the intended hardware, use the correct assignment style, and verify the synthesized implementation through GLS.**

## Conclusion

This module provides a practical understanding of how RTL designs move from functional simulation to synthesized gate-level verification. By studying sensitivity lists, blocking and non-blocking assignments, and GLS, it becomes easier to identify coding practices that can lead to differences between simulation and actual hardware behavior.

The experiments with **Icarus Verilog, GTKWave, and Yosys** demonstrate the complete verification flow and highlight the importance of writing synthesis-friendly, hardware-oriented Verilog code.
