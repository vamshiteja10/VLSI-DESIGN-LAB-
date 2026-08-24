# Module 1 – RTL Design, Simulation, and Synthesis (SKY130)

## 📌 Overview

This module demonstrates the complete **RTL-to-Netlist flow** of a digital design, starting from Verilog RTL coding and functional simulation, followed by synthesis and technology mapping using the **SKY130 standard-cell library**.

A simple **2:1 Multiplexer (`good_mux`)** is used as the example design throughout the module.

The complete flow includes:

```text
Verilog RTL
    ↓
Icarus Verilog
    ↓
VCD Generation
    ↓
GTKWave
    ↓
RTL Verification
    ↓
Yosys Synthesis
    ↓
ABC Technology Mapping
    ↓
SKY130 Standard Cells
    ↓
Gate-Level Netlist
    ↓
Netlist Verification
```

The practical work was carried out as part of a **Digital VLSI / RTL-to-Netlist Synthesis Workshop** using a Linux virtual-machine environment.

---

## 🎯 Objectives

The main objectives of this module are:

* Understand the concept of **RTL design**.
* Write a basic digital circuit using **Verilog HDL**.
* Create a Verilog **testbench** for functional verification.
* Simulate the RTL design using **Icarus Verilog**.
* Generate and analyze **VCD waveform files**.
* Visualize simulation waveforms using **GTKWave**.
* Understand the concept of **RTL synthesis**.
* Use **Yosys** for RTL synthesis.
* Understand **ABC technology mapping**.
* Learn about **SKY130 standard-cell libraries**.
* Generate a technology-mapped **gate-level netlist**.
* Verify the synthesized netlist using simulation.
* Understand the basic relationship between **timing, area, power, and performance**.

---

# 1. Understanding RTL

## What is RTL?

**RTL (Register Transfer Level)** is a way of describing the behavior and functionality of a digital circuit.

RTL focuses on **what the circuit should do**, while synthesis determines how that functionality can be implemented using available standard cells.

In this module, a simple **2:1 multiplexer** is used as the RTL example.

---

## 2. 2:1 Multiplexer Design

The RTL design is implemented using Verilog HDL.

### Verilog RTL

```verilog
module good_mux(i0, i1, sel, y);
    input i0, i1, sel;
    output y;

    assign y = sel ? i1 : i0;
endmodule
```

### Inputs and Output

| Signal | Description        |
| ------ | ------------------ |
| `i0`   | First data input   |
| `i1`   | Second data input  |
| `sel`  | Select input       |
| `y`    | Multiplexer output |

### Multiplexer Operation

| `sel` | Output `y` |
| ----- | ---------- |
| `0`   | `i0`       |
| `1`   | `i1`       |

The Boolean expression of the multiplexer is:

```text
y = sel ? i1 : i0
```

or equivalently:

```text
y = (~sel & i0) | (sel & i1)
```

The RTL describes the required functionality, while the synthesis process later determines the physical logic-cell implementation.

---

# 3. RTL Simulation

Before synthesizing the design, the RTL must be verified through simulation.

Simulation helps confirm that the design behaves according to the intended logic before moving to the synthesis stage.

The tools used for simulation are:

* **Icarus Verilog**
* **GTKWave**

---

## Icarus Verilog

**Icarus Verilog (`iverilog`)** is an open-source Verilog compiler and simulator.

It compiles the design file and testbench and produces a simulation executable.

---

## GTKWave

**GTKWave** is a waveform viewer used to inspect the signal transitions generated during simulation.

The simulation produces a **VCD (Value Change Dump)** file, which contains information about signal changes during the simulation.

GTKWave reads this VCD file and displays the waveforms graphically.

---

# 4. Design File and Testbench

A typical simulation consists of two important Verilog files.

### Design File

The design file contains the actual digital logic.

```text
good_mux.v
```

The design under test is called the **UUT (Unit Under Test)**.

### Testbench

The testbench is used to:

* Instantiate the design.
* Apply input stimulus.
* Generate waveform data.
* Control simulation time.
* End the simulation.

```text
tb_good_mux.v
```

The testbench itself does not represent physical hardware. Instead, it provides inputs to the UUT and observes its outputs.

---

# 5. Simulation Flow

The complete simulation process is:

```text
good_mux.v + tb_good_mux.v
             │
             ▼
        Icarus Verilog
             │
             ▼
           a.out
             │
             ▼
       tb_good_mux.vcd
             │
             ▼
          GTKWave
             │
             ▼
      Waveform Analysis
```

---

# 6. Running the Simulation

## Step 1 – Compile

Use Icarus Verilog to compile the RTL and testbench:

```bash
iverilog good_mux.v tb_good_mux.v
```

This generates the executable:

```text
a.out
```

---

## Step 2 – Run

Execute the generated simulation:

```bash
./a.out
```

This produces the waveform file:

```text
tb_good_mux.vcd
```

---

## Step 3 – Open GTKWave

Open the generated VCD file:

```bash
gtkwave tb_good_mux.vcd
```

GTKWave can now be used to inspect the behavior of the multiplexer.

The basic simulation commands and their outputs are shown in the original workflow.

---

# 7. VCD File Generation

The VCD file is generated from the testbench using the following statements:

```verilog
$dumpfile("tb_good_mux.vcd");
$dumpvars(0, tb_good_mux);
```

### `$dumpfile`

Specifies the name of the VCD file.

### `$dumpvars`

Specifies the signals and hierarchy that should be recorded.

Without these statements, a VCD waveform file will not be generated for GTKWave.

---

# 8. Generating Test Stimulus

The testbench applies changing input values to verify the multiplexer.

```verilog
initial begin
    sel = 0;
    i0 = 0;
    i1 = 0;
    #300 $finish;
end

always #75 sel = ~sel;
always #10 i0  = ~i0;
always #55 i1  = ~i1;
```

The inputs change at different time intervals so that different combinations of `i0`, `i1`, and `sel` can be observed.

The simulation ends after:

```text
300 ns
```

using:

```verilog
$finish;
```

---

# 9. Waveform Analysis Using GTKWave

After opening the VCD file in GTKWave:

1. Select the `tb_good_mux` module.
2. Locate the `uut` instance.
3. Select the required signals.
4. Add the following signals to the waveform viewer:

   * `i0`
   * `i1`
   * `sel`
   * `y`
5. Use **Zoom Fit** to view the complete simulation.
6. Observe the relationship between the inputs and output.

The output `y` should follow the selected input according to the value of `sel`.

---

# 10. Simulation Files

| File              | Description                    |
| ----------------- | ------------------------------ |
| `good_mux.v`      | RTL design                     |
| `tb_good_mux.v`   | Testbench                      |
| `tb_good_mux.vcd` | Generated waveform file        |
| `a.out`           | Compiled simulation executable |

---

# 11. RTL-to-Netlist Synthesis

Once the RTL has been successfully verified, the next stage is **synthesis**.

Synthesis converts the RTL description into a structural **gate-level netlist** using cells available in the target technology library.

The synthesis flow used in this module consists of:

* **Yosys**
* **ABC**
* **SKY130 Standard-Cell Library**

---

## Synthesis Flow

```text
             RTL / Verilog
                  │
                  ▼
                Yosys
                  │
                  ▼
            RTL Synthesis
                  │
                  ▼
           Logic Network
                  │
                  ▼
                 ABC
                  │
                  ▼
        Technology Mapping
                  │
                  ▼
        SKY130 Standard Cells
                  │
                  ▼
               Netlist
```

---

# 12. What is a Netlist?

A **netlist** is a structural representation of a digital circuit.

It describes:

* The cells used in the circuit.
* The connections between cells.
* Input and output ports.
* Internal nets.

After technology mapping, the RTL functionality is represented using actual cells available in the selected standard-cell library.

For this module, the multiplexer can be mapped to the SKY130 cell:

```text
sky130_fd_sc_hd__mux2_1
```

Depending on the mapping flow, equivalent logic could also be implemented using combinations of standard logic cells.

---

# 13. SKY130 Liberty Library

The technology mapping process uses a **Liberty (`.lib`) file**.

A Liberty file contains information about the standard cells available in the technology library.

It includes:

* Cell names
* Logic functions
* Input/output pins
* Timing information
* Propagation delays
* Power information
* Area information
* Drive-strength variants

An example SKY130 cell is:

```text
sky130_fd_sc_hd__mux2_1
```

The `_1` represents a particular drive-strength variant.

---

# 14. Cell Variants and Trade-offs

Standard-cell libraries provide different versions of cells.

### Faster Cells

Faster cells generally provide:

* Lower delay
* Higher drive strength
* Larger area
* Higher power consumption

### Slower Cells

Slower cells generally provide:

* Higher delay
* Lower drive strength
* Smaller area
* Lower power consumption

Therefore, synthesis must select cells according to the requirements of the design.

The main design trade-off is:

```text
Performance ↔ Area ↔ Power ↔ Timing
```

---

# 15. Timing Analysis

Timing is an important consideration when selecting standard cells.

For a flip-flop-to-flip-flop path:

```text
FF A
 │
 ▼
Combinational Logic
 │
 ▼
FF B
```

The setup-time relationship is:

```text
Tclk ≥ Tcq(A) + Tcomb + Tsetup(B)
```

Where:

| Parameter   | Description               |
| ----------- | ------------------------- |
| `Tcq(A)`    | Clock-to-Q delay of FF A  |
| `Tcomb`     | Combinational logic delay |
| `Tsetup(B)` | Setup time of FF B        |

The maximum frequency is:

```text
Fmax = 1 / Tclk(min)
```

A shorter critical path can allow a higher operating frequency.

---

# 16. Hold Timing

Fast cells can sometimes create hold-time problems because data may arrive too quickly.

The hold-time relationship is:

```text
Thold(B) ≤ Tcq(A) + Tcomb
```

If the condition is not satisfied, the receiving flip-flop may capture incorrect data.

Therefore, synthesis flows balance fast and slow cells according to timing requirements.

This is another example of the relationship between:

```text
Performance
     ↕
Area
     ↕
Power
     ↕
Timing
```

---

# 17. Yosys Synthesis

**Yosys** is an open-source synthesis framework used to process the Verilog RTL and generate a gate-level representation.

The major steps include:

1. Reading the RTL.
2. Performing synthesis.
3. Optimizing the logic.
4. Performing technology mapping.
5. Visualizing the circuit.
6. Writing the final netlist.

---

## Read the RTL

```tcl
read_verilog good_mux.v
```

---

## Run Synthesis

```tcl
synth -top good_mux
```

---

## Technology Mapping Using ABC

```tcl
abc -liberty /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The `-liberty` option specifies the Liberty library used during technology mapping.

---

## Visualize the Circuit

```tcl
show
```

---

## Generate the Netlist

```tcl
write_verilog -noattr good_mux_netlist.v
```

---

# 18. ABC Technology Mapping

ABC performs logic optimization and technology mapping as part of the synthesis flow.

```text
RTL
 │
 ▼
Yosys Synthesis
 │
 ▼
Logic Representation
 │
 ▼
ABC Optimization
 │
 ▼
Technology Mapping
 │
 ▼
SKY130 Standard Cells
 │
 ▼
Mapped Netlist
```

For the `good_mux` design, ABC maps the multiplexer to:

```text
sky130_fd_sc_hd__mux2_1
```

The mapped cell has:

```text
Inputs  : i0, i1, sel
Output  : y
```

The functionality remains identical to the original RTL.

---

# 19. Synthesized Multiplexer

The synthesized circuit represents the same 2:1 multiplexer functionality:

```text
       i0 ─────┐
               │
       i1 ─────┤
               │
               ├──── 2:1 MUX ──── y
               │
      sel ─────┘
```

### Logic Function

```text
y = sel ? i1 : i0
```

Equivalent Boolean expression:

```text
y = (~sel & i0) | (sel & i1)
```

The synthesized implementation must preserve the functionality defined by the original RTL.

---

# 20. Netlist Verification

After synthesis, the generated netlist is also verified through simulation.

This ensures that the synthesis process has preserved the functionality of the original RTL design.

```text
RTL / Netlist
      │
      ▼
 Testbench
      │
      ▼
Icarus Verilog
      │
      ▼
    VCD
      │
      ▼
 GTKWave
      │
      ▼
Waveform Verification
```

Because the primary inputs and outputs remain the same, the original testbench can generally be reused for the synthesized netlist.

---

# 21. Complete RTL-to-Netlist-to-Waveform Flow

The complete process covered in this module is:

```text
                    RTL Design
                        │
                        ▼
                  Verilog RTL
                        │
                        ▼
              ┌─────────────────┐
              │ Icarus Verilog  │
              └─────────────────┘
                        │
                        ▼
                  VCD Generation
                        │
                        ▼
                    GTKWave
                        │
                        ▼
                RTL Verification
                        │
                        ▼
                     Yosys
                        │
                        ▼
                 RTL Synthesis
                        │
                        ▼
                      ABC
                        │
                        ▼
             Technology Mapping
                        │
                        ▼
             SKY130 Standard Cells
                        │
                        ▼
              Gate-Level Netlist
                        │
                        ▼
               Netlist Simulation
                        │
                        ▼
                    GTKWave
                        │
                        ▼
               Final Verification
```

---

# 22. Project Directory Structure

A simple repository structure for this module can be organized as:

```text
Module-1-RTL-Design-Simulation-Synthesis/
│
├── README.md
│
├── rtl/
│   └── good_mux.v
│
├── testbench/
│   └── tb_good_mux.v
│
├── simulation/
│   └── tb_good_mux.vcd
│
└── synthesis/
    └── good_mux_netlist.v
```

---

# 23. Files Used

| File                 | Description                       |
| -------------------- | --------------------------------- |
| `good_mux.v`         | RTL design of the 2:1 multiplexer |
| `tb_good_mux.v`      | Verilog testbench                 |
| `tb_good_mux.vcd`    | Generated simulation waveform     |
| `a.out`              | Compiled simulation executable    |
| `good_mux_netlist.v` | Technology-mapped SKY130 netlist  |
| `README.md`          | Module documentation              |

The original module identifies the same core RTL, testbench, waveform, executable, and synthesized-netlist files.

---

# 24. Tools and Technologies

| Tool / Technology  | Purpose                                   |
| ------------------ | ----------------------------------------- |
| **Verilog HDL**    | RTL design                                |
| **Icarus Verilog** | RTL and netlist simulation                |
| **GTKWave**        | Waveform visualization                    |
| **Yosys**          | RTL synthesis                             |
| **ABC**            | Logic optimization and technology mapping |
| **SKY130**         | Standard-cell technology library          |
| **Linux VM**       | Development environment                   |

---

# 25. Key Concepts Learned

This module provides practical understanding of:

* RTL design
* Verilog HDL
* Behavioral modeling
* Testbench development
* Functional simulation
* VCD waveform generation
* GTKWave waveform analysis
* RTL synthesis
* Gate-level netlists
* Liberty files
* Standard-cell libraries
* Technology mapping
* ABC optimization
* SKY130 standard cells
* Drive strength
* Propagation delay
* Setup timing
* Hold timing
* Maximum operating frequency
* Area-power-performance trade-offs
* Netlist visualization
* Post-synthesis verification

---

# 26. Learning Outcome

After completing this module, the complete RTL-to-Netlist process can be understood as:

```text
Design
  ↓
Simulate
  ↓
Verify
  ↓
Synthesize
  ↓
Technology Map
  ↓
Generate Netlist
  ↓
Simulate Netlist
  ↓
Verify Functionality
```

The module demonstrates how a simple Verilog design moves from an abstract RTL description toward an implementation using real standard cells.

---

# 27. Conclusion

This module demonstrates the complete **RTL design, simulation, synthesis, and verification flow** using a 2:1 multiplexer as the example design.

The process begins by writing the multiplexer in **Verilog RTL** and verifying its functionality using **Icarus Verilog**. The generated VCD file is then analyzed using **GTKWave**.

After successful RTL verification, the design is synthesized using **Yosys**. **ABC** performs logic optimization and technology mapping using the **SKY130 standard-cell library**. The resulting design is represented as a technology-mapped gate-level netlist.

For the example used in this module, the multiplexer is mapped to:

```text
sky130_fd_sc_hd__mux2_1
```

Finally, the synthesized netlist is simulated and verified to ensure that the original RTL functionality has been preserved.

The complete flow can therefore be summarized as:

```text
Verilog RTL
     ↓
Icarus Verilog
     ↓
GTKWave
     ↓
RTL Verification
     ↓
Yosys
     ↓
ABC
     ↓
SKY130 Technology Mapping
     ↓
Gate-Level Netlist
     ↓
Netlist Verification
```

This provides a practical foundation for understanding the **RTL-to-Netlist flow used in digital VLSI design**.
