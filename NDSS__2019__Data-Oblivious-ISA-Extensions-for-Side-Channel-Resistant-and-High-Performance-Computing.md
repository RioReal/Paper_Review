# Rui Wang (2021-01-21)

# Paper information
- Title: Data Oblivious ISA Extensions for Side Channel-Resistant and High Performance Computing
- Authors: Jiyong Yu, Lucas Hsiung, Mohamad El Hajj, Christopher W. Fletcher
- Venue: NDSS 2019
- Keywords: data oblivious, ISA, security 

# Paper content

## Summary
### Motivation
The background on the data oblivious program and the threat model is based on two security definitions:

- *Confidential input privacy* means that the adversary can't distinguish two confidential inputs in polynomial time by the data oblivious program's observable execution trace.

- *BitCycle observability* defines the adversary's ability that it has no knowledge of the value in each memory cell but knows the usage of each memory resource.

Against the adversary with BitCycle observability, the data oblivious program should have the proporty of confidential input privacy.

However, today's microarchitectures lead to the insecurity and performance challenges of our data oblivious codes:
- Side-channel attacks can still work if the hardware implements some instructions in a non-oblivious way. Take the instruction *cmove* (conditional move) as an example. It may be completed in a branch+mov so the branch predictor would result in a speculative execution and as a consequnce leak the secret condition.
- Data-dependent optimizations during program execution can also leak the key data. Some multiply/divide, floating point opertations are so complex that they might have some optimizations in their implementations. Thus, attacks would apply a timing side-channel attack on different input data.
- Because of the lack of supporting ISA-level support for data oblivious program, the programmer have to use some simple instructions and then slow down the speed.

The adversary can take advantage of branch, jump, memory speculation, sub-address optimization, input-dependent arithmetic, microcode, data-based compression, data-based speculation, and silent stores, to attack our programs even if we write codes extremely carefully.

This paper proposes a **Data Oblivious ISA Extension** to overcome the challenges above. 

## Data Oblivious Instruction Set Architectures  extension (OISA):

The basic idea of OISA is to divide data and instructions into two parts respectively, public and confidential, safe and unsafe. Only safe instructions can do operations on confidential data. Otherwise, it throws an exception. Both safe and unsafe instructions work on public data without any limitation.


This paper implements the OISA design on RISC-BOOM (Berkery out of order execution). 

* Each data with a size of one word has one label bit, to specify whether the data is secret or not. The labels will be determined by two special instructions *oseal* (mark data as confidential) and *ounseal* (mark data as public) and be propagated when oblivious instructions are executed. These instructions should not be executed speculatively for the sake of preventing malicious attacks.

* The arithmetic operands, random number generation, and *cmove-style* (conditional move) instructions are considered safe, while the branch, jump, load/store operations are considered unsafe.
* To implement safe address operands, they invent *oblivious memory extension*, which contains safe load/store instructions and *CPUID* for accessing *oblivious memory partition size* (OSZ). The hardware designer ought to provide an oblivious memory partition for *ocld/ocst* (oblivious confidential load/store) to load or store confidential data in this storage. OSZ differs in different machines. We need program-level functions to make data oblivious code portable. If the size of the confidential object doesn't exceed the remaining capability of oblivious memory partition (OMP), just simply allocate memory. Otherwise, set the memory type as ORAM or SCAN. The former one applies the ZeroTrace technique (NDSS'18) which has better performance while the latter one just uses normal memory to emulate the data oblivious one. Additionally, they modify OS-interface about the exception handler, context switching, and the system calls.

Since the datapaths for the majority of instructions are already completed, the main work is to add the extension ones. *oseal* and *onunseal* reuse serializing instructions in RISC-BOOM and design one ALU for *cmov* instruction. For OMP, they isolate one region of the L1 cache which can be used as normal memory if no oblivious object is allocated. Finally, there is also a hardware component called a label station to operate on labels as the rules mentioned before.

### Evaluation:
They implement their design prototype written in the Chisel hardware description language and also simulate it by the software named Multi2Sim. The area overhead is about 6.80% for logic and 1.84% for SRAM and 4.25% in total. Compared with OISA, OISA_OMP has 8.8×/1.7× speed up and 3.2×/40.4× slowdown for small/large data sizes.

## Strengths
The idea that solving the security and performance problems of oblivious code at the ISA level is novel.
 The proof of security analysis is based on a mathematical framework.
## Weaknesses
The labels for data will incur the performance overheads for storage.
## Paper presentation


## Thoughts
1. In the implementation of this paper, they use one extra bit for each word data to specify the label. Usually, oblivious code is a small part of our program and confidential data is far less than public ones. Maybe we can set an extra mode for the CPU and only on this mode, it runs OISA. Or we could make a blacklist for the confidential data, so there is no waste on labeling public data.

3.  The label station is in the CPU, but it doesn't use already completed datapaths in RISC-BOOM. Therefore, it increases the area overheads. Maybe we can move it to the memory side, and every read/write request also sends the operands label (safe or unsafe). 
4. As mentioned in the paper, there are two ways to enhance the performance. One is to increase the OMP size and the other is to add more safe oblivious instructions.
