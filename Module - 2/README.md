# Module 2 – Timing Libraries, Hierarchical vs. Flat Synthesis, and Efficient Flop Coding

## Overview

This module explores the next stage of the RTL-to-netlist synthesis flow. It focuses on how timing libraries guide standard-cell selection, how hierarchical and flat design structures affect synthesis, and how different flip-flop coding styles in Verilog influence the resulting hardware.

The module uses the **Sky130 standard-cell library** and **Yosys** for synthesis.

### Topics Covered

* Timing libraries and PVT (Process, Voltage, Temperature)
* Sky130 standard-cell library
* Hierarchical vs. flat synthesis
* Submodules and module instantiation
* Synchronous and asynchronous reset
* Flip-flop initialization
* Yosys synthesis flow
* `dfflibmap` and `abc`
* Technology-mapped netlist generation

---

## 1. Timing Library

A timing library provides the information required by the synthesis tool to select appropriate standard cells for a design.

For each standard cell, the library contains information such as:

* Logic function
* Propagation delay
* Power consumption
* Area
* Input capacitance
* Output drive strength

Libraries are characterized for different operating conditions, such as **slow, fast, and typical** process corners.

---

## 2. Sky130 Library

The standard-cell timing library used in this module is:

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

| Parameter        | Value               |
| ---------------- | ------------------- |
| Technology       | CMOS                |
| Process          | 130 nm              |
| Library          | Sky130 High Density |
| Delay Model      | Lookup Table        |
| Time Unit        | ns                  |
| Voltage Unit     | V                   |
| Power Unit       | nW                  |
| Current Unit     | mA                  |
| Resistance Unit  | kΩ                  |
| Capacitance Unit | pF                  |

### Library Filename

* `sky130` → SkyWater 130 nm process
* `fd_sc` → Fully Digital Standard Cell
* `hd` → High Density
* `tt` → Typical process corner
* `025C` → 25°C
* `1v80` → 1.80 V supply
* `.lib` → Liberty timing-library format

---

## 3. PVT: Process, Voltage, Temperature

**PVT** represents the three major conditions that affect circuit behavior.

### Process

Manufacturing variations cause differences between fabricated chips and wafers.

### Voltage

Supply voltage affects circuit delay, power consumption, switching behavior, and reliability.

### Temperature

Temperature changes can affect propagation delay, leakage current, and overall power consumption.

The library used in this module is characterized at:

```text
tt_025C_1v80
```

which represents:

* Typical process
* 25°C temperature
* 1.80 V supply voltage

PVT analysis helps ensure that the design operates reliably under different operating conditions.

---

## 4. Hierarchical Synthesis

Hierarchical synthesis organizes a design into smaller modules instead of treating the complete design as one large block.

```text
Top Module
   │
   ├── Submodule 1
   │
   └── Submodule 2
```

For example, a design can contain separate AND and OR modules that are instantiated inside a top-level module.

### Advantages

* Easier organization
* Easier debugging
* Module reuse
* Better scalability
* Clear design boundaries

---

## 5. Flat Synthesis

Flat synthesis removes module boundaries and treats the complete design as a single logic structure.

In Yosys, hierarchy can be removed using:

```text
flatten
```

Flattening allows the synthesis tool to optimize logic across previously separate modules.

### Hierarchical vs. Flat Synthesis

```text
Hierarchical Synthesis          Flat Synthesis
        ↓                              ↓
Modules stay separate         Hierarchy removed
        ↓                              ↓
Boundaries preserved       Whole design optimized together
```

Flat synthesis can help the synthesis tool identify redundant logic and perform optimization across module boundaries.

---

## 6. Submodules and Instantiation

A submodule is a smaller module used inside a larger module.

### Submodule

```verilog
module submodule(
    input a,
    input b,
    output y
);

    assign y = a & b;

endmodule
```

### Top Module

```verilog
module top(
    input a,
    input b,
    output y
);

    submodule u1 (
        .a(a),
        .b(b),
        .y(y)
    );

endmodule
```

Here:

* `top` is the top-level module.
* `submodule` is the child module.
* `u1` is the instance name.
* Named port connections explicitly connect the signals.

---

## 7. Flip-Flops

A flip-flop is a sequential storage element used to store one bit of information.

Typical signals include:

| Signal  | Function   |
| ------- | ---------- |
| `D`     | Data input |
| `CLK`   | Clock      |
| `RESET` | Reset      |
| `Q`     | Output     |

Flip-flops provide memory to digital circuits by allowing the circuit to retain state between clock cycles.

The way a flip-flop is described in Verilog directly affects the hardware inferred by Yosys.

---

## 8. Synchronous Reset

A synchronous reset is checked only at the active clock edge.

```verilog
always @(posedge clk)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

The operation can be represented as:

```text
Clock Edge
    ↓
Check Reset
    ↓
Reset = 1 → Q = 0
Reset = 0 → Q = D
```

The reset does not immediately change the output. It takes effect when the next active clock edge occurs.

---

## 9. Asynchronous Reset

An asynchronous reset operates independently of the clock.

```verilog
always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

The behavior is:

```text
Reset asserted
      ↓
Q becomes 0 immediately

        OR

Clock edge occurs
      ↓
If reset = 0
      ↓
Q follows D
```

### Synchronous vs. Asynchronous Reset

| Synchronous Reset                 | Asynchronous Reset                  |
| --------------------------------- | ----------------------------------- |
| Reset depends on the clock        | Reset does not depend on the clock  |
| Checked at the clock edge         | Acts immediately                    |
| `posedge clk`                     | `posedge clk or posedge reset`      |
| Easier to reason about for timing | Useful for immediate initialization |

---

## 10. Flip-Flop Initialization

A flip-flop does not automatically have a known value when simulation starts. Without a reset, its initial state can be unknown (`X`).

A reset condition such as:

```verilog
if (reset)
    q <= 1'b0;
```

provides a known starting state.

---

## 11. Yosys Synthesis Flow

The synthesis flow used in this module is:

```text
RTL Verilog
     ↓
read_verilog
     ↓
hierarchy
     ↓
flatten
     ↓
dfflibmap
     ↓
abc
     ↓
write_verilog
```

### Reading RTL

```text
read_verilog diff_async_set.v
```

This loads the Verilog RTL source into Yosys.

### Flip-Flop Mapping — `dfflibmap`

```text
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

This maps inferred flip-flops to sequential cells available in the Sky130 library.

```text
RTL Flip-Flop
      ↓
dfflibmap
      ↓
Sky130 Flip-Flop Cell
```

### Technology Mapping — `abc`

```text
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

ABC optimizes the combinational logic and maps it to standard cells.

```text
Optimized Logic
      ↓
ABC
      ↓
Technology Mapping
      ↓
Sky130 Standard Cells
```

### `dfflibmap` vs. `abc`

* **`dfflibmap`** → Maps sequential elements such as flip-flops.
* **`abc`** → Optimizes and maps combinational logic.

Both are important parts of the technology-mapping process.

### Writing the Netlist

```text
write_verilog -noattr synthesized.v
```

The `-noattr` option removes synthesis attributes from the generated output netlist.

### Complete Example

```text
read_verilog diff_async_set.v

synth -top async_set

dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib

abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib

write_verilog -noattr synthesized.v
```

---

## 12. Yosys Commands Used

| Command         | Purpose                                   |
| --------------- | ----------------------------------------- |
| `read_verilog`  | Reads Verilog RTL into Yosys              |
| `hierarchy`     | Sets and checks the design hierarchy      |
| `flatten`       | Removes module hierarchy                  |
| `dfflibmap`     | Maps inferred flip-flops to library cells |
| `abc`           | Optimizes and maps combinational logic    |
| `write_verilog` | Writes the synthesized netlist            |

---

## 13. Synchronous vs. Asynchronous Reset – Synthesis

### Synchronous Reset

```verilog
always @(posedge clk)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

### Asynchronous Reset

```verilog
always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

The appropriate reset style depends on the required hardware architecture.

Synchronous reset keeps reset operation associated with the clock, while asynchronous reset allows the flip-flop to be forced into a known state independently of the clock.

---

## Files Used

| File                                | Description                                  |
| ----------------------------------- | -------------------------------------------- |
| `diff_async_set.v`                  | RTL design with asynchronous reset flip-flop |
| `sky130_fd_sc_hd__tt_025C_1v80.lib` | Sky130 timing library at the typical corner  |
| `synthesized.v`                     | Technology-mapped output netlist             |

---

## What This Module Covered

* Timing libraries and their role in synthesis
* PVT variations and operating conditions
* Sky130 standard-cell library
* Hierarchical design and module instantiation
* Flat synthesis and hierarchy removal
* Synchronous and asynchronous flip-flop resets
* Flip-flop initialization
* Yosys synthesis flow
* `dfflibmap` for sequential-cell mapping
* `abc` for combinational optimization and technology mapping
* Generation of a technology-specific netlist

---

## Conclusion

This module extends the RTL-to-netlist concepts introduced earlier by exploring the factors that influence technology mapping. It explains how timing libraries and PVT conditions are used, how hierarchical and flat synthesis affect optimization, and how Verilog flip-flop coding styles influence the synthesized hardware.

Using **Yosys** together with the **Sky130 standard-cell library**, the RTL design can be transformed into a technology-specific netlist through synthesis, flip-flop mapping, combinational optimization, and final netlist generation.
