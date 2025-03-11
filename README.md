# CPU Assembly Operations Reference

This document provides a comprehensive reference for the assembly operations of the custom CPU implemented in Digital.sh.

## Control Signal Bit Positions

Each control signal corresponds to a specific bit position in the control word:

| Bit Position | Control Signal | Hex Value (when only this bit is set) | Description |
|--------------|----------------|--------------------------------------|-------------|
| 0 | HLT | 0x0001 | Halt the CPU |
| 1 | MI | 0x0002 | Memory address register Input |
| 2 | RO | 0x0004 | RAM Output |
| 3 | II | 0x0008 | Instruction register Input |
| 4 | IO | 0x0010 | Instruction register Output |
| 5 | AI | 0x0020 | A register Input |
| 6 | AO | 0x0040 | A register Output |
| 7 | ΣO | 0x0080 | ALU (Sum) Output |
| 8 | ΣS | 0x0100 | ALU (Sum) Subtract mode |
| 9 | BI | 0x0200 | B register Input |
| 10 | OI | 0x0400 | Output register Input |
| 11 | CE | 0x0800 | Counter Enable |
| 12 | CO | 0x1000 | Counter Output |
| 13 | J | 0x2000 | Jump (program counter load) |

## Instruction Cycle Breakdown

### Fetch Cycle (Common to all instructions)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x00 | 0x1002 | CO, MI | Put PC onto address bus |
| 0x01 | 0x0C04 | RO, II, CE | Read from memory to instruction register, increment PC |

### Execute Cycles by Opcode

#### NOP (No Operation - Opcode 0x0)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x02 | 0x0000 | (none) | No operation, return to fetch cycle |

#### LDA (Load A from memory - Opcode 0x1)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x0A | 0x0010 | IO | Put instruction operand on address bus |
| 0x0B | 0x0006 | MI, RO | Load memory address from instruction |
| 0x0C | 0x0024 | RO, AI | Load value from memory into A register |

#### ADD (Add memory to A - Opcode 0x2)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x12 | 0x0010 | IO | Put instruction operand on address bus |
| 0x13 | 0x0206 | MI, RO, BI | Load memory address, prepare B register |
| 0x14 | 0x00A0 | ΣO, AI | Add B to A, store in A |

#### SUB (Subtract memory from A - Opcode 0x3)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x1A | 0x0010 | IO | Put instruction operand on address bus |
| 0x1B | 0x0206 | MI, RO, BI | Load memory address, prepare B register |
| 0x1C | 0x01A0 | ΣO, ΣS, AI | Subtract B from A, store in A |

#### STA (Store A to memory - Opcode 0x4)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x22 | 0x0010 | IO | Put instruction operand on address bus |
| 0x23 | 0x0002 | MI | Load memory address from instruction |
| 0x24 | 0x0044 | RO, AO | Output A register to memory |

#### JMP (Jump to address - Opcode 0x5)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x2A | 0x0010 | IO | Put instruction operand on address bus |
| 0x2B | 0x2000 | J | Jump to address (load PC) |

#### JZ (Jump if Zero - Opcode 0x6)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x32 | Conditional | (Conditional logic) | Jump if zero flag is set |

#### OUT (Output A to output register - Opcode 0x7)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x3A | 0x0440 | AO, OI | Output A register to output register |

#### HLT (Halt the CPU - Opcode 0x8)

| Address | Hex Value | Active Signals | Description |
|---------|-----------|----------------|-------------|
| 0x42 | 0x0001 | HLT | Halt the CPU |

## Opcode Summary

| Opcode | Instruction | Description |
|--------|-------------|-------------|
| 0x0 | NOP | No Operation |
| 0x1 | LDA | Load A from memory |
| 0x2 | ADD | Add memory to A |
| 0x3 | SUB | Subtract memory from A |
| 0x4 | STA | Store A to memory |
| 0x5 | JMP | Jump to address |
| 0x6 | JZ | Jump if Zero (if implemented) |
| 0x7 | OUT | Output A to output register |
| 0x8 | HLT | Halt the CPU |

## Programming Examples

### Example 1: Adding Two Numbers

```assembly
LDA 14    ; Load value from memory address 14
ADD 15    ; Add value from memory address 15
OUT       ; Output the result
HLT       ; Halt the CPU

; Data
14: 5     ; First number (5)
15: 10    ; Second number (10)
```

### Example 2: Counting Loop

```assembly
LDA 20    ; Load counter value
OUT       ; Display current count
SUB 21    ; Decrement by 1
STA 20    ; Store back to counter
JZ 7      ; Jump to end if counter reaches 0
JMP 1     ; Jump back to beginning of loop
HLT       ; Halt when done

; Data
20: 5     ; Counter initial value
21: 1     ; Decrement value
```

## Implementation Notes

This CPU follows a simple fetch-decode-execute cycle with a control ROM driving all operations. The control ROM's contents define the microcode that implements each instruction.

The CPU uses separate address spaces for instruction and data memory. The control signals coordinate data flow between various components including registers, ALU, and memory.

---

*This reference is based on the analysis of the control ROM for a custom CPU implementation in Digital.sh.*
