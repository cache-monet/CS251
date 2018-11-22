# Pipelining

---

## Overview

Breaks down instruction to 5 main stages


**IF** - instruction fetch

**ID** - instruction decode
 * opcode sent to control unit
 * perform read from REG file (both done in parallel)

**EX** - execution
 * use the ALU to perform computations

**MEM** - data access *(mem read/write)*

**WB** - write back 

This allows us to *ideally execute one instruction per clockcycle*

see [Pipeline RT](runtime.md) for more details


We pass down control bits for each instruction through pipeline registers

[Control bits in Pipeline registers](./pictures/control_bits.png)


[Datapath with pipelining](./pictures/datapath_control_bits.png )

Example executes branch in MEM stage, eventually it'll get to ID

---
## Hazards and Forwarding

**Structural hazard** certain instructions cannot overlap with load/store when its at the IF stage (*resource conflicts*)
* *solution*: separate those instructions and data memories
* multiple instructions are tryiing to **use the same hardware on the DP**


**Control Hazard** conditional branch instruction may change **the seq of code is executed**
* PC updated by BRANCH or JUMP instr
* when we start the next instruction?

**Data Hazard** the result of one instr is **need by next** (or subsequent) instruction(s)
* some computation is needed for a subsequent instructions

```mips
...
addi $4, $2, 5
add $1, $4, $4
bne $1, $3, label
...
```

* in certain cases we can resolve this by ***Write first read Second (REG file)***
* other cases we can either **stall the pipeline** with **NOPS** or perform **code rearrangement** to avoid the conflicts in the first place
    * NOPS add additional clockcycles into the pipeline
    * **better idea** use **data forwarding**
```mips
NOP: add $0, $0, $0
```
---
#### Data forwarding / Load-Use Hazard

R-format instructions and any ALU computations can be successfully forwarded to the **immediate following** (no stalls needed)

**Load-Use Hazard** LW data needs **one stall** even with forwarding (needs to read data in MEM stage)
* either stall using NOPs
* OR use code rearrangement and place a line of code that needs execute anyways as **buffer** between LW and data reliance

[Datapath with forwarding hardware](./pictures/data_forwarding.png) 

The MUX chooses between the ***ReadReg** value and the one **computed by the ALU** in the previous instr*

* suplies the **Correct Forward Data**

[Control signals for Forwarding](./pictures/forward_control.png) 

Detect if we have a LW hazard:
* check if instruct at ID or EX is LW (**MemRead is asserted**)
* Either source (rs or rt) in IF/ID must use the same register as LW's rt

```JavaScript
if ((ID/EX).MemRead &&
    (ID/EX).Rt == (IF/ID).Rs ||
    (ID/EX).Rt == (IF/ID).Rt
 )
```
[LW HAZARD](./pictures/lw_hazard.png)


---

#### Control Hazards(Branches)

When the branch instr is in MEM stage, the below can be solved with data forwarding

```mips
add $2, $3, $4
bne $2, $0, -5
```

**Q**: But what is Branch is executed in ID stage?

**A**: A **one cycle delay** is needed IF 
* the **proceeding instr** is a **R-format/addi/subi** AND
* there's a **data dependency**

[Hardware for stall](./pictures/stall_hardware.png)

If Branch success if decided in MEM stage, we must **flush out 3 instructions in IF, ID, and EX**

To avoid excessive flushing: move **branch execution to ID**
* move equality test to ID (avoid ALU) 
* calculate branch address calc to ID 
* [only need to flush the instruction that was just fetched in **IF/ID**](./pictures/branch_flush.png)
    *  the **IF.Flush** control bit turns whatever's in IF into a **NOP**
    * clears all instruction bits in IF/ID register

[Hardware to Execute Branch in ID](./pictures/branch_in_id.png)

**Summary**

After moving Branch to ID, instruction after branch will still always **execute** 
(this instruction is also known as **Branch delay slot**)

How to handle this (possibly errant instruction)

1. MIPS can use compiler to handle them using Code rearrangements/NOPS
2. Alternatively use branch flushing to **zero out control bits in IF/ID register**

Code rearangement guidelines
* Don't swap code with data dependencies
* Don't swap into or out of loops