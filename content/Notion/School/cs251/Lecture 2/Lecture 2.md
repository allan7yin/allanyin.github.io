As mentioned last time, we will be working with following commands:

![[Screenshot_2023-01-14_at_7.06.31_PM.png]]

Each line of such an instruction are encoded in 32-bit (four byte) words in memory. The addresses of the words are `Mem[0]`, `Mem[4]`, etc. We step through by multiples of 4, using the program counter, **PC**. The **PC** is a special register, outside of the regular X0, X1, … X31 we mentioned. The purpose of this is to control order of execution. To move where our program is executing, we just need to update the **PC** ← **PC + 4** and step through the instructions. Below is a more visual representation of where data in an instruction is stored:

![[Screenshot_2023-01-14_at_7.14.25_PM.png]]

In the above representation, the **most significant bit** is denoted 31, and the **least significant bit** is denoted 0. The “opcode” (operation code) tells the processor what to do.

![[Screenshot_2023-01-14_at_7.15.45_PM.png]]

# Assignment 0

**Question 1**

```Assembly
100 ADDI X0, XZR, \#1 
104 ADDI X1, XZR, \#43
108 SUBI X2, XZR, \#2
112 CBZ X2, \#2
116 SUBI X3, X12, \#22
120 B \#20
124 
```

  

**Question 2**

```Assembly
36 LDUR X9, [XZR, \#112]
40 SUBI X9, X9, \#2023 
44 STUR X9, [XZR, \#120]
48 
```

  

**Question 3**

```Assembly
0 ADD X2, XZR, XZR
4 ADD X2, X2, X1
8 SUBI X1, X1, \#1
12 CBNZ X1, \#-2
16 
```

  

**Question 4**

```Assembly
0 ADDI X5, XZR, \#10 // loop counter 
4 ADD X6, XZR, X1 // temp holds x1, as we cannot modify x1 
8 CBZ X5, \#9 // this means, loop is over, did not find match, go make x3 = 0
12 LDUR X4, [X6, \#0] // loads in from array 
16 SUB X4, X4, X2
20 CBZ X4 \#4 // to the area where we make x3 = 1
24 ADDI X6, X6, \#8 // move array index by 1
28 SUBI X5, X5, \#1 // decrement loop counter 
32 B \#-6 // this branches back 
// next line is where we make x3 equal to 1
36 ADDI X3, XZR, \#1
44 B \#2 // goes below the lines setting x3 
48 ADD X3, XZR, XZR
```