
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