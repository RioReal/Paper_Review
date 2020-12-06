# List of recent papers

## keyword: cache replacement

1. Venue: ISCA 2017; link: https://ieeexplore.ieee.org/abstract/document/8192484
> Defend against cache-based side channel attack
2. Venue: MICRO'52; link: https://dl.acm.org/doi/abs/10.1145/3352460.3358319
> Apply DL to the cache replacement
3. Venue: ISCA 2016; link: https://ieeexplore.ieee.org/abstract/document/7551384
> Policy based on Belady's algorithm.

## keyword: dead blocks/cache lines
4. Venue: HPCA 2016; link: https://ieeexplore.ieee.org/abstract/document/7446101
> Manage dead region at runtime.
5. Venue: MICRO'49; link: https://ieeexplore.ieee.org/abstract/document/7783705
> Perceptron learning for reuse **prediction**. Its references about dead block prediction are before 2010. 

## keyword: reuse cache
There are few latest(after 2016) papers in top conference.

6. Venue: MICRO'45; link: https://ieeexplore.ieee.org/abstract/document/6493636
> Reuse cache by using dynamic reuse distances(Protecting Distance)

## keyword: STT-RAM
7. Venue HPCA 2014; link: https://ieeexplore.ieee.org/abstract/document/6835944
> Dead write prediction assisted STT-RAM cache architecture.
>> The key observation is that a significant amount of data written to last-level caches is not actually re-referenced again during the lifetime of the corresponding cache blocks. Such write operations, which we call dead writes, can bypass the cache without incurring extra misses by definition.

8 Venue ISCA 2016; link https://ieeexplore.ieee.org/document/7551386
> Eliminate redundant writes to the LLC in asymmetric LLC

# Summary

- In order to speed up the performance of caches, we need to find a proper cache replacement policy to increase the cache hits. In other words, the cache should keep the data which might be used in the future and evict so-called dead blocks in high priority. 

- Different from the knowledge form the textbook, researchers want the cache to predict the future at runtime rather learn from the history like LRU or clock algorithm. 
Additionally, the cache replacement policy becomes complicated, and some require serveral extra hardware components named dead block predictors for to support them. 
There are also some unbelivable papers using machine learning to predict cache accesses, which seems impossible to be implemented in the real world.

- In 7, they apply the state-of-the-art dead block predictor and analyze on the writeback cachelines in the L3 non-inclusive cache made by STT-RAM. The  key observation is that writeback cachelines are seldom reused, so that they are likely to be dead blocks in LLC. Also someone cares about the extra energe comsuming in the writes in asymmetric LLCs.

- In conclusion, the recent researches in cache replacement become more and more difficult and complicated by adding more hardware components and impossible ML algorithms.  