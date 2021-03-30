# Rui Wang (2021-03-29)

# Paper information
- Title: Designing a Cost-Effective Cache Replacement Policy using Machine Learning
- Authors: Subhash Sethumurugan, Jieming Yin, John Sartori
- Venue: HPCA 2021
- Keywords: cache replacement policy, reinforcement learning

# Paper content

## Summary
### Motivation
All cache replacement policies can be categorized into two parts, PC-based (program counter-based) policy, and non-PC-based policy. With the knowledge of PC, the policy could easily capture the access patterns, leading to better performance than non-PC-based ones like LRU. However, the processor manufacturers never apply a PC-based policy on the real chips, because it is hard to implement the PC-based on the hardware level. The paper aims to design a cache replacement policy without using PC, which can also reach high performance.

### Learning from reinforcement learning-based simulation:
RL (reinforcement learning) is a machine learning paradigm in which an agent tries to navigate through an environment by choosing an action from a set of allowed actions. (R. Sutton and A. Barto, Reinforcement Learning. MIT Press, 1998) In the paper, they use RL-based simulation to learn which features of access, set and cache line play an essential role in the cache replacement policy. 

The results show that those features share high weights:
1. Preuse Distance:
    - Set accesses since last access to the accessed address. (It is difficult to record all accessed address in practical, so this feature would be neglected in the following discussion)
    - Set accesses between last two accesses to the cache line.
2. Line Last Access Type: Type of last access to the cache line (LD, RFO, PR, WB).
3. Line Hits Since Insertion: Number of hits to cache line since its insertion.
4. Recency: Number of hits to cache line since its insertion.

Then the paper proposes their RLR (Reinforcement Learned Replacement) based on their observations above.

### RLR rules
1. For a significant number of cache lines, the reuse distance can be approximated by preuse distance.
2. The type of previous access of a cache line can be used to predict its chance of receiving a hit.
3. Cache lines that have been accessed can be predicted to be accessed again in the future.
4. Recently-inserted cache lines are prioritized for eviction to allow older cache lines to be reused.

## Strengths
1. The algorithm is non-PC-based, has acceptable area overhead, and relies on easy combination logics.

## Weaknesses
1. This paper uses a neural network to study the cache replacement policy's key feature, and get each weight of them. However, the authors use them straightforwardly without explaining why these features are important. 
2. The features might have some internal relationship with each other which the MLP (multi-layer perceptron) too simple to reveal. 
3. The simulation works on a 16-way 2MB LLC cache and supposes L1 and L2 cache use LRU. What about L1 ICache and DCache and L2 Cache? What about other sizes and associativity? What if L1 and L2 take advantage of other replacement policies?

## Thoughts
1. Regardless of weakness1, it is still worth a try to apply the method on the page cache of the file system. This method refers to using RL to learn key features and designing a new replacement policy based on the features.
2. PCA (Principal component analysis) might solve the weakness2. This technique is commonly used for dimensionality reduction by projecting each data point onto only the first few principal components to obtain lower-dimensional data while preserving as much of the data's variation as possible. (wikipedia)
3. As for weakness 3, many experiments are needed. If the size, associativity, and so on don't affect the result significantly, it is good. Otherwise, these features should also be considered in the RL framework. 