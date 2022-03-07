# 6502 Assembly Language for people who know BASIC

## Variables

Unlike BASIC, assembly has several different types of things that you could use when you need a variable:

### Registers

**Registers** are memory built into the CPU to store values and perform operations. Some later CPUs have dozens of registers, for all kinds of uses. The 6502 has 5 single-byte registers 
and one 16-bit register. They are:
- The Program Counter or **PC** holds the 16-bit address of the currently-running instruction. As each instriction gets executed, the PC increments appropriately.

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
    also want to keep in mind that certail comparison operands can affect the V bit; again, we'll get to it. There are special instructions to set, clear, and check the V bit.
    
    7: The Negative, or **N** flag is set when the result of a signed 8-bit arithmetic operation ends up negative. Yep, we'll get to it later.
    
- The *Stack Pointer*, or **SR** register stores the single-byte index into the *stack*. More on that later.

