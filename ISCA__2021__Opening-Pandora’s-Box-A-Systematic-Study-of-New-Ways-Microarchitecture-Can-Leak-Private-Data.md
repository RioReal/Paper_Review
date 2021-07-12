# Rui Wang (2021-6-24)

# Paper information
- Title: Opening Pandora’s Box: A Systematic Study of New Ways Microarchitecture Can Leak Private Data
- Authors: Jose Rodrigo Sanchez Vicarte, Pradyumna Shome, Nandeeka Nayak, Caroline Trippely, Adam Morrisonz, David Kohlbrennerx, Christopher W. Fletcher
- Venue: ISCA 2021
- Keywords: side-channel attack, microarchitecture

# Paper content
With Moore's law fading, microarchitectural optimization gains more attention. However, these optimizations possibly bring security implications at the same time. This paper warns that some security vulnerabilities might already exist previously-proposed microarchitectural optimizations and would affect future chips with might-be-implemented optimizations. To make architects better analyze the potential security implications, they also propose microarchitectural leakage descriptors (MLDs) and classify several types of optimizations.

Totally, the following seven microarchitectural optimization classes are discussed.
1. computation simplification: computation simplifications are utilized to simplify or eliminate some computation executions. For example, zero-skip multiplier will directly (in 0 cycle) give the result 0 without occupying the ALU, if one of the operands is 0.
2. pipeline compression: the data exceeding the most significant bit can be ignored without losing valid information when transmitted through pipelines. The pipeline compression takes advantage of this feature to improve the bandwidth of the processor pipelines.
3. silent stores: silent stores can skip store operations that are detected not to change the contents of the storage devices.
4. computation reuse: some computations can be skipped if their identity exists in a hardware table.
5. value prediction: the hardware might give one speculative result with high confidence before its computation ends.
6. register-file compression: the values in registers can be compressed (shared) because of value locality.
7. data memory-dependent prefetchers: traditional prefetchers can prefetch data according to memory addresses, while the data memory-dependent ones are able to capture access patterns according to data contents.
In conclusion, all the above optimizations are proved to be insecurity and each of them leaks different private data under different circumstances.

To deeply analyze which data might be leaked and under which circumstances, they propose a systematical tool called MLDs. The descriptor takes Inst (instructions), Arch, or UArch as inputs, and returns a number that indicates one possible outcome of the hardware. Take the zero-skip multiplier as an example, if the input (only Inst in this example) has one zero operand, the MLD returns 1, indicating it takes 0 cycle; otherwise, the MLD returns 0 indicating it takes 1 cycle. From the MLD, we can learn that, the information that whether one of the operands in the zero-skip multiplier will be leaked, when we can measure the time the hardware takes.

At the end of the paper, they provide proof-of-concept attacks on silent stores and data memory-dependent prefetchers. In Bitslice AES128 Encrypt (BSAES), the intermediate values of the AddRoundKey step are beyond the capacity of X86 registers, so some of them, including part of the expanded key (by which we can infer the key), are stored in the stack. If silent stores are applied, the write operations take less time, if this written data is exactly what in the cache. The adversary can take brute force way, trying different plaintexts and keys and measuring the time until the time is so small that suggesting silent stores worked. Another instance is that the adversary can access out-of-bound data by utilizing indirect-memory prefetcher (IMP), one of the data memory-dependent prefetchers. Given a number i, this prefetcher might prefetch array_A[i] and even array_b[array_A[i]] if it observed such access patterns. But IMP has no knowledge of bounds, the 3-level IMP will prefetch data without checking array bounds.

## Strengths
With MLDs, we can quickly analyze the potential security implications of the optimizations.

## Weaknesses

## Thoughts
1. Optimizations will treat different inputs in different ways. We call this feature data-dependent. The time or the energy consumption which might be a function of the inputs, which means that any optimization might leak data potentially. It is useless if we just focus on how to create safe hardware optimization components. We need to consider this question in the bigger picture.
2. As for this sentence, "with Moore's law fading, microarchitectural optimization gains more attention", I doubt whether exhausting hardware optimizations is a sustainable way. First, more optimizations are adapted, more instructions would be unsafe. Second, better optimizations might amplify the time or power differences for different inputs. Last but not least, we can not add endless optimizations on CPUs. For one specific program, it is likely that only a small part of all optimizations is used, and the other ones are redundant.