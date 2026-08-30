# VLSI RTL Design and Synthesis – Overall Summary & BabySoc

The **VLSI RTL Design and Synthesis training** provided a practical understanding of how a digital design moves from a **Verilog RTL description to a synthesized gate-level implementation**. The training covered RTL coding, functional simulation, waveform analysis, timing libraries, hierarchical and sequential design, logic optimization, synthesis, technology mapping, Gate-Level Simulation (GLS), and synthesis-oriented coding practices. The complete work was carried out using open-source EDA tools such as **Icarus Verilog, GTKWave, Yosys, and ABC**, along with the **SKY130 standard-cell library**.

## Module 1 – RTL Design, Simulation, and Synthesis

The first module introduced the fundamentals of **RTL design using Verilog HDL**. A simple **2:1 multiplexer** was used as the main example to understand how a digital circuit is described and verified.

The design was first written in Verilog and then a **testbench** was created to apply different input combinations. The RTL was simulated using **Icarus Verilog**, which generated a **VCD (Value Change Dump)** file containing the signal transitions. These waveforms were then viewed and analyzed using **GTKWave** to verify that the multiplexer behaved as expected.

After functional verification, the design was synthesized using **Yosys**. The synthesis process converts the behavioral RTL description into a hardware representation. **ABC** was then used for logic optimization and technology mapping to the **SKY130 standard-cell library**, resulting in a gate-level netlist.

This module established the basic **RTL-to-netlist flow**:

**Verilog RTL → Testbench → Simulation → VCD → GTKWave → Yosys Synthesis → ABC Mapping → SKY130 Cells → Gate-Level Netlist**.

## Module 2 – Timing Libraries, Hierarchical Design, and Sequential Logic

The second module moved deeper into how synthesis tools convert RTL into **technology-dependent hardware**. A major focus was understanding **standard-cell timing libraries and Liberty (`.lib`) files**.

A timing library provides important information about standard cells, including **logic function, propagation delay, area, power, input capacitance, output drive strength, setup time, hold time, and clock-to-Q characteristics**. The training also introduced **PVT (Process, Voltage, Temperature)** conditions, which influence the behavior and reliability of digital circuits.

The module also introduced **hierarchical RTL design**, where a large design is divided into smaller reusable submodules. Hierarchical design improves organization, debugging, scalability, and module reuse. In contrast, **flat synthesis** removes module boundaries and allows the synthesis tool to optimize the complete logic together.

Sequential logic was another important part of this module. **D flip-flops**, synchronous and asynchronous reset behavior, flip-flop initialization, and different coding styles were studied. The way sequential logic is written in Verilog affects the hardware inferred during synthesis.

The Yosys synthesis flow introduced operations such as **`read_verilog`**, hierarchy processing, **`flatten`**, **`dfflibmap`**, **`abc`**, and netlist generation. This helped connect RTL coding with the actual standard cells used in synthesized hardware.

## Module 3 – Combinational and Sequential Logic Optimization

The third module focused on **optimization**, which is one of the most important stages of synthesis. The main objective of optimization is to achieve the required functionality using less hardware while improving **area, power, and performance**.

For **combinational logic**, techniques such as:

* Constant propagation
* Boolean simplification
* Redundant logic removal
* Conditional logic simplification
* K-Map-based simplification
* Quine–McCluskey concepts

were studied.

For example, when a known constant value is propagated through a Boolean expression, unnecessary gates can be removed. This reduces the amount of hardware required and can also reduce switching activity and power consumption.

The module also covered **sequential optimization**. Flip-flops or registers whose outputs can be proven to remain constant may be simplified or removed. Other concepts included **state optimization, register dependencies, counter optimization, unused counter bits, and comparison-logic optimization**.

Through these examples, it became clear that synthesis tools do not simply translate every line of RTL directly into hardware. Instead, they analyze the required functionality and remove logic that is unnecessary while preserving the intended behavior.

## Module 4 – Gate-Level Simulation and Synthesis-Simulation Mismatch

The fourth module focused on verifying whether the synthesized hardware still behaves like the original RTL design. This introduced **Gate-Level Simulation (GLS)**.

In GLS, the synthesized gate-level netlist is simulated using a testbench. The resulting waveform can then be compared with the RTL simulation. This helps verify that synthesis has preserved the intended functionality and can also reveal **synthesis-simulation mismatches**.

A major topic was the importance of **correct sensitivity lists**. An incomplete sensitivity list can cause RTL simulation to behave incorrectly even though the synthesized hardware may represent the intended logic. For combinational procedural logic, using:

`always @(*)`

ensures that changes in all relevant input signals trigger the block.

The module also explained the difference between **blocking (`=`)** and **non-blocking (`<=`)** assignments.

* **Blocking assignments (`=`)** execute immediately and are generally used for combinational logic.
* **Non-blocking assignments (`<=`)** schedule updates and are generally used for sequential logic such as flip-flops and registers.

The general coding rule learned was:

**Combinational Logic → Blocking `=`**

**Sequential Logic → Non-Blocking `<=`**

The overall GLS flow was:

**RTL → RTL Simulation → Synthesis → Gate-Level Netlist → Gate-Level Simulation → Waveform Comparison**

This provided practical experience in **post-synthesis verification**.

## Module 5 – RTL Coding Styles and Synthesis Optimization

The fifth module concentrated on how **RTL coding style directly influences the hardware produced during synthesis**. The main lesson was that RTL should not only be functionally correct but should also describe the intended hardware clearly and completely.

One of the most important concepts was **latch inference**. If a combinational `if` statement does not assign an output for every possible condition, the hardware may need to retain the previous value. This can cause an unintended latch to be inferred during synthesis.

For example, an incomplete condition such as:

`if(i0) y = i1;`

does not specify what happens when `i0` is `0`. Providing an `else` branch or a default assignment ensures that the output receives a value on every execution path.

The same principle applies to **`case` statements**. Incomplete case statements can produce unintended latches, while a **`default` branch** or complete case coverage helps create predictable combinational hardware.

The module also covered:

* MUX and DEMUX implementation
* Procedural `for` loops
* `for generate`
* Hardware replication
* Hierarchical structures
* Ripple-carry adder implementation
* Repeated full-adder generation

An important distinction was made between a **procedural `for` loop**, which describes repeated operations inside procedural logic, and a **`for generate` construct**, which is mainly used to replicate hardware structures during elaboration.

## Overall Understanding

Across all five modules, the training built a complete understanding of the **digital VLSI RTL-to-hardware design flow**.

The overall process can be summarized as:

**RTL Design**
↓
**Verilog Coding**
↓
**Testbench Development**
↓
**Functional Simulation**
↓
**VCD Generation**
↓
**GTKWave Analysis**
↓
**Yosys Synthesis**
↓
**Logic Optimization**
↓
**Technology Mapping**
↓
**SKY130 Standard Cells**
↓
**Gate-Level Netlist**
↓
**Gate-Level Simulation**
↓
**Waveform Verification**

The training demonstrated that successful VLSI design is not only about writing correct Verilog code. The RTL must also be written in a way that allows synthesis tools to generate **efficient, predictable, and functionally correct hardware**. Timing libraries help the tool understand the characteristics of available cells, optimization removes unnecessary hardware, technology mapping converts the optimized logic into standard cells, and GLS verifies the final synthesized implementation.

The major tools used throughout the training were **Verilog for RTL design, Icarus Verilog for simulation, GTKWave for waveform analysis, Yosys for synthesis and optimization, ABC for logic optimization and technology mapping, and SKY130 for standard-cell implementation**.

## Final Conclusion

Overall, the VLSI RTL Design and Synthesis training provided a practical foundation for understanding how a digital circuit progresses from an **RTL description to synthesized hardware**. Starting with basic Verilog coding and functional simulation, the training gradually introduced timing libraries, hierarchical and sequential logic, synthesis optimization, coding best practices, technology mapping, and Gate-Level Simulation.

The most important learning was that **RTL coding style has a direct impact on the hardware generated by synthesis**. Proper sensitivity lists, correct blocking and non-blocking assignments, complete conditional statements, careful `case` statements, and appropriate use of generate constructs help produce hardware that closely matches the intended design.

By completing these modules, the complete relationship between **RTL code, simulation, synthesis, optimization, standard-cell technology mapping, gate-level implementation, and verification** became clear. This establishes a strong practical foundation for further work in **digital VLSI design, RTL development, synthesis, and physical implementation**.
# BabySoC – RTL to Post-Synthesis Gate-Level Verification

## 📌 Overview

This project demonstrates the **front-end ASIC design flow** using a small RISC-V-based System-on-Chip called **BabySoC**.

The design is taken through the complete front-end process, starting from **RTL design and functional simulation**, followed by **logic synthesis, optimization, SKY130 technology mapping, gate-level netlist generation, and post-synthesis simulation**.

The main goal of this project is to verify that the synthesized hardware continues to implement the intended functionality of the original RTL design.

---

## 🧩 1. BabySoC Design Overview

BabySoC is built around three main hardware blocks inside the top-level `vsdbabysoc` module:

| Block       | Function                                                      |
| ----------- | ------------------------------------------------------------- |
| **RVMyth**  | RISC-V processor responsible for digital processing           |
| **AVSDPLL** | Generates the clock used by the processor                     |
| **AVSDDAC** | Converts the processor's digital output into an analog output |

The overall signal flow can be represented as:

```text
        Reference / PLL Control Signals
                     │
                     ▼
                AVSDPLL
                     │
                    CLK
                     │
                     ▼
                  RVMyth
                RISC-V CPU
                     │
               RV_TO_DAC[9:0]
                     │
                     ▼
                  AVSDDAC
                     │
                    OUT
```

### Signal Flow

The main control and data signals move through the SoC as follows:

```text
REF, VCO_IN, ENb_CP, ENb_VCO
              │
              ▼
           AVSDPLL
              │
             CLK
              │
              ▼
           RVMyth
              │
       RV_TO_DAC[9:0]
              │
              ▼
           AVSDDAC
              │
             OUT
```

### Design Hierarchy

```text
vsdbabysoc
├── avsddpll
├── rvmyth
└── avsddac
```

This hierarchy makes it easier to understand how the processor, clock-generation block, and DAC are connected within the BabySoC.

---

# 🔄 2. ASIC Design Flow

The project follows the **front-end portion of the ASIC design flow**:

```text
RTL Design
    ↓
Pre-Synthesis Simulation
    ↓
Logic Synthesis
    ↓
SKY130 Technology Mapping
    ↓
Gate-Level Netlist
    ↓
Post-Synthesis Simulation
    ↓
Static Timing Analysis
    ↓
Physical Design
```

### Project Progress

| Design Stage              | Status      |
| ------------------------- | ----------- |
| RTL Design                | ✅ Completed |
| Pre-Synthesis Simulation  | ✅ Completed |
| Yosys Synthesis           | ✅ Completed |
| SKY130 Technology Mapping | ✅ Completed |
| Gate-Level Netlist        | ✅ Completed |
| Post-Synthesis Simulation | ✅ Completed |
| Static Timing Analysis    | 🔜 Next     |
| Floorplanning             | ⏳ Upcoming  |
| Placement                 | ⏳ Upcoming  |
| Clock Tree Synthesis      | ⏳ Upcoming  |
| Routing                   | ⏳ Upcoming  |
| GDSII Generation          | ⏳ Upcoming  |

The current work successfully covers the flow from **RTL design to post-synthesis gate-level verification**.

---

# 🧪 3. Pre-Synthesis Simulation

Before synthesis, the original RTL design was simulated to verify that the BabySoC behaves as expected.

The simulation was performed using **Icarus Verilog**, and the generated waveform was analyzed using **GTKWave**.

### Important Signals Observed

* `CLK`
* `REF`
* `reset`
* `VCO_IN`
* `VREFH`
* `RV_TO_DAC[9:0]`
* `OUT`

The pre-synthesis simulation acts as the **reference behavior** of the RTL design.

This reference is important because the same functionality can later be checked against the synthesized gate-level implementation.

### Pre-Synthesis Waveform

<img width="1920" height="983" alt="PRE_SYNTH" src="https://github.com/user-attachments/assets/87eb0d2b-1ea6-4e2c-be25-137f9527102e" />


---

# ⚙️ 4. RTL Synthesis Using Yosys

After verifying the RTL functionality, the design was synthesized using **Yosys**.

The BabySoC design was synthesized against the **SKY130 high-density standard-cell library**.

### Technology Library

```text
Library:
sky130_fd_sc_hd

Liberty:
sky130_fd_sc_hd__tt_025C_1v80.lib
```

During synthesis, Yosys performs several transformations to convert the RTL description into an optimized gate-level implementation.

### Important Yosys Operations

| Yosys Operation     | Purpose                             |
| ------------------- | ----------------------------------- |
| `read_verilog`      | Reads the RTL source files          |
| `dfflibmap`         | Maps flip-flops to library cells    |
| `opt`               | Performs logic optimization         |
| `abc`               | Performs technology mapping         |
| `flatten`           | Combines the module hierarchy       |
| `setundef -zero`    | Resolves undefined signals          |
| `clean -purge`      | Removes unused logic                |
| `rename -enumerate` | Renames internal signals            |
| `write_verilog`     | Generates the final netlist         |
| `show`              | Produces a schematic representation |

### Yosys Synthesis

<img width="846" height="712" alt="image" src="https://github.com/user-attachments/assets/a5a8710e-01a6-4d2d-9d45-97a3beaed53d" />


---

# 📊 5. Synthesis Statistics

After synthesis and optimization, Yosys provides statistics describing the resulting hardware.

These statistics give an idea of the **amount and type of logic** present in the synthesized design.

<img width="917" height="763" alt="image" src="https://github.com/user-attachments/assets/ff2faec7-c40b-426a-bed3-1260fe561c28" />


---

# 🔧 6. SKY130 Technology Mapping

Once the RTL has been logically optimized, it is mapped to actual cells available in the **SKY130 standard-cell library**.

This is an important transition in the ASIC flow because the design moves from an abstract RTL representation to a **technology-specific gate-level implementation**.

Some of the standard cells used in the mapped design include:

```text
sky130_fd_sc_hd__nand2_1
sky130_fd_sc_hd__nor2_1
sky130_fd_sc_hd__and2_0
sky130_fd_sc_hd__mux2_1
sky130_fd_sc_hd__xor2_1
sky130_fd_sc_hd__dfrtp_1
```

These standard cells are used to build the synthesized digital logic of the BabySoC.

---

# 🏗️ 7. Technology-Mapped Netlist

The synthesized design can be inspected at different levels of hierarchy.

### Top-Level BabySoC Netlist

<img width="1920" height="983" alt="vsdbabysoc2" src="https://github.com/user-attachments/assets/73ffe8a1-ef85-4986-a5f4-ef624c71bc49" />


### RVMyth CPU Netlist

<img width="843" height="428" alt="image" src="https://github.com/user-attachments/assets/8f72dde8-3d2f-40a2-99cf-5d09b3b3da68" />


### Expanded RVMyth Netlist
<img width="842" height="477" alt="image" src="https://github.com/user-attachments/assets/52cbaa0d-e257-489e-9ddc-d2a8c03be82c" />



### Clock-Gating Netlist

These different netlist views help visualize how the original RTL hierarchy has been transformed into a network of **SKY130 standard cells**.

---

# 🧪 8. Post-Synthesis Gate-Level Simulation

After generating the technology-mapped netlist, the design was simulated again.

Unlike RTL simulation, this stage operates on the **synthesized gate-level representation** of the design.

The simulation uses:

* Synthesized gate-level netlist
* SKY130 standard-cell Verilog models
* Original testbench
* Icarus Verilog

### Post-Synthesis Simulation Flow

```text
        Gate-Level Netlist
                 +
        SKY130 Cell Models
                 +
             Testbench
                 │
                 ▼
          Icarus Verilog
                 │
                 ▼
       post_synth_sim.vcd
                 │
                 ▼
             GTKWave
```

The simulation was performed with the following definitions:

```text
-DPOST_SYNTH_SIM
-DFUNCTIONAL
-DUNIT_DELAY=#1
```

### Post-Synthesis Waveform

<img width="1577" height="791" alt="image" src="https://github.com/user-attachments/assets/c87ca8e5-4e14-412e-aea0-139176b344c3" />


The purpose of this stage is to confirm that the synthesized implementation still behaves correctly when simulated using the actual mapped standard-cell models.

---

# 🔍 9. RTL vs Gate-Level Verification

One of the most important parts of this project is comparing the behavior of the **original RTL design** with the **synthesized gate-level implementation**.

The comparison focuses on important signals such as:

* `CLK`
* `REF`
* `reset`
* `RV_TO_DAC[9:0]`
* `OUT`

Among these signals, `RV_TO_DAC[9:0]` is particularly useful because it represents the digital information being transferred from the processor toward the DAC.

The basic verification idea is:

```text
RTL Simulation
      │
      ▼
Reference Behavior
      │
      │ Compare
      ▼
Gate-Level Simulation
      │
      ▼
Synthesized Behavior
```

If the important signals show the expected matching behavior, it provides evidence that the synthesis process has preserved the intended functionality of the RTL.

### RTL Waveform

```markdown
![RTL Waveform](images/pre_synth_babysoc.png)
```

The matching behavior of the important signals demonstrates that the synthesized implementation continues to perform the intended function for the applied testbench.

---

# ⏱️ 10. Functional GLS vs Timing GLS

Gate-Level Simulation can be broadly considered in two forms.

## Functional Gate-Level Simulation

Functional GLS checks whether the synthesized gates perform the required **logical operations**.

This is the type of gate-level simulation performed in this project.

```text
RTL
 ↓
Synthesis
 ↓
Gate-Level Netlist
 ↓
Functional GLS
 ↓
Logical Verification
```

The main focus is functional equivalence of the synthesized implementation for the applied testbench.

## Timing Gate-Level Simulation

Timing GLS additionally considers **cell and interconnect delays**.

It can be used to investigate timing-related behavior, including setup and hold problems.

Timing analysis is not part of the current stage of this project. The next stage of the flow is **Static Timing Analysis (STA)**.

---

# 🛠️ 11. Tools and Technologies

The following tools and technologies were used during the BabySoC implementation:

| Tool / Technology  | Purpose                             |
| ------------------ | ----------------------------------- |
| **Verilog HDL**    | RTL design                          |
| **Icarus Verilog** | RTL and gate-level simulation       |
| **GTKWave**        | Waveform analysis                   |
| **Yosys**          | Logic synthesis                     |
| **ABC**            | Technology mapping and optimization |
| **SKY130**         | Standard-cell technology            |
| **Liberty `.lib`** | Cell and timing information         |
| **Linux**          | Development environment             |

Each tool plays a specific role in moving the design from RTL toward a technology-mapped implementation.

---

# 📈 12. Current Project Status

The BabySoC implementation has successfully progressed through the following stages:

```text
RTL Design
     ↓
Pre-Synthesis Simulation
     ↓
Yosys Synthesis
     ↓
SKY130 Technology Mapping
     ↓
Gate-Level Netlist
     ↓
Post-Synthesis Simulation
     ↓
Functional Verification
```

### ✅ Current Milestone

**RTL → Post-Synthesis Gate-Level Simulation**

The next major stage is:

```text
Static Timing Analysis
        ↓
Floorplanning
        ↓
Placement
        ↓
Clock Tree Synthesis
        ↓
Routing
        ↓
Physical Verification
        ↓
GDSII Generation
```

---

# 💡 13. Key Learnings

Working through the BabySoC flow provided practical insight into several important concepts in ASIC design.

### 1. Understanding Design Hierarchy

Even a relatively small SoC consists of multiple interconnected blocks.

Understanding the relationship between the **CPU, PLL, and DAC** makes it easier to trace signals and debug the design.

### 2. Synthesis Is More Than RTL-to-Gates Conversion

Synthesis involves several stages, including:

* Logic optimization
* Sequential-cell mapping
* Technology mapping
* Netlist generation

Each stage contributes to transforming the RTL into an optimized hardware implementation.

### 3. Standard Cells Make the Hardware Concrete

At the RTL level, the design is described using hardware constructs.

After technology mapping, those constructs become actual **SKY130 standard-cell instances**.

Inspecting the generated netlist provides a clearer picture of the hardware represented by the RTL.

### 4. Simulation Before and After Synthesis Is Important

Pre-synthesis simulation establishes the expected RTL behavior.

Post-synthesis simulation then checks whether the synthesized implementation continues to produce the expected behavior.

This comparison helps identify synthesis-related functional issues.

### 5. Functional GLS Is Not the End of Verification

Functional GLS checks logical behavior, but it does not establish complete timing closure.

Additional timing analysis is required to determine whether the design satisfies its timing requirements.

---

# 📝 Conclusion

The BabySoC project provides a practical introduction to the **front-end ASIC design flow**, starting with RTL and progressing through simulation, synthesis, optimization, technology mapping, and gate-level verification.

The project first established the expected functionality of the original RTL design through pre-synthesis simulation. The design was then synthesized using **Yosys** and mapped to the **SKY130 standard-cell library**.

The resulting gate-level netlist was subsequently simulated using SKY130 cell models. The behavior of important signals was compared with the original RTL simulation to verify that the synthesis process preserved the intended functionality.

The complete flow can be summarized as:

```text
              BabySoC RTL
                   ↓
       Pre-Synthesis Simulation
                   ↓
            Yosys Synthesis
                   ↓
          Logic Optimization
                   ↓
       SKY130 Technology Mapping
                   ↓
         Gate-Level Netlist
                   ↓
      Post-Synthesis Simulation
                   ↓
       RTL vs GLS Comparison
                   ↓
        Functional Verification ✓
```

This experiment provides hands-on understanding of how a digital SoC moves from an **RTL description to a technology-mapped gate-level implementation**.

The next step in the ASIC flow is **Static Timing Analysis**, followed by physical-design stages such as floorplanning, placement, clock-tree synthesis, routing, physical verification, and finally GDSII generation.
