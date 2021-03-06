# Rui Wang (2021-03-29)

# Paper information
- Title: Designing a Cost-Effective Cache Replacement Policy using Machine Learning
- Authors: Subhash Sethumurugan, Jieming Yin, John Sartori
- Venue: HPCA 2021
- Keywords: cache replacement policy, reinforcement learning

# Paper content
## Summary
The miss rate of the cache depends on its cache replacement policy, the algorithm determining which cache line to be evicted when there is no spare space to insert a new one. All cache replacement policies can be categorized into two parts, *PC-based (program counter-based) policy*, and *non-PC-based policy*. With the knowledge of PC, the policy could easily capture the access patterns, leading to better performance than non-PC-based ones like LRU. However, to transfer the PC from the CPU front-end to the last level cache, extra wire and logic are needed to pass PC throughout LSQ and the higher-level caches, indicating that the expense to design and implement this kind of cache replacement policy in practice is unacceptable, and even the designers need extra work to verify if it works well as expected. That is why the processor manufacturers never apply a PC-based policy on the real chips in practice. To explore other factors in replacements other than PC, the paper takes advantage of the reinforcement learning technique, to find out the key features in replacements excluding PC. Then in the next step, the paper proposes one algorithm called **RLR** (reinforcement learning replacements) based on these features without PC.

This paper applies reinforcement learning on replacements, to study the factors related to the optimal replacement algorithm. Reinforcement learning is a machine learning technique that one agent tries to take optimal actions in the environment and learns from the awards that the action brings. If the quality of each action is calculated by a neural network, we name it Deep Q-Learning. In this model, some information of the access, cache set, and cache line is chosen and encoded as the input. Then a simple neural network with one hidden layer as the agent will consume these inputs and decide on the eviction. The agent is trained to evict the cache line with the longest reuse distance in the cache set after plenty of training episodes on one billion instruction traces per benchmark provided by CRC-2 (2nd Cache Replacement Championship). When training is done, the neural network tells the weights for each feature. The result shows that the following features play a critical role in replacements.
1. Preuse distance:
  - Set accesses since last access to the accessed address. (It is difficult to record all accessed address in practical, so this feature would be neglected in the following discussion)
  - Set accesses between the last two accesses to the cache line. (Experiments reveal that this feature is approximate to the reuse distance)
2. Line last access type: The type of last access to the cache line (Load, Read For Ownership, Prefetch, Writeback).
3. Line hits since insertion: The number of hits to cache line since its insertion.
4. Recency: Order of cache line access for other cache lines in the set.

A priority-based algorithm called RLR based on their observations from the aforementioned features is proposed. This block with the highest priority in their block will be evicted and other blocks with lower priority will be retained. The priority value consists of three parts, its type priority, its hit priority, and its reuse distance priority. If the last access to the line is a prefetch instruction, the type priority will be set to 1; otherwise will be 0. If the cache is not hit since its insertion, the hit priority will be set to 1; otherwise will be 0. If the reuse distance is more than double the average reuse distance in the cache set, the reuse distance priority will be set to 8, because the feature is the key in the observations, otherwise, it will be 0. All the weights are decided by the hill-climbing method on the same workload of the RL part. Finally, the recency is used to break the tie. The old blocks are more likely to be retained compared with the new ones in RLR.

## Strengths
The algorithm is non-PC-based, has acceptable area overhead, and relies on easy combinational logic and sequential logic.

## Weaknesses
1. The features might have some internal relationship with each other which the MLP (multi-layer perceptron) is too simple to reveal. This paper doesn't tell the reader how they encode these input features. In other words, the model is too coarse and unclear.
2. The simulation works on a 16-way 2MB LLC cache and supposes L1 and L2 cache use LRU. What about L1 ICache and DCache and L2 Cache? What about other sizes and associativity? What if L1 and L2 take advantage of other replacement policies? (On 8-way 32KB L1 cache, their RLR is not so good).

## Thoughts
1. Regardless of the weakness that neural network is a black box, it is worth a try to apply the method on the page cache of the file system. This method refers to using RL to learn key features and designing a new replacement policy based on the features.
2. PCA (Principal component analysis) might solve the first weakness. This technique is commonly used for dimensionality reduction by projecting each data point onto only the first few principal components to obtain lower-dimensional data while preserving as much of the data's variation as possible (cited from Wikipedia).
3. As for the second weakness, many experiments are needed. If the size, associativity, and so on don't affect the result significantly, it is good. Otherwise, these features should also be considered in the RL framework. Also, the RLR should be adjusted according to the new observations.