# 16-PARS — 16-Bit Single-Cycle Processor

A custom 16-bit single-cycle processor designed with the **Digital** logic simulator for the Computer Architecture course at Sharif University of Technology. The design uses a Harvard-style memory organization, an 8-register register file, an ALU, a control unit, condition-based instruction execution, and an embedded set of test cases.

> **Project status:** The supplied top-level circuit contains 32 embedded test cases. Run them in Digital before describing the test suite as passing. The filenames of two subcircuits also need to be reconciled with the names referenced by the top-level circuit; see [Setup](#setup-and-running-the-project).

## Academic Context

- **Course:** Computer Architecture & Microprocessors
- **Institution:** Department of Electrical Engineering, Sharif University of Technology
- **Instructor:** Dr. Saeed Bagheri Shourki
- **Author:** Mohammadmehdi Fotouhi Firouzabad
- **Simulator:** [Digital — Digital Logic Designer and Circuit Simulator](https://github.com/hneemann/Digital)

## Overview

16-PARS is an educational, custom 16-bit processor built from digital logic components. It implements a single-cycle datapath and separates instruction memory from data memory. Every instruction includes a 2-bit condition-code field (`CC`) that determines whether the instruction's normal side effects are enabled.

### Main Features

- 16-bit data path and 16-bit instructions
- Single-cycle datapath with an 8-bit program counter (`PC`)
- Separate instruction ROM and data RAM
- Eight 16-bit registers (`R0`–`R7`), including a zero register and a return-address register
- Arithmetic, logical, and shift operations
- Load/store instructions
- Conditional execution, conditional branches, `JR`, and `JAL`
- Four status flags: Zero (`Z`), Negative (`N`), Carry (`C`), and Overflow (`V`)
- Debug access to a selected register and a selected data-memory address
- 32 embedded test cases in the supplied top-level schematic

## Hardware Architecture

| Component | Implementation |
|---|---|
| Instruction width | 16 bits |
| Data width | 16 bits |
| Datapath | Single-cycle |
| Program counter | 8 bits |
| Instruction memory (IMEM) | ROM, 8-bit address, 16-bit word; 256 addressable instruction words |
| Data memory (DMEM) | RAM, 16-bit address, 16-bit word |
| Register file | 8 registers × 16 bits, addressed by 3-bit register indices |
| Status flags | 4 bits: `Z`, `N`, `C`, `V` |
| Memory organization | Harvard-style: separate instruction and data memories |

### Register File

| Register | Role |
|---|---|
| `R0` | Constant zero; writes must not change its value |
| `R1`–`R6` | General-purpose registers |
| `R7` | Return-address register; written by `JAL` with `PC + 1` |

The register file has two normal read paths for source operands and a separate debug selection path. Debug access is intended for observation and does not serve as a third execution-path source operand.

## Instruction Set Architecture (ISA)

Instructions are 16 bits wide. The two most significant bits, `[15:14]`, contain the condition code (`CC`); bits `[13:10]` contain the 4-bit opcode.

### Instruction Formats

```text
R-format:   [ CC (2) | OPCODE (4) | Rd (3) | Rs1 (3) | Rs2 (3) | S (1) ]
I-format:   [ CC (2) | OPCODE (4) | Rd/Rs2 (3) | Rs1 (3) | imm4 (4) ]
U-format:   [ CC (2) | OPCODE (4) | Rd (3) | imm7 (7) ]
B/J-format: [ CC (2) | OPCODE (4) | offset10 (10) ]
JR-format:  [ CC (2) | OPCODE (4) | Rs1 (3) | Reserved (7) ]
SYS-format: [ CC (2) | OPCODE (4) | Reserved (10) ]
```

`B` and `JAL` share the same bit layout, but have different behavior. The `S` bit in R-format controls whether an eligible instruction updates the status flags.

### Instruction Table

| Opcode | Mnemonic | Format | Operation |
|:---:|---|:---:|---|
| `0000` | `ADD` | R | `Rd = Rs1 + Rs2` |
| `0001` | `SUB` | R | `Rd = Rs1 - Rs2`; can be used as a compare operation by targeting `R0` |
| `0010` | `AND` | R | `Rd = Rs1 & Rs2` |
| `0011` | `OR` | R | `Rd = Rs1 OR Rs2` (bitwise OR) |
| `0100` | `XOR` | R | `Rd = Rs1 XOR Rs2` |
| `0101` | `SHL` | R | Logical left shift of `Rs1` by `Rs2[3:0]` |
| `0110` | `SHR` | R | Logical right shift of `Rs1` by `Rs2[3:0]` |
| `0111` | `ADDI` | I | `Rd = Rs1 + sext(imm4)` |
| `1000` | `LW` | I | `Rd = DMEM[Rs1 + sext(imm4)]` |
| `1001` | `SW` | I | `DMEM[Rs1 + sext(imm4)] = Rs2` |
| `1010` | `MOVI` | U | `Rd = sext(imm7)` |
| `1011` | `LUI` | U | `Rd[15:9] = imm7`; `Rd[8:0] = 0` |
| `1100` | `B` | B | If the condition is met: `PC = PC + 1 + sext(offset10)` |
| `1101` | `JAL` | J | If the condition is met: `R7 = PC + 1`, then `PC = PC + 1 + sext(offset10)` |
| `1110` | `JR` | JR | If the condition is met: `PC = Rs1[7:0]` |
| `1111` | `NOP` | SYS | No architectural side effects other than normal sequential PC progression |

`sext(x)` denotes sign extension to 16 bits where applicable. The actual field widths and operation definitions above describe this project's custom ISA; it is not intended to be a complete implementation of an established commercial ISA.

## Condition Codes and Status Flags

The condition code occupies bits `[15:14]` of every instruction and is checked against the current status flags.

| `CC` | Mnemonic | Condition | Meaning |
|:---:|---|---|---|
| `00` | `AL` | Always | Execute unconditionally |
| `01` | `EQ` | `Z = 1` | Execute when the Zero flag is set |
| `10` | `LT` | `N = 1` | Execute when the Negative flag is set, following this processor's condition-code definition |
| `11` | `VS` | `V = 1` | Execute when the Overflow flag is set |

If an instruction's condition is not met, its register writes, memory writes, and control-flow side effects are suppressed, and the processor proceeds to the next sequential instruction. The status flags are updated only by R-format instructions when `S = 1` and the instruction condition is satisfied. Otherwise, the previous flag values are retained.

## Hardware Modules

The supplied files are:

| File | Purpose |
|---|---|
| `403102322_CPU.dig` | Top-level processor: PC, instruction ROM, data RAM, flags, instruction decode/control connections, condition logic, and debug interface |
| `ALU.dig` | 16-bit arithmetic, logical, and shift operations; provides `Result` and the `C`, `Z`, `N`, and `V` flags |
| `ControlUnit.dig` | Decodes the 4-bit opcode and generates the control signals for the datapath |
| `RegisterFile.dig` | 8 × 16-bit register storage, source-register read paths, write control, and debug-register selection |

The current top-level file references subcircuits by the names `ALU_CPU.dig`, `ControlUnit.dig`, and `RegisterFile_CPU.dig`. The uploaded files are named `ALU.dig`, `ControlUnit.dig`, and `RegisterFile.dig`. **Before running the project, make these names consistent**: either rename `ALU.dig` to `ALU_CPU.dig` and `RegisterFile.dig` to `RegisterFile_CPU.dig`, or open the top-level circuit in Digital and update the subcircuit references to match the existing filenames. `ControlUnit.dig` already matches. Keep the referenced files together in the same project directory unless the circuit references are updated accordingly.

The sign-extension and condition-checking logic are currently implemented inside the top-level schematic; separate `condition_checker.dig` and `sign_extender.dig` files are not included among the supplied files.

## Debug Interface

The top-level schematic exposes the following debug signals:

| Signal | Direction in circuit | Width | Purpose |
|---|---|---:|---|
| `RESET` | Input | 1 bit | Resets the processor state |
| `DBG_EN` | Input | 1 bit | Enables the debug-register selection path |
| `DBG_RSEL` | Input | 3 bits | Selects one of the eight registers to inspect |
| `DBG_MADDR` | Input | 16 bits | Selects a data-memory address to inspect |
| `CLK` | Internal clock element | 1 bit | Supplies the processor clock |
| `PC_OUT` | Output | 8 bits | Current program-counter value |
| `IR_OUT` | Output | 16 bits | Current instruction word |
| `FLAGS_OUT` | Output | 4 bits | Current status flags |
| `DBG_ROUT` | Output | 16 bits | Selected register value for debugging |
| `DBG_MOUT` | Output | 16 bits | Selected data-memory value for debugging |

## Verification and Test Coverage

The top-level schematic contains **32 embedded test cases**. Their labels cover the following areas:

- Reset and instruction-fetch behavior, including reset in the middle of a program
- `NOP`, positive and negative `MOVI`, and the constant-zero behavior of `R0`
- Arithmetic and logical instructions: `ADD`, `SUB`, `AND`, `OR`, `XOR`, `SHL`, `SHR`, and `ADDI`
- Flag behavior: Zero, Negative/Borrow, signed overflow, and retaining flags when `.S` is not set
- `LW`/`SW` and conditionally executed instructions
- Conditional branches, forward and backward long branches, and PC behavior
- `LUI`, `JR`, `JAL`, and a conditionally suppressed `JAL`
- Higher-level array-sum tests for positive, mixed-sign, and zero-valued data
- Sorting tests for random, already sorted, reverse-sorted, all-equal, and extreme-value inputs

### Running the Tests

1. Install and launch [Digital](https://github.com/hneemann/Digital).
2. Ensure the referenced subcircuit filenames match the top-level circuit, as described in [Setup](#setup-and-running-the-project).
3. Open `403102322_CPU.dig` in Digital.
4. Use Digital's **Test component** command for the top-level circuit and inspect the result for each test case.
5. Record the result only after running the tests. The presence of embedded test cases does not, by itself, confirm that all tests pass.

## Setup and Running the Project

1. Place the top-level circuit and its subcircuits in the same directory (for example, `src/`).
2. Reconcile the `ALU_CPU.dig` and `RegisterFile_CPU.dig` references with the actual filenames before opening the CPU circuit.
3. Open `403102322_CPU.dig` in Digital.
4. Run the built-in test cases using **Test component**.
5. To inspect processor state during simulation, use `DBG_EN`, `DBG_RSEL`, and `DBG_MADDR`, and observe `DBG_ROUT`, `DBG_MOUT`, `PC_OUT`, `IR_OUT`, and `FLAGS_OUT`.

## Repository Structure

The following is a suggested layout for the files supplied with this project. Keep the `.dig` files together, and ensure the subcircuit references match their actual names.

```text
.
├── README.md
└── src/
    ├── 403102322_CPU.dig
    ├── ALU.dig
    ├── ControlUnit.dig
    └── RegisterFile.dig
```

If you add the course specification PDF under `docs/`, include it only if course policy permits redistribution. Before publishing the circuit files in a public repository, confirm that sharing them is allowed by the course instructions and instructor.

## Limitations and Scope

16-PARS is an educational custom 16-bit single-cycle processor built for a course project. Its instruction encodings, register conventions, condition codes, datapath, and debug signals are specific to this implementation. The README describes the supplied schematics and their embedded tests; it does not claim that the test suite has passed unless the tests have been run successfully in Digital.
