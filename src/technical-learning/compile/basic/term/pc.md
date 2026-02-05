# program counter

## 一、参考

> [program counter](https://en.wikipedia.org/wiki/Program_counter)

## 二、名词

The program counter (PC), commonly called the instruction pointer (IP) in Intel x86 and Itanium microprocessors, and sometimes called the instruction address register (IAR), the instruction counter, or just part of the instruction sequencer, is a processor register that indicates where a computer is in its program sequence.

Usually, the PC is incremented after fetching an instruction, and holds the memory address of ("points to") the next instruction that would be executed.

Processors usually fetch instructions sequentially from memory, but control transfer instructions change the sequence by placing a new value in the PC. These include branches (sometimes called jumps), subroutine calls, and returns. A transfer that is conditional on the truth of some assertion lets the computer follow a different sequence under different conditions.

A branch provides that the next instruction is fetched from elsewhere in memory. A subroutine call not only branches but saves the preceding contents of the PC somewhere. A return retrieves the saved contents of the PC and places it back in the PC, resuming sequential execution with the instruction following the subroutine call.


程序计数器（PC）通常在英特尔 x86 和 Itanium 微处理器中称为指令指针（IP），有时称为指令地址寄存器（IAR）、指令计数器或指令序列器的一部分。它是一个处理器寄存器，指示计算机在程序序列中的位置。

通常，在获取指令后 PC 会自增，并保存下一条将被执行的指令的内存地址。

处理器通常会按顺序从内存中获取指令，但控制传输指令会通过将新值放入 PC 来改变序列。这些包括分支（有时称为跳转）、子例程调用和返回。条件转移（conditional transfer）依据某个条件的真值来让计算机在不同条件下执行不同的指令序列。

分支提供了从内存中获取下一条指令的位置。子例程调用不仅会分支，还会将 PC 的前一个内容保存在某个地方。返回指令会检索保存的 PC 内容，并将其放回 PC，从子例程调用后的下一条指令继续顺序执行


