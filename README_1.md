
# 🏗️ ASIC Synthesis & Physical Design — File/Data Flow

## 📌 Part 1 — Complete Flow, RTL & Synthesis

This repository contains a detailed reference for the **files, data, and information exchanged throughout the ASIC Synthesis and Physical Design flow**.

The complete flow is:

> **RTL → Synthesis → Floorplan → Placement → CTS → Routing → Extraction → STA → Signoff → GDS**

---

# 📚 Table of Contents

- [1. Complete ASIC Flow](#1-complete-asic-flow)
- [2. Types of Data in an ASIC Flow](#2-types-of-data-in-an-asic-flow)
- [3. RTL](#3-rtl)
- [4. RTL Information](#4-rtl-information)
- [5. RTL Hierarchy](#5-rtl-hierarchy)
- [6. RTL vs Netlist](#6-rtl-vs-netlist)
- [7. Synthesis](#7-synthesis)
- [8. Major Synthesis Inputs](#8-major-synthesis-inputs)
- [9. RTL + .lib + .sdc + .upf](#9-rtl--lib--sdc--upf)
- [10. Synthesis Output](#10-synthesis-output)
- [11. Complete Synthesis Data Flow](#11-complete-synthesis-data-flow)
- [12. Important Mental Model](#12-important-mental-model)

---

# 1. 🔄 Complete ASIC Flow

The overall ASIC implementation flow can be represented as:

```text
                         RTL
                      .v / .sv
                          |
                          v
                  RTL ELABORATION
                          |
                          v
                   SYNTHESIS
                          |
             +------------+------------+
             |            |            |
            .lib         .sdc         .upf
             |            |            |
             +------------+------------+
                          |
                          v
                 GATE-LEVEL NETLIST
                          |
                          v
                    FLOORPLAN
                          |
                          v
                     PLACEMENT
                          |
                          v
                        CTS
                          |
                          v
                      ROUTING
                          |
                          v
                         DEF
                          |
                          v
               PARASITIC EXTRACTION
                          |
                          v
                        SPEF
                          |
                          v
                    POST-ROUTE STA
                          |
                          v
                       SIGNOFF
                          |
                          v
                    GDSII / OASIS
                          |
                          v
                       TAPEOUT
```

---

# 2. 🗂️ Types of Data in an ASIC Flow

Not every file in an ASIC flow represents the same type of information.

```text
ASIC FLOW DATA
│
├── Design Description
│   └── RTL
│
├── Logical Library Data
│   └── .lib
│
├── Timing Constraints
│   └── .sdc
│
├── Power Intent
│   └── .upf
│
├── Physical Abstract Data
│   └── LEF
│
├── Design Physical Data
│   └── DEF
│
├── Parasitic Data
│   └── SPEF
│
├── Timing Annotation
│   └── SDF
│
├── Detailed Physical Layout
│   └── GDSII / OASIS
│
├── Technology Data
│   ├── Technology LEF
│   ├── RC Models
│   ├── TLU+
│   ├── ITF
│   └── Technology files
│
└── Signoff Data
    ├── STA
    ├── DRC
    ├── LVS
    ├── IR Drop
    └── EM
```

---

# 3. 📝 RTL

## 3.1 What is RTL?

RTL stands for:

> **Register Transfer Level**

RTL describes the **functional behavior and architectural structure of a digital design** using a hardware description language.

Common RTL languages:

```text
Verilog
SystemVerilog
VHDL
```

Typical RTL files:

```text
*.v
*.sv
*.vhd
```

---

## 3.2 What Does RTL Describe?

RTL describes:

```text
Inputs
Outputs
Registers
Combinational Logic
Sequential Logic
State Machines
Clocking
Reset
Datapaths
Control Logic
Module Hierarchy
```

Example:

```verilog
module counter (
    input  logic       clk,
    input  logic       rst,
    output logic [7:0] count
);

always_ff @(posedge clk) begin
    if (rst)
        count <= 8'b0;
    else
        count <= count + 1'b1;
end

endmodule
```

This tells synthesis:

```text
There is an 8-bit register.
The register is clocked by clk.
Reset initializes the register.
The register increments every clock cycle.
```

But RTL does NOT directly specify:

```text
Which standard-cell DFF should be used?
Where should the DFF be placed?
Which metal layer should connect it?
What is the wire resistance?
What is the wire capacitance?
What is the final physical geometry?
```

Those decisions happen later.

---

# 3.3 RTL Is Technology Independent

Consider:

```verilog
assign y = a & b;
```

The RTL says:

```text
y = a AND b
```

It does not say:

```text
Use AND2_X1
```

or:

```text
Use AND2_X4
```

or:

```text
Use a particular transistor implementation
```

The synthesis tool chooses an implementation from the available technology library.

Therefore:

```text
RTL
 |
 | Functional description
 v
Synthesis
 |
 | Technology mapping
 v
Standard Cells
```

---

# 3.4 RTL Example

```verilog
module alu (
    input  logic [31:0] a,
    input  logic [31:0] b,
    input  logic [2:0]  op,
    output logic [31:0] y
);

always_comb begin

    case (op)

        3'b000:
            y = a + b;

        3'b001:
            y = a - b;

        3'b010:
            y = a & b;

        3'b011:
            y = a | b;

        3'b100:
            y = a ^ b;

        default:
            y = 32'b0;

    endcase

end

endmodule
```

The RTL contains:

```text
Module
Inputs
Outputs
ALU operations
Control signal
Combinational logic
```

---

# 4. 📋 RTL Information

A typical RTL design contains the following information.

## 4.1 Module Information

```verilog
module alu (
    ...
);
```

This defines:

```text
Module Name
Port Interface
```

---

## 4.2 Port Information

Example:

```verilog
input  logic [31:0] a;
input  logic [31:0] b;
output logic [31:0] y;
```

Contains:

```text
Port Name
Direction
Width
Data Type
```

---

## 4.3 Internal Signals

Example:

```verilog
logic [31:0] result;
logic        valid;
```

These represent internal logical signals.

They do not necessarily correspond one-to-one with physical wires after synthesis.

Synthesis may:

```text
Preserve
Rename
Merge
Remove
Optimize
```

signals depending on optimization.

---

## 4.4 Sequential Logic

Example:

```verilog
always_ff @(posedge clk) begin
    q <= d;
end
```

This represents register behavior.

The RTL does not explicitly say:

```text
DFF_X1
```

Instead:

```text
always_ff
    |
    v
Register behavior
    |
    v
Synthesis
    |
    v
DFF_X1 / DFF_X2 / DFF_X4 / ...
```

The final cell choice depends on:

```text
Timing
Area
Power
Fanout
Load
Library Availability
Optimization
```

---

## 4.5 Combinational Logic

Example:

```verilog
assign y = (a & b) | c;
```

The synthesis tool may implement it using:

```text
AND
+
OR
```

or optimize it into a different equivalent implementation.

The final implementation depends on:

```text
Logic Optimization
Technology Library
Timing Constraints
Area Constraints
Power Constraints
Synthesis Strategy
```

---

## 4.6 State Machines

RTL can describe FSMs.

Example:

```text
              +------+
              |      |
              v      |
            IDLE --> READ
              ^        |
              |        v
              +------ WRITE
```

The synthesis tool eventually implements the FSM using:

```text
Flip-Flops
+
Combinational Logic
```

---

## 4.7 Parameters

RTL can contain:

```verilog
parameter WIDTH = 32;
```

Parameters allow reusable designs.

During elaboration, parameter values are resolved.

---

## 4.8 Generate Blocks

RTL can contain:

```verilog
genvar i;

generate
    for (i = 0; i < 16; i = i + 1) begin
        assign y[i] = a[i] & b[i];
    end
endgenerate
```

Generate constructs are elaborated into actual hardware structures before the final synthesized netlist is produced.

---

# 5. 🏛️ RTL Hierarchy

Large designs are hierarchical.

Example:

```text
TOP
│
├── CPU
│   ├── ALU
│   ├── Register_File
│   ├── Control
│   └── LSU
│
├── CACHE
│   ├── Tag_Array
│   └── Data_Array
│
├── PERIPHERALS
│   ├── UART
│   ├── SPI
│   └── GPIO
│
└── PLL
```

The top-level module is generally the starting point for synthesis.

---

## 5.1 Why Hierarchy Matters

Hierarchy enables:

```text
Design Partitioning
IP Reuse
Block-Level Synthesis
Hierarchical Physical Design
Timing Analysis
Power Analysis
Verification
```

For example:

```text
TOP
│
├── CPU
├── SRAM
└── PERIPHERAL
```

Some blocks may eventually become:

```text
Hard Macros
Soft Blocks
Hierarchical Blocks
```

---

# 6. 🔄 RTL vs Netlist

This distinction is extremely important.

## RTL

```verilog
assign y = (a & b) | c;
```

RTL answers:

> **What logic should the circuit perform?**

---

## Gate-Level Netlist

A synthesized implementation could look like:

```verilog
module example (
    input  a,
    input  b,
    input  c,
    output y
);

wire n1;

AND2_X1 U1 (
    .A(a),
    .B(b),
    .Y(n1)
);

OR2_X1 U2 (
    .A(n1),
    .B(c),
    .Y(y)
);

endmodule
```

The netlist answers:

> **Which technology-library cells implement the logic and how are they logically connected?**

---

## 6.1 RTL → Netlist

```text
                    RTL
                     |
                     v
                 Elaboration
                     |
                     v
              Generic Logic
                     |
                     v
             Logic Optimization
                     |
                     v
            Technology Mapping
                     |
                     v
             Gate-Level Netlist
```

---

# 7. 🔨 Synthesis

## 7.1 What is Synthesis?

Synthesis converts:

```text
RTL
```

into:

```text
Technology-Mapped Gate-Level Netlist
```

while attempting to satisfy:

```text
Timing
Area
Power
Design Rules
Functional Equivalence
```

---

# 7.2 Main Synthesis Stages

A simplified synthesis flow is:

```text
RTL
 |
 v
READ / ANALYZE
 |
 v
ELABORATION
 |
 v
GENERIC REPRESENTATION
 |
 v
LOGIC OPTIMIZATION
 |
 v
TECHNOLOGY MAPPING
 |
 v
TIMING OPTIMIZATION
 |
 v
AREA / POWER OPTIMIZATION
 |
 v
GATE-LEVEL NETLIST
```

---

# 7.3 RTL Reading

The synthesis tool first reads the RTL.

Example:

```tcl
read_verilog {
    alu.sv
    register_file.sv
    control.sv
    cpu.sv
}
```

The tool parses:

```text
Modules
Ports
Signals
Expressions
Always blocks
Assignments
Generate constructs
Parameters
```

---

# 7.4 Elaboration

Elaboration resolves the actual design structure.

It determines:

```text
Top Module
Module Instances
Parameters
Generate Blocks
Widths
Constant Expressions
Hierarchical Connectivity
```

Example:

```text
TOP
 |
 +-- ALU
 |
 +-- Register File
 |
 +-- Control
```

After elaboration, the tool has a complete logical representation of the design.

---

# 7.5 Generic Logic

Before technology mapping, the tool may represent logic using generic Boolean operators.

Example:

```text
RTL:

y = (a & b) | c
```

Generic representation:

```text
      AND
     /   \
    a     b
     \   /
      AND
        \
         OR ---- y
        /
       c
```

At this stage the design is not necessarily mapped to a specific standard-cell library.

---

# 7.6 Logic Optimization

The synthesis tool performs optimizations such as:

```text
Boolean Optimization
Constant Propagation
Dead Logic Removal
Redundant Logic Removal
Logic Factoring
Common Subexpression Optimization
MUX Optimization
Arithmetic Optimization
Register Optimization
```

Example:

```verilog
assign y = a & 1'b1;
```

can become:

```verilog
assign y = a;
```

The unnecessary AND operation is removed.

---

# 7.7 Technology Mapping

After generic optimization, logic is mapped to cells from the target library.

Example:

```text
Generic AND
    |
    v
AND2_X1
```

or:

```text
Generic AND
    |
    v
AND2_X2
```

depending on:

```text
Timing
Load
Power
Area
```

---

# 7.8 Cell Sizing

Suppose the library contains:

```text
BUF_X1
BUF_X2
BUF_X4
BUF_X8
```

The synthesis tool can choose different drive strengths.

```text
BUF_X1
  |
  | smaller
  v
BUF_X2
  |
  v
BUF_X4
  |
  | stronger
  v
BUF_X8
```

Larger cells generally provide stronger drive but may increase:

```text
Area
Power
Input Capacitance
```

---

# 7.9 Timing Optimization

Suppose:

```text
FF1 → Logic → FF2
```

has negative slack.

The synthesis tool can attempt:

```text
Cell Upsizing
Buffer Insertion
Logic Restructuring
Logic Replication
Path Optimization
```

Example:

```text
Before:

FF1 → NAND_X1 → NAND_X1 → FF2
```

Possible optimization:

```text
After:

FF1 → NAND_X4 → NAND_X4 → FF2
```

or restructuring the logic.

---

# 7.10 Area Optimization

If area is excessive, synthesis may attempt:

```text
Cell Downsizing
Logic Sharing
Logic Simplification
Register Optimization
Buffer Reduction
```

The challenge is:

```text
Timing
   ↕
Area
   ↕
Power
```

Improving one metric can hurt another.

---

# 7.11 Power Optimization

Power-aware synthesis may optimize:

```text
Switching Activity
Cell Selection
Clock Gating
Buffering
Logic Structure
Leakage
```

Power can be broadly divided into:

```text
Dynamic Power
+
Static / Leakage Power
```

---

# 8. 📂 Major Synthesis Inputs

The major inputs to synthesis are:

```text
RTL
.lib
.sdc
.upf
```

Depending on methodology, additional information may be required.

A simplified view:

```text
             RTL
              |
              |
      +-------+-------+
      |       |       |
     .lib    .sdc    .upf
      |       |       |
      +-------+-------+
              |
              v
          SYNTHESIS
              |
              v
       GATE-LEVEL NETLIST
```

---

# 8.1 RTL

Answers:

> **What functionality should be implemented?**

---

# 8.2 `.lib`

Answers:

> **What cells are available and how do they behave electrically?**

---

# 8.3 `.sdc`

Answers:

> **What timing requirements must the design satisfy?**

---

# 8.4 `.upf`

Answers:

> **What power architecture should the design implement?**

---

# 9. 🔗 RTL + .lib + .sdc + .upf

These files provide different dimensions of information.

```text
                     DESIGN
                       |
       +---------------+---------------+
       |               |               |
      RTL             .lib            .sdc
       |               |               |
       |               |               |
       v               v               v
  Functionality   Cell Capability   Requirements
       |
       |
      .upf
       |
       v
   Power Intent
```

---

## 9.1 RTL

```text
WHAT should the circuit do?
```

---

## 9.2 `.lib`

```text
WHAT cells exist?
HOW fast are they?
HOW much power do they consume?
WHAT loads can they drive?
WHAT timing constraints do they have?
```

---

## 9.3 `.sdc`

```text
WHAT timing must the DESIGN achieve?
```

---

## 9.4 `.upf`

```text
HOW should POWER be managed?
```

---

# 10. 📤 Synthesis Output

The most important synthesis output is:

```text
Gate-Level Netlist
```

But synthesis also generates several reports and databases.

Typical outputs:

```text
Synthesis Outputs
│
├── Gate-Level Netlist
│
├── Timing Reports
│
├── Area Reports
│
├── Power Reports
│
├── QoR Reports
│
├── Constraint Reports
│
├── Design Check Reports
│
└── Synthesis Database
```

---

# 10.1 Gate-Level Netlist

Example:

```verilog
module top (
    input  clk,
    input  a,
    input  b,
    output y
);

wire n1;

AND2_X1 U1 (
    .A(a),
    .B(b),
    .Y(n1)
);

DFF_X1 U2 (
    .D(n1),
    .CK(clk),
    .Q(y)
);

endmodule
```

The netlist now contains actual library cells.

---

# 10.2 Timing Report

A timing report may contain:

```text
Startpoint
Endpoint
Path Type
Clock
Data Arrival Time
Data Required Time
Slack
Cell Delays
Net Delays
```

Example:

```text
Startpoint: FF1/Q
Endpoint:   FF2/D

Path:

FF1/Q
  |
  v
U1
  |
  v
U2
  |
  v
FF2/D

Data Arrival Time = 1.42 ns
Required Time      = 1.80 ns

Slack = 0.38 ns
```

---

# 10.3 Area Report

Area reports can include:

```text
Combinational Area
Sequential Area
Buffer Area
Inverter Area
Total Cell Area
Number of Cells
```

Example:

```text
Combinational Area : 1200 um²
Sequential Area    : 800 um²
Buffer Area        : 300 um²
--------------------------------
Total Area         : 2300 um²
```

Exact report categories depend on the tool.

---

# 10.4 Power Report

Power reports can contain:

```text
Internal Power
Switching Power
Leakage Power
Total Power
```

Conceptually:

```text
Total Power
    |
    +-- Dynamic Power
    |     |
    |     +-- Internal
    |     +-- Switching
    |
    +-- Leakage Power
```

---

# 10.5 QoR Report

QoR = **Quality of Results**

A QoR report may summarize:

```text
Area
Timing
Power
Violations
Cell Count
Buffer Count
Sequential Count
```

Example:

```text
Timing
-------
WNS = +0.12 ns
TNS =  0.00 ns

Area
-----
Total = 12,500 um²

Power
------
Total = 8.2 mW
```

---

# 10.6 Constraint Report

A constraint report can identify:

```text
Unconstrained Paths
Missing Clocks
Invalid Constraints
Ignored Constraints
Input Delay Coverage
Output Delay Coverage
Clock Relationships
```

This is extremely important because a design can appear to have good timing simply because paths were not constrained correctly.

---

# 11. 🔄 Complete Synthesis Data Flow

```text
                         RTL
                       .v / .sv
                           |
                           v
                    RTL ELABORATION
                           |
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
           .lib          .sdc          .upf
             |             |             |
             |             |             |
             +-------------+-------------+
                           |
                           v
                    LOGIC SYNTHESIS
                           |
                           v
                  GENERIC OPTIMIZATION
                           |
                           v
                  TECHNOLOGY MAPPING
                           |
                           v
                   TIMING OPTIMIZATION
                           |
                           v
                    AREA OPTIMIZATION
                           |
                           v
                   POWER OPTIMIZATION
                           |
                           v
                  GATE-LEVEL NETLIST
                           |
              +------------+------------+
              |            |            |
              v            v            v
           Timing        Area         Power
           Report        Report       Report
```

---

# 12. 🧠 Important Mental Model

The easiest way to remember the beginning of the ASIC flow is:

```text
RTL
 |
 | "What should the circuit do?"
 |
 v
Synthesis
 |
 | Uses:
 |    .lib → "What cells can I use?"
 |    .sdc → "What timing must I meet?"
 |    .upf → "What power intent must I implement?"
 |
 v
Gate-Level Netlist
 |
 | "Which library cells implement the logic?"
 |
 v
Physical Design
```

---

# ⭐ Key Takeaways

| File | Main Question |
|---|---|
| `.v / .sv` | What should the design do? |
| `.lib` | How do the available cells behave? |
| `.sdc` | What timing must the design satisfy? |
| `.upf` | What is the intended power architecture? |
| Netlist | Which cells implement the design? |

---

# 🎯 Interview-Level Summary

### RTL

> RTL is a technology-independent description of the functionality and register-transfer behavior of a digital design.

### Synthesis

> Synthesis converts RTL into a technology-mapped gate-level netlist while optimizing for timing, area, power, and design constraints.

### `.lib`

> Liberty describes the logical, timing, power, and electrical characteristics of library cells.

### `.sdc`

> SDC defines the timing constraints and requirements imposed on the design.

### `.upf`

> UPF describes the power intent, including power domains, isolation, retention, level shifting, and power states.

### Netlist

> The gate-level netlist represents the logical connectivity of technology-specific standard cells after synthesis.

---

# 🔑 Core Relationship

```text
                    RTL
                     |
             "Functionality"
                     |
                     v
                  SYNTHESIS
                     ^
          +----------+----------+
          |          |          |
        .lib       .sdc       .upf
          |          |          |
      "Cell       "Timing"   "Power"
     Capability"  "Need"     "Intent"
          |          |          |
          +----------+----------+
                     |
                     v
             GATE-LEVEL NETLIST
                     |
                     v
              PHYSICAL DESIGN
```

---

# 🚀 Next Part

The next section goes into **`.lib` in extreme depth**, including:

```text
Cell
Pin
Direction
Function
Timing Arc
Timing Sense
Related Pin
Rise/Fall Delay
Transition
Slew
Capacitance
Setup
Hold
Recovery
Removal
Clock-to-Q
Min Pulse Width
Max Transition
Max Capacitance
Fanout
Internal Power
Switching Power
Leakage Power
PVT
Operating Conditions
Lookup Tables
NLDM
CCS
ECSM
LVF
OCV/AOCV/POCV interaction
```

> **Part 2 → Liberty `.lib` in extreme depth**
# 🏗️ ASIC Synthesis & Physical Design — File/Data Flow

## 📌 Part 2 — Liberty `.lib` in Extreme Depth

The Liberty file is one of the **most important files in the entire ASIC flow**.

It provides the synthesis, STA, optimization, and physical-design tools with the **logical, timing, electrical, power, and variation information of library cells**.

The key idea is:

> **LEF tells the tool what a cell physically looks like. `.lib` tells the tool how that cell electrically and logically behaves.**

---

# 📚 Table of Contents

- [1. What is a Liberty File?](#1-what-is-a-liberty-file)
- [2. Why `.lib` is Required](#2-why-lib-is-required)
- [3. What Information Does `.lib` Contain?](#3-what-information-does-lib-contain)
- [4. Liberty Hierarchy](#4-liberty-hierarchy)
- [5. Library](#5-library)
- [6. Operating Conditions](#6-operating-conditions)
- [7. Cell](#7-cell)
- [8. Cell Area](#8-cell-area)
- [9. Pins](#9-pins)
- [10. Input Pin Information](#10-input-pin-information)
- [11. Output Pin Information](#11-output-pin-information)
- [12. Logic Function](#12-logic-function)
- [13. Timing Arcs](#13-timing-arcs)
- [14. Timing Sense](#14-timing-sense)
- [15. Timing Tables](#15-timing-tables)
- [16. Cell Delay](#16-cell-delay)
- [17. Slew / Transition](#17-slew--transition)
- [18. Input Capacitance](#18-input-capacitance)
- [19. Output Capacitance](#19-output-capacitance)
- [20. Setup and Hold](#20-setup-and-hold)
- [21. Clock-to-Q](#21-clock-to-q)
- [22. Recovery and Removal](#22-recovery-and-removal)
- [23. Minimum Pulse Width](#23-minimum-pulse-width)
- [24. Maximum Transition](#24-maximum-transition)
- [25. Maximum Capacitance](#25-maximum-capacitance)
- [26. Maximum Fanout](#26-maximum-fanout)
- [27. Power Information](#27-power-information)
- [28. Leakage Power](#28-leakage-power)
- [29. Internal Power](#29-internal-power)
- [30. Switching Power](#30-switching-power)
- [31. PVT Corners](#31-pvt-corners)
- [32. Slow/Fast Libraries](#32-slowfast-libraries)
- [33. NLDM](#33-nldm)
- [34. CCS](#34-ccs)
- [35. ECSM](#35-ecsm)
- [36. LVF](#36-lvf)
- [37. `.lib` and OCV](#37-lib-and-ocv)
- [38. `.lib` vs LEF](#38-lib-vs-lef)
- [39. `.lib` vs SDC](#39-lib-vs-sdc)
- [40. How STA Uses `.lib`](#40-how-sta-uses-lib)
- [41. How Synthesis Uses `.lib`](#41-how-synthesis-uses-lib)
- [42. Complete `.lib` Mental Model](#42-complete-lib-mental-model)

---

# 1. 📖 What is a Liberty File?

A Liberty file normally has the extension:

```text
.lib
```

Liberty is a **standard format for describing the characteristics of semiconductor library cells**.

A standard-cell library may contain:

```text
AND
OR
NAND
NOR
XOR
MUX
INV
BUF
DFF
Latch
Clock-gating cells
Tie cells
Isolation cells
Level shifters
Retention cells
Special cells
```

For example:

```text
INV_X1
INV_X2
INV_X4

NAND2_X1
NAND2_X2
NAND2_X4

DFF_X1
DFF_X2
DFF_X4
```

The `.lib` describes the behavior of each of these cells.

---

# 2. 🎯 Why `.lib` is Required

Suppose synthesis sees:

```text
a ----\
       AND ---- y
b ----/
```

The RTL only tells synthesis:

```text
Perform AND operation.
```

But synthesis needs to know:

```text
Which AND cells exist?
How much area does each cell occupy?
How fast is each cell?
How much input capacitance does it have?
How much power does it consume?
How much load can it drive?
What happens at different slews?
What happens at different loads?
```

That information comes from:

```text
.lib
```

Therefore:

```text
RTL
 |
 | Function
 v
Synthesis
 ^
 |
.lib
 |
 | Cell characteristics
```

---

# 3. 📋 What Information Does `.lib` Contain?

A `.lib` can contain information about:

```text
LOGICAL
├── Cell name
├── Pin name
├── Pin direction
├── Logic function
└── Timing relationships

TIMING
├── Cell delay
├── Output transition
├── Setup
├── Hold
├── Clock-to-Q
├── Recovery
├── Removal
└── Minimum pulse width

ELECTRICAL
├── Input capacitance
├── Output capacitance
├── Maximum capacitance
├── Maximum transition
└── Maximum fanout

POWER
├── Leakage power
├── Internal power
└── Switching-related power

TECHNOLOGY / PVT
├── Process
├── Voltage
└── Temperature

VARIATION
├── OCV-related information
├── AOCV-related information
├── POCV-related information
└── LVF information
```

---

# 4. 🌳 Liberty Hierarchy

A simplified `.lib` structure looks like:

```text
library
│
├── library attributes
│
├── operating_conditions
│
├── wire_load
│
├── cell
│   │
│   ├── cell attributes
│   │
│   ├── pin
│   │   ├── direction
│   │   ├── capacitance
│   │   └── function
│   │
│   ├── pin
│   │   ├── direction
│   │   ├── capacitance
│   │   └── timing
│   │
│   └── internal_power
│
├── cell
│   └── ...
│
└── cell
    └── ...
```

The basic hierarchy is:

```text
Library
  |
  +-- Cell
       |
       +-- Pin
       |
       +-- Timing
       |
       +-- Power
```

---

# 5. 📚 Library

A Liberty file begins with a library definition.

Conceptually:

```liberty
library("my_library") {

    ...

}
```

The library contains all characterized cells.

For example:

```text
MY_LIBRARY
│
├── INV_X1
├── INV_X2
├── INV_X4
├── NAND2_X1
├── NAND2_X2
├── NAND2_X4
├── DFF_X1
├── DFF_X2
└── ...
```

---

# 5.1 Library-Level Information

Library-level information can include:

```text
Library Name
Technology
Units
Voltage
Current
Power
Time
Capacitance
Operating Conditions
Default Attributes
Wire-load Information
Variation Information
```

Units are important because the tools need to know what numerical values mean.

For example:

```text
time        : ns
capacitance : pF
voltage     : V
power       : mW
```

The exact units depend on the library.

---

# 6. 🌡️ Operating Conditions

A library is characterized under a specific operating condition.

The major PVT parameters are:

```text
P = Process
V = Voltage
T = Temperature
```

Therefore:

```text
PVT
 |
 +-- Process
 +-- Voltage
 +-- Temperature
```

Example:

```text
SS
0.72 V
125 C
```

This represents a slow process corner at a specified voltage and temperature.

---

# 6.1 Why PVT Matters

Cell delay changes with PVT.

For example:

```text
Slow process
    +
Low voltage
    +
High temperature
    |
    v
Usually slower cells
```

Whereas:

```text
Fast process
    +
High voltage
    +
Low temperature
    |
    v
Usually faster cells
```

Therefore different `.lib` files are commonly used for different timing corners.

---

# 7. 🧱 Cell

A `cell` block describes one library cell.

Example concept:

```liberty
cell("INV_X1") {

    area : 1.0;

    pin("A") {
        direction : input;
        capacitance : 0.002;
    }

    pin("Y") {
        direction : output;
        function : "!A";
    }
}
```

This tells the tool:

```text
Cell Name = INV_X1
Area = 1.0

Input:
    A

Output:
    Y

Logic:
    Y = NOT(A)
```

---

# 7.1 Cell Name

Example:

```text
INV_X1
INV_X2
INV_X4
```

The naming convention is library-specific.

Typically:

```text
INV
```

means inverter.

```text
X1
X2
X4
```

usually indicates different drive strengths.

It is important to remember:

> The exact meaning of `X1`, `X2`, `X4`, etc. is defined by the library vendor and should not be assumed solely from the name.

---

# 8. 📐 Cell Area

A cell can contain an area attribute.

Conceptually:

```liberty
area : 1.25;
```

This represents the characterized physical cell area according to the library.

Synthesis uses area information during optimization.

Example:

```text
INV_X1 → smaller
INV_X2 → larger
INV_X4 → larger
```

Generally:

```text
Larger Drive Strength
        |
        +--> More Area
        +--> More Input Capacitance
        +--> Potentially More Power
        +--> Potentially Better Timing
```

This creates the classic optimization trade-off:

```text
Timing ↔ Area ↔ Power
```

---

# 9. 🔌 Pins

Each cell contains pins.

Example:

```text
NAND2

A ----\
       NAND ---- Y
B ----/
```

Pins:

```text
A
B
Y
```

Each pin can have attributes.

For example:

```text
Direction
Capacitance
Function
Timing
Power
```

---

# 10. 📥 Input Pin Information

An input pin may contain:

```liberty
pin("A") {

    direction : input;

    capacitance : 0.003;

}
```

Important information:

```text
Pin Name
Direction
Input Capacitance
Max Transition
Clock-related attributes
Power information
```

---

# 10.1 Input Capacitance

Input capacitance represents the capacitive load presented by the cell input.

For example:

```text
Driver
  |
  +-----------> A of NAND2
                 |
                 |
              C_input
                 |
                GND
```

The driving cell sees this capacitance as part of its load.

Therefore:

```text
Higher input capacitance
        |
        v
Higher load on driver
        |
        v
Potentially larger driver delay
```

---

# 11. 📤 Output Pin Information

An output pin may contain:

```text
Direction
Logic Function
Timing
Max Capacitance
Max Transition
Power
```

Example:

```liberty
pin("Y") {

    direction : output;

    function : "!(A & B)";

}
```

---

# 12. 🧮 Logic Function

The `function` attribute describes the logical behavior of an output.

For an inverter:

```text
Y = !A
```

For NAND:

```text
Y = !(A & B)
```

For NOR:

```text
Y = !(A | B)
```

For XOR:

```text
Y = A ^ B
```

This information allows the tool to understand the logical behavior of the cell.

---

# 12.1 Why Function Matters

Suppose the netlist contains:

```text
NAND2_X1
```

The tool needs to know:

```text
Y = !(A & B)
```

This is required for:

```text
Logic optimization
Simulation models
STA logic propagation
Synthesis
Formal analysis
```

---

# 13. ⏱️ Timing Arcs

This is one of the most important concepts in `.lib`.

A timing arc describes how a change at one pin affects another pin.

Example:

```text
       A
       |
       v
    +------+
B ->| NAND |-> Y
    +------+
```

Timing arcs:

```text
A → Y
B → Y
```

Each arc can contain timing information.

---

# 13.1 Timing Arc Structure

Conceptually:

```text
Timing Arc
│
├── Related Pin
├── Timing Type
├── Timing Sense
├── Timing Tables
│   ├── Cell Rise
│   ├── Cell Fall
│   ├── Rise Transition
│   └── Fall Transition
└── Constraints, if applicable
```

---

# 13.2 Related Pin

For an output pin `Y`, the timing arc can specify the input pin that controls the timing relationship.

Conceptually:

```liberty
pin("Y") {

    timing() {

        related_pin : "A";

        ...

    }
}
```

This means:

```text
A → Y
```

---

# 14. 🔄 Timing Sense

Timing sense describes how the output responds logically to an input transition.

Common concepts include:

```text
positive_unate
negative_unate
non_unate
```

---

# 14.1 Positive Unate

Example:

```text
BUF

A ↑
 |
 v
Y ↑
```

Input rising tends to cause output rising.

Therefore:

```text
A ↑ → Y ↑
A ↓ → Y ↓
```

This is:

```text
Positive Unate
```

---

# 14.2 Negative Unate

Example:

```text
INV

A ↑
 |
 v
Y ↓
```

Therefore:

```text
A ↑ → Y ↓
A ↓ → Y ↑
```

This is:

```text
Negative Unate
```

---

# 14.3 Non-Unate

Some logic functions do not have a fixed relationship between input transition and output transition.

Example:

```text
XOR
```

For:

```text
Y = A XOR B
```

the effect of A depends on B.

Therefore XOR is generally treated as:

```text
Non-Unate
```

---

# 15. 📊 Timing Tables

Cell delay is not a single constant.

It depends primarily on:

```text
Input Slew
+
Output Load
```

Therefore libraries often use lookup tables.

Conceptually:

```text
                  Output Load
              0.01  0.02  0.05  0.10
             --------------------------------
Input Slew
  0.01       |  10    12    16    25
  0.02       |  12    14    18    27
  0.05       |  15    18    23    32
```

The STA tool uses interpolation to determine the appropriate value.

---

# 15.1 Why Two Variables?

Cell delay depends strongly on:

```text
Input Slew
```

and:

```text
Output Load
```

Consider:

```text
Fast input
+
Small load
=
Small delay
```

versus:

```text
Slow input
+
Large load
=
Large delay
```

Therefore one fixed delay number would be insufficient.

---

# 16. ⏱️ Cell Delay

Cell delay represents the propagation delay through a standard cell.

Consider:

```text
A ----> NAND ----> Y
```

If:

```text
A changes at t = 1.0 ns
```

and:

```text
Y changes at t = 1.08 ns
```

then approximately:

```text
Cell delay = 0.08 ns
```

But the actual value depends on:

```text
Input transition
Output capacitance
PVT
Cell type
Transition direction
```

---

# 16.1 Cell Rise

When output transitions:

```text
0 → 1
```

the library provides rise-delay information.

Conceptually:

```liberty
cell_rise(...)
```

---

# 16.2 Cell Fall

When output transitions:

```text
1 → 0
```

the library provides fall-delay information.

Conceptually:

```liberty
cell_fall(...)
```

Rise and fall delays are generally different.

---

# 17. 📈 Slew / Transition

Slew is the transition time of a signal.

For example:

```text
Voltage
  |
1 |            ______
  |           /
  |          /
  |         /
0 |________/
  |
  +--------------------> Time
```

The transition from low to high is not instantaneous.

The library characterizes output transition behavior.

---

# 17.1 Why Slew Matters

A slow signal can make downstream cells slower.

Example:

```text
FF
 |
 v
BUF
 |
 v
NAND
 |
 v
FF
```

If the BUF output has poor slew:

```text
Poor slew
   |
   v
NAND delay increases
   |
   v
Path delay increases
```

Therefore STA propagates slew through the timing path.

---

# 18. 🔌 Input Capacitance

Consider:

```text
        Driver
           |
           |
           v
      +----------+
      |   Cell   |
      |           |
      +----------+
           |
          C
```

The driver sees the input capacitance of the receiving cell.

Therefore total load can include:

```text
Pin Capacitance
+
Wire Capacitance
+
Other Loads
```

This directly affects delay.

---

# 19. 📤 Output Capacitance / Load

The output of a cell drives:

```text
Multiple Cell Inputs
+
Interconnect
```

Therefore:

```text
Total Load
=
Pin Capacitances
+
Wire Capacitance
```

Post-route:

```text
Total Load
=
Pin Capacitance
+
Extracted Wire Capacitance
```

This is why post-route timing can differ significantly from pre-route timing.

---

# 20. ⏱️ Setup and Hold

Sequential cells have timing constraints.

Consider:

```text
            +------+
D --------->|      |
CLK -------->| DFF  |----> Q
            +------+
```

---

## 20.1 Setup Time

Data must be stable sufficiently **before** the active clock edge.

```text
             Setup
        <------------>
D  _________
           |___________
                    |
                    |
CLK ________________|↑
                    ^
                Active Edge
```

Setup violation occurs when:

```text
Data arrives too late
```

---

## 20.2 Hold Time

Data must remain stable sufficiently **after** the active clock edge.

```text
CLK ________________|↑________
                    ^
                    |
                 Active Edge

D  ______________________
                    <---->
                     Hold
```

Hold violation occurs when:

```text
Data changes too early
```

---

# 20.3 Important Difference

```text
Setup
    |
    +-- Data must arrive BEFORE capture edge

Hold
    |
    +-- Data must remain stable AFTER capture edge
```

This distinction is fundamental to STA.

---

# 21. ⏱️ Clock-to-Q

For a flip-flop:

```text
CLK ---> DFF ---> Q
          ^
          |
          D
```

After the active clock edge:

```text
CLK edge
   |
   v
Internal FF operation
   |
   v
Q changes
```

The delay between:

```text
Clock edge
```

and:

```text
Q transition
```

is:

```text
Clock-to-Q delay
```

The `.lib` characterizes this.

---

# 21.1 Clock-to-Q Example

Suppose:

```text
Clock edge = 10.00 ns
Q transition = 10.08 ns
```

Then:

```text
Clock-to-Q = 0.08 ns
```

Again, the actual delay depends on:

```text
Clock slew
Output load
PVT
Transition direction
```

---

# 22. 🔄 Recovery and Removal

These constraints are important for asynchronous control signals such as:

```text
Reset
Set
```

Consider:

```text
        +------+
D ----->|      |
RST --->| DFF  |----> Q
CLK --->|      |
        +------+
```

---

## 22.1 Recovery

Recovery checks the relationship between:

```text
Asynchronous control release
```

and:

```text
Active clock edge
```

It is somewhat analogous to setup for an asynchronous control signal.

---

## 22.2 Removal

Removal checks the relationship after the active clock edge.

It is somewhat analogous to hold for an asynchronous control signal.

Therefore:

```text
Recovery → Setup-like check
Removal  → Hold-like check
```

---

# 23. 📏 Minimum Pulse Width

Sequential and clock-related cells can have minimum pulse-width requirements.

For example:

```text
CLK

____        ______
    |______|
       ^
       |
    Pulse Width
```

If the pulse is too short:

```text
Circuit may not operate correctly.
```

The `.lib` can specify minimum required:

```text
High pulse width
Low pulse width
```

---

# 24. 📈 Maximum Transition

Maximum transition specifies the maximum acceptable signal transition time.

Example:

```text
Maximum transition = 100 ps
```

If a signal has:

```text
Transition = 150 ps
```

then:

```text
150 ps > 100 ps
```

and the design has a:

```text
Max Transition Violation
```

---

# 24.1 Why Maximum Transition Matters

Poor slew can cause:

```text
Higher delay
Higher dynamic power
Signal integrity problems
Timing degradation
```

Therefore synthesis and implementation tools try to fix transition violations.

Typical fixes:

```text
Upsize Driver
Insert Buffer
Reduce Fanout
Reduce Wire Length
```

---

# 25. 🔌 Maximum Capacitance

A library cell can specify the maximum capacitance that its output should drive.

Example:

```text
max_capacitance = 0.20 pF
```

Suppose:

```text
Actual Load = 0.30 pF
```

Then:

```text
0.30 pF > 0.20 pF
```

This is a:

```text
Max Capacitance Violation
```

Possible fixes:

```text
Upsize Driver
Buffer the Net
Reduce Fanout
```

---

# 26. 🔢 Maximum Fanout

Fanout represents the number of loads driven by a signal.

Example:

```text
             +--> FF1
             |
Driver ------+--> FF2
             |
             +--> FF3
             |
             +--> FF4
```

Fanout:

```text
4
```

Too much fanout can cause:

```text
Large capacitance
Poor slew
Large delay
High power
```

Possible fix:

```text
Buffer Tree
```

Example:

```text
                Driver
                   |
             +-----+-----+
             |           |
           BUF          BUF
           / \          / \
         FF1 FF2      FF3 FF4
```

---

# 27. ⚡ Power Information

Liberty can contain power characterization.

Major categories:

```text
Leakage Power
+
Internal Power
+
Switching Power
```

Conceptually:

```text
Total Power
│
├── Dynamic Power
│   ├── Internal Power
│   └── Switching Power
│
└── Leakage Power
```

The exact terminology and reporting decomposition can vary by tool and library methodology.

---

# 28. 🔋 Leakage Power

Leakage power occurs even when a cell is not actively switching.

It is related to:

```text
Transistor leakage
Threshold voltage
PVT
Input state
Cell structure
```

For example:

```text
INV_X1
```

may have different leakage depending on:

```text
Input = 0
Input = 1
```

Therefore libraries may characterize leakage for different input states.

---

# 29. ⚡ Internal Power

Internal power represents power consumed within a cell due to internal switching behavior.

For example:

```text
Input transition
      |
      v
Internal transistor switching
      |
      v
Internal power
```

This is different from the energy required to charge the external interconnect.

---

# 30. 🔌 Switching Power

Switching power is associated with charging and discharging capacitances.

The fundamental relationship is approximately:

```text
P_dynamic ∝ α C V² f
```

where:

```text
α = switching activity
C = capacitance
V = voltage
f = frequency
```

The capacitance can include:

```text
Cell input capacitance
Wire capacitance
Other load capacitances
```

---

# 31. 🌡️ PVT Corners

PVT means:

```text
Process
Voltage
Temperature
```

These conditions strongly affect cell behavior.

---

## 31.1 Process

Process variation affects transistor characteristics.

Common corner labels:

```text
SS
TT
FF
```

There can also be mixed corners such as:

```text
SF
FS
```

depending on the library and technology.

---

## 31.2 Voltage

Lower voltage generally reduces drive strength.

Example:

```text
0.88 V
0.80 V
0.72 V
```

---

## 31.3 Temperature

Temperature also changes device behavior.

Examples:

```text
-40°C
25°C
125°C
```

Exact signoff corners depend on the technology and foundry methodology.

---

# 32. 🐌⚡ Slow/Fast Libraries

A design is not analyzed using only one `.lib`.

Different timing corners can use different libraries.

Example:

```text
slow.lib
fast.lib
```

Conceptually:

```text
Slow Corner
    |
    v
Slow Library
    |
    v
Slow Cell Delays
```

and:

```text
Fast Corner
    |
    v
Fast Library
    |
    v
Fast Cell Delays
```

---

# 32.1 Setup and Hold Corners

A simplified rule of thumb is:

```text
Setup
  |
  +--> Usually stressed by slower data paths
```

and:

```text
Hold
  |
  +--> Usually stressed by faster data paths
```

But real signoff uses complete MMMC definitions and library/corner combinations rather than blindly assuming one universal corner.

---

# 33. 📊 NLDM

NLDM =

> **Non-Linear Delay Model**

This is a traditional cell timing model.

The basic concept is:

```text
Input Slew
     +
Output Load
     |
     v
Lookup Table
     |
     v
Cell Delay
```

---

## 33.1 NLDM Cell Rise

Conceptually:

```text
cell_rise() {

    index_1("input_slew");
    index_2("output_load");

    values(
        ...
    );
}
```

The table provides rise delay.

---

## 33.2 NLDM Cell Fall

Similarly:

```text
cell_fall()
```

provides fall delay.

---

## 33.3 NLDM Transition

The library can also characterize:

```text
rise_transition
fall_transition
```

These determine the output slew.

---

# 33.4 NLDM Limitation

NLDM essentially reduces complex waveform behavior into scalar quantities such as:

```text
Slew
Load
Delay
```

For advanced technologies, waveform-based models can provide more accuracy.

This is where:

```text
CCS
ECSM
```

become important.

---

# 34. 🌊 CCS

CCS =

> **Composite Current Source**

CCS provides a more detailed model of cell behavior using current-source based representations.

Instead of only saying:

```text
Input Slew
+
Load
=
Delay
```

the model can better represent:

```text
Voltage waveform
Current behavior
Output waveform
Input slew
Output loading
```

This can improve timing accuracy, especially for advanced nodes.

---

# 34.1 Why CCS Is More Detailed

Real signals are waveforms.

```text
Idealized:

      ______
     /
____/

Real waveform:

      ______
    /
  /
_/
```

The exact waveform can affect:

```text
Delay
Noise
Crosstalk
Receiver behavior
```

CCS can model these effects more accurately than a simple scalar-delay model.

---

# 35. 🌊 ECSM

ECSM =

> **Effective Current Source Model**

It is another advanced library characterization/modeling approach used to represent cell behavior more accurately than simple lookup-table delay models.

The exact implementation depends on the library/tool methodology.

The key idea is:

```text
NLDM
 ↓
Scalar lookup-table based timing

CCS / ECSM
 ↓
More detailed waveform/current-based modeling
```

---

# 36. 📈 LVF

LVF =

> **Library Variation Format**

LVF provides statistical variation information associated with library timing models.

It is especially important for:

```text
Statistical variation
POCV
Advanced variation-aware STA
```

Instead of having only:

```text
Nominal Delay
```

the model can include statistical information such as:

```text
Mean
Sigma
Variation
```

for relevant timing quantities.

---

# 36.1 Why LVF Matters

Suppose a cell delay is approximately:

```text
Mean = 50 ps
Sigma = 5 ps
```

Then variation can be statistically modeled.

Conceptually:

```text
Nominal delay
      |
      +---- variation
      |
      v
Statistical timing analysis
```

This is one of the foundations for modern variation-aware timing analysis.

---

# 37. 🔬 `.lib` and OCV

OCV stands for:

> **On-Chip Variation**

Real chips do not have identical transistors everywhere.

There can be local variations in:

```text
Process
Voltage
Temperature
```

Therefore two cells on the same chip may not have exactly identical delays.

---

## 37.1 Basic OCV

A simple OCV approach can apply derates.

Conceptually:

```text
Nominal Delay
      |
      v
+----------------+
| OCV Derate     |
+----------------+
      |
      v
Early / Late Delay
```

For example:

```text
Early = Nominal × Early Derate

Late  = Nominal × Late Derate
```

---

# 37.2 AOCV

AOCV:

> **Advanced On-Chip Variation**

AOCV considers additional path-related information such as:

```text
Logic depth
Path depth
Distance / spatial effects
```

instead of applying one simple global derate to everything.

---

# 37.3 POCV

POCV:

> **Parametric On-Chip Variation**

POCV models variation statistically.

Conceptually:

```text
Cell Delay
   |
   +-- Mean
   |
   +-- Sigma
   |
   v
Statistical variation
```

LVF can provide the statistical library data used by variation-aware timing methodologies.

---

# 38. 🔄 `.lib` vs LEF

This is a very important interview question.

## `.lib`

Describes:

```text
Logical behavior
Timing
Power
Electrical characteristics
Cell constraints
Variation data
```

---

## LEF

Describes:

```text
Physical abstraction
Cell dimensions
Pin locations
Pin shapes
Metal layers
Obstructions
Sites
Routing information
```

---

## Comparison

| Information | `.lib` | LEF |
|---|---:|---:|
| Logic function | ✅ | ❌ |
| Cell timing | ✅ | ❌ |
| Setup/Hold | ✅ | ❌ |
| Clock-to-Q | ✅ | ❌ |
| Power | ✅ | ❌ |
| Input capacitance | ✅ | ❌ |
| Cell width | ❌ | ✅ |
| Cell height | ❌ | ✅ |
| Pin physical location | ❌ | ✅ |
| Pin metal layer | ❌ | ✅ |
| Routing obstruction | ❌ | ✅ |
| Metal layers | ❌ | ✅ |

---

# 39. 🔗 `.lib` vs SDC

Another very important distinction.

## `.lib`

Describes:

> **What the cells CAN do.**

For example:

```text
DFF clock-to-Q = 80 ps
NAND delay = 30 ps
INV delay = 15 ps
```

---

## `.sdc`

Describes:

> **What the DESIGN MUST achieve.**

For example:

```tcl
create_clock -period 2.0 [get_ports clk]
```

This means:

```text
Design clock period = 2 ns
```

---

## Simple Mental Model

```text
.lib
 |
 | Cell capability
 v

"What can this cell do?"

.sdc
 |
 | Design requirement
 v

"What must my design achieve?"
```

---

# 40. ⏱️ How STA Uses `.lib`

STA needs to calculate path delay.

Consider:

```text
FF1
 |
 v
INV
 |
 v
NAND
 |
 v
BUF
 |
 v
FF2
```

STA needs:

```text
Clock-to-Q of FF1
+
Delay of INV
+
Delay of NAND
+
Delay of BUF
+
Setup time of FF2
```

Where does this information come from?

```text
.lib
```

---

# 40.1 STA Data Flow

```text
             Netlist
                |
                |
                v
             STA Tool
                ^
                |
       +--------+--------+
       |        |        |
      .lib     .sdc     SPEF
       |        |        |
       v        v        v
     Cell     Timing    Wire
    Timing   Requirements RC
       |        |        |
       +--------+--------+
                |
                v
          Timing Analysis
```

Before routing:

```text
Wire parasitics may be estimated.
```

After routing:

```text
SPEF provides extracted parasitics.
```

---

# 40.2 Cell Delay Calculation

Suppose:

```text
Input slew = 30 ps
Output load = 10 fF
```

STA looks up/interpolates the relevant library timing table.

Conceptually:

```text
Input Slew = 30 ps
       +
Output Load = 10 fF
       |
       v
.lib Timing Table
       |
       v
Cell Delay
```

---

# 40.3 Timing Path

Suppose:

```text
FF1 → INV → NAND → BUF → FF2
```

Then:

```text
Launch Clock
     |
     v
   FF1
     |
 Clock-to-Q
     |
     v
    INV
     |
 Cell Delay
     |
     v
   NAND
     |
 Cell Delay
     |
     v
    BUF
     |
 Cell Delay
     |
     v
   FF2
```

The `.lib` provides the cell-level timing information required for these calculations.

---

# 41. 🔨 How Synthesis Uses `.lib`

Synthesis uses `.lib` to decide which cells to instantiate.

Suppose:

```text
RTL:

assign y = a & b;
```

The library contains:

```text
AND2_X1
AND2_X2
AND2_X4
```

Synthesis can select among them.

---

## Example

Suppose the path has a large load.

The tool may select:

```text
AND2_X4
```

instead of:

```text
AND2_X1
```

because the larger cell may provide stronger drive.

But:

```text
AND2_X4
```

may have:

```text
More Area
More Power
More Input Capacitance
```

Therefore synthesis performs optimization.

---

# 41.1 Synthesis Optimization Loop

Conceptually:

```text
Choose Cell
    |
    v
Calculate Timing
    |
    v
Calculate Area
    |
    v
Calculate Power
    |
    v
Check Constraints
    |
    +---- Violations ----> Optimize
    |                         |
    |                         v
    |                    Choose Another Cell
    |
    +---- Clean -----------> Final Netlist
```

---

# 42. 🧠 Complete `.lib` Mental Model

The easiest way to understand Liberty is:

```text
                         .LIB
                          |
        +-----------------+-----------------+
        |                 |                 |
        v                 v                 v
     LOGIC             TIMING             POWER
        |                 |                 |
        |                 |                 |
   Function          Cell Delay          Leakage
   Pin Direction     Transition          Internal
                     Setup               Switching
                     Hold
                     Recovery
                     Removal
                     Clock-to-Q
        |                 |                 |
        +-----------------+-----------------+
                          |
                          v
                     ELECTRICAL
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
          Capacitance  Fanout     Transition
                          |
                          v
                       VARIATION
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
             OCV        POCV         LVF
```

---

# ⭐ The Most Important Concept

Remember this relationship:

```text
                  RTL
                   |
                   v
               SYNTHESIS
                   |
            +------+------+
            |             |
           RTL           .lib
            |             |
       Functionality   Cell Capability
            |             |
            +------+------+
                   |
                   v
            Gate-Level Netlist
```

Then STA:

```text
                 NETLIST
                    |
                    v
                   STA
                    ^
                    |
          +---------+---------+
          |         |         |
         .lib      .sdc      SPEF
          |         |         |
       Cell data  Timing     Wire RC
                    |
                    v
             Timing Analysis
```

---

# 🎯 Interview Questions

## Q1. What is a `.lib` file?

> A Liberty file contains the logical, timing, power, electrical, and sometimes variation-related characterization of library cells used by synthesis and timing-analysis tools.

---

## Q2. Does `.lib` contain physical cell dimensions?

> Generally, no. Physical abstracts such as cell dimensions, pin geometry, and routing obstructions are provided through LEF or related physical library data.

---

## Q3. Does `.lib` contain setup and hold?

> Yes. Sequential-cell Liberty characterization can contain setup, hold, recovery, removal, and other timing constraints.

---

## Q4. Why are there multiple `.lib` files?

> Because cell timing and power characteristics vary with PVT and other characterization conditions, so different libraries/corners are used for different analysis scenarios.

---

## Q5. What determines cell delay?

Primarily:

```text
Input Slew
+
Output Load
+
PVT
+
Cell Characterization
```

---

## Q6. Why are rise and fall delays separate?

Because CMOS pull-up and pull-down behavior is not identical.

Therefore:

```text
Rise Delay ≠ Fall Delay
```

in general.

---

## Q7. What is timing arc?

> A timing arc represents a characterized timing relationship between a related input/control pin and an output or constrained pin.

Example:

```text
A → Y
```

---

## Q8. What is positive unate?

```text
Input rises → Output rises
Input falls  → Output falls
```

Example:

```text
BUF
```

---

## Q9. What is negative unate?

```text
Input rises → Output falls
Input falls  → Output rises
```

Example:

```text
INV
```

---

## Q10. What is non-unate?

The output transition cannot be determined from the input transition alone.

Example:

```text
XOR
```

---

## Q11. Why does STA need `.lib`?

Because STA needs:

```text
Cell delay
Clock-to-Q
Setup
Hold
Recovery
Removal
Transition
Capacitance
```

to calculate timing paths.

---

## Q12. Why does synthesis need `.lib`?

Because synthesis must know:

```text
Which cells exist
Cell area
Cell timing
Cell power
Cell drive characteristics
Cell constraints
```

to perform technology mapping and optimization.

---

# 🔑 Final Summary

```text
                    .LIBERTY
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
     LOGIC           TIMING           POWER
       |               |               |
   Function        Cell Delay       Leakage
   Pins            Transition       Internal
                   Setup            Switching
                   Hold
                   Recovery
                   Removal
                   Clock-to-Q
       |               |               |
       +---------------+---------------+
                       |
                       v
                  ELECTRICAL
                       |
             +---------+---------+
             |         |         |
             v         v         v
          Capacitance Fanout  Max Transition
                       |
                       v
                   VARIATION
                       |
                 +-----+-----+
                 |           |
                POCV        LVF
```

---

# 🚀 Next Part

## Part 3 — SDC `.sdc` in Extreme Depth

The next part will cover:

```text
create_clock
create_generated_clock
Virtual Clock
Input Delay
Output Delay
Clock Latency
Clock Uncertainty
Clock Transition
Clock Groups
False Paths
Multicycle Paths
Max Delay
Min Delay
Max Transition
Max Capacitance
Max Fanout
Case Analysis
Operating Conditions
Input/Output Constraints
Setup Analysis
Hold Analysis
Launch/Capture Relationships
Interface Timing
```

and most importantly:

```text
                 .SDC
                   |
                   v
          "WHAT TIMING MUST
           THE DESIGN MEET?"
```

with detailed timing diagrams and examples.