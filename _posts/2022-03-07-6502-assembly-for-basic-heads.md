# 6502 Assembly Language for people who know BASIC

## Variables

Unlike BASIC, assembly has several different types of things that you could use when you need a variable:

### Registers

**Registers** are memory built into the CPU to store values and perform operations. Some later CPUs have dozens of registers, for all kinds of uses. The 6502 has 5 single-byte registers 
and one 16-bit register. They are:
- The Program Counter or **PC** holds the 16-bit address of the currently-running instruction. As each instruction gets executed, the PC increments appropriately.

- The **Accumulator** is really the only general-purpose register. A typical fetch-modify-store cycle involves reading the contents of a memory address, modifying it (adding a constant value to it, for example), 
  and storing it back into that memory location. The accumulator is a single byte.

- The **X** and **Y** registers work basically like the accumulator, and can perform many of the same functions, but they're really there to act as indexes -- think of them as the
  variables in a FOR loop. X and Y are also single bytes.
  
- The Status Register, or **SR**, is a single byte holding the value of 8 *flags*. Flags are single bits (0=off/1=on) that indicate the state of various things after an operation 
  happens.
  
  The flags are, from right-to-left / lowest-to-highest:
  
    0: The Carry, or **C** flag gets set if the last arithmetic operation resulted in a value that was over 255. Remember that binary (base 2) addition works just like base 10 addition: you add
    the rightmost digit of the first number to the same digit of the second number. If the result is more than one digit (like 12 in base 10 or 3 in base 2), you truncate the result 
    of that first addition to 1 digit, and the rest "carries" to the next digit. Then, for the second-to-the-rightmost-digit, you add the digit of A and the digit of B _and_ the carry. 
    You continue until you run out of digits. If there's still a carry left at the end, the C flag gets set. Note that the next time a math operation is performed it may clear C. It may 
    also be affected by certain comparison operations; more on that later. There are special instructions to set, clear, and check the C bit.

    1: The Zero, or **Z** flag is set when arithmetic operations get a result of 0, or when a compare instruction is *not* equal. The Z flag is used more often than you would think, 
    as you'll see later. There are special instructions to set, clear, and check the Z bit.

    2: The Interrupt, or **I** flag determines whether or not the CPU will respond to certain interrupts. More on interrupts later.

    3: The Decimal, or **D** flag is for Binary Coded Decimal mode. That's a whole topic of its own, but it's honestly not used very often so we'll ignore it for now. 
    
    4: The Break, or **B** flag is always 1 except during a hardware interrupt. It is very rarely used.
    
    5: This flag is unused, except possibly in weird 6502 clones.
    
    6: The overflow, or **V** flag, is set when doing addition and subtraction of *signed* 8-bit numbers. This is a strange and possibly difficult subject so we'll be handing them later. You will 
    also want to keep in mind that certain comparison operands can affect the V bit; again, we'll get to it. There are special instructions to set, clear, and check the V bit.
    
    7: The Negative, or **N** flag is set when the result of a signed 8-bit arithmetic operation ends up negative. Yep, we'll get to it later.
    
- The *Stack Pointer*, or **SR** register stores the single-byte index into the *stack*. More on that later.

### Constants

Sometimes, you don't so much need a variable to hold a value, as to just *label* a set value. This is where **labels** and **constants** come in. A label can stand for anything numeric: the address of a specific line of assembly code, a descriptive name for a specific address (like one of the registers for the VIC II chip), or just a set value. A label can be much more than just the 2 characters you get with BASIC; there's effectively no limit to how long labels can be. And unlike BASIC, long labels aren't any slower than short ones, since the assembler turns it all into numbers in the end anyway. We'll see more of labels for loops and such later, but for our current topic, we'll stick to statements like:
```
TRUE=1
FALSE=0
SPRITE_ENABLE_REGISTER=53269
INTERRUPT_REGISTER=$d019
```
...and so on.

### The Stack

Here's where your mostly-forgotten computer classes come in. Long ago, you might have learned about the data structure known as a **stack** - a sort of container into which you can *push* values, and then *pull* them out. A stack is a "Last In, First Out" structure, which is to say that the current value in the stack is always the one pushed in last, and the last value you can get from the stack is the first one you pushed in, just like a stack of plates or something. In this case, the most common use is for subroutines. Remember how in BASIC you could GOSUB to a spot in your program, and then when that code was finished, it could just RETURN back to where you called, wherever that was? Assembly has exactly the same thing, and the way it works, is that the GOSUB (or *JSR* for assembly) pushes the address of the call to the stack, and then the RETURN (or *RTS*) will pop the return address and go back there. The beauty of this method is that a subroutine can call another subroutine, which can then call a third. As long as everybody returns properly, it all just works. 

There are also instructions to push and pull your own arbitrary values to the stack, to use it as temporary storage or a communication mechanism. One common example would be in s "SWAP(A,B)" routine: push the value of A to the stack, set a to the value of B, and set B to the value on the top of the stack. Another is to add *parameters* to subroutines. In this scenario, you might want to, again, set up our SWAP code. In BASIC, you would have to make A and B program-wide variables, and then call the subroutine:
```
A=V1
B=V2
GOSUB 8200
V1=A
V2=B
```
This is kind of messy, isn't it? And what if, as you go to make changes three months later, you get the routine at 8200 mixed up with the one at 3400 and set B and C instead? In assembly, we can do something like this: (this is basically pseudocode, to give you the idea)
```
PUSH V2
PUSH V1
jsr SWAP
V1=PULL()
V2=PULL()
```

This can work a lot better, and be easier to keep track of, without making the SWAP routine much bigger.

There are a few drawbacks to using the stack, though:
1. There is only one stack, shared among your program and BASIC and the Kernal. Other machines may have very large stacks, but the 6502's is only 255 bytes max. It is *not* very hard to fill it up, and if it fills up, ***BOOM!*** It is possible to build your own stack using custom code, but that has drawbacks of its own.
2. You have to be *very* careful that you pull everything you push, and in the right order. If you're using the stack for temporary storage in a part of your code, and you forget and leave an extra byte or two sitting there when your routine is trying to RETURN, the CPU pulls what is supposed to be the return address off the top of the stack. Instead, it gets whatever random data you put in there, and, well, you guessed it, another ***BOOM!***
3. The stack is quite a bit slower than registers, and usually slower than using **zero-page variables**.


Note that you don't need any "global" variables like A and B here

### Zero-page memory

Here we're getting into the what probably comes to mind when you think of variables -- a variable as just bytes in the computer's RAM. But in this case, it's a special area of RAM. As we saw earlier, the 6502 only has three internal registers, which is pretty lean. As a way of partially making up for that, the 6502's designers made access to the first 256 bytes of RAM (from addresses 0-255), much faster than access to the rest. Of course, that makes that zero page memory very much in demand -- BASIC uses most of it from addresses 0-127, and the Kernal uses most of the rest. So it you're writing a machine language routine to call from BASIC, you are going to have pretty slim pickings to choose from. On the other hand, if you're designing a standalone program, BASIC won't be running after the initial SYS to call the machine code, so you can use pretty much everything from 2 (0 and 1 are reserved by the hardware) to 127.

When you're using RAM for your variables, the pattern is pretty straightforward:
```
SCORE=3482
A(ccumulator)=PEEK(SCORE)
ADD 10 to A
STORE A into SCORE
DISPLAY(SCORE)
```

This is psuedocode, so don't pay much attention to the details, but this code does point out how you need to use memory, whether it's zero page or not. Unlike modern processors, you can't do some thing like:
ADD 10 to memory SCORE

The only way to modify memory with the 6502 is to load the contents of that memory into a register (usually the accumulator), modify the register, and save the register back into memory. 

### Plain memory

Although accessing plain old memory is the slowest of our methods, it is absolutely the most versatile. Assuming you're writing your own stand-alone program, you have almost complete control of how you want your memory to be organized and used. 

Pretty much all assemblers have memory allocation statements that tell the assembler how to organize the code and memory. (NOTE: if you don't remember your hexadecimal very well, find up a good reminder on the internet somewhere. It's just the most sensible way to describe 16-bit addresses. If nothing else, the calculator that comes with Windows, in Programmer mode, is a great tool to convert back & forth.) The start of the program usually looks something like this:
```
.*=$801                     ; start the program at the beginning of the BASIC space. .ORG $801 would mean the same thing
            .byte XX,YY,ZZ...   ; insert a bunch of bytes here that make BASIC think it's a regular program and execute it

start:      jump to real_start

score:      .byte 00,00,00      ; max score is 16 million and something
fuel:       .word $ffff         ; fuel starts at 65536 and goes down from there. A word is 16 bits, or 2 bytes
lives:      .byte 3             ; three lives to start with
score_string: .byte "score:"
MAX_LIVES=3                 ; This is a constant. it doesn't allocate any memory, it just supplies an unchanging value for the program.
...
real_start:
            (start the program)
            
```
There are a few things to note here:
1. Since the C64 starts up in BASIC, you usually need the first few bytes of your program to have the right values to be executed by BASIC. Of course, you don't need that if you are, for example, writing a game to be put on a cartridge.
2. The program will start at the first memory location after the "camouflage". See how the first instruction is to jump over the memory allocations, to the start of the actual code? Unlike BASIC, machine code doesn't make any distinction between memory used to store variables and memory to store code, and if you're not careful, you'll put the CPU in the position of trying to execute your high score or shield percentage as program code. Guess what happens then? (That's right, another ***BOOM!***)
3. While BASIC lets you create a new variable literally anywhere in your program, in assembly, you'll just about always want to define all of your variables at onec, all at the beginning of the program. That having been said, there's nothing necessarily magical about the beginning of memory -- you could put all of your variables at the end, but many assemblers won't let you use a label it doesn't know about yet at that state of the assembly process, and will thus throw an error. Generally, the beginning the the best choice.
4. As a side note, note also the lack of line numbers. While numbered lines were a handy way for the BASIC interpreter to keep track of things, you'll almost certainly be using a PC with literally billions of bytes of memory, so you can use descriptive labels, cut and paste, all of that good stuff.

So, to summarize, the types of storage you can use in your assembly program are:
1) the three **registers**, **X**, **Y**, and the **accumulator**, built into the 6502. There are a bunch of instructions devoted to usung the registers to do stuff.
2) the **stack**, which can be convenient for calling subroutines and holding temporary values, but which is pretty small, and is more than caspable of crashing your program if used incorrectly.
3) **zero page** memory, which is the memory from 2-255, and which gets special treatment from the 6502, making it quite a bit faster to use then regular memory. However it's also in high demand, so if your program has to co-exist with BASIC and the Kernal, the amout available to a game programmer is very limited.
4) **other memory**, which will be the majority of the memory that you'll use. Remember that you need to be careful that your program doesn't go rampaging through your variable storage, or well, you know.

That's about it for variables. Next time, we'll discuss the actual instructions.
