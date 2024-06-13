## MIPS Hardware

We use the 32-bit version of MIPS. The MIPS processor looks like the following:

![[Screenshot_2023-03-04_at_3.46.46_PM.png]]

**⇒ How MIPS Executes Programs**

Programs live in the same space within the RAM as the data they operate on. To run a program, the program sitting on the hard drive) has to be loaded into memory. The OS uses the **Loader** to do this.

- Assume that program is located at start of memory `(0x00)`
- Some additional memory (shaded light grey) is also made available to the program for use. The rest of memory (not shaded) is not made available to the program.
- The loader setts the PC to the starting address of the program (0x00 for now).

![[Screenshot_2023-03-04_at_3.51.36_PM.png]]

Once the loader is done, the **Fetch-Execute Cycle** begins to run. In pseudo-code, the algorithm behaves as follows:

```C++
Algorithm 1: Fetch-Execute Cycle 
1. PC = 0x00
2. while true do
3.    IR = MEM[PC]
4.    PC += 4
5.    Decode and execute instruction in IR
6. end while 
```

Think of it this way. Our MIPS program starts at 0x00, hence why the PC is there. We continually load the PC line’s MIPS instruction into the **Instruction Register (IR)**. Each instruction is 32 bits, as we are using 32-bit architecture. We read 4 bytes at a time, (4bytes is 32 bits), hence we do PC += 4. Once read into **IR**, decoded and executed.

  

## MIPS Machine Language

We have a set of instructions for MIPS. Of course, this is massive, so for this course, we use a very small subset:

![[Screenshot_2023-03-04_at_4.03.14_PM.png]]

It is important to know that for all assembly languages, they are a textual representation of the machine language. In the above, you can see assembly syntax, and machine syntax. These are **1:1.**

### Add and Subtract Instructions

They have the following format:

- **add** **==$d==****,** **==$s==****, $t: 000000** **==sssss==** **ttttt** **==ddddd==** **00000 100000**

Now, notice that we need 5 bits to represent a register (as we only have 32 general purpose registers). We often like to separate each nibble (4 bits), as it makes the relationship to hexadecimal clearer:

- **add** **==$d==****,** **==$s==****, $t: 0000 00****==ss sss==****t tttt** **==dddd d==****000 0010 0000**

We can convert our MIPS instruction, into hexadecimal, by converting 4 bits into it’s hexa-decimal format:

![[Screenshot_2023-03-04_at_4.12.01_PM.png]]

Subtract is essentially the same as addition:

- **sub** **==$d==****,** **==$s==****, $t: 000000** **==sssss==** **ttttt** **==ddddd==** **00000 100010**
- **Note** that at the bit level, the only difference betwen add and subtract, is the second least-significant bit

  

### Jump Instruction

This is an instruction that changes the control flow of a program. It takes the form of **jr $s,** where **s** is a register. What this does is it changes the PC to be whatever is in **s.** We need this at the end of our MIPS program, in the form of **jr $31**, where **$31** contains the return address to exit the MIPS program. The **Loader** stores **$31** as a return address, this is an address a program must jump to, in order to return control back to the loader. It’s like **return** in C++.

- **jr** **==$s==****: 000000** **==sssss==** **00000 00000 00000 001000**
- **jr** **==$31==****: 0000 00****==11 111==****0 0000 0000 0000 0000 1000. In hexadecimal: 0x03e00008**

  

### Load Immediate and Skip Instruction

We can load constants, either hexadecimal or decimal, into a register. We use it as:

- lis $d: 000000 00000 00000 ddddd 00000 010100

This is called the **load immediate and skip** (lis) instruction. It takes the thing immediately after this instruction as data, and skips that line. Hence, we can do:

```Assembly
lis $8
.word 11
lis $9
.word 0xD
add $3, $8, $9
jr $31
```

The above simple program loads in the numbers 11 and 0xD into the regiesters, and computes their sum.

  

### Multiply and Divide

- mult $s, $t
- multu $s, $t
- div $s, $t
- divu $s, $t

There is no destination register since multipying two 32-bit numbers can easily require more than 32-bits to store, hence, it is stored in special resgisters, **hi and lo.**

- when we do **mult $s, $t:** The product is stored in **Hi and Lo,** where the least significant wor dis stored in **Hi,** and the most significant word is stored in **Lo.** For this course, we will assume it will never be greater than 32-bits, so always in **Lo.**
- **multu** is same as mult, just assumes the register values are unsigned
- when we do **div $s, $t:** The queotient is in **Lo,** and the remainder is in **Hi**
- **divu** is the same, just assumes register values are unsigned

  

### Branch on Equal and Not Equal

The assembly instruction **beq $s, $t, i**, compares the values in **$s and $t** and increments **PC** by i number of words if the values are equal. In other words, if ==**$s == $t**== then ==**pc += i * 4**==. The branch on not equal instruction, **bne**, will update PC similarly if the values in $s and $t are not equal. Consider the following:

```Assembly
lis $8
.word 2
lis $9
.word 3
lis $2
.word 11
div$1,$8
mfhi $3
beq $3, $0, 1 ; if $3 == $0 skip 1 instruction 
add $2, $9, $0 ; store 3 in $2
jr $31         ; exit the program
```

Notice that `beq $3, $0, 1` first moves to the next line, and then moves the PC by 4(skips a line). You need to take this into account when deciding how much to branch by.

  

### Set Less Than

The Set Less Than **(slt $d, $s, $t)** instruction sets the value of register **$d to 1** if the value in register **$s is less than the value in register $t** and sets it to be **0 otherwise**. Once again, there is also an unsigned version, **sltu**.

```Assembly
;example, writing a program that computes the absolute value of $1 and stores the result in $1
slt $2, $1, $0; $2 = 1, if $1 is negative 
beq $2, $0, end ; if positive, just return
sub $1, $0, $1 ; if negative, convert to positive 
end:
jr $31 
```

  

**⇒ Labels**

Lablels are words we can use to move branch to, and they hold the address of whatever line they are above. These are called **assembler directives**

![[Screenshot_2023-03-04_at_6.32.35_PM.png]]

### Store Word and Load Word

We can also access memory that is no in the 32 registsers, that is RAM. With this, we have the ability to access a much larger bank of memory, although it is slower. Here are the commands we use for storing and loading from RAM.

  

**⇒ Store Word**

The store word instruction is **sw $t, i($s)** takes the 32-bit value currently in register $t and stores it at MEM[$s + i], where i is a 16-bit Two’s complement immediate.

**⇒ Load Word**

The Load Word instruction **lw $t, i($s)**, loads a word starting at **MEM[$s + i]** and places it into **$t.** The address accessed in memory must be a multiple of 4, we will not have to deal with situations where it is not a multiple of 4. Below is how we would do so:

  

```Assembly
lw $3, 0($1)
sw $3, 0($2)
jr $31

; We begin by loading the value stored at the address in $1 into register 3. 
; Then we store the value we just loaded, i.e., the value that is currently in $3, at the address that is contained in $2.
```

  

### Input and Output from MIPS Program

To read one character from standard input, we do:

```Assembly
lis $1
.word 0xFFFF0004
lw $3, 0($1) ; load one character from standard input into $3, "loading in from standard input"
```

And, to output one character to standard output, we do:

```Assembly
lis $1
.word 0xFFFF000c
lis $2
.word 67     ; ASCII for upper case C
sw $2, 0($1) ; outputs C to standard output, "saving to standard output"
```

  

### Procedures in Assembly Language

Procedures are very important, as they somewhat allow us to define “functions” in MIPS. But, how can we do this? We do this by introducing a stack. We want to make it so that when we call a procedure, when we return from the procedure, their values are not changed. To do this, we push the register values we want to save them to RAM, and then, change how much memory our program is allowed to access. Consider the below:

![[Screenshot_2023-03-04_at_7.26.17_PM.png]]

![[Screenshot_2023-03-04_at_7.26.24_PM.png]]

  

- It turns out, the **Loader** sets **$30** to be the value of the address **just past** the last word of memory alloted to the program. So, **$30** sort of acts like a bookmark, and tells us where the avilable memory ends.
- So, in the top left, is the state before we enter a procedure. So, we want to preserve the program’s current state. So, we push the register’s w used onto the stack, and mode the value $30 to reflect that.

Once the procedure is done, we pop return, and pop the values of stack, and move **$30** again. Here is how we would push:

```Assembly
sw $1, -4($30) ; stores $1 onto the stack
; now, we can use the $1 however we would like 
lis $1
.word 4
sub $30, $30, $1 ; move the pointer down by 4

;; then in order to pop $1 from the stack, we simply lw into it
lis $1
.word 4
add $30, $30, $1
lw $1, -4($30)
```

  

**⇒ Calling and Returning from Procedures**

There is no “formal” definition for a procedure in Assembley. It’s just a label in front of assembly instructions that represent the code for the procedure. But, how do we return from a procedure? We use **jalr (jump and link register)**.

- The instruction **jalr $s** stores the current value of PC into **$s** and sets PC to **$s**
- So, we often will get the address of a label, and then, **jalr** to that label, which brings us to that procedure, and also bumping the PC (before calling procedure) into **$s**

![[Screenshot_2023-03-04_at_7.40.14_PM.png]]

> **Example:** Write a MIPS program to sum numbers from 1 to N and store the result in $3. Make this a procedure. Assume $2 holds the value for N

```Assembly
; $2 holds the number to sum to 
; 3 will hold the number of the sum 
; preserve all register values, except for $3

sw $1, -4($30)
sw $2, -8($30)
sw $4, -12($30)
sw $31, -16($30)

lis $4
.word 16

sub $30, $30, $4

lis $4
.word computeSum
jalr $4

lis $4
.word 16

add $30, $30, $12
lw $1, -4($30)
lw $2, -8($30)
lw $4, -12($30)
lw $31, -16($30)
jr $31

;; the below is the procedure, almost like a function 
computeSum:
add $3, $0, $0 ; sum is 0
lis $1
.word 1

beq $2, $0, break ; if n is 0 now
loop: 
add $3, $3, $2 ; add n to sum 
sub $2, $2, $1 ; decrement n
beq $0, $0, loop ; move back to beginning of loop 

break:
jr $31
```

  

## MIPS Assembler

![[Screenshot_2023-03-04_at_8.23.54_PM.png]]

This is the assembler for the MIPS language. Each section can be broken down, and explained:

1. **Scanning**
    1. This is the phase that uses DFA’s and Simplified Maximal Munch to read the MIPS code, and tokenize it
    2. The result is something like: **LABEL ID REG COMMA REG COMMA REG**
2. **Parsing**
    1. This is after we have the tokens. Therefore, writing a parser for Assembly Language is straightforward; just check that the sequence of tokens for each line of input follows the syntax for some assembly instruction. Essentially, this is like the syntax check, making sure we are adding registers togther that have ‘$’ at the front, etc.
3. **Semantic Analysis**
    1. Once we have performed the syntax checks, the next stage is **Semantic Analysis.** In higher level languages, this would check to make sure variable names are not duplicated, types match, etc. In Assembley, we just make sure **the labels are not duplicated.**
4. **Synthesis**
    1. This is where we convert the Assmbley language into machine language. AKA, we use the table, and convert each assmbley instruction into a 32-bit binary machine instruction

  

However, in Synthesis, there are a couple of things we need to be aware of. Namely, dealing with labels. We can have an instance where we have something like `beq $0, $0, cond`, but, at the point, the label `cond` has not been encountered yet. We can do this through implementing the assembler in **2 passes**.

- During the parsing stage, the assembler will encounter a label, and add it to a **label table,** where it will associate the label with its corresponding address. At this point, the assembler can also ensure the uniqueness of labels; if, when trying to insert a new entry into the symbol table, an entry for this label already exists, the assembler can generate an appropriate error indicating that a duplicate has been detected.
- Later, during Synthesis, when the assembler encounters the branch instruction that uses begin, it can refer to the symbol table. When it encounters something like a branch instruction, it will use the lablel’s address to compute the offset. If it is in a .word directive, the address is simply subbed in

![[Screenshot_2023-03-04_at_11.45.35_PM.png]]

**⇒ Encoding the Instruction**

After we deal with labels in the synthesis stage, it is time to encode the instructions from MIPS assembly language, into machine language. This is a 2 step process:

- Encoding the MIPS instruction into a 32-bit instruction
- Sending the 32-bit instruction to standard output

So, consider the instruction: `add $3, $2, $4`. To encode this, we refer back to the reference sheet, and we get: 000000 ==**00010**== ==**00100**== **==00011==** 00000 100000.

Now, how do we get this? First, obtain the binary version of the register numbers, so ==**2 is 00010**==, ==**4 is 00100**==, and ==**3 is 00011**==. Then, shift these values into their correct place.

![[Screenshot_2023-03-05_at_10.46.29_AM.png]]

The, in order to produce the correct result, we use the logical or for all of these 32-bit numbers, to obtain our encoded instruction.

![[Screenshot_2023-03-05_at_10.47.17_AM.png]]

  

Every time there is an option to incluce a constant, however, we need to be a bit more careful. Consider the instruction `beq $1, $2, -3`. This has the constant -3 inside. Let’s try and encode it.

![[Screenshot_2023-03-05_at_10.48.46_AM.png]]

![[Screenshot_2023-03-05_at_10.49.35_AM.png]]

Now, this is where the problem arises. Notice that the 32-bit version of - has 1 for all bits except for one. If we were to take the logical or of all the above 32-bit values, we would get an incorrect encoding, as everything would be a 1. To fix this, we notice that there are only 16 bits allocated for the offset, i. So, we trim it into a 16-bit number.

![[Screenshot_2023-03-05_at_10.52.25_AM.png]]

Now, once this is done, we can use bitwise OR to find the encoding:

![[Screenshot_2023-03-05_at_10.53.13_AM.png]]

  

**⇒ Generating Output**

Now that we know how to encode our MIPS instructions into Machine language, need to output. Now, we can’t just print the instruction out, as due to ASCII characters, we will get something unexpected. To print, we do:

```Assembly
int instr = (0<<26) | (2<<21) | (4<<16) | (3<<11) | 32; // add $3, $2, $4
unsigned char c = instr >> 24;
cout << c;
c = instr >> 16;
cout << c;
c = instr >> 8;
cout << c;
c = instr;
cout << c;
```

Now, we create an **unsigned character** to hold each bit we are outputting. Say I did this:

```Assembly
int x = 65;
unsigned char c = 65;
cout << x << c;
// this yields 65A
```

When we print an integer, that number is first converted into the ASCII version of ‘65’, and then sent to standard output. On the other hand, unsigned char is saying it is already in ASCII, so the bits are set to standard output as is. In the first code example above, we print it by every 8 bits. So, first, we go all the way to the, printing the first 8 bits. Then the next, and so on.

  

Below are some spring MT MIPS questions:

![[Screenshot_2023-03-05_at_11.39.43_AM.png]]

Solution:

```Assembly
; determine if the array is fibonacci-like, for index i >= 2, so A[i] = A[i-2] + A[i-1]
; $1 holds the address to the beginning of the array, add 4 each time to access next element 
; $2 holds number of elements, store 1 in $3 if fibonacci-like, otherwise store 0

lis $7
.word 2

lis $4 ; this will be the value used to iterate through the array 
.word 4 

lis $10 ; use this to decrement the loop couter 
.word 1

beq $2, $7, true ; true if size is 2
beq $2, $10, true; true if size is 1

lw $5, 0($1) ; this is the first value
add $1, $1, $4
lw $6, 0($1) ; this is the second value 
add $1, $1, $4
lw $8, 0($1);

loop:
beq $2, $7, true ; if we have reached the end, break out of loop 
add $9, $5, $6 ; this is A[i-2] + A[i-1]
sub $9, $9, $8 ; is checking to see if A[i] - (A[i-2] + A[i-1]) == 0
bne $9, $0, false ; can break out of loop, and immediately return false;
; if equal, then, move the numbers up
add $5, $0, $6 ; make A[i-2] equal to A[i-1]
add $6, $8, $0 ; make A[i-1] equal to A[i]
add $1, $1, $4 
lw $8, 0($1) ; read in the next number of the array 
add $7, $7, $10 ; increment loop counter 
beq $0, $0, loop

true:
lis $3
.word 1
beq $0, $0, end

false:
lis $3
.word 0
beq $0, $0, end

end:
jr $31
```

![[Screenshot_2023-03-05_at_11.40.42_AM.png]]

```Assembly
; we have 2 procedures, f and g
; $1 holds the input value v, determine if f(v) >= g(v)

lis $2
.word g

lis $4
.word 4

jalr $2 ; here, coming back from here, reg values may have changed 
add $6, $0, $3 ; this now holds the value of g(v)

lis $2
.word f

jalr $2
add $7, $0, $3 ; this now holds the value of f(v)

; here, we now have values of f(v) and g(v), compare them, and return result 
sub $8, $7, $6
beq $8, $0, one
slt $9, $0, $8
beq $9, $0, zero; means that $8 is negative, which means g(v) > f(v)
beq $0, $0, one ; reaching here means $8 is positive, as $0 < $8

zero:
add $3, $0, $0
beq $0, $0, end

one: 
lis 43
.word 1
beq $0, $0, end

end: 
jr $31
```