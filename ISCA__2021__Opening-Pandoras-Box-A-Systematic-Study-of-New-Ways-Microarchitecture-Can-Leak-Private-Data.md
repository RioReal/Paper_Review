# Rui Wang (2021-06-24)

# Paper information
- Title: Opening Pandora's Box: A Systematic Study of New Ways Microarchitecture Can Leak Private Data
- Authors: Jose Rodrigo Sanchez Vicarte, Pradyumna Shome, Nandeeka Nayak, Caroline Trippely, Adam Morrisonz, David Kohlbrennerx, Christopher W. Fletcher
- Venue: ISCA 2021
- Keywords: side-channel attack, microarchitecture, optimization

# Paper content
With Moore's law fading, microarchitectural optimizations gain more attention. However, these optimizations possibly bring security implications at the same time. This paper warns that there exist some security vulnerabilities in previously-proposed microarchitectural optimizations and might-be-implemented optimizations would affect future chips. To let architects better analyze the potential security implications, the authors also propose microarchitectural leakage descriptors (MLDs), which precisely reveal the privacy leakage and the relative conditions, and classify several types of optimizations.

The following seven microarchitectural optimization classes are discussed. These seven ones roughly cover the most popular optimizations that are already or likely to be applied on commercial CPUs. 
1. computation simplification: computation simplifications are utilized to simplify or eliminate some computation executions. For example, zero-skip multiplier will directly (in 0 cycle) give the result 0 without occupying the ALU, if one of the operands is 0.
2. pipeline compression: the data exceeding the most significant bit can be ignored without losing valid information when transmitted through pipelines. The pipeline compression takes advantage of this feature to improve the bandwidth of the processor pipelines.
3. silent stores: silent stores can skip store operations that are detected not to change the contents of the storage devices.
4. computation reuse: some computations can be skipped if their identity exists in a hardware table.
5. value prediction: the hardware might give one speculative result with high confidence before its computation ends.
6. register-file compression: the values in registers can be compressed (shared) because of value locality.
7. data memory-dependent prefetchers: traditional prefetchers can prefetch data according to memory addresses, while the data memory-dependent ones can capture access patterns according to data contents.
In conclusion, all the above optimizations are proved to be insecure and each of them leaks different private data under different circumstances.

To deeply analyze which data might be leaked and under which circumstances, they propose a systematical tool called MLDs. The optimizations change the hardware resource usage because certain instructions, Arch or UArch states trigger them and the MLDs describe the different observable results like spending time or power consumption. The descriptor takes Inst (instructions), Arch, or UArch as inputs, and returns a number that indicates one possible outcome of the hardware. The number marks each outcome state of the hardware components, from 0 to the capacity of the optimization's channel. Taking the zero-skip multiplier as an example, if the input (only Inst in this example) has one zero operand, the optimization will be triggered, indicating it takes 0 cycle; otherwise, it takes 1 cycle. We can learn that the capacity of the zero-skip multiplier's channel is 2, so we can rewrite the if statement into a function of the Inst: *Inst.operand_1==0 OR Inst.operand_2==0*. The MLD tells us the multiply leaks the information whether there is at least one zero one in the two inputs operands, since the hardware states is a function of this information.

At the end of the paper, they provide proof-of-concept attacks on silent stores and data memory-dependent prefetchers. In Bitslice AES128 Encrypt (BSAES), the intermediate values of the AddRoundKey step are beyond the capacity of X86 registers, so some of them, including part of the expanded key (by which we can infer the key), are stored in the stack. With silent stores applied, the write operations detected not to modify any data in the cache would be ignored, leading to speed up. The adversary can take brute force way, by trying different keys and then measuring the time. The case costing less time suggests the key hits. Another instance is that the adversary can access out-of-bound data by utilizing indirect-memory prefetcher (IMP), one of the data memory-dependent prefetchers. Given number i, this prefetcher might prefetch array\_A[i] (1-level IMP) and even array\_b[array\_A[i]] (2-level IMP) if it observed such access patterns. But IMP does not know bounds, the IMP will prefetch data without checking array bounds. A 3-level IMP (can prefetch array\_c[array\_b[array\_A[i]]]) has the risk of leaking all of the victim memory at worst.

## Strengths
With MLDs, we can quickly analyze the potential security implications of the optimizations.

## Weaknesses
It remains to be seen whether some combinations of those optimizations results in more complex but covert security implications, for there are often several optimizations applied in one processors.

## Thoughts
1. Optimizations will treat different inputs in different ways. We call this feature data-dependent. The time or the energy consumption might be a function of the inputs, which means that any optimization might leak data potentially. It is useless if we just focus on how to create safe hardware optimization components. We need to consider this question in the bigger picture.
2. As for this sentence, "with Moore's law fading, microarchitectural optimization gains more attention", I doubt whether exhausting hardware optimizations is a sustainable way. First, more optimizations are adapted, more instructions would be unsafe. Second, better optimizations might amplify the time or power differences for different inputs. Last but not least, we can not add endless optimizations on CPUs. For one specific program, only a small part of all optimizations is likely used, and the other ones are redundant.