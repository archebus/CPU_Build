# 8-bit CPU Architecture Documentation

## CPU Overview
This document details the architecture of a simple 8-bit CPU implementation with a 4-bit address space, supporting 16 distinct instructions. The CPU features 
- A microcoded control unit with a 5-step instruction cycle
- An 8 bit ALU with carry, and a substract switch
- A register for general use (A) and a register for ALU operation (B)
- 64bits of dedicated Memory to store program OPCODES
- A program counter and simple 5hz clock (configurable)
- A 3 panel display programmed for decimal, that supports negative numbers

## Control Signals

| Signal # | Control Signal | Function |
|----------|---------------|----------|
| 0        | J             | Jump - Controls program counter jumping |
| 1        | CO            | Counter Out - Outputs current program counter step to the bus |
| 2        | CE            | Counter Enable - Increments the program counter 1 step |
| 3        | OI            | Output In - Loads data into the output register for panel display  |
| 4        | BI            | B register In - Loads data into the B register |
| 5        | ΣS            | Sum/Subtract Select - Selects operation for the ALU |
| 6        | ΣO            | Sum Out - Enables the ALU output onto the bus |
| 7        | AO            | A register Out - Outputs A register contents to the bus |
| 8        | AI            | A register In - Loads data from the bus into the A register |
| 9        | IO            | Instruction Out - Outputs instruction register contents to the bus |
| 10       | II            | Instruction In - Loads data from the bus into the instruction register |
| 11       | MO            | Memory Out - Enables the memory output to the bus |
| 12       | MI            | Memory address In - Loads an address into the MAR |
| 13       | HLT           | Halt - Stops the CPU clock |

## Instruction Set
-> Blank codes have not been confirmed working in Control ROM yet <-

| Opcode | Mnemonic | Description | Step 0 | Step 1 | Step 2 | Step 3 | Step 4 |
|--------|----------|-------------|-----------------|-----------------|--------|--------|--------|
| 0x0    | NOP      | No Operation | - | - | - | MI CO | MO II CE |
| 0x1    | LDA      | Load from memory to A | IO MI | MO AI | - | MI CO | MO II CE |
| 0x2    | ADD      | Add memory to A | IO MI | MO BI | ΣO AI | MI CO | MO II CE |
| 0x3    | OUT      | Output A register | AO OI | - | - | MI CO | MO II CE |
| 0x4    | JMP      | Jump to address | - | - | - | - | - |
| 0x5    | STA      | Store A to memory | - | - | - | - | - |
| 0x6    | LDI      | Load immediate to A | - | - | - | - | - |
| 0x7    | JC       | Jump if carry | - | - | - | - | - |
| 0x8    | HLT      | Halt CPU | HLT | - | - | MI CO | MO II CE |
| 0x9    | SUB      | Subtract memory from A | - | - | - | - | - |
| 0xA    | AND      | Bitwise AND | - | - | - | - | - |
| 0xB    | OR       | Bitwise OR | - | - | - | - | - |
| 0xC    | XOR      | Bitwise XOR | - | - | - | - | - |
| 0xD    | LDB      | Load from memory to B | - | - | - | - | - |
| 0xE    | STB      | Store B to memory | - | - | - | - | - |
| 0xF    | JNZ      | Jump if not zero | - | - | - | - | - |

## Instruction Format
Each instruction is 8 bits wide:
- The high 4 bits contain the opcode (0-15)
- The low 4 bits contain the memory address (0-15)

## Memory Architecture
- Address Space: 16 locations (4-bit address)
- Word Size: 8 bits
- Memory is isolated from the main bus via a Memory Address Register (MAR)

## Instruction Cycle
Each instruction executes in up to 5 steps:
1. **Execute 1**: Instruction-specific operation
2. **Execute 2**: Instruction-specific operation
3. **Execute 3**: Instruction-specific operation
4. **Fetch 1**: Load program counter to Memory Address Register
5. **Fetch 2**: Load instruction from memory to Instruction Register

## Control Unit
The control unit consists of:
- A 5-step counter
- A control ROM containing microcode for each instruction
- Logic to map instruction opcodes to appropriate microcode sequences

## Control ROM Layout
The control ROM is organzed as an 8×8 grid (64 cells), with control signal patterns for each step of each instruction:
- Each instruction requires 5 steps
- Each step has a 14-bit control signal pattern
- The opcode determines which 5-step sequence to execute

## Sample Programs

### Simple Addition
```
0x18 ; LDA 8    - Load value from address 8 into A register
0x29 ; ADD 9    - Add value from address 9 to A register
0x30 ; OUT      - Output the result
0x80 ; HLT      - Halt the CPU
0x05 ; Data (5) - First operand at address 8
0x03 ; Data (3) - Second operand at address 9
```

### Memory Copy
```
0x18 ; LDA 8    - Load value from address 8 into A
0x59 ; STA 9    - Store A to address 9
0x80 ; HLT      - Halt the CPU
0x00 ; ...      - (unused)
0x42 ; Data     - Value at address 8 to be copied
0x00 ; Target   - Will contain 0x42 after program runs
```

## Implementation Notes
- All jumps update the program counter immediately
- The ALU operation (addition, subtraction, logic) is determined by the ΣS signal
- The fetch cycle is common to all instructions
- The B register is only used for ALU operationsi
