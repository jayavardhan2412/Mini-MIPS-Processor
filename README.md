# 32-bit Custom MIPS Processor (3-Cycle Architecture)

A custom 32-bit multi-cycle MIPS processor built from scratch in Verilog. This project was developed as part of the CS220 (Computer Organization and Architecture) course at IIT Kanpur under the supervision of Prof. Mainak Chaudhuri.

This project implements a highly optimized **3-Cycle Finite State Machine (FSM)** datapath, completely moving away from a traditional 5-stage pipeline to mitigate control-hazard complexities. It features Register Transfer Level (RTL) control and robust sub-word memory access.

## ABOUT MY CODES

This repository contains the raw Verilog source codes that make up the complete processor architecture. Here is a detailed breakdown of each module and its specific role:

* **`computer.v` (Top-Level Architecture)**
  The main integration module that brings the processor and memory together. It acts as the system wrapper, handling clocking, stalling, and tracking execution cycles (`total_cycles`, `proc_cycles`).
* **`Processor.v` (The Core CPU)**
  Houses the 3-state control FSM (Fetch, Execute, Write-Back). It manages the main datapath, handles big-endian formatting for partial word loads/stores, and contains the custom logic for I/O stalling and keyboard input handshakes.
* **`ALU.v` (Arithmetic Logic Unit)**
  An entirely combinational block that executes all data manipulation. It handles arithmetic (`add`, `sub`), logic (`and`, `or`, `xor`), shifts (`sll`, `srl`), and computes memory addresses and branch targets dynamically in a single cycle.
* **`Memory.v` (Big-Endian Controller)**
  The main memory module. It supports precise byte (`lb`/`sb`), half-word (`lh`/`sh`), and full-word memory operations using a specialized sub-word write command system.
* **`RegisterFile.v` (32x32 Register Array)**
  The CPU's working memory, providing two asynchronous reads and one synchronous write per cycle. It features a hardwired `$0` register to prevent uninitialized cascading.
* **`defs.vh` (Global Definitions)**
  A header file defining all MIPS opcodes, function codes, memory commands, and syscall numbers (`SYS_exit`, `SYS_read`, `SYS_write`), making the RTL code highly readable.

## Instruction Encoding

Each MIPS instruction is 32 bits long. The processor decodes these standard formats combinationally to extract the appropriate fields.

* **R-Format (Register):** Used for instructions with only register operands (add, sub, and, or, xor, nor, sll, srl, sra, sllv, srlv, srav, syscall, slt, sltu, jr, jalr).
* **Encoding:** `opcode (6) | rs (5) | rt (5) | rd (5) | sh_amt (5) | function (6)`
* **I-Format (Immediate):** Used for instructions with an immediate operand (addi, andi, ori, xori, lw, sw, lb, sb, lh, sh, lbu, lhu, lui, beq, bne, bltz, bgez, blez, bgtz, slti, sltiu).
* **Encoding:** `opcode (6) | rs (5) | rt (5) | immediate (16)`
* **J-Format (Jump):** Used for unconditional jumps (j, jal).
* **Encoding:** `opcode (6) | jump_target (26)`

## Execution Flow & Architecture

1. **State 0 (Fetch & Decode):** The instruction is fetched from `Memory.v` using the PC. Operands are asynchronously read from `RegisterFile.v` in the same cycle.
2. **State 1 (Execute):** `ALU.v` performs computations. If a memory instruction is decoded, the effective memory address is calculated here. I/O syscalls transition to special waiting states.
3. **State 2 (Write-Back):** The ALU result, or memory load data, is written back to the destination register. Branch conditions update the PC.

## Big-Endian Sub-Word Memory Access

Standard 32-bit memory handles full words. To support byte and half-word operations:
- **Stores (`sb`, `sh`):** The logic fetches the existing 32-bit word, dynamically masks out the targeted bits, splices in the new sub-word, and issues a write-back.
- **Loads (`lb`, `lbu`, `lh`, `lhu`):** Shifts the requested byte or half-word to the lowest significant bits and applies zero-extension (unsigned) or sign-extension (signed) before writing to the register.
