# Rui Wang, 2020/11/10

# Paper information
- Title: JumpSwitches: Restoring the Performance of Indirect Branches In the Era of Spectre
- Authors: Nadav Amit, Fred Jacobs, Michael Wei
- Venue: USENIX ATC
- Keywords: Spectre, indirect branches

# Paper content

## Summary
* To defend the **Spectre variant2**, which takes advantage of speculative execution caused by indirect calls to leak secret, they design JumpSwitches to transfer unsafe code fragment to safe trampolines.

* The traditional way is **retpolines**, directing speculative execution into a finite loop and returning to actual address when the CPU gets the true value. However, retpolines will result in slowdowns because of the loop. The idea is also inspired by the **indirect call promotions**. Indirect call promotions replace indirect calls with directs ones that have a much lower cost, but it increases code size and the branch targets are not flexible at runtime.

* JumpSwitches ensure the security of speculative branch executions, but also optimize the performance comparing with indirect call promotions. The key idea is similar to indirect call promotions. They invented several types of JumpSwitches like inline, outline, registration, instance ones. Different kinds of JumpSwitches are optimized for different cases so that it can go to learned branch targets without increasing much more code size.

* Code for retpoline:

        call rax_retpoline

        rax_retpoline:
            call target
        capture: 
            pause
            lfence
            jmp

* Code for indirect call promotion:

        cmp ${likely}, %eax
        jnz miss
        call {likely}
        jmp done
        miss: 
            call %eax
        done :

* Code for inline JumpSwitches:

        cmp ${likely}, %eax
        jnz miss
        call {likely rel}
        jmp done
        miss: 
            call {slow rel};
        done :

## Strengths
1. It seems easy to implement so-called JumpSwitches.

1. Many variants are come up with so that they can work in many cases.

## Weaknesses
1. The idea is not so novel.
1. The evaluation doesn't compare JumpSwitches with indirect call promotions.
