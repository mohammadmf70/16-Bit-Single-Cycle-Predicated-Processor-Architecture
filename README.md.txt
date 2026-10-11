# 16-PARS: 16-Bit Single-Cycle Processor Design

A custom 16-bit single-cycle processor designed and simulated using the **Digital** logic simulator for the Computer Architecture course at Sharif University of Technology.

This repository contains the top-level CPU design, hardware submodules, custom Instruction Set Architecture (ISA) specifications, and built-in test components.

---

## 👨‍💻 Author

- **Mohammadmehdi Fotouhi Firouzabad**  
  - 5th Semester Electrical Engineering Student, Sharif University of Technology  

---

## 📌 Project Overview

**16-PARS** is a custom 16-bit processor built using basic digital logic components in Digital simulator. The processor uses a Harvard architecture (separate instruction and data memory) and features conditional execution on every instruction based on a 2-bit condition code field (`CC`).

### Academic Context
- **Course**: Computer Architecture & Microprocessors
- **Institution**: Department of Electrical Engineering, Sharif University of Technology
- **Instructor**: Dr. Saeed Bagheri Shourki
- **Simulator**: [Digital Logic Simulator](https://github.com/hneemann/Digital)

---

## 🏗️ Hardware Architecture

| Specification | Description |
| :--- | :--- |
| **Data & Instruction Width** | 16-bit |
| **Datapath** | Single-Cycle |
| **Memory Architecture** | Harvard Architecture (Separate IMEM and DMEM) |
| **Instruction Memory (IMEM)** | 256 words (8-bit address, 16-bit ROM) |
| **Data Memory (DMEM)** | 256 words active / 16-bit address (16-bit RAM) |
| **Register File** | 8 registers ($R_0$ to $R_7$, 16-bit each) <br> • $R_0$: Always zero (`ZERO`) <br> • $R_1 - R_6$: General Purpose <br> • $R_7$: Return Address (`RA`) written by `JAL` |
| **Flag Register** | 4-bit flags: Zero ($Z$), Negative ($N$), Carry ($C$), Overflow ($V$) |
| **Conditional Execution** | Evaluated via a 2-bit Condition Code (`CC`) field in every instruction |

---

## ⚡ Flag System & Condition Codes

Every instruction contains a 2-bit **Condition Code (`CC`)** at bits `[15:14]`. The processor checks `CC` against the current **Flag Register**:

| CC | Mnemonic | Condition | Description |
| :---: | :---: | :---: | :--- |
| `00` | `AL` | Always | Executes unconditionally (Default) |
| `01` | `EQ` | Equal | Executes if $Z = 1$ |
| `10` | `LT` | Less Than | Executes if $N = 1$ |
| `11` | `VS` | Overflow Set | Executes if $V = 1$ |

If the condition is not met, the instruction acts as a `NOP` (no registers or memory updated, flags remain unchanged, and $PC$ increments by 1).

The **Flag Register** ($Z, N, C, V$) is only updated by R-format instructions when the $S$-bit (bit 0) is set to `1` and the condition `CC` is satisfied.

---

## 📜 Instruction Set Architecture (ISA)

The processor supports 16 instructions categorized into 6 formats:

### Formats
- **R-Format**: `[CC (2b) | OPCODE (4b) | Rd (3b) | Rs1 (3b) | Rs2 (3b) | S (1b)]`
- **I-Format**: `[CC (2b) | OPCODE (4b) | Rd/Rs2 (3b) | Rs1 (3b) | imm4 (4b)]`
- **U-Format**: `[CC (2b) | OPCODE (4b) | Rd (3b) | imm7 (7b)]`
- **B/J-Format**: `[CC (2b) | OPCODE (4b) | offset10 (10b)]`
- **JR-Format**: `[CC (2b) | OPCODE (4b) | Rs1 (3b) | Reserved (7b)]`
- **SYS-Format**: `[CC (2b) | OPCODE (4b) | Reserved (10b)]`

### Instruction Table

| Opcode | Mnemonic | Format | Description |
| :---: | :--- | :---: | :--- |
| `0000` | `ADD` | R | $R_d = R_{s1} + R_{s2}$ |
| `0001` | `SUB` | R | $R_d = R_{s1} - R_{s2}$ (Used for `CMP` when $R_d = R_0$) |
| `0010` | `AND` | R | $R_d = R_{s1} \ \& \ R_{s2}$ |
| `0011` | `OR` | R | $R_d = R_{s1} \ \Vert{} \ R_{s2}$ |
| `0100` | `XOR` | R | $R_d = R_{s1} \ \oplus \ R_{s2}$ |
| `0101` | `SHL` | R | Shift Left $R_{s1}$ by $R_{s2}[3:0]$ |
| `0110` | `SHR` | R | Shift Right $R_{s1}$ by $R_{s2}[3:0]$ |
| `0111` | `ADDI` | I | $R_d = R_{s1} + \text{sext}(imm_4)$ |
| `1000` | `LW` | I | $R_d = \text{DMEM}[R_{s1} + \text{sext}(imm_4)]$ |
| `1001` | `SW` | I | $\text{DMEM}[R_{s1} + \text{sext}(imm_4)] = R_{s2}$ |
| `1010` | `MOVI` | U | $R_d = \text{sext}(imm_7)$ |
| `1011` | `LUI` | U | $R_d[15:9] = imm_7, \ R_d[8:0] = 0$ |
| `1100` | `B` | B | If $CC$ met: $PC = PC + 1 + \text{sext}(offset_{10})$ |
| `1101` | `JAL` | J | If $CC$ met: $R_7 = PC + 1$, $PC = PC + 1 + \text{sext}(offset_{10})$ |
| `1110` | `JR` | JR | If $CC$ met: $PC = R_{s1}[7:0]$ |
| `1111` | `NOP` | SYS | Increments $PC$ by 1 |

---

## 🧩 Hardware Submodules

The project is structured hierarchically into separate `.dig` files:

1. **Top-Level CPU (`NN_CPU.dig`)**: Main processor circuit connecting all components and debug pins.
2. **Register File (`regfile.dig`)**: 8x16-bit register file with two read ports, one write port, and debug routing.
3. **ALU (`alu.dig`)**: Implements arithmetic, logic, and shift operations, and generates $Z, N, C, V$ flags.
4. **Control Unit (`control_unit.dig`)**: Control logic that decodes instruction opcodes into control signals.
5. **Condition Checker (`condition_checker.dig`)**: Evaluates instruction `CC` against current flags to generate the `cond_met` signal.
6. **Sign Extender (`sign_extender.dig`)**: Extends 4-bit, 7-bit, and 10-bit values to 16-bit signed format.

---

## 🔌 Debug Interface

The top-level CPU schematic includes standard debug pins required for inspection and automated verification:

- **Inputs**: `CLK`, `RESET`, `DBG_RSEL` (3-bit), `DBG_EN` (1-bit), `DBG_MADDR` (16-bit)
- **Outputs**: `PC_OUT` (8-bit), `IR_OUT` (16-bit), `FLAGS_OUT` (4-bit), `DBG_ROUT` (16-bit), `DBG_MOUT` (16-bit)

---

## 🚀 How to Run & Test

1. **Prerequisites**: Download and install [Digital Simulator](https://github.com/hneemann/Digital).
2. **Open the Circuit**: Launch **Digital** and open the main CPU circuit file (`src/NN_CPU.dig`).
3. **Run Tests**:
   - Press **`F8`** (or go to **Simulation → Test component**) inside the simulator window to execute all built-in test cases automatically.
   - Verify that all test assertions pass.

---

## 📁 Repository Structure

```text
.
├── docs/                       # Project specification document
│   └── CS_Project_Specification.pdf
├── src/                        # Digital schematic circuit files (.dig)
│   ├── NN_CPU.dig              # Top-level CPU schematic
│   ├── alu.dig                 # ALU submodule
│   ├── regfile.dig             # Register File submodule
│   ├── control_unit.dig        # Control Unit submodule
│   ├── condition_checker.dig   # Condition Evaluator
│   └── sign_extender.dig       # Sign Extender module
└── README.md                   # Project documentation