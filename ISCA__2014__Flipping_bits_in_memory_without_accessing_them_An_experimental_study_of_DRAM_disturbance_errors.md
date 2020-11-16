# Rui Wang, 2020/11/13

# Paper information
- Title: Flipping bits in memory without accessing them: An experimental study of DRAM disturbance errors
- Authors: Yoongu Kim; Ross Daly; Jeremie Kim; Chris Fallin; Ji Hye Lee; Donghyuk Lee; Chris Wilkerson; Konrad Lai; Onur Mutlu
- Venue: ISCA
- Keywords: DRAM

# Paper content

## Summary
* In this paper, they found that recent(2014) *commodity* DRAM chips are vulnerable to disturbance errors. This paper mainly exposed the existence of the errors and proposed some possible solutions.

* Demo code on real system:
        
        codela:    
            mov X, %eax
            mov Y, %ebx
            clflush X
            clflush Y
            mfence
            jmp codela
            
* The value X and Y map to the same bank but different rows in DRAMs. The disturbance errors have two causes. One fact is that **when a wordline's voltage is toggled repeatedly, some cells in nearby rows leak charge at a much faster rate**. Additionally, accessing two addresses repeatedly instead of one address can lead to repeated opening/closing of rows instead of columns.

* By hundreds of experiments, they discovered some factors affecting the disturbance errors. First, the disturbance errors are widespread and depend on the modules' manufacture date. It might result from a new but not reliable process. The refresh interval(RI), activation interval(AI), and the number of activations could affect the error rate as well. Also, the address correlation of an aggressor and its victims causes different effects because physically adjacent rows are more likely to influence each other for some hardware reasons. Last, since only charged cells would lose charge, the data patterns and which logic value represents   "charged" decide the error type.

* They also proposed several solutions to prevent DRAM disturbance error.  PARA(probabilistic adjacent row activation) is the most powerful one of all mentioned ones. When access one row, PARA would activate its adjacent rows with a small probability p. Even with a small p causing less performance overhead, the error rate will decrease significantly.
## Strengths


## Weaknesses


## Thoughts