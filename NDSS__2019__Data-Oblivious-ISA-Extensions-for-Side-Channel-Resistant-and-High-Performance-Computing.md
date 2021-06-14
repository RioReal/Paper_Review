# Rui Wang (2021-01-21)

# Paper information
- Title: Data Oblivious ISA Extensions for Side Channel-Resistant and High Performance Computing
- Authors: Jiyong Yu, Lucas Hsiung, Mohamad El Hajj, Christopher W. Fletcher
- Venue: NDSS 2019
- Keywords: data oblivious program

# Paper content
The data oblivious programs have the ability to defend against the *bitCycle observability* adversary which means the adversary has no knowledge of the value in each memory cell but knows the usage of each memory resource. In other words, the adversary can't distinguish two confidential inputs in polynomial time by the data oblivious program's observable execution trace, and we call such property *confidential input privacy*. However, today's microarchitectures lead to the insecurity and performance challenges of the data oblivious codes. To ensure a secure platform for executing the data oblivious program, this paper starts with **Data Oblivious ISA Extension** and implements it on a RISC-BOOM processor.

The paper lists the following potential vulnerabilities of microarchitectures in the modern processors.
- branch, jump, memory speculation: These might cause data or control flow misprediction and then leak secret by some convert channel.
- sub-address optimization, input-dependent arithmetic: The run time of some arithmetic operations like multiply/divide, floating-point operations depends on the operands. Thus, the adversary could apply a timing side-channel attack on different input data.
- microcode: Take the instruction *cmove* (conditional move) as an example. It may be completed in a branch+mov, so the branch predictor would result in a speculative execution and, as a consequence, it leaks the secret condition.
- data-based compression, silent stores: Data-based compression compresses the register file and the cache to save storage, and silent stores would cut off redundant store operations. These optimizations change the memory state or execution time based on the data, and attackers can get extra information of the program data.

Besides the security problems, the performance is also compromised because of the lack of supporting ISA-level support for data oblivious programs. The programmer has to use some simple instructions instead and slow down the CPU speed.

To address the issues aforementioned, the paper proposes OISA which works as a contract between the hardware and software. The basic idea of OISA is to divide data and instructions into two parts respectively, public and confidential, safe and unsafe. The hardware tells the software which operations are reliable while the software tells the hardware which data should be protected. Only safe instructions can do operations on confidential data. Otherwise, it throws an exception. Both safe and unsafe instructions work on public data without any limitation.

This paper implements the OISA design on the RISC-V Boom (Berkeley Out-of-Order Machine) processor. 
- Each data with a size of one word (64 bits in BOOM) has one label bit to specify whether the data is secret or not. This design makes it simple to check whether the labels satisfy the rule, but all storage mediums would be physically extended to store the labels. Labels could be determined by two special instructions *oseal* (mark data as confidential) and *ounseal* (mark data as public) and be propagated when oblivious instructions are executed. These instructions should not be executed speculatively for the sake of preventing malicious attacks.
- The arithmetic operands, random number generation, and *cmove-style* (conditional move) instructions are considered safe, while the branch, jump, load/store operations are considered unsafe. *cmove* instead of a branch is always applied in data oblivious code. For example, an if statement like this
```
(secret) ? (x = array1(A)) : (x = array2(B))
```
might have different execution traces and then leak some information of secret. We can write it as  
```
z, y = array1(A), array2(B)
(secret) ? (x = z) : (x = y)
```
to access both sides of branches, and use *cmove* to transfer the right result finally.
- To implement safe address operands, they invent *oblivious memory extension*, which contains safe load/store instructions and *CPUID* for accessing *oblivious memory partition size* (OSZ). The hardware designer ought to provide an oblivious memory partition for *ocld/ocst* (oblivious confidential load/store) to load or store confidential data in this storage. OSZ differs in different machines. We need program-level functions to make data oblivious code portable. If the size of the confidential object doesn't exceed the remaining capability of oblivious memory partition (OMP), just simply allocate memory. Otherwise, set the memory type as ORAM (oblivious RAM) or SCAN. The former one applies the ZeroTrace technique (NDSS'18) which has better performance while the latter one just uses normal memory to emulate the data oblivious one and reading the object by scanning its memory. Additionally, they modify OS-interface about the exception handler, context switching, and the system calls.

Since the designs of datapaths for the majority of instructions are already completed in RISC-V, the main work is to add the extension ones. *oseal* and *onunseal* reuse serializing instructions in RISC-BOOM and design one ALU for *cmov* instruction. For OMP, they isolate one region of the L1 cache which can be used as normal memory if no oblivious object is allocated. Finally, there is also a hardware component called a label station in their design which operates on labels as per the rules mentioned before. The label stations are in the execution units and deal with the instructions with labels.

## Strengths
1. The idea that solving the security and performance problems of oblivious code at the ISA level is novel.
2. The proof of security analysis is based on a mathematical framework.

## Weaknesses
<!-- 1. This paper implemented the ISA extension is based on RISC. However, most of the popular commercial CPUs use CISC. It will be a large work to extend the CISC to a data oblivious one. -->
1. The labels for data will incur performance overheads for storage, and the storage device will change a lot too. Usually, oblivious code is a small part of our program and confidential data is far less than public ones. It is unwise to modify a lot on the entire hardware system to satisfy few cases.
2.  Not only the hardware parts but also the application software and operating systems would be modified to satisfy the label and OMP design. The OISA style code is not compatible with the machine which lacks any components and the OS necessary to OISA.
3. The OISA is still unable to handle the security problem brought by data-based compression if these optimizations are on. If all data-dependent optimizations are off, performance could be slow down.

## Thoughts
1. The label design in this paper extends the width of one word (64 bits). This should not be the only way to implement the label idea. First, I think greater granularity like 256 bits could cut off overhead, since the length of plaintext, key, and ciphertext for some classical encryption and decryption algorithms, for example, AES and RSA, is far longer than one word. To ensure security, these algorithms ought to be written in a data oblivious way, but one word one label is not suitable for them. Also, where to store the label is trouble. In this paper, they claim that "labels must be stored alongside each word", resulting in weakness 2. However, if the metadata (in this paper, the label is the metadata we care about) is stored too distant, the CPU has to spend much on fetching the labels from the memory system when the instruction needs them. One possible solution is to organize the label in the padding bits of some data structures. Or we can collect all label information of a program in a list/table, and wait for inquiry from the CPU.
2. Oblivious code is a small part of our execution programs, and we focus on security more than performance. It is acceptable to stop all data-dependent optimization when running data oblivious code. To realize this idea, we can set the CPU one *data oblivious mode* which is determined by one register. In this mode, data-dependent optimizations are off and then prefetch the labels to the cache. And in a CISC CPU, this mode allows only a few instructions to run, so we only need to modify a few datapaths like in a RISC one. The speed would be heavily slow down in data oblivious mode, but this mode would not compromise the performance of the normal mode of the CPU and save the cost of hardware design and verification in the CISC CPU.

